#os包

import "os"
---

##简介

##概览
os包提供了平台独立的接口来操作系统。设计是Unix—-like的的，虽然错误处理Go-like的，失败的调用会返回error类型的值而不是错误代码。常常从error能得到更多的信息。比如，如果通过文件名的调用，比如Open或者Stat，错误会包括失败的文件名（当打印的时候）和*PathError类型，这个类型可以解包然后得到更多信息。

os接口的想法是实现对所有的操作系统统一的操作。这种特性一般不会在一些带有系统特色的syscall包中。

下面是一个简单的例子，打开文件然后读取它。

```go
file, err := os.Open("file.go") // For read access.
if err != nil {
	log.Fatal(err)
}
```

如果打开失败，错误字符串会是不言而喻的，就像这样：
```go
open file.go: no such file or directory
```

文件的数据会被读入字节切片中。读取和写入会从切片参数得到字节数：
```go
data := make([]byte, 100)
count, err := file.Read(data)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("read %d bytes: %q\n", count, data[:count])
```

###变量


###func Chdir
```go
func Chdir(dir string) error
```
切换到`dir`目录。如果出现错误，返回的错误类型为 *PathError类型。

###func Chmod
```go
func Chmod(name string, mode FileMode) error
```
改变名为`name`文件的mode为`mode`。如果文件是一个符号链接，它会改变链接目标文件的mode。如果出现错误，返回的错误类型为 *PathError类型。

###func Chown
```go
func Chown(name string, uid, gid int) error
```
改变名为`name`文件的`uid`和`gid`。如果文件是一个符号链接，它会改变链接目标文件的`uid`和`gid`。如果出现错误，返回的错误类型为 *PathError类型。

###func Chtimes
```go
func Chtimes(name string, atime time.Time, mtime time.Time) error
```
Chtimes 改变访问和修改名为`name`文件的时间，类似于Unix的utime()和utimes()函数。

底层文件系统可能取值的truncate或者round来降低时间单元的准确性。如果出现错误，返回的错误类型为 *PathError类型。

###func Clearenv
```go
func Clearenv()
```
删除所有的环境变量。

###func Environ
```go
func Environ() []string
```
返回环境变量，形式为“key=value”。

###func Exit
```go
func Exit(code int)
```
Exit引起当前程序退出并给出状态码。按照惯例，0代表成功，非零代表错误。程序立马退出，defer函数不会执行。

###func Expand
```go
func Expand(s string, mapping func(string) string) string
```
Expand用mapping 函数指定的规则替换字符串中的${var}或者$var。比如，os.ExpandEnv(s)等效于os.Expand(s, os.Getenv)。

###func ExpandEnv
```go
func ExpandEnv(s string) string
```
ExpandEnv根据当前环境变量的值来替换字符串中的${var}或者$var。如果引用变量没有定义，则用空字符串替换。

###func Getegid
```go
func Getegid() int
```
返回调用者的 effective group id。

###func Getenv
```go
func Getenv(key string) string
```
获取环境变量中与`key`对应的值。如果变量值不存在，返回空。

###func Geteuid
```go
func Geteuid() int
```
返回调用者的user id。

###func Getgid
```go
func Getgid() int
```
返回调用者的 group id。

###func Getgroups
```go
func Getgroups() ([]int, error)
```
返回调用者所属的一些列的groutps的id。

###func Getpagesize
```go
func Getpagesize() int
```
返回底层系统的内存的页面大小。

###func Getpid
```go
func Getpid() int
```
返回调用者的进程id。

###func Getppid
```go
func Getppid() int
```
返回调用者的父进程id。

###func Getuid
```go
func Getuid() int
```
返回调用者的user id。

###func Getwd
```go
func Getwd() (dir string, err error)
```
返回相对于当前目录的根目录名。如果当前努力可以被多个路径获得（因为符号链接），Getwd可能返回其中的一个。

###func Hostname
```go
func Hostname() (name string, err error)
```
返回内核报告的主机名。

###func IsExist
```go
func IsExist(err error) bool
```
返回一个布尔值，它指明`err`错误是否报告了一个文件或者目录已经存在。它被ErrExist和其它系统调用满足。

###func IsNotExist
```go
func IsNotExist(err error) bool
```
返回一个布尔值，它指明`err`错误是否报告了一个文件或者目录不存在。它被ErrNotExist 和其它系统调用满足。

###func IsPathSeparator
```go
func IsPathSeparator(c uint8) bool
```
如果`c`是一个目录分隔符，则返回true。

