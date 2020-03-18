---
title: 'Python01-高阶函数'
date: 2020-03-17 21:03:19
tags:
- python
categories:
- python学习
---

### map函数

python内建map()函数，map()函数接收两个参数，一个是函数，一个是Iterator，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回。

```python
def f(x):
	return x*x
r=map(f,[1,2,3,4,5])
list(r)

output:
[1,4,9,16,25]
```

### reduce函数

再看reduce的用法。reduce把一个函数作用在一个序列[x1, x2, x3, ...]上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算，其效果就是

```python
from functools import reduce
def add(x,y):
	return x+y
reduce(add,[1,3,5,7,9])

output:
25
```

练习1：

利用map和reduce编写一个str2float函数，把字符串'123.456'转换成浮点数123.456：

```python
def str2float(s):
  def f1(i):
    d={'0':0,'1':1,'2':2,'3':3,'4':4,'5':5,'6':6,'7':7,'8':8,'9':9}
    return d[i]
  def f2(a,b):
    return a*10+b
  t=0
  for i in range(len(s)-1):
    if s[i]=='.':
      t=i
      break
  s1=s[:t]
  s2=s[t+1:]
  L=len(s)-1-t
  return reduce(f2,map(f1,s1))+reduce(f2,map(f1,s2))*pow(10,-L)
  
print('str2float(\'123.456\') =', str2float('123.456'))
if abs(str2float('123.456') - 123.456) < 0.00001:
    print('测试成功!')
else:
    print('测试失败!')
```



### filter函数

Python内建的filter()函数用于过滤序列。

和map()类似，filter()也接收一个函数和一个序列。和map()不同的是，filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。

例如，在一个list中，删掉偶数，只保留奇数，可以这么写：

```python
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]
```

把一个序列中的空字符串删掉，可以这么写：

```python
def not_empty(s):
    return s and s.strip()

list(filter(not_empty, ['A', '', 'B', None, 'C', '  ']))
# 结果: ['A', 'B', 'C']
```

可见用filter()这个高阶函数，关键在于正确实现一个“筛选”函数。

练习2：

回数是指从左向右读和从右向左读都是一样的数，例如12321，909。请利用filter()筛选出回数：



```python
def is_palindrome(n):
    return str(n) == str(n)[::-1]
  
output = filter(is_palindrome, range(1, 1000))
print('1~1000:', list(output))
if list(filter(is_palindrome, range(1, 200))) == [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 22, 33, 44, 55, 66, 77, 88, 99, 101, 111, 121, 131, 141, 151, 161, 171, 181, 191]:
    print('测试成功!')
else:
    print('测试失败!')
```



### sorted函数

排序也是在程序中经常用到的算法。无论使用冒泡排序还是快速排序，排序的核心是比较两个元素的大小。如果是数字，我们可以直接比较，但如果是字符串或者两个dict呢？直接比较数学上的大小是没有意义的，因此，比较的过程必须通过函数抽象出来

我们给sorted传入key函数，即可实现忽略大小写的排序：

```python
sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
['about', 'bob', 'Credit', 'Zoo']
```

要进行反向排序，不必改动key函数，可以传入第三个参数reverse=True：

```python
sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
['Zoo', 'Credit', 'bob', 'about']
```

练习3：

```python
#请用sorted()对下述列表分别按名字排序
#按成绩从高到低排序
L = [('Bob', 75), ('Adam', 92), ('Bart', 66), ('Lisa', 88)]
#名称
L2 = sorted(L, key=lambda t: t[0])
#成绩
L2 = sorted(L, key=lambda t: t[1], reverse=True)
```

