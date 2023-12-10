---
title: go坑大勿踩   
date: 2023-12-09 14:36:50   
tags: go   
categories: Go   
excerpt: go作为高级程序设计语言，有其自身的独特性，但也会有容易踩坑的地方；下面将列举一些主要的场景，引以为鉴.
---

### For循环内启动Goroutine问题
以下是一段很常见的代码：
```go
func main() {
	var wg sync.WaitGroup
	envList := []string{"dev", "test", "pre", "prod"}
	for _, env := range envList {
		wg.Add(1)
		go func() {
			defer wg.Done()
			fmt.Println(env)
		}()
	}
	wg.Wait()
}
```
输出的结果始终是prod，因为内部循环goroutine使用的变量一直是循环变量env，而env所代表的内存地址始终就只有一块，也就是说env只能是一个值，不可能代表不同的值。为了解决这个问题，我们需要开辟新的存储空间，来存放每一次的值。修改后的代码如下：
```go
func main() {
	var wg sync.WaitGroup
	envList := []string{"dev", "test", "pre", "prod"}
	for _, env := range envList {
		wg.Add(1)
		go func(envName string) {
			defer wg.Done()
			fmt.Println(envName)
		}(env)
	}
	wg.Wait()
}
```

### For循环struct数组问题
```go
func main() {
	type DD struct {
		Name string `json:"name"`
	}
	ddList := []DD{
		{
			Name: "aaa",
		},
		{
			Name: "bbb",
		},
		{
			Name: "ccc",
		},
	}
	var nameList []*DD
	for _, dd := range ddList {
		nameList = append(nameList, &dd)
	}
	fmt.Printf("%+v", nameList)
}
```
上述的nameList里的，每一个元素都是一样的，指向同一片内存空间。因为你仍然是将循环对象的引用地址追加到nameList里去，所以肯定是相同的。为解决这个问题，应该如下操作：
```go
func main() {
	type DD struct {
		Name string `json:"name"`
	}
	ddList := []DD{
		{
			Name: "aaa",
		},
		{
			Name: "bbb",
		},
		{
			Name: "ccc",
		},
	}
	var nameList []*DD
	for _, dd := range ddList {
		tmpDD := dd    // 也是通过申请新的存储空间来解决
		nameList = append(nameList, &tmpDD)
	}
	fmt.Printf("%+v", nameList)
}
```
但是修改前的代码，如何使用的是指针数组，就不存在上述的问题，请思考下这是为什么，代码如下：
```go
func main() {
	type DD struct {
		Name string `json:"name"`
	}
	ddList := []*DD{
		{
			Name: "aaa",
		},
		{
			Name: "bbb",
		},
		{
			Name: "ccc",
		},
	}
	var nameList []*DD
	for _, dd := range ddList {
		nameList = append(nameList, dd)
	}
	fmt.Printf("%+v", nameList)
}
```
如果修改前的代码里，ddList是基本数据类型，而nameList同样是指针数组，同样有一样的问题，代码如下：
```go
func main() {
	ddList := []string{"aaa", "bbb", "ccc"}
	var nameList []*string
	for _, dd := range ddList {
		nameList = append(nameList, &dd)
	}
	fmt.Printf("%+v", nameList)
}
```
上述同样是由于使用循环变量dd的内存空间导致的，而这个内存空间的始终是固定不变的。如果最开始的代码，这样改，那也没问题：
```go
func main() {
	type DD struct {
		Name string `json:"name"`
	}
	ddList := []DD{
		{
			Name: "aaa",
		},
		{
			Name: "bbb",
		},
		{
			Name: "ccc",
		},
	}
	var nameList []DD
	for _, dd := range ddList {
		nameList = append(nameList, dd)
	}
	fmt.Printf("%+v", nameList) // 每个值都是不一样的
}
```
有些人可能搞不明白了，为啥区别这么大呢，其实也能理解：for的循环变量始终是取值的操作逻辑，当ddList是指针数组时，那循环变量dd的值始终是循环的每一个地址，每一个都是不同的；如果ddList非指针数组，切记不能把他的指针传递给调用方，因为此时他值所在的内存地址始终是固定的，不会变。总计一句话：<strong>for循环非指针数组时，不可直接取地址给依赖方</strong>
