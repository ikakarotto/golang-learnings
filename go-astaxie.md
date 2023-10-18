https://learnku.com/docs/build-web-application-with-golang/about-this-book/3151


## Go环境变量

```
export GOROOT=/usr/local/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
export GOPATH=$HOME/gopath (可选设置)

```

## Go目录规范

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



### 如何编写应用包

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



### 编译应用

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



## 获取远程包

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

————————————————
原文作者：Go &#25216;&#26415;&#35770;&#22363;&#25991;&#26723;&#65306;&#12298;Go Web &#32534;&#31243;&#65288;&#65289;&#12299;
转自链接：https://learnku.com/docs/build-web-application-with-golang/012-gopath-and-workspace/3154
版权声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请保留以上作者信息和原文链接。





## Go命令

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



## Go 开发工具

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

#### Go 关键字

```text
break    default      func    interface    select
case     defer        go      map          struct
chan     else         goto    package      switch
const    fallthrough  if      range        type
continue for          import  return       var
```





## Go 语言基础

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



### 定义变量

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



### 常量

```go
const constantName = value
const Pi float32 = 3.1415926
const Pi 3.1415926
const i = 10000
const MaxThread = 10
const prefix = "astaxie_"
```

常量类型：https://go.dev/ref/spec#Constants  http://c.biancheng.net/view/18.html

| 类型           | 常用类型                   | 说明                  |
| -------------- | -------------------------- | --------------------- |
| boolean        | true  false(默认)          | 布尔型                |
| rune           |                            | 符文类型、int32的别称 |
| integer        |                            | 整型                  |
| floating-point | float32  float64(默认)     | 浮点型                |
| complex        | complex64 complex128(默认) | 复数                  |
| string         |                            | 字符串                |

整数类型有无符号和有符号两种。

Go同时支持`int`和`uint`，这两种类型的长度相同，但具体长度取决于不同编译器的实现。

`rune`, `int8`, `int16`, `int32`, `int64`

`byte`, `uint8`, `uint16`, `uint32`, `uint64`

`rune` 是 `int32` 的别称，`byte` 是 `uint8` 的别称

不同类型的变量不允许互相赋值或操作



声明一个多行的字符串，使用反撇号 **`** 来声明

```go
m := `hello
    world`
```



分组声明：go可使用 ( ) 括号来声明一组变量和常量

```
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



Go 之所以会那么简洁，是因为它有一些默认的行为：

大写字母开头的变量是可导出的，也就是其它包可以读取的，是公有变量；小写字母开头的就是不可导出的，是私有变量。
大写字母开头的函数也是一样，相当于 class 中的带 public 关键词的公有函数；小写字母开头的就是有 private 关键词的私有函数。



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

对于`slice`有几个有用的内置函数：

> `len` 获取 `slice` 的长度
>
> `cap` 获取 `slice` 的最大容量
>
> `append` 向 `slice` 里面追加一个或者多个元素，然后返回一个和 `slice` 一样类型的 `slice`
>
> `copy` 函数 `copy` 从源 `slice` 的 `src` 中复制元素到目标 `dst` ，并且返回复制的元素的个数



### map 字典

`map` 的格式为 `map[keyType]valueType`

```go
var numbers map[string]int
numbers := make(map[string]int)
numbers["one"] = 1
numbers["two"] = 2
numbers["three"] = 3
fmt.Println("n3:", numbers["three"])
```

> `map` 是无序的，所以每次打印出来的 `map` 都可能会不一样，它不能通过 `index` 获取，必须通过 `key` 获取
>
> `map` 的长度是不固定的，也就是和 `slice` 一样，也是一种引用类型
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
```



##### make、new 操作

`make` 用于内建类型(`map`、`slice` 和 `channel`)的内存分配。

`new`用于各种类型的内存分配。

内建函数 `new` 本质上说跟其他语言中的同名函数功能一样：`new(T)` 分配了**零值**的 `T` 类型的内存空间，并且返回其地址，即一个 `*T` 类型的值。用Go的术语说，它返回了一个指针，指向新分配的类型 `T` 的零值。

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



### 函数

函数的格式如下：

```go
func functionName(input1 type1, input2 type2) (output1 type1, output2 type2) {
    // some staements
    return output1, output2
}
```


























