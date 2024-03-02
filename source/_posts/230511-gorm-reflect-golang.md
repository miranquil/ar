---
title: 一次折腾 Golang 反射和 Gorm 的经历
date: 2023-05-11 17:01:29
tags:
  - Golang
categories:
  - Algorithm & Programming
---

事情的起初是一个很常见的需求：批量更新多条记录的相同字段，每条记录对应的字段值不同因此无法批量 Update。看着没啥难度却没想到从开头到结束整整花了一天的时间，遂有此文。

<!--more-->

首先尝试了 gorm 自带的 `Save()`，按理说 gorm 本身会自动识别零值不去更新，这样直接创建一个实例数组挨个赋值后调 `Save()` 就可以了，如：

```go
data := make([]records, 0, len(n))
for i:=0; i!=len(data); i++ {
    data[i].column = values[i]
}
return db.Save(data).Error
```

现实很骨感：`Save()` 只支持类似思路的**单个**字段更新，敢往里塞数组这货会直接批量创建记录……

重新整理了一下思路，想到可以手搓 SQL 用 `UPDATE` 的 `CASE WHEN THEN` 语法来批量更新，控制一下每次更新的记录数量就行了。

翻了半天 gorm 浅显易懂的文档压根没有关于`CASE WHEN THEN`的内容，讲真要不是 gorm 有 `Exec` 方法我都想考虑换 ORM……~~其实早就想换了~~

实现时候又碰见了两个问题，一个是使用`UPDATE`的话需要往 SQL 中传入表名，如何获取表名需要一个通用的函数（不能指望每个表都手动实现`TableName()`），于是又花时间实现了一个通用的函数。~~具体内容足够再水一篇。~~ 另外就是需要一个通用的将数字转换为字符串的函数，把结果最后拼接成 SQL。

## SQL

麻烦的点主要是需要识别每个新值的类型，如果是字符串或者`Time`的话需要往 SQL 里塞引号，于是写了个根据反射判断类型并转为字符串的函数：

这里要注意一下不能简单的用`%v`或`%f`去格式化打印浮点型，因为结果可能是科学记数法。

```go
func ParseNumberToString(n any) (result string, ok bool) {
    var integer, decimal32, decimal64 bool
    switch n.(type) {
    // common
    case int:
        integer = true
    // ...
    case float32:
        decimal32 = true
    // ...

    if integer {
        return fmt.Sprintf("%d", n), true
    }
    if decimal32 {
        decimalValue, _ := n.(float32)
        return strconv.FormatFloat(float64(decimalValue), 'f', -1, 32), true
    }
    if decimal64 {
        decimalValue, _ := n.(float64)
        return strconv.FormatFloat(decimalValue, 'f', -1, 64), true
    }
    return "", false
}
```

在拼接 SQL 的时候检查是否是字符串 or `Time`，如果两者都不是直接调它就行了。主要内容大概这样：

```go
query := fmt.Sprintf("UPDATE %s SET", tableName)

conditions := make([]string, 0, len(columnValuesMap))
for column, rawValues := range columnValuesMap {
    condition := fmt.Sprintf(" %s = CASE id ", column)
    args := make([]string, 0, len(ids))
    for i := 0; i != len(rawValues); i++ {
        var stringValue string
        switch rawValues[i].(type) {
        case string:
            stringValue = rawValues[i].(string)
        case time.Time:
            stringValue = rawValues[i].(time.Time).Format("2006-01-02 15:04:05")
        default:
            strValue, ok := reflects.ParseNumberToString(rawValues[i])
            // 这里不能直接写 stringValue 就极度的恶心好吧，想写还要声明一遍 ok 再把：=改成=！
            if !ok {
                return fmt.Errorf("unknown type: %T", rawValues[i])
            }
            stringValue = strValue
        }
        args = append(args, fmt.Sprintf(" WHEN %d THEN %s", ids[i], stringValue))
    }
    condition += strings.Join(args, " ")
    conditions = append(conditions, condition)
}

query += strings.Join(conditions, " END, ") + " END WHERE id IN (?)"
```

处理一下可能的数据不合法情况如值和 id 数量不一样，字段名为空啥的就能顺利的拼接出 SQL 了。

拼 SQL 没花多少功夫，真正费时间的在于那个生成表名的函数……

## TableName

gorm 这个框架本身拥有一套自己生成表名的函数，并定义了一个`Tabler`接口，其中包含一个`TableName() string`方法来返回表名。用户可以自行实现`TableName()`，若未实现则会使用 gorm 自己的规则。

