---
layout:     post
title:      "High Performance Python | 高性能编程"
subtitle:   " \"Cython, Numba, Polars...\""
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

* TOC
{:toc}


---
## **1. Cython**

### Basic Usage

<u>Notes: 本部分涉及到的纯C++版本是将c++文件导入到cython版本的pyx中，是对cython进行编译，与pybind11直接编译c++成python对象是不同的</u>

Compile .pyx file and import the compiled module into .py file

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

进一步优化c++函数：py调用时传入np.array, c++中使用NumPy C API操作numpy数组。避免list转vector。

```cython
@cython.boundscheck(False)
@cython.wraparound(False)
cpdef np.ndarray[double] row_sum_cpp(np.ndarray[np.double_t, ndim=2] df):
    cdef vector[double] result = row_sum_cpp_original(df)
    return np.array(result)
```

```c++
#include <vector>
#define NPY_NO_DEPRECATED_API NPY_1_7_API_VERSION
#include <numpy/arrayobject.h>

std::vector<double> row_sum_cpp_original(PyArrayObject* np_array){
    int nrows = PyArray_DIM(np_array, 0);
    int ncols = PyArray_DIM(np_array, 1);
    std::vector<double> result(nrows, 0.0);

    for(std::size_t i = 0; i < nrows; i++){
        for(std::size_t j = 0; j < ncols; j++){

            // 从 NumPy 数组获取值 该宏返回void指针，强制转换为double*，再解引用获取值
            double value = *(double*)PyArray_GETPTR2(np_array, i, j);
            result[i] += value;
        }
    }
    return result;
}
```

```python
>>>
Python function time:6.40712s
Cython function time:0.00473s
Cython C++ function time:0.01663s
```

耗时大幅缩减。

### 引用计数管理

python自动管理内存，Cpython主要通过简单引用计数来实现，自动垃圾收集器定期运行以清理无法访问的引用周期，引用计数到达零时对象被清理（无论动态类型还是静态类型）。

一般来说，从Python接收一个NumPy数组时，不拥有该数组的内存，不能手动释放它。当返回一个NumPy数组给Python时，也放弃了对该数组的内存的所有权。在这两种情况下，内存的管理都是由Python的垃圾回收器负责的。只需要确保正确地处理引用计数，以防止在还需要使用数组的时候它被回收：`Py_INCREF`和`Py_DECREF`

```c++
#include <Python.h>
#include <numpy/arrayobject.h>

// 计算输入数组的平方
static PyObject* square(PyObject* self, PyObject* args)
{
    PyArrayObject* input_array;

    // 解析输入参数（这里假设只有一个参数，且它是一个NumPy数组）
    if (!PyArg_ParseTuple(args, "O!", &PyArray_Type, &input_array)) {
        return NULL;
    }

    // 增加输入数组的引用计数，以防止它在还在使用它的时候被垃圾回收
    Py_INCREF(input_array);

    // 获取输入数组的尺寸
    npy_intp size = PyArray_SIZE(input_array);

    // 创建一个新的NumPy数组来保存结果
    PyArrayObject* output_array = (PyArrayObject*)PyArray_SimpleNew(PyArray_NDIM(input_array), PyArray_DIMS(input_array), NPY_DOUBLE);
    if (output_array == NULL) {
        Py_DECREF(input_array);
        return NULL;
    }

    // 计算平方
    for (npy_intp i = 0; i < size; i++) {
        double* input_data = (double*)PyArray_GETPTR1(input_array, i);
        double* output_data = (double*)PyArray_GETPTR1(output_array, i);

        *output_data = (*input_data) * (*input_data);
    }

    // 我们已经完成了对输入数组的使用，所以可以减少其引用计数
    Py_DECREF(input_array);

    // 返回结果数组
    // 注意：这里不应该减少输出数组的引用计数，因为我们想要返回它给Python
    // 当Python接收到这个返回值时，它将负责管理这个数组的内存
    return (PyObject*)output_array;
}

```

为什么要增加引用计数：

> 当你在C/C++扩展中创建一个Python对象（比如一个NumPy数组），并把它返回给Python代码，你实际上是把这个对象的所有权转移给了Python。此时，你需要增加该对象的引用计数。这样做主要是为了防止在该对象返回给Python代码之前，Python的垃圾回收器误认为这个对象没有被引用（因为引用计数是0）而提前把它销毁。

