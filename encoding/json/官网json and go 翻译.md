# JSON与Go

标签（空格分隔）： 未分类

---

`http://rgyq.blog.163.com/blog/static/316125382013934153244/`

翻译自 JSON and Go(`http://blog.golang.org/json-and-go`)

JSON与Go

##介绍

JSON(JavaScript Object Notation)是一种简单的数据交换格式。从语法上来说，它综合了JavaScript的对象(objects)和列表(lists)。通常用于在web后端和运行在浏览器中的JavaScript程序之间通信，不过也可以用在很多其他的地方。官方主页，json.org，提供了对标准的详尽说明。

使用json 包可以轻松地在Go程序中读写JSON数据

##编码

通过函数Marshal编码JSON数据。

func Marshal(v interface{}) ([]byte, error)
给定Go的数据结构， Message，
```go
type Message struct {
    Name string
    Body string
    Time int64
}
```
以及Message的一个实例
```go
m := Message{"Alice", "Hello", 1294706395881547000}
```
通过json.Marshal就可以得到一个JSON格式的m：
```go
b, err := json.Marshal(m)
```
如果一切顺利，err将会是nil，b将会是一个包含JSON数据的 []byte：
```go
b == []byte(`{"Name":"Alice","Body":"Hello","Time":1294706395881547000}`)
```
只有能够被表示成合法的JSON的数据才会被编码：

JSON objects 只支持字符串作为 keys；要对 Go map 类型编码，必须是这是形式map[string]T（其中T任意一种 json 包支持的 Go 类型）
Channel， complex 以及函数不能被编码
不支持循环的数据结构；这会导致 Marshal 进入死循环
指针会被编码成它们指向的值（或者null如果指针是nil）

##解码

通过函数Unmarshal解码JSON数据。
```go
func Unmarshal(data []byte, v interface{}) error
```
首先，声明一个变量用于存放解码后的数据
```go
var m Message
```
然后调用json.Unmarshal，传递参数JSON数据的[]byte以及指向m的指针
```go
err := json.Unmarshal(b, &m)
```
如果b中包含匹配m的合法的JSON，那么函数调用之后，err值为nil，b中的数据会存到结构体m中，就像下面这样的赋值：
```go
m = Message{
    Name: "Alice",
    Body: "Hello",
    Time: 1294706395881547000,
}
```
Unmarshal是如何定义存放解码的数据的呢？对于一个给定的 JSON key"Foo"，Unmarshal会查询结构体的域来寻找（in order of preference）：

一个带有标签"Foo" 的可导出域（更多关于结构体标签见Go spec）
一个名为"Foo" 的可导出域，或者
一个名为"FOO" 或者 "FoO 或者其他大小写的匹配"Foo"的可导出域
当 JSON 数据结构跟 Go 类型不一致会怎么样呢？
```go
b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
var m Message
err := json.Unmarshal(b, &m)
```
Unmarshal只会解码在目标类型中出现的域。在上面的例子中，m只有Name域会被填充，Food域将被忽略。这种行为在你想从一个大的JSON blob中选择几个指定的域时会特别有用。这也意味着Unmarshal不会影响目标结构体中的不可导出域。

但是，如果你事先并不了解JSON数据的结构，又该如何呢？

Generic JSON with interface{}

interface{}类型（空接口）表示一个没有方法的接口。每一个Go类型至少实现了0个方法，因此都符合空接口。

空接口可以作为一个通用的容器类型：
```go
var i interface{}
i = "a string"
i = 2011
i = 2.777
```
类型断言可以访问底层的具体类型：
```go
r := i.(float64)
fmt.Println("the circle's area", math.Pi*r*r)
```
或者，如果底层的类型是未知的，通过类型switch来确定类型：
```go
switch v := i.(type) {
case int:
    fmt.Println("twice i is", v*2)
case float64:
    fmt.Println("the reciprocal of i is", 1/v)
case string:
    h := len(v) / 2
    fmt.Println("i swapped by halves is", v[h:]+v[:h])
default:
    // i isn't one of the types above
}
```
json 包使用 map[string]interface{} 和 []interface{} 来存储任意 JSON objects 以及 arrays；它很乐意将任意合法的 JSON blob unmarshal 到普通的 interface{} 中。默认的具体 Go 类型是：
```go
bool JSON booleans
float64 JSON numbers
string JSON strings
nil JSON null
```

##解码任意数据

