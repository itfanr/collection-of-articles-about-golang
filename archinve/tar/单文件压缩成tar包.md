archinve/tar包例子：

示例1：

单个文件的压缩（压缩后格式为`.tar`）：
例子用到的主要函数：

 - func Create(name string) (file *File, err error)
 - func Stat(name string) (fi FileInfo, err error)
 - func FileInfoHeader(fi os.FileInfo, link string) (*Header, error)
 - func (tw *Writer) WriteHeader(hdr *Header) error
 - func Copy(dst Writer, src Reader) (written int64, err error)

`.tar`文件的组成：存储在`.tar`文件中的每个文件都由两部分组成：文件信息（`tar.Header`结构体，其实就是linux下执行`ll`命令得到的文件信息）和文件内容。和linux文件系统管理文件类似，inode保存了文件相关的创建人、权限、文件大小等信息，文件的内容保存在block中。

先创建一个os.Writer对象，将该对象传给tar.NewWriter（构造函数），这样就可以将打包后的数据写入文件中。

向`.tar`文件中写入文件要分两步：第一步写入文件信息，第二步写入文件数据。对于目录来说，由于没有内容可写，所以只需要写入目录信息即可。`tar.FileInfoHeader`函数可以直接通过 `os.FileInfo`创建`tar.Header`，并自动填写 `tar.Header` 中的大部分信息。

注意：一定要执行 tw.Close() 操作，因为 tar.Writer 使用了缓存，tw.Close() 会将缓存中的数据写入到文件中，同时 tw.Close() 还会向 .tar 文件的最后写入结束信息，如果不关闭 tw 而直接退出程序，那么将导致 .tar 文件不完整。先关闭tar.Writer，再关闭os.Wtirer，如果使用defer，注意LIFO。

```
package main

import (
	"archive/tar"
	"errors"
	"fmt"
	"io"
	"os"
	"path"
)

func main() {
	TarFile := `C:\Users\itfanr\Desktop\test_tar.tar`
	src := "C:\\Users\\itfanr\\Desktop\\test_tar.txt"
	if err := Tar(src, TarFile, false); err != nil {
		fmt.Println(err)
	}
}

// src 是要打包的文件
// dstTar 是要生成的 .tar 文件的路径
// overwrite标记如果 dstTar 文件存在，是否覆盖。如果存在但不覆盖，则放弃打包。
func Tar(src string, dstTar string, overwrite bool) (err error) {
	// 清理路径字符串
	src = path.Clean(src)
	// 判断要打包的文件或目录是否存在
	if !Exists(src) {
		return errors.New("要打包的文件不存在：" + src)
	}
	// 判断目标文件是否存在
	if FileExists(dstTar) {
		if overwrite { // 不覆盖已存在的文件
			return errors.New("目标文件已经存在：" + dstTar)
		} else { // 覆盖已存在的文件
			if er := os.Remove(dstTar); er != nil {
				return er
			}
		}
	}
	// 创建空的目标文件,也可以用os.OpenFile函数
	//函数原型：
	//func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
	fw, er := os.Create(dstTar)
	if er != nil {
		return er
	}
	defer fw.Close()
	// 创建 tar.Writer，执行打包操作
	tw := tar.NewWriter(fw)
	defer func() {
		// 这里要判断 tw 是否关闭成功，如果关闭失败，则 .tar 文件可能不完整
		if er := tw.Close(); er != nil {
			err = er
		}
	}()
	// 获取文件信息
	fi, er := os.Stat(src)
	if er != nil {
		return er
	}
	// 开始打包
	tarFile(src, tw, fi)
	return nil
}

func tarFile(srcFile string, tw *tar.Writer, fi os.FileInfo) (err error) {
	// 获取要打包的文件所在位置和名称
	_, srcRelative := path.Split(path.Clean(srcFile))
	// 写入文件信息
	hdr, er := tar.FileInfoHeader(fi, "")
	if er != nil {
		return er
	}
	hdr.Name = srcRelative
	if er = tw.WriteHeader(hdr); er != nil {
		return er
	}
	// 打开要打包的文件，准备读取
	fr, er := os.Open(srcFile)
	if er != nil {
		return er
	}
	defer fr.Close()
	// 将文件数据写入 tw 中
	copied, er := io.Copy(tw, fr)
	if er != nil {
		return er
	}
	if copied < fi.Size() {
		//fmt.Errorf函数也可以用来创建错误信息
		//原型：func Errorf(format string, a ...interface{}) error
		msg := "已经写入%d 字节 , 应写入 %d 字节"
		return fmt.Errorf(msg, copied, fi.Size())
	}
	return nil
}

// 判断档案是否存在
func Exists(name string) bool {
	_, err := os.Stat(name)
	return err == nil || os.IsExist(err)
}

// 判断文件是否存在
func FileExists(filename string) bool {
	fi, err := os.Stat(filename)
	return (err == nil || os.IsExist(err)) && !fi.IsDir()
}

// 判断目录是否存在
func DirExists(dirname string) bool {
	fi, err := os.Stat(dirname)
	return (err == nil || os.IsExist(err)) && fi.IsDir()
}
```
参考：
  [1] http://www.cnblogs.com/golove/p/3454630.html "Golang创建 .tar.gz 压缩包"
  [2] https://leanpub.com/go-thestdlib "Go, The Standard Library
Real Code. Real Productivity. Master The Go Standard Library"		