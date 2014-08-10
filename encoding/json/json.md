#json包

import "encoding/json"

---

##简介

##概览
json实现了JSON对象（RFC4627中定义）的编码和解码。JSON和Go值之间的映射在Marshal和Unmarshal函数的文档中描述。可以参考文章` http://golang.org/doc/articles/json_and_go.html `理解json包。


###func Compact
```go
func Compact(dst *bytes.Buffer, src []byte) error
```
Compact向`dst`中增加JSON-encoded `src`，同时消除那些微不足道的空格。

###func HTMLEscape
```go
func HTMLEscape(dst *bytes.Buffer, src []byte)
```
HTMLEscape向`dst`中增加JSON-encoded `src`，同时字符串中的<, >, &, U+2028 and U+2029 字符转换为 \u003c, \u003e, \u0026, \u2028, \u2029，这样JSON可以安全地嵌入在HTML <脚本>标记。

未完。。。

###func Indent
```go
func Indent(dst *bytes.Buffer, src []byte, prefix, indent string) error
```
Indent向`dst`中增加带有缩进形式的JSON-encoded `src`。

未完。。。

###func Marshal
```go
func Marshal(v interface{}) ([]byte, error)
```

>只有能够被表示成合法的JSON的数据才会被编码：
- JSON objects 只支持字符串作为 keys；要对 Go map 类型编码，必须是这是形式map[string]T（其中T任意一种 json 包支持的 Go 类型）
- Channel， complex 以及函数不能被编码
- 不支持循环的数据结构；这会导致 Marshal 进入死循环
- 指针会被编码成它们指向的值（或者null如果指针是nil）

###func MarshalIndent
```go
func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
```
MarshalIndent和Marshal很像，但是使用了缩进`indent`来格式化输出。

###func Unmarshal
```go
func Unmarshal(data []byte, v interface{}) error
```

>Unmarshal是如何定义存放解码的数据的呢？对于一个给定的 JSON key"Foo"，Unmarshal会查询结构体的域来寻找（in order of preference）：
- 一个带有标签"Foo" 的可导出域（更多关于结构体标签见Go spec）
- 一个名为"Foo" 的可导出域，或者
- 一个名为"FOO" 或者 "FoO 或者其他大小写的匹配"Foo"的可导出域

###type Decoder struct
```go
type Decoder struct {
}
```
Decoder类型读取并解码输入流中的JSON对象。

###func NewDecoder
```go
func NewDecoder(r io.Reader) *Decoder
```
NewDecoder返回一个新的dedocer，这个decoder从`r`读取。

这个decoder引入它自己的缓冲，从`r`读取的数据可能超过JSON值的要求。

###func (*Decoder) Buffered
```go
func (dec *Decoder) Buffered() io.Reader
```
Buffered返回一个reader，这个reader的数据残留在Decoder的缓冲中。reader是有效的，直到下次调用Decode。

###func (*Decoder) Decode
```go
func (dec *Decoder) Decode(v interface{}) error
```
Decode读取下一个JSON编码的值，然后存储到`v`所指的值中。

想了解更多关于JONS向Go值转换的信息，请看Unmarshal 函数的文档。

###func (*Decoder) UseNumber
```go
func (dec *Decoder) UseNumber()
```
UseNumber引起Decoder来将一个数字（作为一个Number而不是一个float64数值）解码成一个接口（interface{}）。

###type Encoder struct
```go
type Encoder struct {
}
```
Encoder向输出流写入JSON对象。

###func NewEncoder
```go
func NewEncoder(w io.Writer) *Encoder
```
NewEncoder返回一个写入`w`的新的encoder。

###func (*Encoder) Encode
```go
func (enc *Encoder) Encode(v interface{}) error
```
Encode将JSON编码的`v`写入到流中，接下来是换行符。

想了解更多关于Go值向JSON转换的信息，请看Marshal 函数的文档。

###type InvalidUTF8Error struct
```go
type InvalidUTF8Error struct {
    S string // the whole string value that caused the error
}
```
在GO1.2之前，当Marshal 尝试编码带有无效UTF-8序列的字符串时，会返回一个InvalidUTF8Error。在Go 1.2中，Marshal通过用 rune类型的U+FFFD（Unicode replacement）来替换非法的字节这种方法来强迫字符串转为UTF-8。这种错误不会产生但是为了向后兼容而保留下来。

