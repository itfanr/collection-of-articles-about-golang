# ���ļ������zip�ļ�

��ǩ���ո�ָ����� tar 

---
ʾ��1��

�����ļ���ѹ����ѹ�����ʽΪ`.zip`����

�����õ�����Ҫ������

 -  func NewWriter(w io.Writer) *Writer
 -  func (w *Writer) Create(name string) (io.Writer, error)
 -  func (w *Writer) CreateHeader(fh *FileHeader) (io.Writer, error)

����zip����tar�����ƣ���ͬ����zip����һ������Ĺ���������ٵذ��ļ�д��ѹ���������Ǵӵõ�NewWriter�õ�Writer����󣬿���ֱ��ʹ��Create��������zip�ļ�д�����ݣ�����Ҫ�Լ�д�ļ�Ԫ���ݡ���Ȼ����Ҳ�����Լ���CreateHeader����ȥдFileHeader��

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
	//��Ϥlinux��ͬѧ�϶�֪��λ���
	flags := os.O_WRONLY | os.O_CREATE | os.O_TRUNC
	//0644��unmask��Ϣ����ʾ�û����û��顢�������Ȩ��
	file, err := os.OpenFile("test_zip.zip", flags, 0644)
	if err != nil {
		log.Fatalf("Ϊ����ѹ��������zip�ļ�ʧ��", err)
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
		return fmt.Errorf("���ļ�%sʧ�ܣ�%s", filename, err)
	}
	defer file.Close()
	iw, err := zw.Create(filename)
	if err != nil {
		return err
	}
	//�˴������д��ѹ�������ļ��ֽ�����
	//FileHeaderû���ļ���size��Ϣ��
	if _, err := io.Copy(iw, file); err != nil {
		return fmt.Errorf("���ļ� %s д�� zipѹ����ʧ��: %s", filename, err)
	}
	return nil
}

```
ע�⣺Create����Ҫ�����name���������·����������б��`/`����Ҳ����·���ʼ�������̷�����`/`��Create������CreateHeaderҪ���´ε���Create��CreateHeader����Closeǰ���뽫�ļ�����д��io.Writer��




