---
layout:     post
title:      "My Trading Framework Development | 实盘框架设计"
subtitle:   " \"Every little bit helps\""
date:       2023-08-25 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
mathjax: true
catalog: true
tags:
    - Notes
    - Software Design
    - Quant
---

# Design Draft of My Real-Time Trading Framework

2023-07-01

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

* TOC
{:toc}

### 1 Observer Mode 观察者模式
目的：创建实例时完成观察关系注册绑定，从而在某些事件逻辑触发后，自动完成观察者和被观察者之间的通知和响应，而无需手动显式触发。比如策略完成下单后，TradingEngine可以自动通知Logger, DatabaseManager和Notifier模块完成相应功能，如日志记录、数据库更新和消息推送，而无需子策略开发者显式地费事地编写下单成功后的通知代码。

观察者Observer: 数据库观察者 / 日志观察者 

被观察者Subject: TradeEngine

抽象基类Subject含有添加观察者、移除观察者和通知观察者notify_observer三个方法，其中通知方法会调用所有观察者对象的update方法。执行对应逻辑；

TradeEngine类实现Subject类，实现以上方法，下单函数order在完成执行后调用notify方法，通知相关的观察者；

抽象基类Observer要求实现抽象方法update，DatabaseManager实现Observer类作为观察者，一旦被被观察者调用update，执行对应的具体逻辑，比如存入数据库；

在main脚本中实例化TradeEngine后register观察者们。
<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/%E5%AE%9E%E7%9B%98%E6%A1%86%E6%9E%B6/ER%20(2).png?raw=true" alt="image-ER" style="zoom:100%;" />


### 2 多进程通信的对象传递问题

主进程的某些实例对象无法简单地通过传递给子进程，即使通过进程间通信Queue也不可以。主要原因是无法序列化pickle，比如数据库连接池对象等。



### 3 异步下任务打断的问题

背景：获取数据、下单等网络IO都基于异步方式，作为协程调用。同时，希望能够支持，telegram端随时可以给策略管理器及子策略发送指令。这个指令应能够被实时响应，如策略正在运行next协程，以处理策略核心部分，即获取数据、计算信号、报撤单等，这个过程应该能够被打断。又比如，本次循环next核心部分已经运行完毕，策略是基于15min Kline的，处于休眠等待下一次kline更新也应该能够被实时打断。

而无论是python的多线程和async异步都没有如Java多线程interrupt打断抛出异常的机制。

改进过程：

```python
async def next_function():
    print("Executing next function for 3秒...")
    await asyncio.sleep(3)  # 模拟 next 函数需要 5 秒钟来完成
    # print("Next function completed.")
    return False

async def sleep_for(seconds):
    print(f"Sleeping for {seconds} seconds...")
    await asyncio.sleep(seconds)
    # print("Sleep completed.")
    return False

async def check_command(command_queue):
    if not command_queue.empty():
        command = command_queue.get()
        print(command)
        if command == 'exit':
            print("收到received.")
            return True
    return False

async def execute_async(command_queue):
    SLEEP = 5  # 模拟休眠 10 秒

    for i in range(10):
        done, _ = await asyncio.wait(
            [next_function(), check_command(command_queue)],
            return_when=asyncio.FIRST_COMPLETED
        )
        # if any(task.result() for task in done):
        #     break

        done, _ = await asyncio.wait(
            [sleep_for(SLEEP), check_command(command_queue)],
            return_when=asyncio.FIRST_COMPLETED
        )
        # if any(task.result() for task in done):
        #     break

async def send_exit_command(command_queue):
    for i in range(3):
        await asyncio.sleep(8)  # 模拟在 8 秒后发送 exit 指令
        command_queue.put('exit')
        print("Exit command sent.")

if __name__ == "__main__":
    command_queue = Queue()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.gather(
        execute_async(command_queue),
        send_exit_command(command_queue)
    ))
```

这个版本导致执行时间为3秒的next协程和5秒的sleep协程快速的交替输出，并不是期待的3+5+3+5的时间关系。

原因：`asyncio.sleep`是非阻塞的，导致next协程和sleep协程各自在asyncio.sleep被挂起，交出控制权，注意此时没有阻塞同时也没有返回。而check_command快速的执行完成并返回，结合FIRST_COMPLETED的设置，导致事件循环继续向下执行。

