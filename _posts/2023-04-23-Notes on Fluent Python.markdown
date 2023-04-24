---
layout:     post
title:      "Notes - Fluent Python"
subtitle:   " \"Never underestimate Python because of its simplicity.\""
date:       2023-04-23 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Python
    - Notes
---

> “Beautiful is better than ugly.”


# Notes on Fluent Python

2023-04-23

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

[TOC]



## 1 Python Data Structure

```c++
int main(){

}
```



## 2 Arrays and Sequence





3 Generic Programming and Template Classes



## 16 Coroutine



About the usage of namedtuple, it is a data structure in Python allowing user to create tuples with column names.

```python
from collections import namedtuple

# Result = namedtuple('Result', ['count', 'average'])
Result = namedtuple('Result', 'count average') # declare a tuple type called Result

r = Result(1,2) # create an instance of the created type
print(r.count) # print the value of attribute 'count'
print(r.average)

# Output
>>> 1
>>> 2
```

When specify the field names for the namedtuple, not only we can pass a list containing these names but also we could pass a string with format "name1 name2 name3".

**Benefits for using namedtuple:**

1. More readable and maintainable due to the field names. And enable user to access elements directly with their field names.

2. More reliable. Once created, the filed name cannot be modified.

3. Faster access. Namedtuple bases on classes rather than dictionaries.

4. Substitute for a class to store values.

5. Easy to copy with _replace method.

   ```python
   r2 = r._replace(count=3)
   print(r2)
   
   >>> Result(count=3, average=2)
   ```