为什么要减少引用计数：

> 假设你在 C/C++ 扩展中创建了一个 Python 对象，并且增加了其引用计数，但是你忘记了在你不再需要这个对象的时候减少其引用计数。在这种情况下，即使你的 C/C++ 代码不再使用这个对象，Python 的垃圾回收器也不会销毁这个对象，因为它的引用计数不是零。这就导致了内存泄漏，你的程序会持续占用更多的内存，最终可能会导致内存耗尽。

引用计数管理的场景：

> 1. 你创建了一个新的Python对象，并返回给Python环境：当你创建了一个新的Python对象并准备返回给Python环境时，你需要调用`Py_INCREF`来增加这个对象的引用计数。这是因为当你创建一个Python对象时，它的引用计数默认是1，当你返回这个对象给Python环境时，你其实是将它的所有权交给了Python环境，所以你需要增加引用计数。否则，如果Python环境在你返回这个对象之后立即将其赋值给另一个变量，原来的对象就可能被销毁，导致新变量引用了一个无效的对象。
> 2. 你保留了一个Python对象的引用：如果你在C++代码中保留了一个Python对象的引用（例如，将它保存在一个全局变量或者一个静态变量中），你需要调用`Py_INCREF`来增加这个对象的引用计数。这是因为Python的垃圾回收器会在一个对象的引用计数降到0时销毁这个对象，所以如果你没有增加引用计数，对象可能在你还需要它的时候被销毁。
> 3. 你不再需要一个Python对象的引用：如果你不再需要一个Python对象的引用（例如，你保存在全局变量或者静态变量中的对象不再需要），你需要调用`Py_DECREF`来减少这个对象的引用计数。这样，Python的垃圾回收器就可以在适当的时候销毁这个对象。
>
> 需要注意的是，调用`Py_DECREF`时需要小心，因为它可能会导致对象被立即销毁。如果你在调用`Py_DECREF`之后还需要访问这个对象，就会导致未定义的行为。所以，在调用`Py_DECREF`之后，你应该立即将所有指向这个对象的指针设置为NULL，以防止意外地访问已经被销毁的对象。



## 2. Pybind11

### Compile a simple example

将C++代码编译并导入到python

基本步骤：

1. 安装pybind11软件包和库，安装一个新的python环境。这一步是既对环境安装软件包，使得c++可以找到该库。也需要对python环境安装pybind11的python库。

```shell
brew install pybind11

# 路径：
# usr/local/Cellar/pybind11/2.10.3

brew install python3
```

2. 编写c++代码

```c++
#include <pybind11/pybind11.h>

int add(int i, int j){
    return i+j;
}

PYBIND11_MODULE(example, m){
    m.def("add", &add);
}
```

但是此时第一行导入会报错，一是找不到pybind11，二是找到了pybind11后提示里面的代码找不到python.h头文件

首先对项目目录下的.vscode文件夹中的c_cpp_properties.json进行修改。

使用`find /usr/local -name Python.h`命令查找刚刚安装的python3头文件路径：

/usr/local/Cellar/python@3.10/3.10.9/Frameworks/Python.framework/Versions/3.10/include/python3.10/

​	注意：使用`python3-config --includes`该命令是另外一个目录。

由于pybind11的目录就在/usr/local/include中，所以也要加上

```json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**",
                "/Users/admin/Documents/coding/environment/eigen-3.4.0/",  
                // Add the following dirs
                "/usr/local/Cellar/python@3.10/3.10.9/Frameworks/Python.framework/Versions/3.10/include/python3.10/**",
                "/usr/local/include/**"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/g++",  // 编译器路径
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "gcc-x64"
        }
    ],
    "version": 4
}
```

