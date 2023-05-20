---
layout:     post
title:      "Notes - Fluent Python"
subtitle:   " \"Pythonic\""
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

Magic Method is Python's style.

### 2 Arrays and Sequence





3 Generic Programming and Template Classes



## Part III: Classes and Protocols

### Chapter 13(旧10)：Interfaces, Protocols and ABCs 接口，协议和抽象基类

Summary: From Duck Typing to Swan Typing, and Static Type Checking:

<mark>Duck Typing</mark> focuses on an object's behavior, specifically whether it implements a certain protocol, contains the required methods, signatures, and semantics. For example, if an object implements `__len__`, it can be used with the `len()` function. The built-in `len()` function doesn't care about the object's type; it only cares about whether it implements `__len__`.

The best practice is often to use a try-except approach, but there are scenarios where it may not be suitable. For instance, when multiple unrelated classes (e.g., Painter, Lottery) have a function named `draw`, and a function takes an object as a parameter and directly calls its `draw` function within a try-except block, it may not be caught by the try-except block, and the behavior becomes unpredictable.

This leads to <mark>Swan Typing</mark>, where subclasses explicitly inherit from abstract base classes (ABCs) and declare the abstract interfaces within them (even if some interfaces are not utilized). The `isinstance` and `issubclass` functions can then be used to determine if an object is an instance or subclass. However, it's important to note that just because `isinstance` or `issubclass` returns True, it doesn't necessarily mean it's a Swan Type. This is because if the base class implements the `subclasshook` special method, requiring that the subclass's `__dict__` must contain a specified method name (the class's `__dict__` is different from an object's `__dict__` as it contains methods), it can pass the `isinstance` test. Therefore, using `subclasshook` without explicit inheritance, or using only `register` (which doesn't automatically inherit base class attributes and methods) to become a virtual subclass, or simply having the same function name, can all pass the `issubclass` check. (Hence, it is recommended to use explicit inheritance to ensure consistent behavior or `register` . 

<mark>Here, I'm a bit confused</mark> because registering alone cannot guarantee the same behavior. Perhaps the author means when you register, you should know which methods you need to implement proactively? Maybe the intended use target are different for `register` and `subclasshook`. In any case, using `subclasshook` in the base class is unnecessary, troublesome, and cannot guarantee that a class with the same name actually has the desired behavior.)

However, a new problem arises. Some classes may have a function name but the function itself doesn't execute successfully; it may raise an exception instead of behaving as expected. For example, the `complex` class has a `__float__` function, but calling this function doesn't actually convert a complex object into a float; it raises an exception instead. However, `complex` would still be considered a subclass of the `Protocol` `SupportsFloat` because the protocol requires implementing such a `float` function, and `complex` does have that function. This situation, where the behavior differs but `isinstance` still passes, has its risks.

This leads to <mark>static type checking</mark>. Protocols, which inherit from `typing.Protocol`, can be created to declare the function signature types and return types, providing strict constraints. Tools like mypy can perform static type checking at compile time. Additionally, the `@runtime_checkable` decorator can be used to check types at runtime. This approach using static type checking and protocols provides better extensibility and readability compared to using `isinstance` and `issubclass` in the code.

**<u>Summary 从鸭子类型到白鹅类型，再到静态类型检查：</u>**

鸭子类型，只关注对象的行为，即有无实现特定的协议，是否包含所需的方法、签名和语义即可作为子类来直接使用，比如实现了\__len\__就可以使用len()，该内置函数并不关心用户传进来的对象的类型，而只关心其有无实现\__len\__。

因此最佳实践是直接try except，但是有些场景不适用。如当多个本不相关的类(Painter, Lottery)都有相同名字的函数draw，某个函数接收参数对象，其函数体内的try except直接调用该参数对象的draw函数，不会被tryexcept捕获，行为无法预估。

因此引出白鹅类型，白鹅类型即将子类显式地继承抽象基类，明确声明ABC内的抽象接口(尽管有些接口可能使用不到)。此时可以使用isinstance和issubclass，来判断是否为子类或实例。但注意，尽管白鹅类型使用这两个函数来判断，但反过来使用这两个函数return True的并不一定是白鹅类型。因为当基类实现了subclasshook特殊方法，要求子类的\__dict\__中必须包含指定的某个方法名（类的\__dict\__ 与对象的不同，对象的仅包含属性，但类的包含方法），就可以通过isinstance测试。因此，不通过显式继承，而通过register（不会自动继承基类的属性和方法）成为虚拟子类，甚至只是自己有这个函数名，都会通过issubclass的检查。（因此推荐，显式继承以确保子类相同的行为，*或register*，这里我有点疑惑，仅仅register也不能保证行为一样，也许作者意思是当你register的时候你确实知道你该主动实现那些方法？可能作用的对象不同，register是对子类进行操作而subclasshook是对基类操作。总之就是，在基类使用subclasshook没有必要，麻烦且不能保证拥有改名字的类真的有想要的行为）

此时，又产生了新的问题，有些类，拥有某个函数名，但该函数并不会成功执行，而是raise Exception，但仍能通过isinstance(经过测试，这种必须是个抽象基类+该abc定义了subclasshook方法)。比如complex类拥有\__float\__函数，但调用该函数并不能真正的将一个复数对象转换成float，而是抛出异常。这种情况下，complex仍会被视作为Protocol SupportsFloat的子类，因为该协议要求实现这样一个float函数，complex的确有这个函数。行为模式不同但能通过isinstance，是有隐患的。

由此引出static type checking。创建继承自typing.protocol的协议类，声明函数的签名类型，返回类型进行约束，必须完全一致。mypy静态类型检查工具，该工具在编译时进行检查，同时也可以加上@runtime_checkable来在运行时检查类型。比使用isinstance issubclass这种代码的方式要更加的可扩展、可读性。



### Chapter 14 Inheritance: For Better or for Worse 继承优缺点

Handling multiple inheritance principles:

1. Distinguish between interface inheritance and implementation inheritance: Determine whether the purpose of creating a subclass is to define "what" the subclass is or to avoid code duplication.
2. Explicitly represent interfaces using abstract base classes (ABCs): If a class's purpose is to define an interface, explicitly declare it as a subclass of `abc.ABC` or inherit from other abstract base classes.
3. Reuse code using `mixins`: If a class's purpose is to avoid code duplication and provide method implementations for multiple unrelated subclasses, define that class as a mixin class to package the methods. Mixin classes should not be instantiated directly.
4. Clearly indicate mixins in the name: The class name should include the term "mixin" to indicate its purpose.
5. Abstract base classes can be mixins: Mixin classes can also be subclasses of abstract base classes because abstract base classes can have method implementations. However, the reverse is not true.
6. Avoid subclassing from multiple concrete classes: When a concrete class uses multiple inheritance, it should, at most, inherit from one concrete class, while other superclasses should preferably be abstract base classes or mixins.
7. Provide aggregate classes for users: For multiple inheritance, create aggregate classes that do not provide any class definition themselves.

处理多重继承的原则：

1. 区分接口继承和实现继承：创建子类的目的到底是为了定义子类『是什么』的还是为了避免代码重复？

2. 使用抽象基类显式表示接口：若一个类的目的是定义接口，要明确声明为abc.ABC的子类或继承其他抽象基类

3. 通过mixin来重用代码：若一个类的目的是避免代码重复，为多个不想管的子类提供方法实现，不强调『是什么』的关系，要将该类明确定义为 mixin class以打包方法。<u>混入类不应实例化。</u>

4. 在名称中明确指名mixin。类名中应包括mixin字样。

5. 抽象基类可以是mixin class，因为抽象基类中也可以实现方法，但反过来不成立

6. 不要子类化多个具体类。一个具体类进行多重继承，最多应只继承一个具体类，其他的超类最好都是抽象基类或混入类

7. 为用户提供聚合类aggregate class。多重继承，自身不提供任何类定义。

```python
class A:
    def method_a(self):
        print("Method A")

class B:
    def method_b(self):
        print("Method B")

class C:
    def method_c(self):
        print("Method C")

class D:
    def method_d(self):
        print("Method D")

class AggregateClass(A, B, C, D):
    pass

obj = AggregateClass()
obj.method_a()  # 调用超类 A 的方法
# 输出：Method A

obj.method_b()  # 调用超类 B 的方法
# 输出：Method B

obj.method_c()  # 调用超类 C 的方法
# 输出：Method C

obj.method_d()  # 调用超类 D 的方法
# 输出：Method D

```

8. 优先使用对象组合，而非类继承：即将类的实例作为其他类的属性来调用方法。

```python
class Engine:
    def __init__(self, horsepower):
        self.horsepower = horsepower

    def start(self):
        print("Engine started.")

    def stop(self):
        print("Engine stopped.")

class Car:
    def __init__(self, make, model, engine):
        self.make = make
        self.model = model
        self.engine = engine

    def start(self):
        print(f"{self.make} {self.model} is starting the engine.")
        self.engine.start()

    def stop(self):
        print(f"{self.make} {self.model} is stopping the engine.")
        self.engine.stop()

engine = Engine(200)
car = Car("Toyota", "Camry", engine)

car.start()
# 输出：
# Toyota Camry is starting the engine.
# Engine started.

car.stop()
# 输出：
# Toyota Camry is stopping the engine.
# Engine stopped.

```



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

  5. The event loop then check the event queue and since spin has higher priority than slow, it would be executed as the first one. 存疑 现在看起来是slow先拿到控制权
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



**Flexible Object Creation with \__new__**

new方法先于对象创建，init在对象创建之后执行

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

