---
title: Python编码规范   
date: 2021/03/28 15:20:00   
comments: true   
tags:
  - Python   
categories: Python   
excerpt: 好的编码风格，能够让你的代码更优雅、更简洁，还能避免很多坑.
---
### 变量交换
```python
>>> a = 1
>>> b = 2
>>> tmp = a
>>> a = b
>>> b = tmp
# pythonic
>>> a, b = b, a
```
### 循环遍历区间元素
```python
for i in [0, 1, 2, 3, 4, 5]:
    print i
# 或者
for i in range(6):
    print i
# pythonic
for i in xrange(6):
    print i
```
### 带有索引位置的集合遍历
```python
colors = ['red', 'green', 'blue', 'yellow']
 
for i in range(len(colors)):
    print i, '--->', colors[i]
# pythonic
for i, color in enumerate(colors):
    print i, '--->', color
```
### 字符串连接
```python
# 字符串连接时，普通的方式可以用 + 操作
names = ['raymond', 'rachel', 'matthew', 'roger',
         'betty', 'melissa', 'judith', 'charlie']
s = names[0]
for name in names[1:]:
    s += ', ' + name
print s
# pythonic
print ', '.join(names)
```
### 打开/关闭文件
```python
f = open('data.txt')
try:
    data = f.read()
finally:
    f.close()
# pythonic
with open('data.txt') as f:
    data = f.read()
```
### 列表推导式
```python
result = []
for i in range(10):
    s = i  2
    result.append(s)
# pythonic
[i2 for i in xrange(10)]
```
### 善用装饰器
```python
def web_lookup(url, saved={}):
    if url in saved:
        return saved[url]
    page = urllib.urlopen(url).read()
    saved[url] = page
    return page
# pythonic
import urllib # py2
# import urllib.request as urllib # py3
def cache(func):
    saved = {}
    def wrapper(url):
        if url in saved:
            return saved[url]
        else:
            page = func(url)
            saved[url] = page
            return page
    return wrapper
@cache
def web_lookup(url):
    return urllib.urlopen(url).read()
```
### 合理使用列表
```python
names = ['raymond', 'rachel', 'matthew', 'roger',
         'betty', 'melissa', 'judith', 'charlie']
names.pop(0)
names.insert(0, 'mark')
# pythonic
from collections import deque
names = deque(['raymond', 'rachel', 'matthew', 'roger',
               'betty', 'melissa', 'judith', 'charlie'])
names.popleft()   # deque 是一个双向队列的数据结构，删除元素和插入元素会很快
names.appendleft('mark')
```
### 序列解包
```python
p = 'vttalk', 'female', 30, 'python@qq.com'
name = p[0]
gender = p[1]
age = p[2]
email = p[3]
# pythonic
name, gender, age, email = p
```
### 遍历字典的 key 和 value
```python
# 方法一
for k in d:
    print k, '--->', d[k]
# 方法二
for k, v in d.items():
    print k, '--->', v
# pythonic
for k, v in d.iteritems():
    print k, '--->', v
```
### 链式比较操作
```python
age = 18
if age > 18 and x < 60:
    print("yong man")
# pythonic
if 18 < age < 60:  
    print("yong man")
>>> False == False == True  # 这也是一个链式比较
False
```
### if/else 三目运算
```python
if gender == 'male':
    text = '男'
else:
    text = '女'
# pythonic
text = '男' if gender == 'male' else '女'
```
### 真值判断
```python
if attr == True:
    do_something()
 
if len(values) != 0:  # 判断列表是否为空
    do_something()
# pythonic
if attr:
    do_something()
 
if values:
    do_something()
```
### for/else语句
```python
flagfound = False
for i in mylist:
    if i == theflag:
        flagfound = True
        break
    process(i)
if not flagfound:
    raise ValueError("List argument missing terminal flag.")
# pythonic
for i in mylist:
    if i == theflag:
        break
    process(i)
else:
    raise ValueError("List argument missing terminal flag.")
```
### 字符串格式化
```python
s1 = "foofish.net"
s2 = "vttalk"
s3 = "welcome to %s and following %s" % (s1, s2)
# pythonic
s3 = "welcome to {blog} and following {wechat}".format(blog="foofish.net", wechat="vttalk")
```
### 列表切片
```python
items = range(10)
# 奇数
odd_items = []
for i in items:
    if i % 2 != 0:
        odd_items.append(i)
# 拷贝
copy_items = []
for i in items:
    copy_items.append(i)
pythonic
# 第1到第4个元素的范围区间
sub_items = items[1:4]
# 奇数
odd_items = items[1::2]
# 拷贝
copy_items = items[::] 或者 items[:]
```
### 善用生成器
```python
def fib(n):
    a, b = 0, 1
    result = []
     while b < n:
        result.append(b)
        a, b = b, a+b
    return result
# pythonic
def fib(n):
    a, b = 0, 1
    while a < n:
        yield a
        a, b = b, a + b
# 上面是用生成器生成费波那契数列。生成器的好处就是无需一次性把所有元素加载到内存，只有迭代获取元素时才返回该元素，而列表是预先一次性把全部元素加载到了内存。此外用 yield 代码看起来更清晰。
```
### 获取字典元素
```python
d = {'name': 'foo'}
if d.has_key('name'):
    print(d['name'])
else:
    print('unkonw')
# pythonic
d.get("name", "unknow")
```
### 预设字典默认值
```python
# 通过 key 分组的时候，不得不每次检查 key 是否已经存在于字典中。
data = [('foo', 10), ('bar', 20), ('foo', 39), ('bar', 49)]
groups = {}
for (key, value) in data:
    if key in groups:
        groups[key].append(value)
    else:
        groups[key] = [value]
# pythonic
#　第一种方式
groups = {}
for (key, value) in data:
    groups.setdefault(key, []).append(value) 
# 第二种方式
from collections import defaultdict
groups = defaultdict(list)
for (key, value) in data:
    groups[key].append(value)
# dict.fromkeys('hello', 'world')
```
### 字典推导式
```python
# 在python2.7之前，构建字典对象一般使用下面这种方式，可读性非常差
numbers = [1,2,3]
my_dict = dict([(number,number*2) for number in numbers])
print(my_dict)  # {1: 2, 2: 4, 3: 6}
# pythonic
numbers = [1, 2, 3]
my_dict = {number: number * 2 for number in numbers}
print(my_dict)  # {1: 2, 2: 4, 3: 6}
 
# 还可以指定过滤条件
my_dict = {number: number * 2 for number in numbers if number > 1}
print(my_dict)  # {2: 4, 3: 6}

```
### 交换两个变量值
```python
a,b = b,a
```
### 去掉list中的重复元素
```python
old_list = [1,1,1,3,4]
new_list = list(set(old_list))
```
### 翻转一个字符串
```python
s = 'abcde'
ss = s[::-1]
```
### 用两个元素之间有对应关系的list构造一个dict
```python
names = ['jianpx', 'yue']
ages = [23, 40]
m = dict(zip(names,ages))
```
### 将数量较多的字符串相连，如何效率较高，为什么
```python
fruits = ['apple', 'banana']
result = ''.join(fruits)
# python字符串效率问题之一就是在连接字符串的时候使用‘+’号，例如 s = ‘s1’ + ‘s2’ + ‘s3’ + ...+’sN’，总共将N个字符串连接起来， 但是使用+号的话，python需要申请N-1次内存空间， 然后进行字符串拷贝。原因是字符串对象PyStringObject在python当中是不可变 对象，所以每当需要合并两个字符串的时候，就要重新申请一个新的内存空间 （大小为两个字符串长度之和）来给这个合并之后的新字符串，然后进行拷贝。 所以用+号效率非常低。建议在连接字符串的时候使用字符串本身的方法 join（list），这个方法能提高效率，原因是它只是申请了一次内存空间， 因为它可以遍历list中的元素计算出总共需要申请的内存空间的大小，一次申请完
```
### 多用lambda函数
```python
variable = lambda:  True
```
### 巧用 or 子句
```python
variable =  a  or  b
```
### 如何检测用户有没有传进来你想要的参数呢？
> 一个函数需要测试某个可选参数是否被使用者传递 进来。这时候需要小心的是你不能用某个默认值比如`None`、`0`或者`False`值来测试用 户提供的值因为这些值都是合法的值，是可能被用户传递进来的 。因此，你需要其他的解决方案了。
为了解决这个问题，你可以创建一个独一无二的私有对象实例，就像上面的`_no_value`变量那样。在函数里面，你可以通过检查被传递参数值跟这个实例是否一样来判断。这里的思路是用户不可能去传递这个 实例作为输入。因此，这
里通过检查这个值就能确定某个参数是否被传递进来了。  

> 这里对`object()`的使用看上去有点不太常见。`object`是`python`中所有类的基类。你可以创建`object`类的实例，但是这些实例没什么实际用处，因为它并没有任何有用 的方法，也没有哦任何实例数据 因为它没有任何的实例字典，你甚至都不能设置任何 属性值 。你唯一能做的就是测试同一性。这个刚好符合我的要求，因为我在函数中就 只是需要一个同一性的测试而已。
```python
_no_value = object()
def  spam(a,  b=_no_value):
    if  b is  _no_value:
         print('No b value supplied')
```
