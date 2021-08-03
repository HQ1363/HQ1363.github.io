---
title: go 一百问   
date: 2021-04-01 16:02:13   
comments: true   
tags: go   
categories: Go基础   
excerpt: 好的编码风格，能够让你的代码更优雅、更简洁，还能避免很多坑.
---

## Go基础
### Q1: 为啥需要私有`goproxy`？
#### 上下文
> 我们知道在大陆的网络环境是无法访问到`http://golang.org`等`google`的网站的。但在开发日常中使用的很多依赖包或系统包依赖都是在`google`的服务器上。为了解决无法加载依赖的问题，国内也有很多种解决方案。一种是使用`http://goproxy.io`或七牛主导的`http://goproxy.cn`。
在企业里，有很多情况是生产网络或测试网络环境是无法正常访问外网的，为了解决这个问题可能需要自己搭建一个`proxy`来管理依赖包。
#### 可选配置
```bash
export GOPROXY=https://mirrors.aliyun.com/goproxy/
export GOPROXY=https://proxy.golang.org,direct
export GOPROXY=https://goproxy.io
export GOPROXY=https://gonexus.dev
export GOPROXY=https://athens.azurefd.net
export GOPROXY=https://gocenter.io
export GOPROXY=https://goproxy.cn
```
### Q2: `Make`和`New`的异同？
- slice、map和channel，使用make
- array、struct和所有的值类型，使用new
> 内置函数 new 计算类型的⼤小，为其分配零值内存，返回指针。⽽ make 会被编译器翻译成具体的创建函数，由其分配内存和初始化成员结构，返回对象⽽⾮指针。new和make都是在堆上分配内存，只是行为有所不同。new分配完后会返回指向其的内存地址(指针)，make是返回整个数值/对象。

### Q3: 数组和切片陷阱
#### 陷阱一
```golang
func foo(a [2]int) {
	a[0] = 200
}

func main() {
	a := [2]int{1, 2}
	foo(a)
	fmt.Println(a)
}
```
改
```go
func foo(a *[2]int) {
	(*a)[0] = 200
}

func main() {
	a := [2]int{1, 2}
	foo(&a)
	fmt.Println(a)
}
```
或
```go
func foo(a []int) {
	a[0] = 200
}

func main() {
	a := []int{1, 2}
	foo(a)
	fmt.Println(a)
}
```
#### 陷阱二
```go
func foo(a []int) {
	a = append(a, 1, 2, 3, 4, 5, 6, 7, 8)
	a[0] = 200
}

func main() {
	a := []int{1, 2}
	foo(a)
	fmt.Println(a)
}
```
改
```go
func foo(a []int) []int {
	a = append(a, 1, 2, 3, 4, 5, 6, 7, 8)
	a[0] = 200
	return a
}

func main() {
	a := []int{1, 2}
	a = foo(a)   // 可读性更好
	fmt.Println(a)
}
```
或
```go
func foo(a *[]int) {
	*a = append(*a, 1, 2, 3, 4, 5, 6, 7, 8)
	(*a)[0] = 200
}

func main() {
	a := []int{1, 2}
	foo(&a)
	fmt.Println(a)
}
```
### Q4: for vs for...range的性能问题
> 与 for 不同的是，range 对每个迭代值都创建了一个拷贝。因此如果每次迭代的值内存占用很小的情况下，for 和 range 的性能几乎没有差异，但是如果每个迭代值内存占用很大，两者的差距就很明显了.
#### 陷阱一
```go
// for 语句中的迭代变量在每次迭代中都会重用, 即 for 中创建的闭包函数接收到的参数始终是同一个变量, 在`goroutine`开始执行时都会得到同一个迭代值
func main() {
    data := []string{"one", "two", "three"}

    for _, v := range data {
        go func() {
            fmt.Println(v)
        }()
    }

    time.Sleep(3 * time.Second)
    // 输出 three three three
}
```
改
```go
func main() {
    data := []string{"one", "two", "three"}

    for _, v := range data {
        vCopy := v
        go func() {
            fmt.Println(vCopy)
        }()
    }

    time.Sleep(3 * time.Second)
    // 输出 one two three
}
```
或
```go
func main() {
    data := []string{"one", "two", "three"}

    for _, v := range data {
        go func(in string) {
            fmt.Println(in)
        }(v)
    }

    time.Sleep(3 * time.Second)
    // 输出 one two three
}
```
### Q5: 字符串如何实现高效拼接?
> 一般我们拼接字符串时，会使用如下几种方式：
>
> - 使用操作符`+`，此方式最差
> - 使用`fmt.Sprintf`
> - 使用`strings.Builder`，此方式最佳
> - 使用`bytes.Buffer`
> - 使用`[]byte`
> 感兴趣的朋友，可以使用`benchmark`做下测试

### Q6: 常见导致`Go`程序`OOM`的情况
- 递归调用函数导致栈溢出
- `Goroutine`永久不退出，单个协程一般占4k，若堆积，就会导致`OOM`

### Q7: 如何退出协程?
> 超时陷阱
```go
func doBadthing(done chan bool) {
	time.Sleep(time.Second)
	done <- true
}

func timeout(f func(chan bool)) error {
	done := make(chan bool)
	go f(done)
	select {
	case <-done:
		fmt.Println("done")
		return nil
	case <-time.After(time.Millisecond):
		return fmt.Errorf("timeout")
	}
}
timeout(doBadthing)
```
_上述`doBadthing`协程，永久不会退出，会死锁的，针对上述问题，可以有哪些解决办法?_
- 思路一：保证协程能够执行完毕，不至于一直阻塞，可以给`channel`设置缓冲区
  eg: `done := make(chan bool, 1)`