考虑这样一个存放在变量b中的JSON数据：
```go
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
···
当不知道数据结构时，我们可以将它Unmarshal到interface{} ：
```go
var f interface{}
err := json.Unmarshal(b, &f)
```
现在f中的值就是一个map，key为字符串，值就是以空接口存储的自身：
```go
f = map[string]interface{}{
    "Name": "Wednesday",
    "Age":  6,
    "Parents": []interface{}{
        "Gomez",
        "Morticia",
    },
}
```
要访问这样的数据，我们通过类型断言来访问f底层的map[string]interface{}：
```go
m := f.(map[string]interface{})
```
通过 range 语句来迭代map，并通过类型选择来访问具体类型的值：
```go
for k, v := range m {
    switch vv := v.(type) {
    case string:
        fmt.Println(k, "is string", vv)
    case int:
        fmt.Println(k, "is int", vv)
    case []interface{}:
        fmt.Println(k, "is an array:")
        for i, u := range vv {
            fmt.Println(i, u)
        }
    default:
        fmt.Println(k, "is of a type I don't know how to handle")
    }
}
```
通过这种方式我们可以访问未知的 JSON 数据，同时还获得了类型安全的好处。

##引用类型

现在定义一个Go类型来包含上一个例子中的数据：
```go
type FamilyMember struct {
    Name    string
    Age     int
    Parents []string
}

    var m FamilyMember
    err := json.Unmarshal(b, &m)
```
可以正常地将数据 unmarshal 到FamilyMember中，但是如果仔细观察就能看到有意思的事情发生了。通过 var 语句我们分配了一个FamilyMember结构体，然后将指向结构体的指针传递给Unmarshal，但是现在Parents还是一个nil的slice。为了填充Parents域，Unmarshal在内部分配了一个新的slice。这是Unmarshal 支持reference（pointers，slices和maps）类型的典型做法。

考虑 unmarshal 数据到这个结构：
```go
type Foo struct {
    Bar *Bar
}
```
如果在JSON object 中有一个域 Bar，那么Unmarshal就会分配一个新的Bar并填充。如果没有，Bar就是一个nil指针。

根据这个可以得到一个规则：如果应用接收几个不同的消息类型，你可能会像下面这样定义”receiver”结构：
```go
type IncomingMessage struct {
    Cmd *Command
    Msg *Message
}
```
取决于要通信的消息类型，发送方以top-level JSON object 填充 Cmd 域和/或者Msg域。Unmarshal在将数据解码到IncomingMessage结构中时，只会分配在出现在JSON 数据中的结构。要知道处理哪个消息，程序员只需简单的测试下Cmd还是Msg不是nil。

##流式编解码

json 包提供了Decoder 和 Encoder 用来支持JSON 数据流的读写。函数NewDecoder 和NewEncoder 封装了io.Reader和io.Writer 接口类型。
```go
func NewDecoder(r io.Reader) *Decoder
func NewEncoder(w io.Writer) *Encoder
```
下面是一个从标准输入读入连续的JSON object的实例程序，每个结构体只留下Name域，然后把objects写到标准输出：
```go
package main

import (
    "encoding/json"
    "log"
    "os"
)

func main() {
    dec := json.NewDecoder(os.Stdin)
    enc := json.NewEncoder(os.Stdout)
    for {
        var v map[string]interface{}
        if err := dec.Decode(&v); err != nil {
            log.Println(err)
            return
        }
        for k := range v {
            if k != "Name" {
                delete(v, k)
            }
        }
        if err := enc.Encode(&v); err != nil {
            log.Println(err)
        }
    }
}
```
由于读写操作的普遍性，类型Encode和Decoder可以用于多种场合，例如读写HTTP 链接，WebSockets或者文件。

##参考

更多信息请参考[json package documentation]。[jsonrpc] 包中的源文件给出了一个使用json的例子。

作者 Andrew Gerrand

2013.10.3@深圳 坪洲

附一段解析json字符串的代码
```go
package main

import (
    "fmt"
    "encoding/json"
)

type desc struct {
    Lang string `json:"lang"`
    Content string `json:"content"`
}
type DescSlice struct {
    Desc []desc `json:"body"`
}

func main() {
    app1 := `{"lang":"ch", "content":"1233456"}`
    var  info1 desc
    err := json.Unmarshal([]byte(app1), &info1)
    if err != nil {
        fmt.Printf("error is %v\n", err)
    } else {
        fmt.Printf("%v\n", info1)
    }


    app2 := `[{"lang":"ch01","content":"1233456"},{"lang":"ch02","content":"1233456"}]`
    var info2 []desc
    err = json.Unmarshal([]byte(app2), &info2)
    if err != nil {
        fmt.Printf("error is %v\n", err)
    } else {
        fmt.Printf("%v\n", info2)
    }

    app3 := `{"body":[{"lang":"ch01","content":"1233456"},{"lang":"ch02","content":"1233456"}]}`
    info3 := DescSlice{}
    err = json.Unmarshal([]byte(app3), &info3)
    if err != nil {
        fmt.Println("error is %v\n", err)
    } else {
        fmt.Printf("%v\n", info3)
    }
}
```



