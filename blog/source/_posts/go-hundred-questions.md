---
title: go 一百问   
date: 2021-04-01 16:02:13   
comments: true   
tags: go   
categories: Python   
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
```shell script
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
```bash
func foo(a []int) {
	a[0] = 200
}

func main() {
	a := []int{1, 2}
	foo(a)
	fmt.Println(a)
}
```
陷阱二
```shell script
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
```shell script
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
```shell script
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
```shell script
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
```shell script
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
```shell script
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

