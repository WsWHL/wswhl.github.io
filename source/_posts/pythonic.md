---
title: 优雅的Python代码就该这样写
date: 2020-04-05 14:32:39
categories:
  - Python
tags:
  - Pythonic
---
# 简述
Python本就是很高效的开发语言，相比其他语言而言已经很简洁明了。但在实际的编写中，往往会忽略这一点，本该用很简洁高效的写法实现，不仅能够为我们省去繁重的编写工作量，更能利于后期代码的可阅读性和维护。这里就列举一些常用的简洁高效的写法，反例就不在这里列举。
<!--more -->
# Pythonic
1.变量交换
``` Python
a = 1
b = 2

a, b = b, a
print('a:%d, b:%d' % (a, b))
```
打印输出：
> a:2, b:1

2.可迭代对象
``` Python
for i in range(3):
	print(i)
```
打印输出：
>0
>1
>2

*`range`是python3的写法，在python2中使用的是`xrange`*

3.带索引遍历
``` Python
arr = [1, 3, 5, 7, 9]

for i, num in enumerate(arr):
	print('index:%d, value:%d' % (i, num))

```
打印输出：
>index:0, value:1
>index:1, value:3
>index:2, value:5
>index:3, value:7
>index:4, value:9

4.列表推导
``` Python
arr = [i for i in range(10) if i % 2 == 0]

print(arr)
```
打印输出：
>[0, 2, 4, 6, 8]

5.字符串拼接
``` Python
names = ['张三', '李四', '王五']

print(','.join(names))
```
打印输出
>张三,李四,王五

6.zip创建键值对
``` Python
keys = ['name', 'sex', 'age']
values = ['tim', 'male', 23]

user = dict(zip(keys, values))
print(user)
```
打印输出：
>{'name': 'tim', 'sex': 'male', 'age': 23}

7.遍历字典key和value
``` Python
user = {'name': 'tim', 'sex': 'male', 'age': 23}

for key, value in user.items():
	print('%s = %s' % (key, value))
```
打印输出：
>name = tim
>sex = male
>age = 23

8.有效值判断
``` Python
name = 'this blog'
langs = ['zh', 'en', 'jp']
user = {'name': 'tim', 'sex': 'male', 'age': 23}

if name and langs and user:
	print('True')
```
*空字符串、空列表、空字典都会返回`False`*
打印输出：
>True

9.三元运算
``` Python
# 第一种写法
a = 5
b = 1 if a > 0 else 0
print('b =', b)

# 第二种写法
c = (0, a)[a > 0]
print('c = ', c)
```
*第二种是比较晦涩的写法，其实利用的就是逻辑运算结果真或假，程序本身解释便是1或0，然后按照索引取对应的值*
打印输出：
>b = 1
>c =  5

10.打开文件
使用`with`打开文件操作完成后会自动关闭文件，无需手动关闭。
``` Python
with open('test.txt') as f:
	print(f.read())
```