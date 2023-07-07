
    title:  "Go语言中string的长度，字节，字符与编码"
    date:   2023-07-3
    desc:   "Go语言中string的长度，字节，字符与编码"
    tags:   String

---

### 问题缘由

由一次企业微信接口调用失败引起的思考。 公司产品中接入了企业微信的直播功能，而在企业微信对外开放的直播接口中，有一个字段是有长度限制的，标注为**最多支持100个utf8字符**。当时没有注意，结果测试发现接口不通，返回无效的字段。当时第一反应就是长度超出了，但是这个所谓的**100个tf8字符**指的到底是什么？而有些公共接口里面标注的是字节，这就开始有点儿要引人注意了，于是引出下面的一些思考

![image-20230706092315292](https://sso.image-lh.love/202307060923552.png)



### 什么是字符串?

引用官方文档的说明：Go中的字符串实际上就是字节切片(slice)，它不保存任何编码格式的数据，单纯的只保存字节(Byte)

*文档地址：*[Strings, bytes, runes and characters in Go](https://go.dev/blog/strings)

## What is a string?

Let’s start with some basics.

In Go, a string is in effect a read-only slice of bytes. If you’re at all uncertain about what a slice of bytes is or how it works, please read the [previous blog post](https://blog.golang.org/slices); we’ll assume here that you have.

It’s important to state right up front that a string holds *arbitrary* bytes. It is not required to hold Unicode text, UTF-8 text, or any other predefined format. As far as the content of a string is concerned, it is exactly equivalent to a slice of bytes.



### 字节？ 字符？ 

我们一直非常小心地使用“字节”和“字符”这两个词

- **字符（character）**

  字符是一种通用叫法，将数字、汉字、英文、符号等统称为字符，就好比将中文，英文，日文，葡萄牙语等世界上其他中几千种语言统称为**语音**。而不用去区分各种语言的发音，语法和格式等。所以上面说的**100个字符**表示100个任意字符。如 `abc`  和 `我和你` 都是3个字符

  

- **字节（Byte）**

  字节的概念就比较复杂，是计算机的一种存储单位。一个字节通常是8位(bit)长，Byte是从0-255的无符号类型，不能表示负数

  > 百度百科

  ![image-20230706144341254](https://sso.image-lh.love/202307061443901.png)

![image-20230706150114006](https://sso.image-lh.love/202307061501050.png)

### 字符到底占几个字节？ 



一般情况下一个英文或一个符号占一个字节，一个汉字占两个字节。但是依据编码格式不同，情况会有差异：

- **ASCII ：**一个英文字母（不分大小写）为一个字节，一个中文汉字为两个字节。
- **UTF-8 ：**一个英文字为一个字节，一个中文为三个字节。
- **Unicode：**一个英文为一个字节，一个中文为两个字节。
- **符号：**英文标点为一个字节，中文标点为两个字节。例如：英文句号 **.** 占1个字节的大小，中文句号 **。**占2个字节的大小。
- **UTF-16：**一个英文字母字符或一个汉字字符存储都需要 2 个字节（Unicode 扩展区的一些汉字存储需要 4 个字节）。
- **UTF-32 ：**世界上任何字符的存储都需要 4 个字节。



### 常见编码格式： ASCII 、UTF-8、Unicode 


**1.ASCII码**
这是美国在19世纪60年代的时候为了建立英文字符和二进制的关系时制定的编码规范，它能表示128个字符，其中包括英文字符、阿拉伯数字、西文字符以及32个控制字符。它用一个字节来表示具体的字符，但它只用后7位来表示字符（2^7=128），最前面的一位统一规定为0。



**2.扩展的ASCII码**
原本的ASCII码对于英文语言的国家是够用了，但是欧洲国家的一些语言会有拼音，这时7个字节就不够用了。因此一些欧洲国家就决定，利用字节中闲置的最高位编入新的符号。比如，法语中的é的编码为130（二进制10000010）。这样一来，这些欧洲国家使 用的编码体系，可以表示最多256个符号。但这时问题也出现了：不同的国家有不同的字母，因此，哪怕它们都使用256个符号的编码方式，代表的字母却不一样。比如，130在法语编码 中代表了é，在希伯来语编码中却代表了字母Gimel (ג)，在俄语编码中又会代表另一个符号。但是不管怎样，所有这些编码方式中，0—127表示的符号是一样的，不一样的只是128—255的这一段。这个问题就直接促使了Unicode编码的产生。



**3.Unicode符号集**
正如上一节所说，世界上存在着多种编码方式，同一个二进制数字可以被解释成不同的符号。因此，要想打开一个文本文件，就必须知道它的编码方式，否则用错误的编码方式解读，就会出现乱码。为什么电子邮件常常出现乱码？就是因为发信人和收信人使用的编码方式不一样。而Unicode就是这样一种编码：它包含了世界上所有的符号，并且每一个符号都是独一无二的。比如，U+0639表示阿拉伯字母Ain，U+0041表示英语的大写字母A，U+4E25表示汉字“严”。具体的符号对应表，可以查询unicode.org，或者专门的汉字对应表 。很多人都说Unicode编码，但其实Unicode是一个符号集（世界上所有符号的符号集），而不是一种新的编码方式。
但是正因为Unicode包含了所有的字符，而有些国家的字符用一个字节便可以表示，而有些国家的字符要用多个字节才能表示出来。即产生了两个问题：第一，如果有两个字节的数据，那计算机怎么知道这两个字节是表示一个汉字呢？还是表示两个英文字母呢？第二，因为不同字符需要的存储长度不一样，那么如果Unicode规定用2个字节存储字符，那么英文字符存储时前面1个字节都是0，这就大大浪费了存储空间。
上面两个问题造成的结果是：1）出现了unicode的多种存储方式，也就是说有许多种不同的二进制格式，可以用来表示unicode。2）unicode在很长一段时间内无法推广，直到互联网的出现。



**4.UTF-8**
互联网的普及，强烈要求出现一种统一的编码方式。UTF-8就是在互联网上使用最广的一种unicode的实现方式。其他实现方式还包括UTF-16和UTF-32，不过在互联网上基本不用。重复一遍，这里的关系是，UTF-8是Unicode的实现方式之一。
UTF-8最大的一个特点，就是它是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。
UTF-8的编码规则很简单，只有两条：
1）对于单字节的符号，字节的第一位设为0，后面7位为这个符号的unicode码。因此对于英语字母，UTF-8编码和ASCII码是相同的。
2）对于n字节的符号（n>1），第一个字节的前n位都设为1，第n+1位设为0，后面字节的前两位一律设为10。剩下的没有提及的二进制位，全部为这个符号的unicode码。


**5.GBK/GB2312/GB18030**
GBK和GB2312都是针对简体字的编码，只是GB2312只支持六千多个汉字的编码，而GBK支持1万多个汉字编码。而GB18030是用于繁体字的编码。汉字存储时都使用两个字节来储存。



### Go语言中怎么获取字符串长度？  len() 函数返回的长度如何解读？ 



:question: 问：len函数返回的长度是多少？

我们在日常编码中习惯于用len()函数判断字符串的长度,如下代码是在UTF-8编码格式下的代码:

 Go 语言的字符串都以 UTF-8 格式保存

![image-20230707091311787](https://sso.image-lh.love/202307070913920.png)

````go
func test_string_len() {
	str := "你好abc123"
	fmt.Printf("len(str): %v\n", len(str))
}

// len(str): 12
// 所以len函数返回的是字节长度，并不是有些人认为的字符长度
````

**印证了上一小节说的UTF-8下汉字占3个字节，英文和数字占一个字节**



:question: 问： 那我们如何获取字符串的字符长度？

如果使用len()函数返回的字节长度来做判断的话，针对接口中描述的**100字符长度**又该怎么处理呢。

- utf8.RuneCountInString

  ````go
  import (
  	"fmt"
  	"unicode/utf8"
  )
  
  func test_string_len() {
  	str := "你好abc123"
  	fmt.Printf("len: %v\n", len(str))                    // len: 12
  	fmt.Printf("len: %v\n", utf8.RuneCountInString(str)) // len: 8
  }
  ````

- string转换为[]byte、[]rune

  由于string转换为[]rune是unicode编码，rune表示unicode表中的值
  
  ````go
  func test_string_convert() {
  	str := "你好abc123"
  	bytes := []byte(str)
  	runes := []rune(str)
  	fmt.Printf("bytes: %v\n", bytes)           // bytes: [228 189 160 229 165 189 97 98 99 49 50 51]
  	fmt.Printf("runes: %v\n", runes)           // runes: [20320 22909 97 98 99 49 50 51]
  	fmt.Printf("len(bytes): %v\n", len(bytes)) // len(bytes): 12
  	fmt.Printf("len(runes): %v\n", len(runes)) // len(runes): 8
  }
  ````
  
  - rune类型是什么?
  
    Go语言将rune这个词定义为int32类型的别名，代表string中字符对应的unicode表中的编码点(code-point)
  
    对string进行for循环，会输出rune值，
  
    ````go
    const nihongo = "你好啊"
    
    func test_string_loop() {
    	for index, runeValue := range nihongo {
    		fmt.Printf("%#U  position %d\n", runeValue, index)
    	}
    	for index, runeValue := range nihongo {
    		fmt.Printf("%v  position %d\n", runeValue, index)
    	}
    }
    
    // 输出结果
    U+4F60 '你'  position 0
    U+597D '好'  position 3
    U+554A '啊'  position 6
    20320  position 0
    22909  position 3
    21834  position 6
    ````
  
    

:question: 如何截取指定长度的字符串？

 如果我想从字符串 `你好abc123` 截取出`你好ab`，代码该怎么写呢？

  ````go
  func test_string_split() {
  	str := "你好abc123"
  	runes := []rune(str)
  	//索引值: 左包含，右不包含
  	splitRunes := runes[:5]
  	splitStr := string(splitRunes)
  	fmt.Printf("splitStr: %v\n", splitStr) // splitStr: 你好abc
  }
  ````



  ### string之  for



````go
const nihongo = "你好啊"

func test_string_loop() {
	for index, runeValue := range nihongo {
		fmt.Printf("%#U  position %d\n", runeValue, index)
	}
	for index, runeValue := range nihongo {
		fmt.Printf("%v  position %d\n", runeValue, index)
	}
}

// 输出结果
U+4F60 '你'  position 0
U+597D '好'  position 3
U+554A '啊'  position 6
20320  position 0
22909  position 3
21834  position 6
````



### 总结

- 在Go中，string是一个字节切片
- Go源代码始终是UTF-8格式
- rune为int32的别名，string中的rune表示unicode中的代码点（code-point）

- string的字符长度可以使用utf8.RuneCountInString获取，也可以转换为[]rune切片获取长度
- len()函数针对string返回的是utf8编码的字节长度