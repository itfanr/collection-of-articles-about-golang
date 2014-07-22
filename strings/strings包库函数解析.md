# strings包

标签（空格分隔）： os

---

#简介

strings包内包括的方法可用于检查序列的单个字符、比较字符串、搜索字符串、提取子字符串、创建字符串副本并将所有字符全部转换为大写或小写。

#概览

strings包实现了操作字符串的简单函数。

#内容

##函数

###func Contains(s, substr string) bool
当且仅当此字符串s包含字符串substr时，返回 true。

###func ContainsAny(s, chars string) bool
如果s包含chars中的任意一个字符，则返回true。

###func ContainsRune(s string, r rune) bool
如果s包含chars中的任意一个rune类型字符，则返回true。

###func Count(s, sep string) int
返回字符串s中包含的sep字符的数目。特别注意如果sep为空格时的情形。
###func EqualFold(s, t string) bool
比较字符串s和t，当采用UTF-8编码，两者全部小写时若相等，则返回ture。

###func Fields(s string) []string
按字符串s内的空格或者连续的空格来切割字符串s，并返回子串组成的切片。空格由unicode.IsSpace来定义。如果s只包含空格，那么返回空切片。

###func FieldsFunc(s string, f func(rune) bool) []string
按字符串s内的满足某种要求的rune字符来切割字符串s，并返回子串组成的切片。这种要求通过函数`f`来自定义。如果s只包含空格，那么返回空切片。

###func HasPrefix(s, prefix string) bool
判断s是否以prefix为前缀。如果是，则返回true。

###func HasSuffix(s, suffix string) bool
判断s是否以prefix为后缀。如果是，则返回true。

###func Index(s, sep string) int
返回字符串sep在字符串s中第一次出现处的索引。如果s中不包含sep，则返回-1。

###func IndexAny(s, chars string) int
返回字符串sep中的任意字符在字符串s中第一次出现处的索引。如果s中不包含sep中的所有字符，则返回-1。

###func IndexByte(s string, c byte) int
返回byte字符c在字符串s中第一次出现处的索引。如果s中不包含sep，则返回-1。

###func IndexFunc(s string, f func(rune) bool) int
返回符合某种要求的字符在字符串s中第一次出现处的索引，这种要求由函数`f`自定义。如果s中不存在这样的字符，则返回-1。

###func IndexRune(s string, r rune) int
返回rune字符c在字符串s中第一次出现处的索引。如果s中不包含sep，则返回-1。

###func Join(a []string, sep string) string
把string切片连接成一个字符串并返回，切片元素用字符串sep连接。

###func LastIndex(s, sep string) int
返回字符串sep在字符串s中最后一次出现处的索引。如果s中不包含sep，则返回-1。

###func LastIndexAny(s, chars string) int
返回字符串sep中的任意字符在字符串s中最后一次出现处的索引。如果s中不包含sep中的所有字符，则返回-1。

###func LastIndexFunc(s string, f func(rune) bool) int
返回符合某种要求的字符在字符串s中最后一次出现处的索引，这种要求由函数`f`自定义。如果s中不存在这样的字符，则返回-1。

###func Map(mapping func(rune) rune, s string) string
按照某种规则将字符串s中的每个字符做映射处理，然后返回字符串s。这个规则通过函数mapping自定义。如果mapping返回负数，则该字符被丢弃。

###func Repeat(s string, count int) string
返回一个由count个字符串s组成的新字符串

###func Replace(s, old, new string, n int) string
将字符串s中的n个old字符串用new来替换。如果n小于0，则对要替换的old字符串个数没有限制。

###func Split(s, sep string) []string
以字符串sep来拆分字符串s，然后返回由该字符串拆分形成的子字符串切片。如果sep为空，那么按照UTF-8编码分隔每一个字符，等效于SplitN函数当count取值为-1的情形。

###func SplitAfter(s, sep string) []string
以字符串sep来拆分字符串s，然后返回由该字符串拆分形成的子字符串切片,除了最后的切片元素，每个元素以sep结尾。如果sep为空，那么按照UTF-8编码分隔每一个字符，等效于SplitN函数当count取值为-1的情形。

###func SplitAfterN(s, sep string, n int) []string
以字符串sep来拆分字符串s，然后返回由该字符串拆分形成的子字符串切片,除了最后的切片元素，每个元素以sep结尾。如果sep为空，那么按照UTF-8编码分隔每一个字符。

如果n大于零，最多有n个子串，如果n等于零，结果为空，如果n小于0，返回所有的string切片。