3. 编译c++代码

   一度报错大量的涉及pybind11，clang: error: linker command failed with exit code 1的error

   ```shell
   c++ -O3 -Wall -shared -std=c++11 -fPIC \
     -I/usr/local/opt/python@3.10/Frameworks/Python.framework/Versions/3.10/include/python3.10 \
     example.cpp -o example$(python3-config --extension-suffix) \
     -L/usr/local/opt/python@3.10/Frameworks/Python.framework/Versions/3.10/lib \
     -lpython3.10
   ```

   > -O3: 这是编译器优化级别的选项。-O3 表示使用高级别的优化，以最大程度地提高生成的代码的性能。
   >
   > -Wall: 启用所有警告。这个选项会在编译时显示潜在的代码问题和错误。
   >
   > -shared: 告诉编译器生成一个共享库，这是 Python 扩展模块所需的。
   >
   > -std=c++11: 指定使用 C++11 标准来编译代码。这表示代码中可以使用 C++11 标准引入的特性。
   >
   > -fPIC: 生成位置无关的代码。这对于共享库非常重要，因为它允许库加载到不同的内存位置。
   >
   > -I/usr/local/opt/python@3.10/Frameworks/Python.framework/Versions/3.10/include/python3.10: 指定 Python 头文件的路径，以便编译器知道在哪里找到 Python 的头文件。
   >
   > example.cpp: 您的 C++ 源代码文件的名称。
   >
   > -o example$(python3-config --extension-suffix): 指定生成的共享库文件的输出名称。python3-config --extension-suffix 是一个命令，用于获取 Python 扩展模块的文件扩展名（通常是 .so 或 .pyd）并将其附加到 example。
   >
   > 以下这两个参数设置应该是重复的：
   >
   > -L/usr/local/opt/python@3.10/Frameworks/Python.framework/Versions/3.10/lib: 指定 Python 库文件的路径，以便编译器知道在哪里找到 Python 库。
   >
   > -lpython3.10: 告诉编译器链接到 Python 3.10 库，以便在共享库中使用 Python API。

官方的命令：

```shell
c++ -O3 -Wall -shared -std=c++11 -fPIC $(python3 -m pybind11 --includes) example.cpp -o example$(python3-config --extension-suffix)
```

Both worked. 得到.so文件



4. 编写python代码：确保python脚本与.so文件在同一个目录下

   ```python
   import example
   
   result = example.add(1,2)
   print(result)
   ```

   或进入terminal：python3>>>

此处可能需要将vscode的python编译器手动选择为新安装的python10，因此需要设置该解释器的目录`which python3`：/usr/local/bin/python3

并且也需要设置vscode的code runner这个插件中关于python执行器的设置，也需要设置完整的解释器路径



> 我想知道which python3; find /usr/local -name Python.h;python3-config --includes 这三个分别得到的是什么？
> ChatGPT
>
> which python3：这个命令将显示系统中默认的 Python 3 解释器的路径。它告诉您当前默认的 Python 3 版本的可执行文件的位置。
>
> find /usr/local -name Python.h：这个命令在 /usr/local 目录下搜索文件名为 Python.h 的文件。通常，Python.h 是 Python C API 头文件的一部分，用于开发 Python C 扩展模块。它会返回任何找到的文件路径。
>
> python3-config --includes：这个命令用于查询 Python 解释器的配置信息，包括头文件和库文件的位置。--includes 选项告诉它输出包含文件路径。输出将包含编译 C/C++ 代码所需的头文件路径。
>
> 这三个命令的输出将根据您的系统和配置而有所不同，具体的输出路径取决于您的系统和安装位置。您可以执行这些命令来查看在您的环境中它们的具体输出。
>
> 
>
>
> 那么 /usr/local/Cellar/python@3.10/3.10.9/Frameworks/Python.framework/Versions/3.10/include/python3.10和/usr/local/opt/python@3.10/Frameworks/Python.framework/Versions/3.10/lib分别是什么？
>
> /usr/local/Cellar/python@3.10/3.10.9/Frameworks/Python.framework/Versions/3.10/include/python3.10：这个路径是 Python 3.10 安装的头文件目录。在这里，您可以找到用于编译和开发 Python C/C++ 扩展模块的头文件，包括 Python.h。这些头文件包含了 Python 的 C API，允许您将 C/C++ 代码与 Python 3.10 交互。
>
> /usr/local/opt/python@3.10/Frameworks/Python.framework/Versions/3.10/lib：这个路径是 Python 3.10 安装的库文件目录。它包含了 Python 3.10 的共享库文件，允许您将自己编译的 C/C++ 扩展模块与 Python 3.10 运行时链接在一起，以便在 Python 中使用它们。
>
> 这两个路径是 Python 3.10 安装的一部分，对于开发和编译 Python C/C++ 扩展模块非常重要，因为它们包含了与 Python 3.10 交互所需的头文件和库文件。