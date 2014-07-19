# ioutil包

---

ioutil包含有一些公用的I/O工具函数。 

变量 

var Discard io.Writer = devNull(0) 
Discard是一个io.Writer，对其进行的所有Write呼叫都会成功但不会做任何实际的操作。 

func NopCloser(r io.Reader) io.ReadCloser 
NopCloser返回一个包装r参数而来的ReadCloser接口，该接口仅提供Close方法。 

func ReadAll(r io.Reader) ([]byte, error) 
ReadAll从r读取直到遇到error或EOF并返回读取的数据。 成功的调用返回的err为nil，而不是EOF。因为ReadAll定义为从资源读取数据直到EOF，它不会将从r读取的EOF视为应该报告的错误。 

func ReadDir(dirname string) ([]os.FileInfo, error) 
ReadDir接受dirname指定的目录，并返回一个有序的、子目录信息的列表。 

func ReadFile(filename string) ([]byte, error) 
ReadFile从filename指定的文件中读取数据并返回文件的内容。 成功的调用返回的err为nil，而不是EOF。因为ReadFile定义为从资源读取数据直到EOF，它不会将从r读取的EOF视为应该报告的错误。 

func TempDir(dir, prefix string) (name string, err error) 
TempDir在指定的目录里创建一个新的、使用prfix作为前缀的临时文件夹，并返回文件夹的路径。 如果dir是空字符串，TempDir使用默认用于临时文件的目录（参见os.TempDir函数）。 如果多个程序调用该函数的话，将会创建不同的临时目录（因此是线程安全的）。调用本函数的程序有责任在不需要临时文件夹时摧毁它。 

func TempFile(dir, prefix string) (f *os.File, err error) 
TempFile在dir目录下创建一个新的、使用prefix为前缀的临时文件，并以读写模式打开该文件并返回os.File指针。 如果dir是空字符串，TempFile使用默认用于临时文件的目录（参见os.TempDir函数）。 如果多个程序调用该函数的话，将会创建不同的临时文件（因此是线程安全的）。调用本函数的程序有责任在不需要临时文件时摧毁它。 

func WriteFile(filename string, data []byte, perm os.FileMode) error 
WriteFile向filename指定的文件中写入数据。如果文件不存在将按给出的权限创建该文件，否则本函数会在写入数据之前截断文件（即清空之）。
参考：

 1. http://www.open-open.com/lib/view/open1352160969485.html