###func IsPermission
```go
func IsPermission(err error) bool
```
返回一个布尔值，它指明`err`错误是否报告了权限不足。它被ErrPermission 和其它系统调用满足。

###func Lchown
```go
func Lchown(name string, uid, gid int) error
```
改变了文件的`gid`和`uid`。如果文件是一个符号链接，它改变的链接自己。如果出错，则会是*PathError类型。

###func Link
```go
func Link(oldname, newname string) error
```
创建了一个指向`oldname`文件的硬链接`newname`。如果出错，将是*LinkError类型。

###func Mkdir
```go
func Mkdir(name string, perm FileMode) error
```
创建了新的目录，带有权限位`perm`。如果出错，将是*PathError类型。

###func MkdirAll
```go
func MkdirAll(path string, perm FileMode) error
```
MkdirAll创建目录，并带有必须的父目录，返回空或者返回一个错误。MkdirAll创建的所有的目录都会使用权限位`perm`。如果`path`已经是一个目录，MkdirAll将不会做什么并返回空。

###func NewSyscallError
```go
func NewSyscallError(syscall string, err error) error
```
NewSyscallError返回一个SyscallError 错误，带有给出的系统调用名字和详细的错误信息。为了方便，如果err为空，NewSyscallError 返回空。 

###func Readlink
```go
func Readlink(name string) (string, error)
```
返回符号链接的目标。如果出错，将会是 *PathError类型。

###func Remove
```go
func Remove(name string) error
```
删除文件或者目录。如果出错，将是*PathError类型。

###func RemoveAll
```go
func RemoveAll(path string) error
```
删除目录和其子文件（如果存在）。删除它所能删除的东西，并返回遇到的第一个错误。如果`path`不存在，RemoveAll 返回空（不是error。

###func Rename
```go
func Rename(oldpath, newpath string) error
```
重命名（移动）文件。可能使用OS-specific限制。

###func SameFile
```go
func SameFile(fi1, fi2 FileInfo) bool
```
报告`f1`和`f2`是否是同一个文件。比如，在Unix系统，这意味着两个底层结构的设备和inode域是完全相同的；在其他系统，可能要基于路径名字来识别。SameFile只应用与本包的Stat返回的结果。在其他情形将会返回false。

###func Setenv
```go
func Setenv(key, value string) error
```
设置环境变量。如果出错，则返回它。

###func Symlink
```go
func Symlink(oldname, newname string) error
```
创建了一个指向`oldname`的符号链接`newname`。如果出错，将会是*LinkError类型。

###func TempDir
```go
func TempDir() string
```
返回默认的临时文件目录。

###func Truncate
```go
func Truncate(name string, size int64) error
```
改变文件的。如果文件是符号链接，它改变的符号链接的目标文件。如果出错，将会是 *PathError类型。

###type File struct
```go
type File struct {
}
```
File代表打开的文件描述符。

###func Create
```go
func Create(name string) (file *File, err error)
```
创建文件，mode为0666（umask之前），如果文件已经存在则截断。如果成功，返回文件的方法可以用来进行I/O操作，文件描述符的mode为O_RDWR。如果出错，将会是 *PathError类型。

###func NewFile
```go
func NewFile(fd uintptr, name string) *File
```
给定文件描述符和`name`，返回一个新文件。

###func Open
```go
func Open(name string) (file *File, err error)
```
打开文件来读。如果成功，返回文件上的方法可以被用来读取，文件描述符的mode为O_RMONLY。如果出错，将会是 *PathError类型。

###func OpenFile
```go
func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
```
OpenFile是一个通用的open调用，大多数应该用Open或者Create代替。它打开文件带有特定的flag（O_RDONLY等）和perm（0666等），如果合适的话。
如果成功，返回文件的方法可以用来进行I/O操作。如果出错，将会是 *PathError类型。

###func Pipe
```go
func Pipe() (r *File, w *File, err error)
```
返回连接着的一对文件。从`r`读取字节然后写入`w`。它返回文件和错误（存在的）。

###func (*File) Chdir
```go
func (f *File) Chdir() error
```
改变工作目录到`file`，它必须是一个目录。如果出错，将会是 *PathError类型。

###func (*File) Chmod
```go
func (f *File) Chmod(mode FileMode) error
```
Chmod改变文件的mode。如果出错，将会是 *PathError类型。

###func (*File) Chown
```go
func (f *File) Chown(uid, gid int) error
```
改变文件的`uid`和`gid`。如果出错，将会是 *PathError类型。

###func (*File) Close
```go
func (file *File) Close() error
```
关闭文件，使它为I/O可用。如果有错，则返回。

