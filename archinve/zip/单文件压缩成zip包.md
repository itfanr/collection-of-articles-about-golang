# 单文件打包成zip文件

标签（空格分隔）： tar 

---
示例1：

单个文件的压缩（压缩后格式为`.zip`）：

例子用到的主要函数：

 -  func NewWriter(w io.Writer) *Writer
 -  func (w *Writer) Create(name string) (io.Writer, error)
 -  func (w *Writer) CreateHeader(fh *FileHeader) (io.Writer, error)

创建zip包和tar包类似，不同的是zip包有一个方便的工具让你快速地把文件写入压缩包。我们从得到NewWriter得到Writer对象后，可以直接使用Create方法来向zip文件写入内容，不需要自己写文件元数据。当然，你也可以自己用CreateHeader函数去写FileHeader。

```
package main

import (
	"archive/zip"
	"fmt"
	"io"
	"log"
	"os"
)

func main() {
	filename := "test_zip.txt"
	//熟悉linux的同学肯定知道位或吧
	flags := os.O_WRONLY | os.O_CREATE | os.O_TRUNC
	//0644是unmask信息，表示用户、用户组、其他组的权限
	file, err := os.OpenFile("test_zip.zip", flags, 0644)
	if err != nil {
		log.Fatalf("为创建压缩包，打开zip文件失败", err)
	}
	defer file.Close()
	zw := zip.NewWriter(file)
	defer zw.Close()
	if err := zipFile(filename, zw); err != nil {
		log.Fatal(err)
	}

}

func zipFile(filename string, zw *zip.Writer) error {
	file, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("打开文件%s失败：%s", filename, err)
	}
	defer file.Close()
	iw, err := zw.Create(filename)
	if err != nil {
		return err
	}
	//此处不检查写入压缩包的文件字节数。
	//FileHeader没有文件的size信息。
	if _, err := io.Copy(iw, file); err != nil {
		return fmt.Errorf("将文件 %s 写入 zip压缩包失败: %s", filename, err)
	}
	return nil
}

```
注意：Create函数要求参数name必须是相对路径（包含正斜杠`/`），也就是路径最开始不能是盘符或者`/`。Create函数和CreateHeader要求，下次调用Create、CreateHeader或者Close前必须将文件内容写入io.Writer。