同时，wait也是非阻塞的，<mark>注意这里，wait的确是非阻塞的，因此事件循环不同于同步编程中将整个程序阻塞，但是代码会不会向下执行取决于有没有其他的异步任务</mark>。return_when针对的是wait中的任务的返回值到底是什么时候，`FIRST_COMPLETED or ALL_COMPLETED`，这个的设置只会影响return的值~~（并且也会影响所谓的效果，即`sys.exit(0)`)~~的早晚，而不会因为没有完成而阻塞。



版本2： 将`asyncio.slee`p改为`time.sleep`以实现阻塞。但是虽然能够把时间阻塞住，但是也直接把整个代码和事件循环阻塞住了，指令甚至无法发出，更别说接受指令和响应。还是恢复成`asyncio.sleep`



版本3：

```python
async def exit():
    # sys.exit(0)
    print(datetime.now().strftime('%H:%M:%S'), "假退出")

async def check_command(command_queue):
    if not command_queue.empty():
        command = command_queue.get()
        if command == 'exit':
            print(datetime.now().strftime('%H:%M:%S'), "收到exit, 准备终止进程")
            await exit()


    return False

async def execute_async(command_queue):
    SLEEP = 5  # 模拟休眠 10 秒

    for i in range(10):
        done, _ = await asyncio.wait(
            [next_function(), check_command(command_queue)],
            return_when=asyncio.ALL_COMPLETED
        )
        # if any(task.result() for task in done):
        #     break
        # await asyncio.sleep(3)
        done, _ = await asyncio.wait(
            [sleep_for(SLEEP), check_command(command_queue)],
            return_when=asyncio.ALL_COMPLETED
        )
        # if any(task.result() for task in done):
        #     break
        # await asyncio.sleep(SLEEP)
async def send_exit_command(command_queue):
    for i in range(3):
        await asyncio.sleep(6)  # 模拟在 6 秒后发送 exit 指令
        command_queue.put('exit')
        print(datetime.now().strftime('%H:%M:%S'), "发出退出指令")
        
>>> 
21:17:06 开始执行next 3秒...
21:17:09 Sleeping for 5 seconds...
21:17:12 发出退出指令
21:17:14 收到exit, 准备终止进程
21:17:14 假退出
```

问题：改为了ALL_COMPLETED以期待防止check_command快速执行完成返回而导致错误地进入下一次循环。避免了交替输出的问题。但是却导致了12秒发出退出指令，14秒才收到，即sleep完整的5秒结束后才收到并响应。那么就意味着当子策略休眠15分钟的时候，退出指令只能在15min休眠结束后才能生效。关于这个问题，其实是被蒙蔽了。在for循环后加一个print("---------")则有如下输出：

```python
>>>
15:40:18  开始执行next 3秒...
15:40:21  Sleeping for 5 seconds...
15:40:24 发出退出指令
---
15:40:26 收到exit, 准备终止进程
15:40:26 假退出
15:40:26  开始执行next 3秒...
15:40:29  Sleeping for 5 seconds...
15:40:30 发出退出指令
```

说明21+5到26秒的这个收到的exit，实际上是下一次循环中的next协程那里的wait中的check_command去做的，即21秒的这个wait部分中的check_command的确是并发处理的，然而已经早早的发现没收到新消息所以返回了（还没到24秒就return了）。<u>所以并不是说协程check_command中一定要等wait返回才能将sys.exit()生效。</u>

**另外**，关于ALL_COMPLETED为什么会可以避免交替输出，看起来是“阻塞”了：实际上并没有阻塞，事件循环依然是活跃的。但是这个场景下，~~由于没有其他的异步任务被创造出来需要处理，所以事件循环看起来像是停滞了~~，<mark>(这句话半对半不对，确实是没有其他异步任务要处理，但是这不是导致看起来停滞的原因，真正的原因是wait就是使得程序暂停了，而FIRST_COMPLETED的快速返回导致wait完成并没有看起来有等待的效果)</mark> 因为ALL的原因导致即便check_command快速完成，该wait任务也并没有真的返回，程序处于等待。而FIRST_COMPLETED场景下，由于check_command快速返回满足了wait完成的条件，导致该任务完成，向下执行。

**新的疑惑是**：确实ALL_COMPLETED下代码不会继续向下执行，比如从next wait处执行到sleep wait处，又或者直接进入下一次循环，那不是还是卡在了这里，阻塞了？

目前的理解：

