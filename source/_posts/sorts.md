---
title: 常用排序算法锦集
date: 2020-04-05 14:39:05
categories:
  - 算法
tags:
  - sort
---
### 随机数组
使用numpy生成一维随机数组。

``` Python
from numpy import random

print(random.randint(100, size=20))
```
输出：
> [96 53 14 44 39  0 93  2 59 69 22 56 44 81 98  6 93 81 42 34]
<!--more -->
### 冒泡排序
冒泡排序是指从第一个元素开始，依次与后面的元素比较，如果比其大就进行交换。直到每个元素遍历比较完成为止。

算法：
``` Python
def sort(arrs):
    """冒泡排序"""
    length = len(arrs)
    if length == 0 or length < 2:
        return arrs
    
    for i in range(length):
        for j in range(i + 1, length):
            if(arrs[i] > arrs[j]):
                arrs[i], arrs[j] = arrs[j], arrs[i]
				
    return arrs
```

### 选择排序
选择排序是指假设第一个元素为最小，依次与后面元素比较，找出其中最小值的索引，然后与其进行交换。直到每个元素遍历比较完成为止。

算法：
``` Python
def sort(arrs):
    """选择排序"""
    if arrs is None or len(arrs) < 2:
        return arrs
    
    for i, num in enumerate(arrs):
        min = i
        for j in range(i + 1, len(arrs)):
            if arrs[j] < arrs[min]:
                min = j
        if min != i and arrs[min] < num:
            arrs[i], arrs[min] = arrs[min], arrs[i]
            
    return arrs
```

### 插入排序
插入排序是指从第二个元素开始，依次与其前面的元素，从右到左顺序比较，如果比其大则插入到对应元素前面去，***交换对应索引***继续与前面元素以此比较。

算法：
``` Python
def sort(arrs):
    """插入排序"""
    if arrs is None or len(arrs) <= 0:
        return []
    
    for i in range(1, len(arrs)):
        for j in reversed(range(i)):
            if arrs[i] < arrs[j]:
                arrs[i], arrs[j] = arrs[j], arrs[i]
                i = j
    
    return arrs
```

> 不同算法之间由于其时间复杂度不同，其性能也有所差异。