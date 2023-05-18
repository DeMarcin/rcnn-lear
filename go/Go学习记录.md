# Go学习记录

## 安装

### linux

wget https://golang.google.cn/dl/go1.18.3.linux-amd64.tar.gz

wget https://golang.google.cn/dl/go1.20.1.linux-amd64.tar.gz

tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz

vi /etc/profile

```
export GOROOT=/usr/local/go

export PATH=$PATH:$GOROOT/bin

export GOPATH=$HOME/go
```

使环境变量及时生效 `source /etc/profile`变量解释：
GOROOT： 类似于JAVA_HOME，Go的执行文件所在目录
GOPATH： 从go 1.8开始，GOPATH 环境变量现在有一个默认值，如果它没有被设置。 它在Unix上默认为$HOME/go,
$GOPATH 目录约定有三个子目录：

- src 存放源代码（比如：.go .c .h .s等）
- pkg 编译后生成的文件（比如：.a）
- bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中，如果有多个gopath，那么使用${GOPATH//😕/bin:}/bin添加所有的bin目录）

从 Go1.11 开始, Go 官方加入 Go Module 支持, Go1.12 成为默认支持; 从此告别源码必须放在 Gopath。

由于google被阻拦，所以要设置代理

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

## Go Module

基本现在安装的go版本都是1.8之后的了，go1.8之前第三方依赖目录再`GOPATH`，1.8之后go引入了module管理第三方依赖，因此不需要`GOPATH`了

> **go 常用命令**
>
> ```bash
> go mod init 初始化模块，例如 go mod init github.com/wjp2013/hello
> go build, go test 和其它构建命令会自动为 go.mod 添加新的依赖
> go get 改依赖关系的所需版本(或添加新的依赖关系)
> go list -m all 列出当前模块及其所有依赖项
> go mod tidy 拉取缺少的模块，移除不用的模块
> ```
> **不常用命令**
>
> ```sh
> go mod download 下载依赖包到本地 cache
> go mod edit 编辑 go.mod 文件，选项有 -json、-require 和 -exclude，可以使用帮助 go help mod edit
> go mod graph 打印模块依赖图
> go mod vendor 将依赖复制到 vendor 目录
> go mod verify 验证依赖是否正确
> go mod why 解释为什么需要依赖
> ```
>
> 

## 变量

在包级别声明的变量会在main入口函数执行前完成初始化，局部变量将在声明语句被执行到的时候完成初始化。

`var 变量名字 类型 = 表达式`

```go
var str string
var num int = 0
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```



### 简短变量   

用于声明和初始化**局部变量**，变量的类型根据表达式来自动推导。

var形式的声明语句往往是用于需要显式指定变量类型的地方，或者因为变量稍后会被重新赋值而初始值无关紧要的地方。

变量中必须至少声明一个新的变量

`名字 := 表达式`

```go
freq := rand.Float64() * 3.0
t := 0.0
i, j := 0, 1 //初始化一组变量

in, err := os.Open(infile) //初始化in和err
out, err := os.Create(outfile)	//初始化out，赋值err
```

**注意区分**`i, j = j, i // 交换 i 和 j 的值`



### 指针

指针的值存放的是地址，该地址指向了另一个**变量的值**，如下图指针p存放的是地址，该地址为变量x存放的值。通过间接运算符`*`获取指针指向的值，取址运算符`&`能去的该地址的值

![image-20220317165544043](https://xiejy-image.oss-cn-shanghai.aliyuncs.com/markdown-image/image-20220317165544043.png)

```go
x := 12
p := &x         // 指针p，p的值为x的地址
fmt.Println(p)  // 0xc000102090
fmt.Println(*p) // 12
fmt.Println(&p)	// 0xc00012c028

*p = 2          // equivalent to x = 2
fmt.Println(x)  // "2"
```

#### 空指针

未被初始化的指针

```go
var p *int
*p --> err
```



#### 野指针:被一片无效的地址空间初始化

野指针是一种指向内存位置是不可知的指针，一般是由于指针变量在声明时没有初始化所导致的。

```go
var p *int // 指针声明未初始化，野指针/空指针
fmt.Printf("%p\n", ptr) // 0x0
// *p = 0xffl879129   无效的
// *p = 10  panic 无效的内存地址或 nil 指针解引用 Pointer Dereference，野指针不能进行 *p 取值
fmt.Println(ptr)  // nil

var a int = 10
p = &a	// 可以重新指向变量 a 的地址
*p = 2   // 即可修改变量a 内存地址的值
fmt.Println(a) // 2
```

#### 悬空指针

悬空指针是指它最初指向的内存空间已经被释放掉了，在 Go 语言中，如果函数内变量的地址作为函数返回值，编译器就会判断出该变量在函数调用完成后还需要被引用，因此不在栈中为其分配存储空间，而是在堆中为其分配存储空间，这样的变量在函数调用完成后其占用的存储空间不会被立刻释放，也就避免出现悬空指针这种情况。

```go
type Command struct {
    Name    string    // 指令名称
    Var     *int      // 指令绑定的变量
    Comment string    // 指令的注释
}

func newCommand(name string, varref *int, comment string) *Command {
    return &Command{
        Name:    name,
        Var:     varref,
        Comment: comment,
    }
}

cmd = newCommand(
    "version",
    &version,
    "show version",
)
```



## 复合数据类型

### 数组

默认情况下，数组的每个元素都被初始化为元素类型对应的零值，对于数字类型来说就是0。我们也可以使用数组字面值语法用一组值来初始化数组：

```go
var q [3]int = [3]int{1, 2, 3}
var r [3]int = [3]int{1, 2}
fmt.Println(r[2]) // "0"
```

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。因此，上面q数组的定义可以简化为

```go
q := [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```



### 切片Slice

Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作[]T，其中T为slice中元素的类型；slice的语法和数组很像，只 是没有固定长度而已。

一个slice由三个部分构成：**指针、长度和容量**。

+ 指针指向第一个slice元素对应的底层数组元素的**地址**，要注意的是slice的第一个元素并不一定就是数组的第一个元素。
+ 长度对应slice中元素的数目；长度不能超过容量。len函数返回长度
+ 容量一般是从slice的开始位置到底层数据的结尾位置。cap函数返回容量



字符串的切片操作和[]byte字节类型切片的切片操作是类似的。都写作**x[m:n]**



+ 子切片和k母切片共享底层的内存空间，修改子切皮会反映到母切片上，在子切片执行append会把新元素放到母切片预留的内存w空间上。

+ 当子切片不断append，耗完母切片预留的内存空间，子切片跟母切片就会发生内存分离，此后两个切片灭有任何联系。

+ go语言函数传参都是值传递，即切片会把切片的{arrayPointer, len, cap}这3个字段拷贝一份传进来
+ 由于传的是底层数组的指针，所以可以直接修改底层数组里的元素。