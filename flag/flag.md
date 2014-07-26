# flag 包
import "flag"

---

##简介

##概览
flag包实现了命令行的参数解析。

使用flag.String()、Bool()、Int()等来定义flags。
以下声明了一个整形的flag。 `-flagname`存储在一个int型指针`ip`中。
```go
import "flag"
var ip = flag.Int("flagname", 1234, "help message for flagname")
```
当然你可以这样，使用Var()函数将flag绑定到一个变量上面。
```go
var flagvar int
func init() {
	flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")
}
```
或者，你可以创建自定义的flags，它满足指针接收，然后将它们一起进行flag解析：
```go
flag.Var(&flagVal, "name", "help message for flagname")
```
对于这样的flags,默认值就是变量的初始值。
定义了所有的flags，然后调用`flag.Parse()`进行命令行参数解析。

Flags可以直接使用。如果你用flags本身，他们就都是指针，如果你将他们绑定到变量，他们都是值。

```go
fmt.Println("ip has value ", *ip)
fmt.Println("flagvar has value ", flagvar)
```
解析完成以后，flag之后的参数就成了flag.Args()切片或者单独的flag.Args(i)。参数的索引从0直到flag.NArg()-1。

命令行语法：
```go
-flag
-flag=x
-flag x  // non-boolean flags only
```
一个或者两个减号都可以，是等效的。最后一种形式对于bool flags是禁止的，因为如果一个文件名为`0`、`false`，`cmd -x *`的意思会改变。

你必须使用`-flag=false`形式来关闭bool flag。在第一个non-flag参数之前或者在终止符`'--'`停止Flag解析。
整形flags接收1234, 0664, 0x1234 ，而且可以是负值。bool flags可以是：`1, 0, t, f, T, F, true, false, TRUE, FALSE, True, False`。

Duration flags接收任何符合time.ParseDuration的有效的输入参数。

顶层的函数控制默认的命令行flags集合。FlagSet的方法模拟了顶层函数定义了命令行 允许一个定义flags独立的集合，例如在一个命令行接口来实现子命令
。FlagSet的方法模拟了顶层函数定义了命令行flag集合。

##变量
```go
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)
```
命令行是一个默认的命令行flags集合，从os.Args解析而来。顶层函数比如BoolVar、Arg和on都包含了命令行方法。
```go
var ErrHelp = errors.New("flag: help requested")
```
当flag-help被触发，但是没有flag定义，返回ErrHelp。

```go
var Usage = func() {
    fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
    PrintDefaults()
}
```
Usage指向标准错误输出，打印出用法错误信息。这个函数是一个变量，可以指向一个自定义的函数。

###func Arg
```go
func Arg(i int) string
```

###func Args
```go
func Args() []string
```

###func Bool
```go
func Bool(name string, value bool, usage string) *bool
```

###func BoolVar
```go
func BoolVar(p *bool, name string, value bool, usage string)
```

###func Duration
```go
func Duration(name string, value time.Duration, usage string) *time.Duration
```

###func DurationVar
```go
func DurationVar(p *time.Duration, name string, value time.Duration, usage string)
```

###func Float64
```go
func Float64(name string, value float64, usage string) *float64
```

###func Float64Var
```go
func Float64Var(p *float64, name string, value float64, usage string)
```

###func Int
```go
func Int(name string, value int, usage string) *int
```

###func Int64
```go
func Int64(name string, value int64, usage string) *int64
```

###func Int64Var
```go
func Int64Var(p *int64, name string, value int64, usage string)
```

###func IntVar
```go
func IntVar(p *int, name string, value int, usage string)
```

###func NArg
```go
func NArg() int
```

###func NFlag
```go
func NFlag() int
```

###func Parse
```go
func Parse()
```

###func Parsed
```go
func Parsed() bool
```

###func PrintDefaults
```go
func PrintDefaults()
```

###func Set
```go
func Set(name, value string) error
```

###func String
```go
func String(name string, value string, usage string) *string
```

###func StringVar
```go
func StringVar(p *string, name string, value string, usage string)
```

###func Uint
```go
func Uint(name string, value uint, usage string) *uint
```

###func Uint64
```go
func Uint64(name string, value uint64, usage string) *uint64
```

###func Uint64Var
```go
func Uint64Var(p *uint64, name string, value uint64, usage string)
```

###func UintVar
```go
func UintVar(p *uint, name string, value uint, usage string)
```

###func Var
```go
func Var(value Value, name string, usage string)
```

