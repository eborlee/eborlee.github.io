---
layout:     post
title:      "Notes - Fluent Python 流畅的Python"
subtitle:   " \"Pythonic\""
date:       2023-04-29 12:00:00
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


* TOC
{:toc}




### 1 Python Data Structure

Magic Method is Python's style.

### 2 Arrays and Sequence





3 Generic Programming and Template Classes



<!-- ## Part III: Classes and Protocols -->
## **Part III: Classes and Protocols**


### 13(旧10)：Interfaces, Protocols and ABCs 接口，协议和抽象基类

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



### 14 Inheritance: For Better or for Worse 继承优缺点

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



## **Part IV Control Flow 控制流程**

### 19 Concurrency Models in Python 并发模型

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

> **Ebor: 我有个疑问，当slow内部执行到is_prime_async 挂起之后，这个函数内部判断是否为质数的代码是否在运行？在被谁运行？即被哪个线程或者进程之类的东西在运行？这个判断质数的代码如果还在运行的话，为什么事件循环还可以安排其他代码运行？**
>
> Chatgpt: 在Python的`asyncio`库中，协程的执行是由事件循环来调度的。当一个协程在`await`表达式处挂起时，它的状态被保存下来，然后事件循环可以选择其他可以运行的协程来执行。
>
> 在你的例子中，当`is_prime_async`函数在`await`表达式处挂起时，它的状态被保存下来，然后事件循环可以选择其他可以运行的协程，比如`spin`协程。这就是为什么`spin`的旋转不会被阻塞。
>
> 然而，这并不意味着`is_prime_async`函数内部的代码在挂起期间仍在运行。实际上，当一个协程在`await`表达式处挂起时，它的执行被暂停，直到事件循环再次选择它继续执行。这就是为什么我们说协程是协作式的：它们需要显式地让出控制权，以便事件循环可以调度其他协程。
>
> 在你的例子中，`is_prime_async`函数内部的代码在挂起期间不会运行。只有当事件循环再次选择`slow`协程，并在`await is_prime_async`表达式处恢复执行时，`is_prime_async`函数内部的代码才会继续运行。
>
> 需要注意的是，Python的`asyncio`库是单线程的，它使用一个单独的线程（通常是主线程）来运行所有的协程。因此，`is_prime_async`函数内部的代码是在同一个线程中运行的，即使它在`await`表达式处挂起和恢复多次。
>
> 
>
> **Ebor: 那遇到了await is_prime_async之后该函数被挂起，意思是不会进入到这个函数内部 就会直接挂起吗？那么什么时候会真正运行这个prime_async？**
>
> Chatgpt: 在Python的`asyncio`库中，当你在协程中使用`await`关键字调用另一个协程时，当前协程会立即挂起，然后开始执行被调用的协程。这意味着，当你在`slow`协程中调用`await is_prime_async(5_000_111_000_222_021)`时，`slow`协程会立即挂起，然后开始执行`is_prime_async`协程。
>
> 然而，如果`is_prime_async`协程内部也有`await`表达式，那么`is_prime_async`协程也会在那里挂起，然后事件循环可以选择其他可以运行的协程来执行。这就是为什么`spin`协程可以在`is_prime_async`协程挂起期间运行。
>
> 当`is_prime_async`协程完成执行（也就是到达了协程函数的末尾或者遇到了`return`语句），控制权会返回到调用它的协程，也就是`slow`协程。然后，`slow`协程会在`await is_prime_async(5_000_111_000_222_021)`表达式处恢复执行。
>
> 需要注意的是，`is_prime_async`协程的执行是由事件循环来调度的，就像所有其他的协程一样。这意味着，即使`is_prime_async`协程在`await`表达式处挂起，事件循环也可以在稍后的某个时间点选择它继续执行。
>
> 
>
> Ebor: **这是is_prime_async的实现。请你结合这个代码继续分析。是否 is_prime_async协程和spin协程由于各自都有 await asyncio.sleep，所以交替让出控制权再交替获得控制权？**
>
> Chatgpt: 是的，你的理解是正确的。在这个`is_prime_async`协程的实现中，有一个`await asyncio.sleep(0)`表达式。这个表达式会立即挂起`is_prime_async`协程，然后事件循环可以选择其他可以运行的协程来执行。这就是为什么`spin`协程可以在`is_prime_async`协程挂起期间运行。
>
> 当`await asyncio.sleep(0)`表达式的等待时间结束（在这个例子中，等待时间是0，所以它会立即结束），`is_prime_async`协程会被事件循环选择继续执行，直到它再次遇到`await asyncio.sleep(0)`表达式或者到达协程函数的末尾。
>
> 同样，`spin`协程中的`await asyncio.sleep(.1)`表达式也会导致`spin`协程挂起，然后事件循环可以选择其他可以运行的协程来执行。这就是为什么`is_prime_async`协程可以在`spin`协程挂起期间运行。
>
> 因此，`is_prime_async`协程和`spin`协程会交替让出控制权，然后再交替获得控制权。这就是Python的`asyncio`库提供的异步I/O支持的核心部分，它允许你编写可以并发执行的代码，而不需要使用线程或进程。
>
> 
>
> **Ebor: 上述两个协程交替获取和让出控制权的过程中，是否supervisor和slow协程都不会参与到这个获取的过程中？因为slow需要等待is_prime_async返回，而supervisor需要等待slow返回？**
>
> ChatGPT: 是的，你的理解是正确的。在`is_prime_async`协程和`spin`协程交替获取和让出控制权的过程中，`supervisor`协程和`slow`协程都不会参与到这个获取的过程中。
>
> 这是因为`supervisor`协程在`await slow()`表达式处挂起，等待`slow`协程完成；而`slow`协程在`await is_prime_async(5_000_111_000_222_021)`表达式处挂起，等待`is_prime_async`协程完成。只有当`is_prime_async`协程完成，`slow`协程才能恢复执行，然后当`slow`协程完成，`supervisor`协程才能恢复执行。
>
> 因此，只有`is_prime_async`协程和`spin`协程在这个过程中是活跃的，它们会交替获取和让出控制权。这就是Python的`asyncio`库提供的异步I/O支持的核心部分，它允许你编写可以并发执行的代码，而不需要使用线程或进程。
>
> 

