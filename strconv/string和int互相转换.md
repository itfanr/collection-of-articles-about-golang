# string和int转换

标签（空格分隔）： 类型 转换

---
用到的函数：

 - func Atoi(s string) (i int, err error)
 - func Itoa(i int) string
 - func FormatInt(i int64, base int) string
 - func ParseInt(s string, base int, bitSize int) (i int64, err error)

示例1 ：
int转string类型：

```
	var i1 int = 107
	var s1 string = strconv.FormatInt(int64(i1), 2)
	//等效于FormatInt(i, 10)
	var s2 string = strconv.Itoa(i1)
	var s3 string = strconv.FormatInt(int64(i1), 16)
	var s4 string = strconv.FormatInt(int64(i1), 36)
	fmt.Println(s1, s2, s3, s4) //	1101011 107 6b 2z
```

示例2 ：
string 转int类型：
```
	s1 := "15"
	s2 := "101" //0101也是可以的
	//strconv.Atoi(s1)也可以
	//如果 base 为 0，则根据字符串的前缀判断进位制（0x:16，0:8，其它:10）
	i1, err := strconv.ParseInt(s1, 10, 0)
	if err != nil {
		panic(err)
	}
	i2, err := strconv.ParseInt(s1, 16, 0)
	if err != nil {
		panic(err)
	}
	i3, err := strconv.ParseInt(s2, 2, 0)
	if err != nil {
		panic(err)
	}
	fmt.Print(i1, i2, i3) //返回15 21 5
}

```

```
	fmt.Println(strconv.ParseInt("127", 10, 8)) // 127 <nil>
	fmt.Println(strconv.ParseInt("128", 10, 8)) // parsing "128": value out of range
```

```
	fmt.Println(strconv.ParseInt("FF", 16, 0))//255 <nil>
	fmt.Println(strconv.ParseInt("0xFF", 16, 0)) //parsing "0xFF": invalid syntax
```

```
    //有时候Aoti函数更方便，通常使用这个函数，而不使用 ParseInt
	fmt.Println(strconv.Atoi("2147483647"))//	2147483647 <nil>
	fmt.Println(strconv.Atoi("2147483648"))// parsing "2147483648": value out of range
```

参考：
 1. http://blog.csdn.net/wh_luosangnanka5/article/details/9664959 
 2. http://www.blogjava.net/oathleo/archive/2013/08/30/403488.html
 3. http://www.25kx.com/art/2210174
 