###func Visit
```go
func Visit(fn func(*Flag))
```

###func VisitAll
```go
func VisitAll(fn func(*Flag))
```

###type ErrorHandling
```go
type ErrorHandling int
```

###type Flag
```go
type Flag struct {
    Name     string // name as it appears on command line
    Usage    string // help message
    Value    Value  // value as set
    DefValue string // default value (as text); for usage message
}
```

###func Lookup
```go
func Lookup(name string) *Flag
```

###type FlagSet
```go
type FlagSet struct {
    // Usage is the function called when an error occurs while parsing flags.
    // The field is a function (not a method) that may be changed to point to
    // a custom error handler.
    Usage func()
    // contains filtered or unexported fields
}
```

###func NewFlagSet
```go
func NewFlagSet(name string, errorHandling ErrorHandling) *FlagSet
```

###func (*FlagSet) Arg
```go
func (f *FlagSet) Arg(i int) string
```

###func (*FlagSet) Args
```go
func (f *FlagSet) Args() []string
```

###func (*FlagSet) Bool
```go
func (f *FlagSet) Bool(name string, value bool, usage string) *bool
```

###func (*FlagSet) BoolVar
```go
func (f *FlagSet) BoolVar(p *bool, name string, value bool, usage string)
```

###func (*FlagSet) Duration
```go
func (f *FlagSet) Duration(name string, value time.Duration, usage string) *time.Duration
```

###func (*FlagSet) DurationVar
```go
func (f *FlagSet) DurationVar(p *time.Duration, name string, value time.Duration, usage string)
```

###func (*FlagSet) Float64
```go
func (f *FlagSet) Float64(name string, value float64, usage string) *float64
```

###func (*FlagSet) Float64Var
```go
func (f *FlagSet) Float64Var(p *float64, name string, value float64, usage string)
```

###func (*FlagSet) Init
```go
func (f *FlagSet) Init(name string, errorHandling ErrorHandling)
```

###func (*FlagSet) Int
```go
func (f *FlagSet) Int(name string, value int, usage string) *int
```

###func (*FlagSet) Int64
```go
func (f *FlagSet) Int64(name string, value int64, usage string) *int64
```

###func (*FlagSet) Int64Var
```go
func (f *FlagSet) Int64Var(p *int64, name string, value int64, usage string)
```

###func (*FlagSet) IntVar
```go
func (f *FlagSet) IntVar(p *int, name string, value int, usage string)
```

###func (*FlagSet) Lookup
```go
func (f *FlagSet) Lookup(name string) *Flag
```

###func (*FlagSet) NArg
```go
func (f *FlagSet) NArg() int
```

###func (*FlagSet) NFlag
```go
func (f *FlagSet) NFlag() int
```

###func (*FlagSet) Parse
```go
func (f *FlagSet) Parse(arguments []string) error
```

###func (*FlagSet) Parsed
```go
func (f *FlagSet) Parsed() bool
```

###func (*FlagSet) PrintDefaults
```go
func (f *FlagSet) PrintDefaults()
```

###func (*FlagSet) Set
```go
func (f *FlagSet) Set(name, value string) error
```

###func (*FlagSet) SetOutput
```go
func (f *FlagSet) SetOutput(output io.Writer)
```

###func (*FlagSet) String
```go
func (f *FlagSet) String(name string, value string, usage string) *string
```

###func (*FlagSet) StringVar
```go
func (f *FlagSet) StringVar(p *string, name string, value string, usage string)
```

###func (*FlagSet) Uint
```go
func (f *FlagSet) Uint(name string, value uint, usage string) *uint
```

###func (*FlagSet) Uint64
```go
func (f *FlagSet) Uint64(name string, value uint64, usage string) *uint64
```

###func (*FlagSet) Uint64Var
```go
func (f *FlagSet) Uint64Var(p *uint64, name string, value uint64, usage string)
```

###func (*FlagSet) UintVar
```go
func (f *FlagSet) UintVar(p *uint, name string, value uint, usage string)
```

###func (*FlagSet) Var
```go
func (f *FlagSet) Var(value Value, name string, usage string)
```

###func (*FlagSet) Visit
```go
func (f *FlagSet) Visit(fn func(*Flag))
```

###func (*FlagSet) VisitAll
```go
func (f *FlagSet) VisitAll(fn func(*Flag))
```

###type Getter
```go
type Getter interface {
    Value
    Get() interface{}
}
```

###type Value
```go
type Value interface {
    String() string
    Set(string) error
}
```