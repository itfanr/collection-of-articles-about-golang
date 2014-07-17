archinve/tar�����ӣ�

ʾ��1��

�����ļ���ѹ����ѹ�����ʽΪ`.tar`����
�����õ�����Ҫ������

 - func Create(name string) (file *File, err error)
 - func Stat(name string) (fi FileInfo, err error)
 - func FileInfoHeader(fi os.FileInfo, link string) (*Header, error)
 - func (tw *Writer) WriteHeader(hdr *Header) error
 - func Copy(dst Writer, src Reader) (written int64, err error)

`.tar`�ļ�����ɣ��洢��`.tar`�ļ��е�ÿ���ļ�������������ɣ��ļ���Ϣ��`tar.Header`�ṹ�壬��ʵ����linux��ִ��`ll`����õ����ļ���Ϣ�����ļ����ݡ���linux�ļ�ϵͳ�����ļ����ƣ�inode�������ļ���صĴ����ˡ�Ȩ�ޡ��ļ���С����Ϣ���ļ������ݱ�����block�С�

�ȴ���һ��os.Writer���󣬽��ö��󴫸�tar.NewWriter�����캯�����������Ϳ��Խ�����������д���ļ��С�

��`.tar`�ļ���д���ļ�Ҫ����������һ��д���ļ���Ϣ���ڶ���д���ļ����ݡ�����Ŀ¼��˵������û�����ݿ�д������ֻ��Ҫд��Ŀ¼��Ϣ���ɡ�`tar.FileInfoHeader`��������ֱ��ͨ�� `os.FileInfo`����`tar.Header`�����Զ���д `tar.Header` �еĴ󲿷���Ϣ��

ע�⣺һ��Ҫִ�� tw.Close() ��������Ϊ tar.Writer ʹ���˻��棬tw.Close() �Ὣ�����е�����д�뵽�ļ��У�ͬʱ tw.Close() ������ .tar �ļ������д�������Ϣ��������ر� tw ��ֱ���˳�������ô������ .tar �ļ����������ȹر�tar.Writer���ٹر�os.Wtirer�����ʹ��defer��ע��LIFO��

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

// src ��Ҫ������ļ�
// dstTar ��Ҫ���ɵ� .tar �ļ���·��
// overwrite������ dstTar �ļ����ڣ��Ƿ񸲸ǡ�������ڵ������ǣ�����������
func Tar(src string, dstTar string, overwrite bool) (err error) {
	// ����·���ַ���
	src = path.Clean(src)
	// �ж�Ҫ������ļ���Ŀ¼�Ƿ����
	if !Exists(src) {
		return errors.New("Ҫ������ļ������ڣ�" + src)
	}
	// �ж�Ŀ���ļ��Ƿ����
	if FileExists(dstTar) {
		if overwrite { // �������Ѵ��ڵ��ļ�
			return errors.New("Ŀ���ļ��Ѿ����ڣ�" + dstTar)
		} else { // �����Ѵ��ڵ��ļ�
			if er := os.Remove(dstTar); er != nil {
				return er
			}
		}
	}
	// �����յ�Ŀ���ļ�,Ҳ������os.OpenFile����
	//����ԭ�ͣ�
	//func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
	fw, er := os.Create(dstTar)
	if er != nil {
		return er
	}
	defer fw.Close()
	// ���� tar.Writer��ִ�д������
	tw := tar.NewWriter(fw)
	defer func() {
		// ����Ҫ�ж� tw �Ƿ�رճɹ�������ر�ʧ�ܣ��� .tar �ļ����ܲ�����
		if er := tw.Close(); er != nil {
			err = er
		}
	}()
	// ��ȡ�ļ���Ϣ
	fi, er := os.Stat(src)
	if er != nil {
		return er
	}
	// ��ʼ���
	tarFile(src, tw, fi)
	return nil
}

func tarFile(srcFile string, tw *tar.Writer, fi os.FileInfo) (err error) {
	// ��ȡҪ������ļ�����λ�ú�����
	_, srcRelative := path.Split(path.Clean(srcFile))
	// д���ļ���Ϣ
	hdr, er := tar.FileInfoHeader(fi, "")
	if er != nil {
		return er
	}
	hdr.Name = srcRelative
	if er = tw.WriteHeader(hdr); er != nil {
		return er
	}
	// ��Ҫ������ļ���׼����ȡ
	fr, er := os.Open(srcFile)
	if er != nil {
		return er
	}
	defer fr.Close()
	// ���ļ�����д�� tw ��
	copied, er := io.Copy(tw, fr)
	if er != nil {
		return er
	}
	if copied < fi.Size() {
		//fmt.Errorf����Ҳ������������������Ϣ
		//ԭ�ͣ�func Errorf(format string, a ...interface{}) error
		msg := "�Ѿ�д��%d �ֽ� , Ӧд�� %d �ֽ�"
		return fmt.Errorf(msg, copied, fi.Size())
	}
	return nil
}

// �жϵ����Ƿ����
func Exists(name string) bool {
	_, err := os.Stat(name)
	return err == nil || os.IsExist(err)
}

// �ж��ļ��Ƿ����
func FileExists(filename string) bool {
	fi, err := os.Stat(filename)
	return (err == nil || os.IsExist(err)) && !fi.IsDir()
}

// �ж�Ŀ¼�Ƿ����
func DirExists(dirname string) bool {
	fi, err := os.Stat(dirname)
	return (err == nil || os.IsExist(err)) && fi.IsDir()
}
```
�ο���
  [1] http://www.cnblogs.com/golove/p/3454630.html "Golang���� .tar.gz ѹ����"
  [2] https://leanpub.com/go-thestdlib "Go, The Standard Library
Real Code. Real Productivity. Master The Go Standard Library"		