> Ebor
>
> 就是说，阻塞的概念不是完全等同于 代码不向下执行，而是强调没有其他任务可以插足。所以其实阻塞和等待都有 代码不向下执行的效果，但是强调的是这个不向下执行的期间，能否利用这段时间做其他任务？
>
> ChatGPT
>
> 是的，你的理解是正确的。阻塞和等待都可能导致代码暂时不向下执行，但它们之间有一个关键区别：
>
> - 阻塞：在阻塞期间，整个线程都会被挂起，无法执行其他任务。这意味着，如果你的程序是单线程的，那么整个程序都会暂停，直到阻塞操作完成。
> - 异步等待：在异步等待期间，虽然特定的任务（或协程）可能暂时不会向下执行，但事件循环仍然可以运行其他任务。这意味着你可以在等待一个任务完成的同时执行其他任务。
>
> 所以，阻塞和异步等待都可能导致代码暂时不向下执行，但异步等待允许在这段时间内执行其他任务，而阻塞则不允许。这就是为什么异步编程通常更高效：它允许你更充分地利用系统资源。



版本4：引入cancel()，通过判断checkcommand这个任务在不在done中，如果在意味着已经处理完了，一定要等待next或sleep任务完成才向下。并且通过checkcommand本身的返回结果来判断有无检查到消息，如有，取消next或sleep任务。

```python
async def check_command(command_queue):
    if not command_queue.empty():
        command = command_queue.get()
        if command == 'exit':
            print(datetime.now().strftime('%H:%M:%S'), "收到exit, 准备终止进程")
            # for task in tasks_to_cancel:
            #     task.cancel()
            await exit()
            return True
    return False

async def execute_async(command_queue):
    SLEEP = 5  # 模拟休眠 10 秒


    check_command_task = asyncio.create_task(check_command(command_queue))

    for i in range(10):
        next_task = asyncio.create_task(next_function())
        check_command_task = asyncio.create_task(check_command(command_queue))
        done, _ = await asyncio.wait(
            [next_task, check_command_task],
            return_when=asyncio.FIRST_COMPLETED
        )
        if check_command_task in done:
            await next_task
            # next_task = asyncio.create_task(next_function())

        # if any(task.result() for task in done):
        #     break
        # await asyncio.sleep(3)
        sleep_task = asyncio.create_task(sleep_for(SLEEP))
        done, _ = await asyncio.wait(
            [sleep_task, check_command_task],
            return_when=asyncio.FIRST_COMPLETED
        )
        if check_command_task in done:
            ifExit = check_command_task.result()
            print(datetime.now().strftime('%H:%M:%S'), ifExit)
            if ifExit:
                sleep_task.cancel()
            else:
                await sleep_task
            # sleep_task = asyncio.create_task(sleep_for(SLEEP))
        # if any(task.result() for task in done):
        #     break
        # await asyncio.sleep(SLEEP)
>>>
21:52:27  开始执行next 3秒...
21:52:30  Sleeping for 5 seconds...
21:52:30 False   # 其实这里也是与上面一样的道理，check_command协程并发的快速执行完了还没到真的有指令的时候
21:52:33 发出退出指令
21:52:35  开始执行next 3秒...
21:52:35 收到exit, 准备终止进程
```

本质还是checkcommand执行太快且1次性的问题



版本5：FIRST_COMPLETED，改为while True的方式，并且将与checkcommand配对的另一个任务传给checkcommand使其可以在检查到消息的情况下在内部cancel掉另一个任务。关于谁先返回的问题：如果next或者sleep先于checkcommand返回了，那自然也说明没检查到新消息，而如果checkcommand能够先返回，则说明一定是检查到消息需要处理了。

```python
async def check_command(command_queue, task_to_cancel):
    while True:
        if not command_queue.empty():
            command = command_queue.get()
            if command == 'exit':
                print(datetime.now().strftime('%H:%M:%S'), "收到exit, 准备终止进程")
                # for task in tasks_to_cancel:
                #     task.cancel()
                task_to_cancel.cancel()
                await exit()
                return True
    return



async def execute_async(command_queue):
    SLEEP = 5  # 模拟休眠 10 秒


    # check_command_task = asyncio.create_task(check_command(command_queue))

    for i in range(10):
        next_task = asyncio.create_task(next_function())
        check_command_task = asyncio.create_task(check_command(command_queue, next_task))
        done, _ = await asyncio.wait(
            [next_task, check_command_task],
            return_when=asyncio.FIRST_COMPLETED
        )
        if check_command_task in done:
            ifExit = check_command_task.result()
            print(datetime.now().strftime('%H:%M:%S'), ifExit)
            if ifExit:
                next_task.cancel()
            else:
                await next_task
            # next_task = asyncio.create_task(next_function())

        # if any(task.result() for task in done):
        #     break
        # await asyncio.sleep(3)
        sleep_task = asyncio.create_task(sleep_for(SLEEP))
        done, _ = await asyncio.wait(
            [sleep_task, check_command_task],
            return_when=asyncio.FIRST_COMPLETED
        )
        if check_command_task in done:
            ifExit = check_command_task.result()
            print(datetime.now().strftime('%H:%M:%S'), ifExit)
            while True:
                if ifExit:
                    sleep_task.cancel()
                else:
                    await sleep_task
                    break
            # sleep_task = asyncio.create_task(sleep_for(SLEEP))
        # if any(task.result() for task in done):
        #     break
        # await asyncio.sleep(SLEEP)
```