###func (*File) Fd
```go
func (file *File) Fd() uintptr
```
返回Unix文件描述符。

###func (*File) Name
```go
func (f *File) Name() string
```
返回通过Open给出的文件的名字。

###func (*File) Read
```go
func (f *File) Read(b []byte) (n int, err error)
```
将文件的len(b)字节的数据读入字节切片。返回已经读取的字节数和错误（如果存在）。EOF意味着读取了零字节并设置`err`为io.EOF。

###func (*File) ReadAt
```go
func (f *File) ReadAt(b []byte, off int64) (n int, err error)
```

###func (*File) Readdir
```go
func (f *File) Readdir(n int) (fi []FileInfo, err error)
```

###func (*File) Readdirnames
```go
func (f *File) Readdirnames(n int) (names []string, err error)
```

###func (*File) Seek
```go
func (f *File) Seek(offset int64, whence int) (ret int64, err error)
```

###func (*File) Stat
```go
func (file *File) Stat() (fi FileInfo, err error)
```

###func (*File) Sync
```go
func (f *File) Sync() (err error)
```

###func (*File) Truncate
```go
func (f *File) Truncate(size int64) error
```

###func (*File) Write
```go
func (f *File) Write(b []byte) (n int, err error)
```

###func (*File) WriteAt
```go
func (f *File) WriteAt(b []byte, off int64) (n int, err error)
```

###func (*File) WriteString
```go
func (f *File) WriteString(s string) (ret int, err error)
```

###type FileInfo interface
```go
type FileInfo interface {
    Name() string       // base name of the file
    Size() int64        // length in bytes for regular files; system-dependent for others
    Mode() FileMode     // file mode bits
    ModTime() time.Time // modification time
    IsDir() bool        // abbreviation for Mode().IsDir()
    Sys() interface{}   // underlying data source (can return nil)
}
```

###func Lstat
```go
func Lstat(name string) (fi FileInfo, err error)
```

###func Stat
```go
func Stat(name string) (fi FileInfo, err error)
```

###func (FileMode) IsDir
```go
func (m FileMode) IsDir() bool
```

###func (FileMode) IsRegular
```go
func (m FileMode) IsRegular() bool
```

###func (FileMode) Perm
```go
func (m FileMode) Perm() FileMode
```

###func (FileMode) String
```go
func (m FileMode) String() string
```

###type LinkError struct
```go
type LinkError struct {
    Op  string
    Old string
    New string
    Err error
}
```

###func (*LinkError) Error
```go
func (e *LinkError) Error() string
```

###type PathError struct
```go
type PathError struct {
    Op   string
    Path string
    Err  error
}
```

###func (*PathError) Error
```go
func (e *PathError) Error() string
```

###type ProcAttr struct
```go
type ProcAttr struct {
    Dir string
    Env []string
    Files []*File

    Sys *syscall.SysProcAttr
}
```

###type Process struct
```go
type Process struct {
    Pid int
}
```

###func FindProcess
```go
func FindProcess(pid int) (p *Process, err error)
```

###func StartProcess
```go
func StartProcess(name string, argv []string, attr *ProcAttr) (*Process, error)
```

###func (*Process) Kill
```go
func (p *Process) Kill() error
```

###func (*Process) Release
```go
func (p *Process) Release() error
```

###func (*Process) Signal
```go
func (p *Process) Signal(sig Signal) error
```

###func (*Process) Wait
```go
func (p *Process) Wait() (*ProcessState, error)
```

###type ProcessState struct
```go
type ProcessState struct {
}
```

###func (*ProcessState) Exited
```go
func (p *ProcessState) Exited() bool
```

###func (*ProcessState) Pid
```go
func (p *ProcessState) Pid() int
```

###func (*ProcessState) String
```go
func (p *ProcessState) String() string
```

###func (*ProcessState) Success
```go
func (p *ProcessState) Success() bool
```

###func (*ProcessState) Sys
```go
func (p *ProcessState) Sys() interface{}
```

###func (*ProcessState) SysUsage
```go
func (p *ProcessState) SysUsage() interface{}
```

###func (*ProcessState) SystemTime
```go
func (p *ProcessState) SystemTime() time.Duration
```

###func (*ProcessState) UserTime
```go
func (p *ProcessState) UserTime() time.Duration
```

###type Signal interface
```go
type Signal interface {
    String() string
    Signal() // to distinguish from other Stringers
}
```

###type SyscallError struct
```go
type SyscallError struct {
    Syscall string
    Err     error
}
```

###func (*SyscallError) Error
```go
func (e *SyscallError) Error() string
```