如果想不破坏现有的逻辑，那就只能把 gorm 自己的规则翻出来。stackoverflow 上有人也问过 [类似的问题](https://stackoverflow.com/questions/51999441/how-to-get-a-table-name-from-a-model-in-gorm)，稍加查询得到了如下代码：

```go
func (t Table) TableName() string {
    stmt := &gorm.Statement{DB: DB}
    stmt.Parse(t)
    return stmt.Schema.Table
}
```

因为项目有些表已经手动实现了 TableName，所以想着不破坏现有逻辑的情况下尝试直接为每个表都会继承的基础类添上这个方法。

但陷阱在于，`stmt.Parse(t)` 本身也会调用 `TableName`，于是 test 的时候直接真的 stack overflow 了……

后来查阅了一下`Parse`到底在干嘛，发现还有第二种方案：

```go
func (t Table) TableName() string {
    stmt := &gorm.Statement{DB: DB}
    namer := stmt.NamingStrategy
    modelType := reflect.Indirect(t).Type()
    modelValue := reflect.New(modelType)
    return namer.TableName(modelType.Name())
}
```

完美！直接把`Parse`的大部分过程跳过了！

然而妄想给基础类实现这个方法会导致传入的 modelType 永远都是基础类，于是每个表的 TableName 都成了`base_model`……

无奈只能写一个往里传实际类型的。由于之前为了统一数据库逻辑，把所有数据库相关的表方法都改成了泛型函数，于是也要整一个泛型版出来。

```go
func GetGormTableName[T any]() string {
    var t T
    stmt := &gorm.Statement{DB: DB}
    namer := stmt.NamingStrategy

    tType := reflect.ValueOf(t).Type()
    tTypeName := tType.Name()
    return namer.TableName(tTypeName)
}
```

看起来不错，唯一可惜就是不能通过方法调用。

然而又在测试时候发现了问题：泛型会直接把 T 的类型带入函数（废话），这就导致如果 T 是一个指针类型，那么`var t T`本质上直接创建了一个空指针，然后把这玩意给`reflect.ValueOf()`会直接 panic……

虽然应该没有人吃饱了撑的写`GetGormTableName[*Table]()`，但就怕有人在今天这种日子给他打了 50。于是……

通过`reflect.Type`的`Elem()`方法可以拿到指针指向的元素类型，再用`reflect.New(elem)`可以创建一个新的指向该类型的指针，最后用`Elem().Interface()`就可以获取对应的实例了。

```go
func GetGormTableName[T any]() string {
    var t T
    stmt := &gorm.Statement{DB: global.DB}
    namer := stmt.NamingStrategy

    var tTypeName string
    if reflect.ValueOf(t).Kind() == reflect.Ptr {
        tElem := reflect.TypeOf(t).Elem()
        newPtr := reflect.New(tElem)
        tInstance := newPtr.Elem().Interface()
        tTypeName = reflect.ValueOf(tInstance).Type().Name()
    } else {
        tType := reflect.ValueOf(t).Type()
        tTypeName = tType.Name()
    }
    return namer.TableName(tTypeName)
}
```

顺便还写了一个判断当前类型是否拥有`TableName()`方法的函数，如果有的话直接返回对应的值。

```go
func GetImplementedTableName[T any]() (result string, ok bool) {
    var t T
    var funcMayExist reflect.Value
    // 这里也涉及对于方法接收者是否为指针类型的判断，如果接收者为指针，通过对象本身调用方法也会报错，那么直接把传入的对象取地址。
    if reflect.ValueOf(t).Kind() != reflect.Ptr {
        funcMayExist = reflect.ValueOf(&t).MethodByName("TableName")
    } else {
    // 这里同样为传入的指针类型创建一个实例对象。
        tType := reflect.TypeOf(t).Elem()
        tPtr := reflect.New(tType)
        instance := tPtr.Elem().Interface()
        funcMayExist = reflect.ValueOf(instance).MethodByName("TableName")
    }
    if funcMayExist.IsValid() {
        values := funcMayExist.Call(nil)
        if len(values) != 0 {
            if values[0].Kind() == reflect.String {
                return values[0].String(), true
            }
        }
    }

    return "", false
}
```

忙了一天差不多算是完工了，现在 Golang 的泛型我怎么感觉怎么像是用反射换皮的……（还没换完整，不然为啥不支持结构体泛型？）