await会使得调用者交出控制权，被挂起（不是停止），await的函数在后台运行。

> `总结协程和线程版本：
> 1.asyncio.Task约等于threadings.Thread
> 2.Task对象驱动一个协程对象，Thread对象invokes a callable
> 3.协程通过明确的await来交出控制权
> 4.不需要自己实例化Task，而是通过create_task
> 5.当asyncio.create_task返回一个task对象，已经排期准备好run了。而Thread对象需要明确的start函数调用来run
> 6.多线程版本的supervisor中，slow只是一个普通函数，被主线程调用；而异步版的supervisor里，slow是一个由await驱动的协程
> 7.终止线程没有明确api，需要通过设置类似done event然后判断状态这种方式；而协程可以使用Task.cancel()实例方法，其raise CancelledError在await处
> 8.supervisor协程必须由主线程通过哦asyncio.run来启动`

By the way, what is the difference between time.time() and time.perf_counter():

> 适用场景和精度有所不同。`time.time()` 返回的是自纪元（Epoch，通常为1970年1月1日00:00:00）以来的秒数。这个时间是按照墙上时钟的时间计算的，可能会受到许多因素的影响，比如系统时钟的调整。因此，`time.time()` 更适用于获取实际的日期和时间。`time.perf_counter()` 返回的是一个性能计数器的值，这个计数器提供了最高可用的计时器分辨率（在各个平台上都是如此）。与 `time.time()` 不同，`time.perf_counter()` 返回的时间是单调的（即只增不减），并且不受系统时钟调整的影响。因此，`time.perf_counter()` 更适合用来测量短时间内的代码运行时间。



