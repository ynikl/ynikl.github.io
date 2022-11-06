---
title: "Python 基本语法快速学习"
date: 2022-10-09T17:34:39+08:00
publishDate: 2022-10-09T17:34:39+08:00
draft: false
tags:
- python
---

## 学习路径

- 数据结构
- 流程控制
- 代码组织
- 工程化

## 数据结构

- strings
- int
- list
- tuple
- dictionary

如何查看一个基本的类型对象拥有的方法
``` 
name = "ada"

dir(name)
```

`dir` 获取对象的属性

### 字符串

``` 
name = "ada lovelace"
```

拼接, 用`+`

``` 
first_name = "ada"
last_name = "lovelace"
full_name = first_name + " " + last_name
```

- 声明字符串中 `'`单引号和`"` 双引号没有区别, 可以用来相互替换
- 多行字串使用`'''`连续三个单引号或双引号声明

字符串格式化

format

``` 
>>> 'Hey {name}, there is a 0x{errno:x} error!'.format(
...     name=name, errno=errno)
'Hey Bob, there is a 0xbadc0ffee error!'
```

'f'string 模版

``` 
>>> f'Hello, {name}!'
'Hello, Bob!'
```

### 数字

- 整数 : 不带小数点
- 浮点数 : 带小数点, 存在精度问题

操作符

- `+ - * \ %`

与字符串的转换:

- `str()` 数字转化成字符串
- `int()` 字符串转化成整数
- `float()` 字符串转化成浮点数

### 逻辑

布尔值

- True
- False

与或非

- and
- or
- not

比较

- ==
- !=
- <=
- >= 
- <
- >

转化

- `bool(1)` // True
- `bool("")` // False

### 列表

#### 创建

声明

``` python
people = ['alice', 'fred', 'ian']
```

range
``` 
for i in range(1, 5):
	print(i)
	
// 直接创建一个数组
intList = list(range(1, 5)
```

列表解析

``` 
squares = [ value * 2 for value in range(1, 11)]
```

切片
``` 
newPeoples = people[:3]
```

#### 操作

遍历
``` python
for person in people:
	print(person)
```

### 元组

创建

使用小括号创建, 创建之后内部值无法修改

``` 
aTuple = (1, 3, 5)
```

遍历

``` 
for a in aTuple:
	print(a)
```

### 字典

声明
``` 
peopleCity = { "ian":"xiamen", "jinx":"beijing"}
```

取值
``` 
print(peopleCity["ian"])
```

存在修改, 不存在添加
```
peopleCity["newguy"] = "newcity"
```

删除
```
del peopleCity["ian"]
```

遍历
``` 
for person, city in peopleCity.items():
	print(person)
	print(city)

```


## 流程控制

### if

if...else...

``` 
if True :
	print("")
elif True:
	print("if if ")
else:
	print("")
```

判断是否在列表
``` 
if person in people: 
```

判断列表是否为空
```
if people :
	print("not empty")

```

以下略:

- while
- break
- continue

## 代码组织

### 函数

#### 定义
``` 
def func_name():
	print("function bdoy")
```

#### 普通形参
```
def greating(name, words):
	...
```

#### 关键字参数
```
greating(naem='ian', words='hello')
```

#### 默认值
``` 
def greating(name, words='hello'):
	...
```

#### 返回值
```
def getMeFive():
	return 5
```

#### 可变参数
``` 
def make_pizza(*toppings):
	...
```

使用`*`表示可变参数, 内部生成一个元组传递到函数内部

### 模块

模块为文件, 使用`import`导入模块, 假设有`hello.py`文件, 里有`greating`函数

#### 导入模块 

```
import hello

hello.greating()
```

不用写前缀

```
from hello import *
```

#### 导入函数

```
from hello import greating

greating()
```

### 别名

函数

```
from hello import greating as gt
```

模块
```
import hello as ho
```

### 类

面向对象

#### 声明

``` 
class Dog():

def __init__(self, name, age):
		"""初始化属性name和age"""
		self.name = name
		self.age = age 6
		
def sit(self):
	"""模拟小狗被命令时蹲下""" 
	print(self.name.title() + " is now sitting.")

```

初始化函数 `__init___` 

指向自身引用 `self`

#### 实例化

```
my_dog = Dog("kiki", 3)
my_dog.site()

// 访问属性
print(my_dog.name)
```

访问不存在的属性, 会出错

#### 继承

```
class Car():

	def __init__(self):
```

继承

```
class ElectricCar(Car):

	def __init__(self):
		super().init()
```

#### 从模块导入

与导入方法一致

```
from dog import Dog
```

## 工程化

### 文件

``` 
wiht open(filename) as file_obj:
```

json 转存和加载

- json.dump
- json.load

### 异常

捕获异常

``` 
try:
	print(5/0)
except ZeroDivisionError:
	pass
else:
	...
```

### 单元测试

继承 `unittest`

``` 
ipmort unittest

calss NameTestCase(unittest.TestCase):

	def test_func_name(self):
		// self.assertEqual()
		...

unittest.main()
```

