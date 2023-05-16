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



### 1 Python Data Structure

```c++

```



### 2 Arrays and Sequence





### 3 Generic Programming and Template Classes

## Part III: Classes and Protocols

### Chapter 13(旧10)：Interfaces, Protocols and ABCs

**<u>Summary 从鸭子类型到白鹅类型，再到静态类型检查：</u>**

鸭子类型，只关注对象的行为，即有无实现特定的协议，是否包含所需的方法、签名和语义即可作为子类来直接使用，比如实现了\__len\__就可以使用len()，该内置函数并不关心用户传进来的对象的类型，而只关心其有无实现\__len\__。

因此最佳实践是直接try except，但是有些场景不适用。如当多个本不相关的类(Painter, Lottery)都有相同名字的函数draw，某个函数接收参数对象，其函数体内的try except直接调用该参数对象的draw函数，不会被tryexcept捕获，行为无法预估。

因此引出白鹅类型，白鹅类型即将子类显式地继承抽象基类，明确声明ABC内的抽象接口(尽管有些接口可能使用不到)。此时可以使用isinstance和issubclass，来判断是否为子类或实例。但注意，尽管白鹅类型使用这两个函数来判断，但反过来使用这两个函数return True的并不一定是白鹅类型。因为当基类实现了subclasshook特殊方法，要求子类的\__dict\__中必须包含指定的某个方法名（类的\__dict\__ 与对象的不同，对象的仅包含属性，但类的包含方法），就可以通过isinstance测试。因此，不通过显式继承，而通过register（不会自动继承基类的属性和方法）成为虚拟子类，甚至只是自己有这个函数名，都会通过issubclass的检查。（因此推荐，显式继承以确保子类相同的行为，*或register*，这里我有点疑惑，仅仅register也不能保证行为一样，也许作者意思是当你register的时候你确实知道你该主动实现那些方法？可能作用的对象不同，register是对子类进行操作而subclasshook是对基类操作。总之就是，在基类使用subclasshook没有必要，麻烦且不能保证拥有改名字的类真的有想要的行为）

此时，又产生了新的问题，有些类，拥有某个函数名，但该函数并不会成功执行，而是raise Exception，但仍能通过isinstance。比如complex类拥有\__float\__函数，但调用该函数并不能真正的将一个复数对象转换成float，而是抛出异常。这种情况下，complex仍会被视作为Protocol SupportsFloat的子类，因为该协议要求实现这样一个float函数，complex的确有这个函数。行为模式不同但能通过isinstance，是有隐患的。

由此引出static type checking。创建继承自typing.protocol的协议类，声明函数的签名类型，返回类型进行约束，必须完全一致。mypy静态类型检查工具，该工具在编译时进行检查，同时也可以加上@runtime_checkable来在运行时检查类型。比使用isinstance issubclass这种代码的方式要更加的可扩展、可读性。




### 19 Concurrency Models in Python

Basic compare the usage differences among multi-thread, multi-process and coroutine:

**Multi-Thread:**

```python
import itertools
import time
from threading import Thread, Event

def spin(msg: str, done: Event)->None:
    for char in itertools.cycle(r"\|/-"):
      	# \r Reset the cursor in the console to keep the output replace itself again and again
        status = f'\r{char} {msg}' 
        print(status, end="", flush=True)
        if done.wait(.1): # if the event is set by another thread, return True; if timeout elapses, False
            break
    blanks = " " * len(status)
    print(f'\r{blanks}\r', end = "")


def slow() -> int:
    time.sleep(3)
    return 42

def supervisor() -> int:
    done = Event()
    spinner = Thread(target=spin, args=("thinking", done))
    print(f'spinner object:{spinner}')
    spinner.start()
    result = slow() # block the main thread until slow returns
    done.set() # set the Event flag to True to end the infinite loop
    spinner.join()
    return result

def main() -> None:
    result = supervisor()
    print(f'Answer:{result}')

if __name__ == "__main__":
    main()
```

**Multi-Process:**

```python
import itertools
import time
from multiprocessing import Process, Event
from multiprocessing import synchronize

# Here multiprocessing.Event is a function not a class like threading.Event
# But it does return a synchronize.Event instance
def spin(msg: str, done: synchronize.Event) -> None:
    for char in itertools.cycle(r"\|/-"):
        status = f'\r{char} {msg}'
        print(status, end="", flush=True)
        # if the event is set by another thread, return True; if timeout elapses, False
        if done.wait(.1): 
            break
    blanks = " " * len(status)
    print(f'\r{blanks}\r', end = "")


def slow() -> int:
    time.sleep(3)
    return 42

def supervisor() -> int:
    done = Event()
    spinner = Process(target=spin, args=("thinking", done))
    print(f'spinner object:{spinner}')
    spinner.start()
    result = slow() # block the main thread until slow returns
    done.set() # set the Event flag to True
    spinner.join()
    return result

def main() -> None:
    result = supervisor()
    print(f'Answer:{result}')

if __name__ == "__main__":
    main()
```

Coroutine:

```python
import itertools
import time
import asyncio


async def spin(msg: str)->None:
    for char in itertools.cycle(r"\|/-"):
        status = f'\r{char} {msg}'
        print(status, end="", flush=True)
        try:
            await asyncio.sleep(.1) # do not use time.sleep in order not to block the event loop
        except asyncio.CancelledError:
            break
    blanks = " " * len(status)
    print(f'\r{blanks}\r', end = "")


async def slow() -> int:
    await asyncio.sleep(3)
    return 42

async def supervisor() -> int:
    spinner = asyncio.create_task(spin("thinking")) # return a Task object added into the event loop
    print(f'spinner object:{spinner}')
    result = await slow()
    spinner.cancel() # Task.cancel would raise a CancelledError exception
    return result

def main() -> None:
    result = asyncio.run(supervisor()) # create and run a event loop
    print(f'Answer:{result}')

if __name__ == "__main__":
    main()
```

**<u>Bulletpoints:</u>**

- Asyncio.create_task(coro()): Called from a coroutine to schedule another coroutine to execute eventually. This call does not suspend the current coroutine. Returns a **Task** instance.

- await coro(): Called from a coroutine to transfer control to the coroutine object returned by coro(). **This suspends the current coroutine until the body of coro returns.**

- My current understanding about the execution sequence: 

  1. Basically, supervisor, spin, slow, they are all coroutines. 加了async都是协程对象

  2. In main function, a event loop is created and add the coroutine supervisor into the queue. At this time, the supervisor gets the control.

  3. In supervisor function, a Task called spin is created and added into the queue but it does not get the control right away.

  4. Keyword await causes the supervisor to be suspended and return the control back to the event loop.

  5. The event loop then check the event queue and since spin has higher priority than slow, it would be executed as the first one.
  6. Spin and supervisor would get the control alternatively.
  7. （？）Once the slow() returns, the supervisor would continute to execute, i.e. cancel the spin. 
  8. Then the spin is ended.
  9. Finally, the supervisor also quited and the event loop terminated.







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




## Part V: MetaProgramming

### 22 Dynamic Attributes and Properties

```python
import keyword
from urllib.request import urlopen
import warnings
import os
import json
from collections import abc

# URL = 'http://www.oreilly.com/pub/sc/osconfeed' # The url is not available any more
URL = 'https://api.github.com/users'

JSON = os.path.join(os.getcwd(), 'data/osconfeed.json')


def load():
    # check if the file exists
    if not os.path.exists(JSON):
        msg = f"downloading {URL} to {JSON}"
        warnings.warn(msg)   # If we need download, show a warning
        with urlopen(URL) as remote, open(JSON, 'wb') as local:  # Use two context manager
            local.write(remote.read())

    # open the JSON file
    with open(JSON) as fp:
        return json.load(fp)  # Parse Json file, return a Python object
      

feed = load()
feed = {f'user{i}': feed[i] for i in range(len(feed))}
print(feed['user0']['login']) # Cumbersome syntax
```

Since the feed['a'] ['b'] [0] ['c'] is really cumbersome and we want to get the attributes like what we could do in JavaScript: feed.a.b[0].c. Therefore, construct a dict-like class named FrozenJSON to implement this function.

```python
class FrozenJSON:

    def __init__(self, mapping):
        # ensure to pass a dict or a dictable object, and create a copy
        # self.__data = dict(mapping)
        self.test = "hit the attr test"
        self.__data = {}
        
        # To check if a key in the mapping is the reserved key in Python.
        # If yes, rename it
        for key, value in mapping.items():
            if keyword.iskeyword(key):
                key += "_"
            self.__data[key] = value

    def __getattr__(self, item):
        # 1st-edition:
        # if hasattr(self.__data, item):
            # return getattr(self.__data, item)
        # else:
            # return FrozenJSON.build(self.__data[item])
          
        # In 2nd-edition, new codes:
        try:
            return getattr(self.__data, item)
        except AttributeError:
            return FrozenJSON.build(self.__data[item])
    
    def __dir__(self):
        return self.__data.keys()

    @classmethod
    def build(cls, obj):
        if isinstance(obj, abc.Mapping):
            # 如果本次调用找到的属性并不是简单的一个值，而是一个字典，比如user0的value 那就返回这个字典对象
            # 把这个obj字典也包装成一个FrozenJson对象，使其支持__getattr__
            # 此处应该是递归了，即新的这个FJ对象又调用了自己的getattr以找到下一个属性
            # print("1", cls(obj))

            return cls(obj)
        elif isinstance(obj, abc.MutableSequence):
            return [cls.build(item) for item in obj]
        else:
            # 直到调用的属性变成了一个值，返回值
            # print("2", obj)
            return obj
```



Flexible Object Creation with \__new__

```python
class FrozenJsonNew:

    def __new__(cls, arg):
        if isinstance(arg, abc.Mapping):
            return super().__new__(cls)  # call
        elif isinstance(arg, abc.MutableSequence):
            return [cls(item) for item in arg]
        else:
            return arg

    def __init__(self, mapping):
        # ensure to pass a dict or a dictable object, and create a copy
        # self.__data = dict(mapping)
        self.test = "hit the attr test"
        self.__data = {}
        for key, value in mapping.items():
            if keyword.iskeyword(key):
                key += "_"
            self.__data[key] = value

    def __getattr__(self, item):
        if hasattr(self.__data, item):
            return getattr(self.__data, item)
        else:
            return FrozenJsonNew(self.__data[item])  # just call the constructor of the class
```



```python
feed = FrozenJsonNew(feed)

print(feed.user0.login)
print(feed.test)


# feed2 = FrozenJSON({'name': 'john', 'class': '59'})
# print(feed2.class_)
```