- 思路二：仿照主函数`timeout`，也利用`select`来尝试发送，发送失败也能理解返回
```go
func doGoodthing(done chan bool) {
	time.Sleep(time.Second)
	select {
	case done <- true:
	default:
		return
	}
}
```

### Q8: 可以强制`kill`掉`goroutine`吗?
_答案是不能，`goroutine`只能自己退出，而不能被其他`goroutine`强制关闭或者杀死_

### Q9: 如何优雅正确的关闭通道?
> 我们知道一个已经关闭的channel，如果尝试再次close，会导致panic，虽然可以通过recover使程序恢复正常，但很粗鲁；作为文明人，自然要礼貌.
```go
type MyChannel struct {
	C    chan T
	once sync.Once
}
func NewMyChannel() *MyChannel {
	return &MyChannel{C: make(chan T)}
}
func (mc *MyChannel) SafeClose() {
	mc.once.Do(func() {
		close(mc.C)
	})
}
```

### Q10: 请列举一些你所知道的内置函数?
- len、cap
- close、copy、append
- panic、recover
- new、make

### Q11: Go语言的执行过程是?
![go_import](../images/go_import.png)

### Q12: map如何顺序读取?
_map是无序的，不能顺序读取，要想顺序读取，第一个要解决的问题就是，把ｋｅｙ变得有序，然后通过key取值。_

### Q13: 说出一个context包的用途
- 避免`Goroutine`内存泄漏
```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())

    ch := func(ctx context.Context) <-chan int {
        ch := make(chan int)
        go func() {
            for i := 0; ; i++ {
                select {
                case <- ctx.Done():
                    return
                case ch <- i:
                }
            }
        } ()
        return ch
    }(ctx)

    for v := range ch {
        fmt.Println(v)
        if v == 5 {
            cancel()
            break
        }
    }
}
```
_下面的 for 循环停止取数据时，就用 cancel 函数，让另一个协程停止写数据。如果下面 for 已停止读取数据，上面 for 循环还在写入，就会造成内存泄漏。_

### Q14: 如何跳出for select循环?
_通常在for循环中，使用break可以跳出循环，但是注意在go语言中，for select配合时，break 并不能跳出循环_
```go
func ForSelectLoop(ch chan bool){
 EXIT:
    for  {
        select {
        case v, ok := <-ch:
            if !ok {
                fmt.Println("close channel", v)
                break EXIT   //goto EXIT2
            }

            fmt.Println("ch val =", v)
        }
    }
    //EXIT2:
    fmt.Println("exit ForSelectLoop")
}
```

### Q15: 哪些情况，会导致go触发异常?
- NPE，空指针异常，对空指针做了解析引用
- 索引溢出/下标越界
- 除数为0
- 调用panic函数

### Q16: Slice的原理是啥?
> 切片是基于数组实现的，它的底层是数组，它自己本身非常小，是只有3个字段的struct类型：
>
> - 指向底层数据的指针
> - 切片的长度
> - 切片的容量

### Q17: Map的底层实现是基于什么数据结构?
`散列表`

### Q18: 多个defer函数同时存在时，程序会如何处理?
- `先进后出，后进先出的栈处理方式`
- `defer`在`return`语句之后执行，但在函数退出之前，`defer`可以修改返回值

### Q19: 空Select有什么用，如何避免死锁?
- 阻塞主协程
- 主线程内，存在其他运行的协程即可

### Q20: 空结构体struct{}占内存空间吗，可以在哪些场景下使用
- `struct{}`不占用内存空间，`unsafe.Sizeof`可说明

实用场景
- 基于map实现set，对于集合来说，只需要map的键，而不需要值
- 不发送任何数据的信道，只用来通知子协程(goroutine)执行任务，或只用来控制协程并发度
- 仅包含方法的结构体，虽说可以为任意数据结构绑定方法，但其他类型都需要占用额外的内存空间

### Q21: Go里面，有异常类型的概念吗？
> Go 没有异常类型，只有错误类型（Error），通常使用返回值来表示异常状态

### Q22: 什么是 rune 类型?
> 正常ASCII码的所有值，只需要7bit就能全部表示，但只能表示英文字母在内的128个字符；为了统一世界上所有的语言，引入了Unicode编码，它是ASCII的超集；
> 它能表示所有字符，而Go里面，unicode称之为rune，是int32类型的别名；在Go语言中，字符串的底层表示是byte(8bit)序列，而非rune(32 bit)序列。
> Go的默认字符串编码方式是UTF8，一个汉字占3个字节。下面的输出，你看懂了吗?
```go
fmt.Println(len("Go一百问")) // 11
fmt.Println(len([]rune("Go一百问"))) // 5
```

### Q23: 字符串打印时，`%v`和`%+v`的区别
_%v 和 %+v 都可以用来打印 struct 的值，但%v 仅打印各个字段的值，%+v 还会打印各个字段的名称_

### Q24: Go中如何定义enum枚举值
_使用常量(const)来表示枚举值_
```go
type EnumType int32
const (
	XXXXX EnumType = iota
	YYYYY
	ZZZZZ
	DDDDD
)
```
