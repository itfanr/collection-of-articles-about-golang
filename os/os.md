#os包

---

##简介

##概览


###变量


###func Chdir
```go
func Chdir(dir string) error
```

###func Chmod
```go
func Chmod(name string, mode FileMode) error
```

###func Chown
```go
func Chown(name string, uid, gid int) error
```

###func Chtimes
```go
func Chtimes(name string, atime time.Time, mtime time.Time) error
```

###func Clearenv
```go
func Clearenv()
```

###func Environ
```go
func Environ() []string
```

###func Exit
```go
func Exit(code int)
```

###func Expand
```go
func Expand(s string, mapping func(string) string) string
```

###func ExpandEnv
```go
func ExpandEnv(s string) string
```

###func Getegid
```go
func Getegid() int
```

###func Getenv
```go
func Getenv(key string) string
```

###func Geteuid
```go
func Geteuid() int
```

###func Getgid
```go
func Getgid() int
```

###func Getgroups
```go
func Getgroups() ([]int, error)
```

###func Getpagesize
```go
func Getpagesize() int
```

###func Getpid
```go
func Getpid() int
```

###func Getppid
```go
func Getppid() int
```

###func Getuid
```go
func Getuid() int
```

###func Getwd
```go
func Getwd() (dir string, err error)
```

###func Hostname
```go
func Hostname() (name string, err error)
```

###func IsExist
```go
func IsExist(err error) bool
```

###func IsNotExist
```go
func IsNotExist(err error) bool
```

###func IsPathSeparator
```go
func IsPathSeparator(c uint8) bool
```

###func IsPermission
```go
func IsPermission(err error) bool
```

###func Lchown
```go
func Lchown(name string, uid, gid int) error
```

###func Link
```go
func Link(oldname, newname string) error
```

###func Mkdir
```go
func Mkdir(name string, perm FileMode) error
```

###func MkdirAll
```go
func MkdirAll(path string, perm FileMode) error
```

###func NewSyscallError
```go
func NewSyscallError(syscall string, err error) error
```

###func Readlink
```go
func Readlink(name string) (string, error)
```

###func Remove
```go
func Remove(name string) error
```

###func RemoveAll
```go
func RemoveAll(path string) error
```

###func Rename
```go
func Rename(oldpath, newpath string) error
```

###func SameFile
```go
func SameFile(fi1, fi2 FileInfo) bool
```

###func Setenv
```go
func Setenv(key, value string) error
```

###func Symlink
```go
func Symlink(oldname, newname string) error
```

###func TempDir
```go
func TempDir() string
```

###func Truncate
```go
func Truncate(name string, size int64) error
```

###type File struct
```go
type File struct {
}
```

###func Create
```go
func Create(name string) (file *File, err error)
```

###func NewFile
```go
func NewFile(fd uintptr, name string) *File
```

###func Open
```go
func Open(name string) (file *File, err error)
```

###func OpenFile
```go
func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
```

###func Pipe
```go
func Pipe() (r *File, w *File, err error)
```

###func (*File) Chdir
```go
func (f *File) Chdir() error
```

###func (*File) Chmod
```go
func (f *File) Chmod(mode FileMode) error
```

###func (*File) Chown
```go
func (f *File) Chown(uid, gid int) error
```

###func (*File) Close
```go
func (file *File) Close() error
```

###func (*File) Fd
```go
func (file *File) Fd() uintptr
```

###func (*File) Name
```go
func (f *File) Name() string
```

###func (*File) Read
```go
func (f *File) Read(b []byte) (n int, err error)
```

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

