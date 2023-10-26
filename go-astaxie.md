https://learnku.com/docs/build-web-application-with-golang/about-this-book/3151


### 1.0 Go环境变量

```
export GOROOT=/usr/local/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
export GOPATH=$HOME/gopath (可选设置, 但很重要, 建议设置)

```

### Go目录规范

我们在安装 Go 的时候看到需要设置 GOPATH 变量，Go 从 1.1 版本到 1.7 必须设置这个变量，而且不能和 Go 的安装目录一样，这个目录用来存放 Go 源码，Go 的可运行文件，以及相应的编译之后的包文件。所以这个目录下面有三个子目录：**src、bin、pkg**

从 go 1.8 开始，GOPATH 环境变量现在有一个默认值，如果它没有被设置。 它在 Unix 上默认为 $HOME/go，在 Windows 上默认为 %USERPROFILE%/go。



go 命令依赖一个重要的环境变量：$GOPATH

GOPATH 允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候 Windows 是分号，Linux 系统是冒号，当有多个 GOPATH 时，默认会将 go get 的内容放在第一个目录下。

GOPATH 目录约定有三个子目录：

> src 存放源代码（比如：.go .c .h .s 等）
>
> pkg 编译后生成的文件（比如：.a）
>
> bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中，如果有多个 gopath，那么使用 ${GOPATH//://bin:}/bin 添加所有的 bin 目录）



GOPATH 下的 src 目录就是接下来开发程序的主要目录，所有的源码都是放在这个目录下面，那么一般我们的做法就是一个目录一个项目。

例如: $GOPATH/src/mymath 表示 mymath 这个应用包或者可执行应用，这个根据 package 是 main 还是其他来决定，main 的话就是可执行应用，其他的话就是应用包，这个会在后续详细介绍 package。

所以当新建应用或者一个代码包时都是在 src 目录下新建一个文件夹，文件夹名称一般是代码包名称，当然也允许多级目录。

例如在 src 下面新建了目录 **$GOPATH/src/github.com/astaxie/beedb** 那么这个包路径就是 **"github.com/astaxie/beedb"**，包名称是最后一个目录 beedb



### 1.2 如何编写应用包

```bash
cd $GOPATH/src
mkdir mymath
vim sqrt.go
```

```go
// $GOPATH/src/mymath/sqrt.go源码如下：
package mymath

func Sqrt(x float64) float64 {
    z := 0.0
    for i := 0; i < 1000; i++ {
        z -= (z*z - x) / (2 * x)
    }
    return z
}
```



#### 编译应用

上面我们已经建立了自己的应用包，如何进行编译安装呢？有两种方式可以进行安装

> 1、只要进入对应的应用包目录，然后执行 `go install`，就可以安装了
>
> 2、在任意的目录执行如下代码 `go install mymath`

安装完之后，我们可以进入如下目录

```bash
cd $GOPATH/pkg/${GOOS}_${GOARCH}
// 可以看到如下文件
mymath.a
```

这个 .a 文件是应用包，那么我们如何进行调用呢？

接下来我们新建一个应用程序来调用这个应用包

新建应用包 mathapp

```bash
cd $GOPATH/src
mkdir mathapp
cd mathapp
vim main.go
```

```go
package main

import (
    "mymath"
    "fmt"
)

func main() {
    fmt.Printf("Hello, world.  Sqrt(2) = %v\n", mymath.Sqrt(2))
}
```

可以看到这个的 package 是 main，import 里面调用的包是 mymath，这个就是相对于 $GOPATH/src 的路径，如果是多级目录，就在 import 里面引入多级目录，如果你有多个 GOPATH，也是一样，Go 会自动在多个 $GOPATH/src 中寻找。

如何编译程序呢？进入该应用目录，然后执行 `go build`，那么在该目录下面会生成一个 mathapp 的可执行文件：mathapp

```bash
./mathapp
Hello, world.  Sqrt(2) = 1.414213562373095
```

> ***go: go.mod file not found in current directory or any parent directory; see 'go help modules'***

- `go env -w GO111MODULE=auto`
- `go mod init XXX` ，初始化Go moudle

如何安装该应用，进入该目录执行 go install，那么在 $GOPATH/bin/ 下增加了一个可执行文件 mathapp, 还记得前面我们把 $GOPATH/bin 加到我们的 PATH 里面了，这样可以在命令行输入如下命令就可以执行 mathapp



#### 获取远程包

go 语言有一个获取远程包的工具就是 `go get`，目前 go get 支持多数开源社区 (例如：github、googlecode、bitbucket、Launchpad)

对应的开源平台采用不同的源码控制工具，例如 github 采用 git、googlecode 采用 hg，所以要想获取这些源码，必须先安装相应的源码控制工具

> go get -u 参数可以自动更新包，而且当 go get 的时候会自动获取该包依赖的其他第三方包

```bash
go get github.com/astaxie/beedb
```

通过上面获取的代码在我们本地的源码相应的代码结构如下

```text
$GOPATH
  src
   |--github.com
          |-astaxie
              |-beedb
   pkg
    |--相应平台
         |-github.com
               |--astaxie
                    |beedb.a
```

go get 本质上可以理解为首先第一步是通过源码工具 clone 代码到 src 下面，然后执行 `go install`

在代码中如何使用远程包，很简单的就是和使用本地包一样，只要在开头 import 相应的路径就可以

```go
import "github.com/astaxie/beedb"
```

程序的整体结构
通过上面建立的我本地的 mygo 的目录结构如下所示

```text
bin/
    mathapp
pkg/
    平台名/ 如：darwin_amd64、linux_amd64
         mymath.a
         github.com/
              astaxie/
                   beedb.a
src/
    mathapp
          main.go
    mymath/
          sqrt.go
    github.com/
           astaxie/
                beedb/
                    beedb.go
                    util.go
```

从上面的结构我们可以很清晰的看到，bin 目录下面存的是编译之后可执行的文件，pkg 下面存放的是应用包，src 下面保存的是应用源代码







### 1.3 Go命令

https://learnku.com/docs/build-web-application-with-golang/013-go-command/3155

- go build  编译代码
- go clean  清除编译文件
- go fmt  格式化代码
- go get  动态获取远程代码包
- go install  编译代码并移动到对应目录
- go test
- go tool
- go generate
- godoc
- go version
- go env
- go list
- go run



```
go build -o readfile.exe readfile.go
go build readfile.go
```



```
Windows
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build

Linux
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build
```



### 1.4 Go 开发工具

- LiteIDE
- Sublime Text + GoSublime + gocode
- Visual Studio Code
- Atom
- GoLand
- Vim
- Vmacs
- Eclipse
- IntelliJ IDEA

##### 使用Sublime Text

1、安装Package Control：Ctrl + ` 打开命令行，输入以下代码安装 Package Control

```python
import  urllib.request,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();urllib.request.install_opener(urllib.request.build_opener(urllib.request.ProxyHandler()));open(os.path.join(ipp,pf),'wb').write(urllib.request.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())
```

打开首选项，即可找到 Package Control

2、安装插件：Ctrl+Shift+p 打开 Package Controll 输入 pcip（即 “Package Control: Install Package” 的缩写）。

安装SidebarEnhancements 和 Go Build，安装之后记得重启 Sublime 生效

安装GoSublime：https://www.cnblogs.com/rainight/p/10244748.html 、 https://packagecontrol.io/packages/GoSublime

```
cd "C:\Users\root\AppData\Roaming\Sublime Text 3\Packages"
git clone https://margo.sh/GoSublime
sublime:
|    sh.bootstrap: 
|                > 	error building gosubl\sh-bootstrap.go: cannot find package "gosubl/sh-bootstrap.go" in any of:
|                > 	C:\programs\Go\src\gosubl\sh-bootstrap.go (from $GOROOT)
|                > 	D:\Root\gowork\src\gosubl\sh-bootstrap.go (from $GOPATH)
|                > 
|                > 	error running bin\gosubl-sh-bootstrap.exe: The system cannot find the path specified.

```



## Go Start

### 2.0 Go 关键字

```text
break    default      func    interface    select
case     defer        go      map          struct
chan     else         goto    package      switch
const    fallthrough  if      range        type
continue for          import  return       var
```





### 2.1 Go 语言基础

```go
package main

import "fmt"

func main() {
    fmt.Printf("Hello, world or 你好，世界 or καλημ ́ρα κóσμ or こんにちはせかい\n")
}
```

Go 程序是通过 package 来组织的

package <pkgName> （在我们的例子中是 `package main` ）这一行告诉我们当前文件属于哪个包，而包名 main 则告诉我们它是一个可独立运行的包，它在编译后会产生可执行文件。除了 main 包之外，其它的包最后都会生成 *.a 文件（也就是包文件）并放置在 $GOPATH/pkg/$GOOS_$GOARCH 中（以 Mac 为例就是 $GOPATH/pkg/darwin_amd64）

> 每一个可独立运行的 Go 程序，必定包含一个 `package main`，在这个 `main` 包中必定包含一个入口函数 `main`，而这个函数既没有参数，也没有返回值。



### 2.2 Go 基础

#### 定义变量

```
var variableName type
var vname1,vname2,vname3 type
var variableName type = value
var vname1,vname2,vname3 type = v1,v2,v3
var vname1,vname2,vname3 = v1,v2,v3
vname1,vname2,vname3 := v1,v2,v3
```



`:=` 这个符号直接取代了 `var` 和 `type`, 这种形式叫做简短声明。不过它有一个限制，那就是**简短声明只能用在函数内部**；在函数外部使用则会无法编译通过，所以**一般用 `var` 方式来定义全局变量**。

`_` (下划线)是个特殊的变量名，任何赋予它的值都会被丢弃。

```go
_, b := 34, 35
```



#### 常量

```go
const constantName = value
const Pi float32 = 3.1415926
const Pi 3.1415926
const i = 10000
const MaxThread = 10
const prefix = "astaxie_"
```

常量类型：https://go.dev/ref/spec#Constants  http://c.biancheng.net/view/18.html

#### 内置基础类型

| 类型           | 常用类型                   | 说明                  |
| -------------- | -------------------------- | --------------------- |
| boolean        | true  false(默认)          | 布尔型                |
| rune           |                            | 符文类型、int32的别称 |
| integer        |                            | 整型                  |
| floating-point | float32  float64(默认)     | 浮点型                |
| complex        | complex64 complex128(默认) | 复数                  |
| string         |                            | 字符串                |

#### 数值类型

整数类型有无符号和有符号两种。

Go同时支持`int`和`uint`，这两种类型的长度相同，但具体长度取决于不同编译器的实现。

`rune`, `int8`, `int16`, `int32`, `int64`

`byte`, `uint8`, `uint16`, `uint32`, `uint64`

`rune` 是 `int32` 的别称，`byte` 是 `uint8` 的别称

不同类型的变量不允许互相赋值或操作。

##### 数值类型——rune

微软bing解释：*`rune` 是 Go 语言的一种特殊数字类型，是 `int32` 的别名，表示字符的 UTF-8 的编码值，用于区分字符值和整数值。它用于处理 Unicode 字符，支持 Unicode 的 1,114,112 个码点。在 Go 语言中，字符可以被分成两种类型处理：对占 1 个字节的英文类字符，可以使用 `byte`（或者 `uint8`）；对占 1 ~ 4 个字节的其他字符，可以使用 `rune`（或者 `int32`），如中文、特殊符号等。*

百度文心一言解释： `rune` 类型：*在Go语言中，`rune`是一种基本类型，用于表示Unicode字符。在Go中，Unicode字符被视为单个Unicode码点（code point），而`rune`就是对Unicode码点的别名。实际上，`rune`是`int32`类型的别名，用于表示从0到4字节的Unicode码点。*

cnblog博主解释：*[详解 Go 中的 rune 类型 - 技术颜良 - 博客园 (cnblogs.com)](https://www.cnblogs.com/cheyunhua/p/16007219.html)*

```go
package main  
  
import "fmt"  
  
func main() {  
    str := "Hello, 世界！" // 包含中文字符的字符串  
  
    // 将字符串逐个读取为rune类型  
    for _, r := range str {  
        fmt.Println(r) // 输出每个Unicode字符的码点
    }
    fmt.Println(len([]rune(str)))

    // 使用内置函数 len() 统计字符串长度
    fmt.Println(len("Go语言编程"))  // 输出：14  
    // 转换成 rune 数组后统计字符串长度
    fmt.Println(len([]rune("Go语言编程")))  // 输出：6
    fmt.Println([]rune("Go语言编程")[:4]) // 输出：[71 111 35821 35328]
    fmt.Println(string([]rune("Go语言编程")[:4])) // 输出：Go语言
    fmt.Println("Go语言编程",len("Go语言编程")) // 14
    fmt.Println("Go语言编程"[:4]) // go��
    fmt.Println("Go语言编程"[:5]) // go语
    fmt.Println("Go语言编程"[:6]) // go语�
    fmt.Println("Go语言编程"[:7]) // go语��
    fmt.Println("Go语言编程"[:8]) // go语言
}
/*
72
101
108
108
111
44
32
19990
30028
65281
10

72: H (U+0048)  
101: e (U+0065)  
108: l (U+006C)  
108: l (U+006C)  
111: o (U+006F)  
44: , (U+002C)  
32:   (U+0020)  
19990: 世 (U+4E2D)  
30028: 界 (U+540D)  
65281:  (U+E331)

*/

/*
上述代码中，我们定义了一个包含中文字符的字符串str。然后，使用range循环逐个遍历字符串中的字符，将其转换为rune类型并输出每个字符的码点。

注意，由于中文字符和英文字符的编码方式不同，中文字符的码点范围可能超过了int32类型的范围。在这种情况下，Go语言会自动将中文字符的码点扩展到int64类型。因此，在实际使用中，可以放心地处理Unicode字符的码点，而无需考虑超过int32范围的问题。
*/
```

##### Boolean 布尔型

`bool` 的值为 `true` 或 `false`，默认为 `false`

##### 字符串 string

Go 中的字符串都是采用 `UTF-8` 字符集编码。
字符串是用一对双引号 `""` 或反引号 ` `` ` 括起来定义

声明一个多行的字符串，使用反撇号 **`** 来声明

```go
m := `hello
    world`
```

在 Go 中字符串是不可变的

```go
	s := "hello"
	fmt.Println(s)

	fmt.Println(s[0]) // 104
	//s[0] = "c" // cannot assign to s[0] (strings are immutable)

	c := []byte(s)
	fmt.Println(c) // [104 101 108 108 111]
	fmt.Printf("%T\n", c) // []uint8
	c[0] = 99
	fmt.Println(string(c)) // cello
```

##### 错误类型 error

Go 内置有一个 `error` 类型，专门用来处理错误信息，Go 的 package里面还专门有一个包 `errors` 来处理错误：

```go
package main
import "fmt"
import "errors"

func main() {
    err := errors.New("Error: Unknown type")
    if err != nil {
        fmt.Print(err) // Error: Unknown type  结尾没有换行符
        fmt.Println(err) // Error: Unknown type  结尾有换行符
        // fmt.Printf(err) // cannot use err (type error) as type string in argument to fmt.Printf
        fmt.Printf("%v", err) // Error: Unknown type  结尾没有换行符
    }
}
```



分组声明：go可使用 ( ) 括号来声明一组变量和常量

```go
import (
    "fmt"
    "os"
)

var (
    i int
    pi float32
    str string
)

const (
    thread = 8
    pi = 3.1415926
    prefix = "Go_"
)
```

![img](https://cdn.learnku.com/build-web-application-with-golang/images/2.2.basic.png?raw=true)

#### itoa 枚举

Go里面有一个关键字 `itoa`，这个关键字用来声明 `enum` 的时候采用，它默认开始值是0，const中每增加一行加1

```go
const a = iota
const c,d = iota,iota
```



##### Go 程序设计的一些原则

Go 之所以会那么简洁，是因为它有一些默认的行为：

- 大写字母开头的变量是可导出的，也就是其它包可以读取的，是公有变量；小写字母开头的就是不可导出的，是私有变量。
- 大写字母开头的函数也是一样，相当于 class 中的带 public 关键词的公有函数；小写字母开头的就是有 private 关键词的私有函数。



#### array
##### 一维数组
`array` 就是数组，它的定义方式如下：

```go
var arr [n]type

var arr [10]int
arr[0] = 11
arr[1] = 22
```

> n 表示数组的长度，type 表示存储元素的类型
>
> string 数组未赋值的元素默认为空，int 数组未赋值的元素默认为0
>
> 数组长度也是数组类型的一部分，因此`[3]int`与`[4]int`是不同的类型
>
> 数组不能改变长度
>
> n 可以换成 `...` 来自动计算数组长度
>
> 数组之间的赋值是值的赋值，即当把一个数组作为参数传入函数的时候，传入的其实是该数组的副本，而不是它的指针。如果要使用指针，那么就需要用到后面介绍的 `slice` 类型了。

**数组可以使用另一种 `:=` 来声明**

```go
a := [3]int{1,2,3}
b := [10]int{1,2,3,4}
c := [...]int{4,5,6}
```
##### 二维数组

```go
doubleArray := [2][4]int{[4]int{1,2,3,4}, [4]int{5,6,7,8}}

// 简化内部的类型
easyArray := [2][4]int{{1,2,3,4},{5,6,7,8}}
```

#### slice

在很多应用场景中，数组并不能满足我们的需求。在初始定义数组时，我们并不知道需要多大的数组，因此我们就需要 “动态数组”。在 Go 里面这种数据结构叫 `slice`

`slice` 并不是真正意义上的动态数组，而是一个引用类型。`slice` 总是指向一个底层 `array`，`slice` 的声明也可以像 `array` 一样，只是不需要长度。

```go
var fslice []int
slice := []byte {'a','b','c','d'}
```

对于`slice`有几个有用的内置函数：

> `len` 获取 `slice` 的长度
>
> `cap` 获取 `slice` 的最大容量
>
> `append` 向 `slice` 里面追加一个或者多个元素，然后返回一个和 `slice` 一样类型的 `slice`
>
> `copy` 函数 `copy` 从源 `slice` 的 `src` 中复制元素到目标 `dst` ，并且返回复制的元素的个数
>
> 对 `slice` 的 `slice` 可以在 cap 范围内扩展

```go
    // var srcslice = [6]byte{'a','b','c','d','e','f'} // [6]uint8
    var srcslice = []byte{'a','b','c','d','e','f'} // []uint8
    // srcslice = ['a','b','c','d','e','f'] // syntax error: unexpected comma, expecting ]
    srcslice = []byte{'a','b','c','d','e','f','g','a','b'}
    var dstslice1, dstslice2 []byte
    dstslice1 = srcslice[:2]
    dstslice2 = srcslice[2:5]
    fmt.Printf("%T %T %T\n", srcslice, dstslice1, dstslice2)
    fmt.Printf("dstslice1: %s\n", dstslice1) // ab
    fmt.Printf("dstslice2: %s\n", dstslice2) // cde
    fmt.Printf("dstslice1: %s\n", dstslice1[:5]) // abcde
```

从 Go 1.2 开始 slice 支持了三个参数的 `slice`：arraydata[i:j:k]。之前我们一直采用这种方式在 `slice` 或者 `array` 基础上来获取一个 `slice`：arraydata[i:j]

```go
	var arr [10]int // len=10, cap=10
	fmt.Printf("%d\n", arr)
	fmt.Printf("%d %d\n", len(arr), cap(arr))

	sli := arr[2:4] // len=4-2, cap=10-2
	fmt.Printf("%d\n", sli)
	fmt.Printf("%d %d\n", len(sli), cap(sli))

	sli2 := arr[2:4:6] // len=4-2, cap=6-2
	fmt.Printf("%d\n", sli2)
	fmt.Printf("%d %d\n", len(sli2), cap(sli2)) // 2 4
```

从以上例子可以看出，slice[i:j:k] 的 len 为 j-i , cap 为 k-i



### map 字典

`map` 的格式为 `map[keyType]valueType`

```go
var numbers map[string]int
numbers = make(map[string]int)
numbers["one"] = 1
numbers["two"] = 2
numbers["three"] = 3
fmt.Println("n3:", numbers["three"])

books := make(map[string]int)
books["english"] = 3

var d1 = map[string]int{"root":1000,"admin":1001}
d2 := map[string]int{"root":1000, "administrator": 1002}

```

> `map` 是无序的，所以每次打印出来的 `map` 都可能会不一样，它不能通过 `index` 获取，必须通过 `key` 获取
>
> `map` 的长度是不固定的，也就是和 `slice` 一样，也是一种**引用类型**
>
> 内置的 `len` 函数同样适用于 `map`
>
> 在 Go 中，没有值可以安全地进行并发读写，它不是 thread-safe，在多个 go-routine 存取时，必须使用 mutex lock 机制

- `map` 的初始化可以通过 `key:val` 的方式初始化值，同时 `map` 内置有判断是否存在 `key` 的方式
- 通过 `delete` 删除 `map` 的元素

```go
rating := map[string]float32{"C":5, "Go":4.5, "Python": 4.5, "C++": 2}
csharpRating, ok := rating["C#"]
if ok {
    fmt.Println("C# is in the map and its rating is ", csharpRating)
} else {
    fmt.Println("We have no rating associated with C# in the map")
}

delete(rating,"C")
rating["Php"] = 4
fmt.Println(rating)

sql, ok := rating["sql"]
fmt.Printf("%f %t\n", sql, ok) // 0.000000 false
```

因为`map` 是一种引用类型，如果两个 `map` 同时指向一个底层，那么一个改变，另一个也相应的改变：

```go
m := make(map[string]string)
m["Hello"] = "Bonjour"
m1 := m
m1["Hello"] = "Salut"  // 现在m["hello"]的值已经是 Salut 了
```



#### make、new 操作

`make` 用于内建类型(`map`、`slice` 和 `channel`)的内存分配。

`new`用于**各种类型**的内存分配。

内建函数 `new` 本质上说跟其他语言中的同名函数功能一样：`new(T)` 分配了**零值**的 `T` 类型的内存空间，并且返回其地址，即一个 `*T` 类型的值。用Go的术语说，它返回了一个**指针**，指向新分配的类型 `T` 的零值。

**省流：```new 返回指针```。**

内建函数 `make(T, args)` 与 `new(T)` 有着不同的功能，make只能创建`map`、`slice` 和 `channel`，并且返回一个有初始值（非零）的 `T` 类型，而不是 `*T` 。本质来讲，导致这三个类型有所不同的原因是指向数据结构的引用在使用前必须被初始化。例如，一个 `slice` ，是一个包含指向数据（内部 `array` ）的指针、长度和容量的三项描述符；在这些项目被初始化之前，`slice` 为 `nil`。对于`map`、`slice` 和 `channel` 来说，`make` 初始化了内部的数据结构，填充适当的值。

**省流：```make 返回初始化后的(非零)值```**



##### 零值

关于“零值”，所指并非是空值，而是一种“变量未填充前”的默认值，通常为0.

以下罗列部分类型的“零值”：

| 类型    | 初始值 |
| ------- | ------ |
| int     | 0      |
| int8    | 0      |
| int32   | 0      |
| int64   | 0      |
| uint    | 0x0    |
| rune    | 0      |
| byte    | 0x0    |
| float32 | 0      |
| float64 | 0      |
| boot    | false  |
| string  | ""     |



#### 格式化输出

| 格式化   | 解释                                                        |
| -------- | ----------------------------------------------------------- |
| %d       | 整型以十进制方式显示                                        |
| %b       | 整型以二进制方式显示                                        |
| %o       | 整型以八进制方式显示                                        |
| %x       | 十六进制小写字母                                            |
| %X       | 十六进制大写字母                                            |
| %u       |                                                             |
| %s       | 字符串                                                      |
| %T       | 输出 Go 语言语法格式的类型和值                              |
| %t       | 输出布尔型值                                                |
| %f       | 浮点数  %.2f                                                |
| %e       |                                                             |
| %g       |                                                             |
| %q       | 输出 raw 格式                                               |
| %v       | 按值的本来值输出，例如使用%v占位符输出map、数组和切片的值   |
| %+v  %-v | 在 `%v` 基础上，对结构体字段名和值进行展开                  |
| %#v      | 输出 Go 语言语法格式的值                                    |
| %U       | Unicode 字符。fmt.Printf("%U", 1234) // 输出: U+1234        |
| %c       | 以Unicode字符形式输出字符。fmt.Printf("%c", 'A') // 输出: A |
| %p       | 指针，十六进制方式显示                                      |
|          |                                                             |



### 流程控制

##### if 语句

```go
if x > 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is less than 10")
}

if integer == 3 {
    fmt.Println("The integer is equal to 3")
} else if integer < 3 {
    fmt.Println("The integer is less than 3")
} else {
    fmt.Println("The integer is greater than 3")
}
```

Go 的 `if` 还有一个强大的地方就是条件判断语句里面允许声明一个变量，**这个变量的作用域只能在该条件逻辑块内**，其他地方就不起作用了

```go
// 计算获取值 x, 然后根据 x 返回的大小，判断是否大于 10。
if x := computedValue(); x > 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is less than 10")
}

// 这个地方如果这样调用就编译出错了，因为 x 是条件里面的变量
fmt.Println(x)
```



##### goto 语句

```go
func myFunc() {
    i := 0
Here: // 这行的第一个词, 以冒号结束为标签
    println(i)
    time.Sleep(1 * time.Second)
    i++
    goto Here // 跳转到Here
}
```



##### for 语句
Go 里面最强大的一个控制逻辑就是 `for`，它既可以用来循环读取数据，又可以当作 `while` 来控制逻辑，还能迭代操作。它的语法如下：

```go
for expression1; expression2; expression3 {
    //...
}
```
`expression1`、`expression2` 和 `expression3` 都是表达式，其中 `expression1` 和 `expression3` 是变量声明或者函数调用返回值之类的，`expression2` 是用来条件判断，`expression1` 在循环开始之前调用，`expression3` 在每轮循环结束之时调用。

```go
func main() {
    // 使用表达式进行变量定义并赋值, 但该变量不可在for循环外使用
	for i := 0; i < 6; i++ {
		println(i)
	} // 0 1 2 3 4 5
	// println(i) //undefined: i

    // 使用表达式对多个变量定义并赋值
    for i,j := 0, 1; i < 3; i++ {
    	println(i,j)
    } // 0 1, 1 1, 2 1

    // 在for循环外定义变量
	var j int
	println(j) // 0
	for j < 3 {
		println(j)
		j++
	} // 0 1 2
	println(j) // 3
}
```

```go
    // 使用for遍历map中的元素的三个例子
    rating := map[string]float32{"C":5, "Go":4.5, "Python": 4.5, "C++": 2}
    csharpRating, ok := rating["C#"]
    fmt.Println(reflect.TypeOf(csharpRating),reflect.TypeOf(ok))
    fmt.Println(csharpRating, ok)
    for k,v := range rating {
    	fmt.Println("key: ", k)
    	fmt.Println("value: ", v)
    }

    for k := range rating {
    	fmt.Println("key: ", k)
    }

    for _,v := range rating {
    	fmt.Println("value: ", v)
    }
```

##### break continue

break 跳出当前循环

continue 跳过本次循环



#### switch

```go
switch sExpr {
case expr1:
    some instructions
case expr2:
    some other instructions
case expr3:
    some other instructions
default:
    other code
}
```

- `sExpr` 和 `expr1`、`expr2`、`expr3` 的类型必须一致
- 表达式不必是常量或整数，执行的过程从上至下，直到找到匹配项
- 如果 `switch` 没有表达式，它会匹配 `true`
- Go 里面 switch 默认相当于每个 case 最后带有 break，匹配成功后不会自动向下执行其他 case，而是跳出整个 switch, 但是可以使用 fallthrough 强制执行后面的 case 代码
- 当 switch 语句的表达式是一个常量时，可以省略switch 语句中的表达式

```go
    switch num := 2; num {
    case 1:
        fmt.Println("数字是1")
    case 2:
        fmt.Println("数字是2")
    case 3:
        fmt.Println("数字是3")
    default:
        fmt.Println("数字不是1、2或3")
    }

    num := 2
    switch num+1 {
    case 1:
        fmt.Println("原数字是0")
    case 2:
        fmt.Println("原数字是1")
    case 3:
        fmt.Println("原数字是2")
    default:
        fmt.Println("原数字不是1、2或0")
    }

    switch time.Now().Weekday() {
    case time.Saturday, time.Sunday:
      fmt.Println("It's the weekend")
    default:
      fmt.Println("It's a weekday")
    }

    age := 34
    switch {
    case age < 18:
        fmt.Println("younger")
    case age > 18:
        fmt.Println("older than 18")
    default:
        fmt.Println("age is not valid")
    }
```



### 函数

函数的格式如下：

```go
func functionName(input1 type1, input2 type2) (output1 type1, output2 type2) {
    // some statements
    return output1, output2
}
```

- 关键字 func 用来声明一个函数 funcName
- 函数可以有一个或者多个参数，每个参数后面带有类型，通过 , 分隔
- 函数可以返回多个值
- 上面返回值声明了两个变量 output1 和 output2，如果你不想声明也可以，直接就两个类型
- 如果只有一个返回值且不声明返回值变量，那么你可以省略 包括返回值 的括号
- 如果没有返回值，那么就直接省略最后的返回信息
- 如果有返回值， 那么必须在函数的外层添加 return 语句
- 如果返回值在定义函数时已经定义了变量名及其类型，返回时可以省略变量名，直接使用 return

```go
func maxnum(x,y int) int {
	if x > y {
		return x
	}
	return y
}

func Swapnum(x,y int) (int, int) {
	return y, x
}

func swapnum(x,y int) (e,f int) {
	e, f = y, x
    return
}

func main() {
	a := 2
	b := 6
	c := -3

    max_ab := maxnum(a, b)
	fmt.Printf("%d\n", max_ab)
	fmt.Printf("%d\n", maxnum(a, c))

    i, j := Swapnum(a, b)
	fmt.Printf("%d %d\n", i, j)
    
    m, n := swapnum(a, b)
    fmt.Printf("%d %d\n", m, n)
}
```

##### 变参

在 Go 语言中，可以使用可变参数来传递任意数量的参数。可变参数使用三个点（...）表示，它们必须是函数的最后一个参数。

```go
package main

import "fmt"

func sum(numbers ...int) {
    sum := 0
    for _, num := range numbers {
        sum += num
    }
    fmt.Println("Sum:", sum)
}

func printargs(numbers ...int) {
	for index, i := range numbers {
		fmt.Println(index, i)
	}
}

func users(groupname string, users ...string) {
	for _, username := range users {
		fmt.Println(groupname, username)
	}
}

func main() {
    sum(1, 2, 3) // 输出 "Sum: 6"
    sum(4, 5, 6, 7, 8, 9) // 输出 "Sum: 39"
    sum(10, 10, 10, 10, 10) // 输出 "Sum: 50"
    printargs(1,2,4,3,5)
    users("administrator", "root", "admin", "ops", "sre", "dev", "secure")
}
```

##### 传值与传指针

当我们传一个参数值到被调用函数里面时，实际上是传了这个值的一份 copy，当在被调用函数中修改参数值的时候，调用函数中相应实参不会发生任何变化，因为数值变化只作用在 copy 上。

```go
package main
import "fmt"

func add(x int) int {
	x = x + 1
	return x
}

func main() {
	var a int
	a = 2
	fmt.Println(a) // 2
	b := add(a)
	fmt.Println(b) // 3
	fmt.Println(a) // 2
}
```

我们知道，变量在内存中是存放于一定地址上的，修改变量实际是修改变量地址处的内存。只有 `add1` 函数知道 `x` 变量所在的地址，才能修改 `x` 变量的值。所以我们需要将 `x` 所在地址 `&x` 传入函数，并将函数的参数的类型由 `int` 改为 `*int`，即改为指针类型，才能在函数中修改 `x` 变量的值。此时参数仍然是按 copy 传递的，只是 copy 的是一个指针。

```go
package main
import "fmt"

func add(x *int) int {
	*x = *x + 1
	return *x
}

func main() {
	var a int
	a = 2
	fmt.Println(a) // 2
	b := add(&a)
	fmt.Println(b) // 3
	fmt.Println(a) // 3
}
```

传指针的好处：

- 传指针使得多个函数能操作同一个对象
- 传指针比较轻量级（8 bytes），只是传内存地址，我们可以用指针传递体积大的结构体。如果用参数值传递的话，在每次 copy 上面就会花费相对较多的系统开销（内存和时间）。
- Go 语言中 `channel`、`slice`、`map` 这三种类型的实现机制类似指针，所以可以直接传递，而不用去地址后传递指针。（注：若函数需改变 `slice` 的长度，则仍需要取地址传递指针）



##### defer 延迟语句

在函数中添加多个 defer 语句，当函数执行到最后时，这些 defer 语句会按照逆序执行，最后该函数返回。

```go

func ReadWrite() bool {
    file.Open("file")
    defer file.Close()
    if failureX {
        return false
    }
    if failureY {
        return false
    }
    return true
}
```

如果有很多调用 `defer`，那么 `defer` 是采用后进先出模式，所以如下代码会输出 `4 3 2 1 0`

```go

func main() {
	numlist := []int{1,2,3,4,5}
	for index, i := range numlist {
		defer fmt.Printf("%d,%d\n", index, i)
	}

	for i := 0; i < 3; i++ {
		defer fmt.Printf("%d\n", i)
	}

	// 2 1 0
	// 4,5 3,4 2,3 1,2 0,1
}
```

##### 函数作为值、类型

在 Go 中函数也是一种变量，我们可以通过 `type` 来定义它，它的类型就是所有拥有相同的参数，相同的返回值的一种类型

```go
type typeName func(input1 inputType1 , input2 inputType2 [, ...]) (result1 resultType1 [, ...])
```

```go

package main

import "fmt"

type testInt func(int) bool // 声明了一个函数类型

func isOdd(integer int) bool {
    if integer%2 == 0 {
        return false
    }
    return true
}

func isEven(integer int) bool {
    if integer%2 == 0 {
        return true
    }
    return false
}

// 声明的函数类型在这个地方当做了一个参数

func filter(slice []int, f testInt) []int {
    var result []int
    for _, value := range slice {
        if f(value) {
            result = append(result, value)
        }
    }
    return result
}

func main(){
    slice := []int {1, 2, 3, 4, 5, 7}
    fmt.Println("slice = ", slice)
    odd := filter(slice, isOdd)    // 函数当做值来传递了
    fmt.Println("Odd elements of slice are: ", odd)
    even := filter(slice, isEven)  // 函数当做值来传递了
    fmt.Println("Even elements of slice are: ", even)
}
```



##### panic 和 recover

> Go 没有像 Java 那样的异常机制，它不能抛出异常，而是使用了 panic 和 recover 机制。
> 一定要记住，你应当把它作为最后的手段来使用，也就是说，你的代码中应当没有，或者很少有 panic 的东西。

**panic**

是一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。
当函数 `F` 调用 `panic`，函数 `F` 的执行被中断，但是 `F` 中的延迟函数会正常执行，然后 `F`返回到调用它的地方。在调用的地方，`F` 的行为就像调用了 `panic`。这一过程继续向上，直到发生 `panic` 的 `goroutine` 中所有调用的函数返回，此时程序退出。
恐慌可以直接调用 `panic` 产生。也可以由运行时错误产生，例如访问越界的数组。

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

**recover**

是一个内建的函数，可以让进入令人恐慌的流程中的 `goroutine` 恢复过来。`recover` 仅在延迟函数中有效。在正常的执行过程中，调用 `recover` 会返回 `nil`，并且没有其它任何效果。如果当前的 `goroutine` 陷入恐慌，调用 `recover` 可以捕获到 `panic` 的输入值，并且恢复正常的执行。

```go
func throwsPanic(f func()) (b bool) {
    defer func() {
        if x := recover(); x != nil {
            b = true
        }
    }()
    f() // 执行函数f，如果f中出现了panic，那么就可以恢复回来
    return
}
```



##### main 函数和 init 函数

Go 里面有两个保留的函数：

- `init` 函数（能够应用于所有的 `package` ）

- `main` 函数（只能应用于 `package main`）。

这两个函数在定义时不能有任何的参数和返回值。虽然一个 `package` 里面可以写任意多个 `init` 函数，但这无论是对于可读性还是以后的可维护性来说，我们都强烈建议用户在一个 `package` 中每个文件只写一个 init 函数。

Go 程序会自动调用 `init()` 和 `main()`，所以你不需要在任何地方调用这两个函数。每个 `package` 中的 `init` 函数都是可选的，但 `package main` 就必须包含一个 `main` 函数。

程序的初始化和执行都起始于 `main` 包。如果 `main` 包还导入了其它的包，那么就会在编译时将它们依次导入。

有时一个包会被多个包同时导入，那么它只会被导入一次（例如很多包可能都会用到 `fmt` 包，但它只会被导入一次。

当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行 `init` 函数（如果有的话），依次类推。

等所有被导入的包都加载完毕了，就会开始对 `main` 包中的包级常量和变量进行初始化，然后执行 `main` 包中的 `init` 函数（如果存在的话），最后执行 `main` 函数。下图详细地解释了整个执行过程：

![img](https://cdn.learnku.com/build-web-application-with-golang/images/2.3.init.png?raw=true)

#### import

通常导入方式：

```go
import "fmt"
import "os"

import (
    "fmt"
    "os"
)
```

调用：

```go
fmt.Println("hello world")
```

上面这个 fmt 是 Go 语言的标准库，其实是去 `GOROOT` 环境变量指定目录下去加载该模块

当然 Go 的 import 还支持如下两种方式来加载自己写的模块：

- 相对路径：```import "./model"``` // 当前文件同一目录的 model 目录，但是不建议这种方式来 import

- 绝对路径：```import "shorturl/model"``` // 加载 gopath/src/shorturl/model 模块

**特殊的 import**

1、点操作

```go
import (
    . "fmt"
)
// 这个点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名，也就是前面你调用的 fmt.Println ("hello world") 可以省略的写成 Println ("hello world")
```

2、别名操作

```go
import(
    f "fmt"
)
// 别名操作的话调用包函数时前缀变成了我们的前缀，即 f.Println ("hello world")
```

3、_ 操作

```go
import (
    "database/sql"
    _ "github.com/ziutek/mymysql/godrv"
)
// _操作其实是引入该包，而不直接使用包里面的函数，而是调用了该包里面的 init 函数。

```



#### struct

Go 语言中，也和 C 或者其他语言一样，我们可以声明新的类型，作为其它类型的属性或字段的容器。例如，我们可以创建一个自定义类型 `person` 代表一个人的实体。这个实体拥有属性：姓名和年龄。这样的类型我们称之 `struct`。

结构体定义：

```go
type person struct {
    name string
    age int
}
```

声明并使用 struct 的方式：

```go
// 声明类型并赋值
var P person
P.name, P.age = "Tom", 25

// 按照顺序提供初始化值
P := person{"Tom", 25}

// 通过 field:value 的方式初始化，这样可以任意顺序
P := person{age: 25, name: "Tom"}

// 通过 new 函数分配一个指针，此处 P 的类型为 *person
P := new(person) // 相当于 var P *person   P = new(person)
P.name, P.age = "Tom", 25
fmt.Printf("%v\n", P) // &{Tom 25}
fmt.Printf("%v\n", *P) // {Tom 25}
fmt.Printf("%v\n", &P) // 0xc000006028

var P *person
P = new(person)
P.name, P.age = "Sam", 23
fmt.Printf("%v\n", *P) // {Sam 23}
```



```go
package main

import "fmt"

// 声明一个新的类型
type person struct {
    name string
    age int
}

// 比较两个人的年龄，返回年龄大的那个人，并且返回年龄差
// struct 也是传值的
func Older(p1, p2 person) (person, int) {
    if p1.age>p2.age {  // 比较 p1 和 p2 这两个人的年龄
        return p1, p1.age-p2.age
    }
    return p2, p2.age-p1.age
}

func main() {
    var tom person

    // 赋值初始化
    tom.name, tom.age = "Tom", 18

    // 两个字段都写清楚的初始化
    bob := person{age:25, name:"Bob"}

    // 按照 struct 定义顺序初始化值
    paul := person{"Paul", 43}

    tb_Older, tb_diff := Older(tom, bob)
    tp_Older, tp_diff := Older(tom, paul)
    bp_Older, bp_diff := Older(bob, paul)

    fmt.Printf("Of %s and %s, %s is older by %d years\n",
        tom.name, bob.name, tb_Older.name, tb_diff)

    fmt.Printf("Of %s and %s, %s is older by %d years\n",
        tom.name, paul.name, tp_Older.name, tp_diff)

    fmt.Printf("Of %s and %s, %s is older by %d years\n",
        bob.name, paul.name, bp_Older.name, bp_diff)
}
```



##### struct 匿名字段

Go 支持只提供类型，而不写字段名的方式，也就是匿名字段，也称为嵌入字段。

当匿名字段是一个 struct 的时候，那么这个 struct 所拥有的全部字段都被隐式地引入了当前定义的这个 struct。

匿名字段不仅仅是 struct 类型哦，所有的内置类型和自定义类型都是可以作为匿名字段。

如果我们想访问重载后对应匿名类型里面的字段，可以通过匿名字段名来访问。

```go
package main

import (
    "fmt"
    "reflect"
)

type Group struct {
    Name string
}

type City map[string]string

type Person struct {
    Group // 匿名字段
    Cityinfo City
    Name string
    Age  int
    idField int
    mapField map[string]int
}

func main() {
    p := Person{
        Group: Group{"Fale"},
        Cityinfo: map[string]string{"CountryCode":"CN", "CityCode":"GD"},
        Name: "John",
        Age:  30,
        idField: 123,
        mapField: map[string]int{"key": 456},
    }

    t := reflect.TypeOf(p)
    fmt.Printf("%v\n", t) // main.Person

    for i := 0; i < t.NumField(); i++ {
        f := t.Field(i)
        if !f.Anonymous {
            fmt.Printf("nameField: %s %v\n", f.Name, f.Type)
        } else {
            fmt.Printf("anonymousField: %s %v\n", f.Name, f.Type)
        }
    }
    fmt.Printf("%v\n", p) // {{Fale} map[CityCode:GD CountryCode:CN] John 30 123 map[key:456]}
}
```

```go
package main

import (
    "fmt"
    "reflect"
)

type Group struct {
    Name string
}

type City map[string]string

type Person struct {
    Group // 匿名字段
    Cityinfo City
    Name string
    Age  int
    idField int
    mapField map[string]int
}

func main() {
    p := Person{
        Group: Group{"Fale"},
        Cityinfo: map[string]string{"CountryCode":"CN", "CityCode":"GD"},
        Name: "John",
        Age:  30,
        idField: 123,
        mapField: map[string]int{"key": 456},
    }

    t := reflect.TypeOf(p)
    fmt.Printf("%v\n", t) // main.Person
    

    for i := 0; i < t.NumField(); i++ {// NumField() 方法，返回结构体中的字段数量
        f := t.Field(i)
        // fmt.Printf("%v\n", reflect.TypeOf(f)) // reflect.StructField
        // 标识获取 Person 结构体中的第 i 个字段的信息，赋值给变量 f
        // f 的属性有 Name Type Tag Offset Index Anonymous , 例如 mapField map[string]int  56 [5] false
        // fmt.Printf("%v\n", f)
        fmt.Printf("%s %s %v %d %d %t\n", f.Name, f.Type, f.Tag, f.Offset, f.Index, f.Anonymous)
        if !f.Anonymous {
            fmt.Printf("nameField: %s %v\n", f.Name, f.Type)
        } else {
            fmt.Printf("anonymousField: %s %v\n", f.Name, f.Type)
        }
    }
    fmt.Printf("%v\n", p) // {{Fale} map[CityCode:GD CountryCode:CN] John 30 123 map[key:456]}
}
```



```go
package main
import "fmt"

type Subject []string

type Class struct {
	name string
	classid   int
}

type Person struct {
	name string
	age  int
	Class
	subject Subject
}

func Older(user1, user2 Person) (Person, int) {
	if user1.age > user2.age {
		return user1, user1.age - user2.age
	}
	return user2, user2.age - user1.age
}

func main() {
	P := Person{
		"John",
		25,
		Class{
			classid: 3,
			name: "Primary2",
		},
		[]string{"chinese","math"},
	}
	fmt.Printf("%v\n", P) // {John 25 {Primary2 3} [chinese math]}

	Sarry := new(Person)
	Sarry.name, Sarry.age = "Sarry", 23
	Sarry.Class.name, Sarry.classid = "Primary3", 3
	Sarry.subject = []string{"chinese","math","english"}
	fmt.Printf("%v\n", Sarry) //&{Sarry 23 {Primary3 3} [chinese math english]}
	fmt.Printf("%v\n", *Sarry) //{Sarry 23 {Primary3 3} [chinese math english]}

	var Tom Person
	Tom = Person{
		"Tom",
		20,
		Class{
			"Primary2",
			3,
		},
		[]string{"chinese","math"},
	}
	fmt.Printf("%v\n", Tom) // {Tom 20 {Primary2 3} [chinese math]}

	older1, age_larger1 := Older(Tom, *Sarry)
	fmt.Printf("The older of Tom and Sarry is: %s, %s is older than %d years.\n", older1.name, older1.name, age_larger1) // The older of Tom and Sarry is: Sarry, Sarry is older than 3 years.

	Tom.age += 5
	older2, _ := Older(Tom, *Sarry)
	fmt.Printf("The older of Tom and Sarry is: %s, %s's classname is %s, subject is %v.\n", older2.name, older2.name, older2.Class.name, older2.subject) // The older of Tom and Sarry is: Tom, Tom's classname is Primary2, subject is [chinese math].
}
```



```go
package main
import "fmt"

type User struct {
	Name string
	age  int
	int
}

type Subjects []string
type Student struct {
	User
	int
	string
	Subjects
}//匿名字段不仅仅是 struct 类型哦，所有的内置类型和自定义类型都是可以作为匿名字段。

func main() {
	Tom := Student{
		User{
			"Tom",
			23,
			999,
		},
		10,
		"ChineseSubjector",
		[]string{"Chinese","math"},
	}

	fmt.Printf("%s\n", Tom) // {{Tom %!s(int=23) %!s(int=999)} %!s(int=10) ChineseSubjector [Chinese math]}
	fmt.Printf("%s %d\n", Tom.Name, Tom.age) // Tom 23
	fmt.Printf("%v\n", Tom.Subjects) // [Chinese math]
	fmt.Printf("%v\n", Tom.int) // 10
	fmt.Printf("%v\n", Tom.User.int) // 999
}
```











































