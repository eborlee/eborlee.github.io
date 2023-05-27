---
layout:     post
title:      "Pandas Tricks and Optimizations"
subtitle:   " \"Tricks, Cython, Numba, Polars...\""
date:       2023-05-20 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
mathjax: true
catalog: true
tags:
    - Python
    - Pandas
    - Cython
    - C++
---

# Pandas Tricks & Optimization

2023-04-23

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

[TOC]



### **1. Cython**

Basic Usage: Compile .pyx file and import the compiled module into .py file

​	(1) Original python function: 对大型DataFrame每列求和

```python
import numpy as np

def row_sum(df):
    return df.apply(lambda x:np.sum(x), axis=1)
```

​	(2)Transform to Cython function:

```cython
import numpy as np
cimport numpy as np
# import pandas as pd # Cython不支持导入pandas 故DataFrame类型对象的类型声明只能是object
cimport cython

@cython.boundscheck(False) # 关闭数组边界检查
@cython.wraparound(False) # 关闭负索引检查。此两种都可以降低开销，但要确保代码中没有相关问题
cpdef np.ndarray[double] row_sum_cy(object df):
  	# 声明变量
    cdef Py_ssize_t i, j, nrows = df.shape[0], ncols = df.shape[1]
    cdef np.ndarray[double] result = np.zeros(nrows)
    cdef np.ndarray[double, ndim=2] values = df.values

    for i in range(nrows):
        for j in range(ncols):
            result[i] += values[i,j]

    return result
```

​	(3) Or use C++ function: write C++ function in .cpp file and import it into .pyx file

```c++
// #include <vector>

std::vector<double> row_sum_cpp_original(std::vector<std::vector<double> >& data){
    std::vector<double> result(data.size(), 0.0);
		std::size_t dataSize = data.size();
	
  	// 使用了size_t类型
    for(std::size_t i = 0; i < dataSize; i++){

        const std::vector<double>& rowData = data[i];
        std::size_t rowSize = rowData.size();

        for(std::size_t j = 0; j < rowSize; j++){
            result[i] += rowData[j];
        }
    }
    return result;
}
```

```cython
from libcpp.vector cimport vector

cdef extern from "row_sum.cpp":
    # 声明为cython的function，这里的函数名要与c++中一致
    vector[double] row_sum_cpp_original(vector[vector[double]] data)

@cython.boundscheck(False)
@cython.wraparound(False)
cpdef np.ndarray[double] row_sum_cpp(object df):
    cdef vector[vector[double]] data = df.values.tolist() # pylist转cvector有性能消耗
    cdef vector[double] result = row_sum_cpp_original(data)
    return np.array(result)
```

   (4) Write setup.py to for compilation:	

```python
from setuptools import setup
from Cython.Build import cythonize
import numpy as np

setup(
  	# 必须此处加上language="c++" 才能用c++编译器编译而不是c编译器，否则引入c++标准库报错
    ext_modules=cythonize("row_sum_cy.pyx",language="c++"), 
    # 由于我的gcc和clang都无法自动找到numpy和标准库的位置，手动指定其位置
    include_dirs=[np.get_include(), 
                  # 该目录由echo | clang++ -v -E -x c++ -得到，但通过ls /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include/c++/v1/
也可以
                  "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1"],
    language='c++', # 没用
)


```

​	(5) Compile in terminal: 编译成功后得到python可执行的模块

```shell
python setup.py build_ext --inplace
```

​	(6) Test:

```python
import row_sum # 原始python文件
import row_sum_cy # 编译成功的cython模块
import time
import pandas as pd
import numpy as np

df = pd.DataFrame(np.random.rand(100000,50))

# py版本
start = time.time()
result1 = row_sum.row_sum(df)
print("Python function time:", np.round(time.time()-start,5))

# 直接在pyx中编写的cython版本
start = time.time()
result2 = row_sum_cy.row_sum_cy(df)
print("Cython function time:", np.round(time.time()-start,5))

# 引入的c++函数版本
start = time.time()
result3 = row_sum_cy.row_sum_cpp(df)
print("Cython C++ function time:", np.round(time.time()-start,5))

```

```python
>>>
Python function time: 6.86697
Cython function time: 0.00467
Cython C++ function time: 0.91394
```

Cython提升显著。但是引入纯c++函数反倒没cdef编写的函数快。与py list转c++ vector带来的性能消耗有关系。

也可以`python setup.py bdist_wheel `会自动把编译后的依赖打包进去，使用时不再需要编译器和Cython，但是wheel也不再跨平台。预编译，安装时不需要Cython，便捷。但是mac打包的wheel不能在windows和linux上安装，反之亦然。