###func (*InvalidUTF8Error) Error
```go
func (e *InvalidUTF8Error) Error() string
```

###type InvalidUnmarshalError struct
```go
type InvalidUnmarshalError struct {
    Type reflect.Type
}
```
InvalidUnmarshalError 描述了一个无效的传给Unmarshal的参数（传给Unmarshal的参数必须是一个非空指针）。

###func (*InvalidUnmarshalError) Error
```go
func (e *InvalidUnmarshalError) Error() string
```

###type Marshaler interface
```go
type Marshaler interface {
    MarshalJSON() ([]byte, error)
}
```
Marshaler 是一个接口。实现它的类型可以将它们自己编码为一个有效的JSON。

###type MarshalerError struct
```go
type MarshalerError struct {
    Type reflect.Type
    Err  error
}
```

###func (*MarshalerError) Error
```go
func (e *MarshalerError) Error() string
```

###type Number
```go
type Number string
```
一个Number代表了JSON数字。

###func (Number) Float64
```go
func (n Number) Float64() (float64, error)
```
Float64返回float64的数字。

###func (Number) Int64
```go
func (n Number) Int64() (int64, error)
```
返回int64的数字。

###func (Number) String
```go
func (n Number) String() string
```
返回数字的字符串形式。

###type RawMessage
```go
type RawMessage []byte
```
RawMessage是一个原始的编码的JSON对象。它实现了Marshaler 和Unmarshaler ，并可以用来延迟JSON解析或者JSON编码预计算。

###func (*RawMessage) MarshalJSON
```go
func (m *RawMessage) MarshalJSON() ([]byte, error)
```
返回`*m`作为`m`的JSON编码。

###func (*RawMessage) UnmarshalJSON
```go
func (m *RawMessage) UnmarshalJSON(data []byte) error
```
UnmarshalJSON 设置`*m`为data的一份拷贝。

###type SyntaxError struct
```go
type SyntaxError struct {
    Offset int64 // error occurred after reading Offset bytes
}
```
SyntaxError 描述了JSON的语法错误。

###func (*SyntaxError) Error
```go
func (e *SyntaxError) Error() string
```

###type UnmarshalFieldError struct
```go
type UnmarshalFieldError struct {
    Key   string
    Type  reflect.Type
    Field reflect.StructField
}
```
UnmarshalFieldError 描述了一个JSON对象key，这个key引起了一个没有导出的（因此为不可写的）的结构成员（ struct field）（不再使用，为了兼容性而保留）。

###func (*UnmarshalFieldError) Error
```go
func (e *UnmarshalFieldError) Error() string
```

###type UnmarshalTypeError struct
```go
type UnmarshalTypeError struct {
    Value string       // description of JSON value - "bool", "array", "number -5"
    Type  reflect.Type // type of Go value it could not be assigned to
}
```
UnmarshalTypeError描述了这样的一个JSON值，它不适合特定GO类型的值。

###func (*UnmarshalTypeError) Error
```go
func (e *UnmarshalTypeError) Error() string
```
Unmarshaler 是一个接口，实现它的接口可以unmarshal 一个它们自己的JSON描述。输入可以假设为一个JSON值的有效的编码。

UnmarshalJSON 如果想返回后仍然保留数据，它必须复制JSON数据。

###type Unmarshaler interface
```go
type Unmarshaler interface {
    UnmarshalJSON([]byte) error
}
```
当Marshal 尝试编码一个不支持的值类型，将返回UnsupportedTypeError 。

###type UnsupportedTypeError struct
```go
type UnsupportedTypeError struct {
    Type reflect.Type
}
```

###func (*UnsupportedTypeError) Error
```go
func (e *UnsupportedTypeError) Error() string
```

###type UnsupportedValueError struct
```go
type UnsupportedValueError struct {
    Value reflect.Value
    Str   string
}
```

###func (*UnsupportedValueError) Error
```go
func (e *UnsupportedValueError) Error() string
```

