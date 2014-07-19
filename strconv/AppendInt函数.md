# AppendInt 函数

---
函数说明：
func AppendInt(dst []byte, i int64, base int) []byte

// AppendInt 将 int 型整数 i 转换为字符串形式，并追加到 dst 的尾部
// i：要转换的字符串
// base：进位制
// 返回追加后的 []byte

示例1：
```
package main
import (
	"fmt"
	"strconv"
)
func main() {
	b1 := make([]byte, 0)
	b2 := make([]byte, 0)
	b3 := make([]byte, 0)

	//9的ASCII码是57 ，7的ASCII码是55
	fmt.Println(strconv.AppendInt(b1, 97, 10))//[57 55]
	//原来的slice并没有改变
	fmt.Printf("%s\n", strconv.AppendInt(b1, 97, 10))//97

	fmt.Println(strconv.AppendInt(b2, 97, 16))//[54 49]
	fmt.Println(string(strconv.AppendInt(b2, 97, 16)))//61

	fmt.Println(strconv.AppendInt(b3, 97, 2))//	[49 49 48 48 48 48 49]
	//相当于十进制转二进制
	fmt.Println(string(strconv.AppendInt(b3, 97, 2))) //1100001

}

```

参考：
 1. http://www.cnblogs.com/golove/p/3262925.html