> **Ebor:**
>
> asyncio异步的方式是否对于程序员来说，对程序的执行控制要比多线程方式好？我的意思是，是否存在这种差异，即多线程更加取决于cpu层面调度的随机性，而异步主要在代码层面，所以更可控？
>
> **ChatGPT:**
>
> 你的理解是基本正确的。多线程和异步编程都是处理并发的方法，但是它们的工作方式和控制粒度有所不同。
>
> 多线程：当你使用多线程时，线程的调度（即决定何时暂停一个线程并切换到另一个线程）是由操作系统的线程调度器控制的。这使得线程在某种程度上是“无序”的，因为你不能直接控制哪个线程会在特定的时间点执行。这种情况可能会导致复杂的并发问题，例如竞态条件和死锁，需要通过同步原语（例如锁和信号量）来解决。
>
> 异步编程：在异步编程（例如使用asyncio）中，你更直接地控制何时发生上下文切换。当你在协程中使用await关键字时，你正在告诉Python：“现在可以切换到其他任务，我将在这里等待”。这种方式可以使你更好地预测程序的行为，因为上下文切换发生的位置是明确的（即await表达式的位置）。这也使得编写复杂的并发代码变得更简单，因为你不需要使用锁和其他同步原语。
>
> 然而，这并不意味着异步编程总是优于多线程。这两种方法都有各自的适用场景。例如，对于I/O密集型任务，异步编程通常是一个很好的选择，因为它可以在等待I/O操作完成时执行其他任务。对于CPU密集型任务，多线程或多进程可能是更好的选择，因为它们可以利用多核CPU的优势。



### 20 Concurrent Executors 并发执行器

future / concurrent.futures / 

future这个概念是concurrent.futures和asyncio的重要概念，两个库都实现了各自的future类

concurrent.futures 是 Python 的一个并发执行模块，提供了高级别的异步执行接口。该模块实现了 ThreadPoolExecutor 和 ProcessPoolExecutor 这两个类，它们是 Executor 类的子类，用于管理线程池和进程池。这个库提供了一个高级接口，使得异步编程变得更加容易。（根据我的理解，CPU密集型用这个库的ProcessPoolExecutor，IO密集型还是用asyncio异步的写法效率更高，可能多线程还是比单线程资源消耗大？）

比较：依序下载 vs 多线程map vs 多线程submit+as completed

```python
import time
from pathlib import Path
from typing import Callable
import httpx

"""
顺序下载
"""
POP20_CC = ('CN IN US ID BR PK NG BD RU JP '
            'MX PH VN ET EG DE IR TR CD FR').split()

BASE_URL = 'http://www.fluentpython.com/data/flags'
DEST_DIR = Path('downloaded')


def save_flag(img: bytes, filename: str):
    # path = DEST_DIR / filename
    # with open(path, 'wb') as fp:
    #     fp.write(img)
    (DEST_DIR / filename).write_bytes(img)


def get_flag(cc: str):
    url = f'{BASE_URL}/{cc.lower()}/{cc.lower()}.gif'.lower()
    resp = httpx.get(url, timeout=5, follow_redirects=True)
    resp.raise_for_status()
    return resp.content


def download_many(cc_list):
    for cc in sorted(cc_list):
        img = get_flag(cc)
        path = cc.lower() + '.gif'
        # print(path)
        save_flag(img, path)
        print(cc, end=' ')
    return len(cc_list)


def main(download_many):
    DEST_DIR.mkdir(exist_ok=True)
    t0 = time.perf_counter()
    count = download_many(POP20_CC)
    elapsed = time.perf_counter() - t0
    print(f'\n{count} flags downloaded in {elapsed:.2f}s')


if __name__ == '__main__':
    main(download_many)
```

















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