22:08:41  开始执行next 3秒。在此之后就卡死了。

while True循环在check_command函数中会导致该任务永不结束，除非遇到exit命令。这样，asyncio.wait在等待check_command和next_task或sleep_task中的一个完成时，会一直等待。



最终版：FIRST_COMPLETED，while True循环检查有无新消息，每次循环后asyncio.sleep(0)交出控制权一次。并且目前看起来checkcommand内部的cancel和if done中的cancel都是多余的，因为注释掉也不会有影响。因为FIRST_COMPLETED+while True+asyncio.sleep(0)的方式使得checkcommand不会一次性过早的返回以满足wait的条件，同时如果next或者sleep协程先完成，那正是说明没有指令进来。<mark>但是！wait指令能够得到满足从而向下执行，但是旧的任务checkcommand并不会自动的取消！依然在事件循环中，随着循环运行次数越来越多，造成资源泄露，特别是如果next和sleep协程得不到取消，更会造成策略逻辑上的混乱，还是需要两处的cancel！！</mark>

（我猜测实际场景中next还是会造成指令的延时，因为比如计算过程中不会释放控制权，而sleep是没这个问题的）

```python
import asyncio
import os
import sys
import time
from queue import Queue
from datetime import datetime
async def next_function():
    print( datetime.now().strftime('%H:%M:%S'), " 开始执行next 3秒...")
    await asyncio.sleep(3)  # 模拟 next 函数需要 5 秒钟来完成
    # time.sleep(3)
    # print("Next function completed.")
    return False

async def sleep_for(seconds):
    print( datetime.now().strftime('%H:%M:%S'), f" Sleeping for {seconds} seconds...")
    await asyncio.sleep(seconds)
    # time.sleep(seconds)
    # print("Sleep completed.")
    return False

async def exit():
    sys.exit(0)
    print(datetime.now().strftime('%H:%M:%S'), "假退出")

async def check_command(command_queue, task_to_cancel):
    while True:
        if not command_queue.empty():
            command = command_queue.get()
            if command == 'exit':
                print(datetime.now().strftime('%H:%M:%S'), f"{task_to_cancel.get_name()}收到exit, 准备终止进程")
                task_to_cancel.cancel()

                await exit()
                return True
        await asyncio.sleep(0)
    return



async def execute_async(command_queue):
    SLEEP = 6  # 模拟休眠 10 秒
    for i in range(10):
        next_task = asyncio.create_task(next_function())
        next_task.set_name("协程next")
        check_command_task = asyncio.create_task(check_command(command_queue, next_task))

        done, _ = await asyncio.wait(
            [next_task, check_command_task],
            return_when=asyncio.FIRST_COMPLETED
        )

        if next_task in done:
            check_command_task.cancel()

        sleep_task = asyncio.create_task(sleep_for(SLEEP))
        sleep_task.set_name("协程sleep")
        check_command_task2 = asyncio.create_task(check_command(command_queue, sleep_task))

        done, _ = await asyncio.wait(
            [sleep_task, check_command_task2],
            return_when=asyncio.FIRST_COMPLETED
        )
        if sleep_task in done:
            check_command_task2.cancel()

async def send_exit_command(command_queue, elapsed=4):

    await asyncio.sleep(elapsed)  # 模拟在 4 秒后发送 exit 指令
    command_queue.put('exit')
    print(datetime.now().strftime('%H:%M:%S'), "发出退出指令")



if __name__ == "__main__":
    command_queue = Queue()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.gather(
        execute_async(command_queue),
        send_exit_command(command_queue,8),
        send_exit_command(command_queue,5)
    ))

```



### 4 是否采用事件驱动的设计