---
layout:     post
title:      "Notes - Projects Learning | 项目学习笔记"
subtitle:   " \"C++ Implementations and HFT System Designs\""
date:       2023-09-23 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
    - Quant
    - Python
    - Notes
---

> “”


# Projects Learning

2023-06-01

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

* TOC
{:toc}

## Project Sources:

Async Trading Framework,  aat: https://github.com/AsyncAlgoTrading/aat

CppTrader: https://github.com/chronoxor/CppTrader/tree/master

C++ Memory Pool: https://github.com/cacay/MemoryPool

[Spartan]: https://github.com/rigtorp/spartan	"Sparan is a collection of components for High-Frequency trading. It&#39;s goal is to provide components for exchange connectivity and simulation."

Lightning Trader: https://gitee.com/lightning-trader/lightning-futures

### 1) Python

##### 异步+回调的事件驱动机制：



关于是否要使用异步的事件驱动结构：

鉴于我自己的项目是一个用于中低频的python开发的，使用异步+事件驱动+事件循环的意义不是很大。

对于异步的需求仅仅在于



##### periodic定时器

##### 使用动态属性为类的方法添加属性并赋默认值

setattr(EventHandler.onTrade, "_original", 1)

对应hasattr

注意这种方式对象并不会获得类方法的属性

### 2) Speed Up: C++

**Structure：**

**Specific usages：**

编译：Pybind11

RAII智能指针：std::shared_prt & std::unique_ptr

static修饰

#### 高频订单簿的管理：使用内存池

简单的内存池：Boost库或CppCommon `#include "memory/allocator_pool.h"`

```c++
#include <boost/pool/object_pool.hpp>

struct MyClass {
    int a, b, c;
    MyClass(int x, int y, int z) : a(x), b(y), c(z) {}
};

int main() {
    boost::object_pool<MyClass> pool;
    
    MyClass* obj = pool.malloc();
    // 使用obj
    // ...

    pool.destroy(obj);
}
```

CppTrader中的使用：这个项目是用于做交易平台开发，比如交易所的撮合引擎，而非高频交易的交易框架。

MarketManager.h构造函数中创建多个内存池：

```c++
// Bid/Ask price levels
CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _level_memory_manager;
CppCommon::PoolAllocator<LevelNode, CppCommon::DefaultMemoryManager> _level_pool;

// Symbols
CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _symbol_memory_manager;
CppCommon::PoolAllocator<Symbol, CppCommon::DefaultMemoryManager> _symbol_pool;
Symbols _symbols;

// Order books
CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _order_book_memory_manager;
CppCommon::PoolAllocator<OrderBook, CppCommon::DefaultMemoryManager> _order_book_pool;
OrderBooks _order_books;

// Orders
CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _order_memory_manager;
CppCommon::PoolAllocator<OrderNode, CppCommon::DefaultMemoryManager> _order_pool;
Orders _orders;
```

OrderBooks是一个`typedef std::vector<OrderBook*> OrderBooks;`

这里先声明了某个pool中对象的类型，比如OrderBook。而OrderBook类的有参构造函数的形参为`MarketManager& manager, const Symbol& symbol`

以orderbook为例，create语句传的正是OrderBook类构造函数所需的参数。

```c++
ErrorCode MarketManager::AddOrderBook(const Symbol& symbol)
{
    assert(((symbol.Id < _symbols.size()) && (_symbols[symbol.Id] != nullptr)) && "Symbol not found!");
    if ((_symbols.size() <= symbol.Id) || (_symbols[symbol.Id] == nullptr))
        return ErrorCode::SYMBOL_NOT_FOUND;

    // Get the symbol by Id
    Symbol* symbol_ptr = _symbols[symbol.Id];

    // Resize the order book container
    if (_order_books.size() <= symbol.Id)
        _order_books.resize(symbol.Id + 1, nullptr);

    // Create a new order book
    OrderBook* order_book_ptr = _order_book_pool.Create(*this, *symbol_ptr);

    // Insert the order book
    assert((_order_books[symbol.Id] == nullptr) && "Duplicate order book detected!");
    if (_order_books[symbol.Id] != nullptr)
    {
        // Release the order book
        _order_book_pool.Release(order_book_ptr);
        return ErrorCode::ORDER_BOOK_DUPLICATE;
    }
    _order_books[symbol.Id] = order_book_ptr;

    // Call the corresponding handler
    _market_handler.onAddOrderBook(*order_book_ptr);

    return ErrorCode::OK;
}

```



```c++
ErrorCode MarketManager::AddSymbol(const Symbol& symbol)
{
    // Resize the symbol container
    if (_symbols.size() <= symbol.Id)
        _symbols.resize(symbol.Id + 1, nullptr);

    // Create a new symbol
    Symbol* symbol_ptr = _symbol_pool.Create(symbol);

    // Insert the symbol
    assert((_symbols[symbol.Id] == nullptr) && "Duplicate symbol detected!");
    if (_symbols[symbol.Id] != nullptr)
    {
        // Release the symbol
        _symbol_pool.Release(symbol_ptr);
        return ErrorCode::SYMBOL_DUPLICATE;
    }
    _symbols[symbol.Id] = symbol_ptr;

    // Call the corresponding handler
    _market_handler.onAddSymbol(*symbol_ptr);

    return ErrorCode::OK;
}
```







异常处理对性能的影响

类型别名的使用

订单簿-链表的使用

订单簿的管理：std::map



#### 本地订单簿维护：Spartan

`BestPrice` 类

这个类用于存储最优买入和卖出价格及其数量。它有四个成员变量：`bidqty`、`bid`、`ask`和`askqty`，分别表示买入数量、买入价格、卖出价格和卖出数量。

`OrderBook` 类

这个类用于管理一个交易品种的订单簿。它有两个主要的数据结构：

- `buy_` 和 `sell_`：这两个是 `boost::container::flat_map` 类型的成员，用于存储买单和卖单。这里使用 `flat_map` 主要是因为它在内存中是连续存储的，访问效率更高。
- `Level` 结构：用于存储订单的价格、数量和序列号。

`OrderBook` 类还有几个主要的成员函数：

- `GetBestPrice()`：获取最优买入和卖出价格。
- `Add()` 和 `Reduce()`：添加和减少订单。
- `IsCrossed()` 和 `UnCross()`：检查订单簿是否交叉（即买价高于或等于卖价）并解除交叉。

`Feed` 模板类

这个类是整个系统的核心，用于管理多个 `OrderBook`。它有几个主要的数据结构：

- `books_`：一个 `OrderBook` 对象的数组，每个交易品种一个。
- `symbols_`：一个 `HashMap`，用于存储交易品种标识符与其在 `books_` 数组中的索引。

`Feed` 类的主要成员函数包括：

- `Subscribe()`：订阅一个新的交易品种并返回其 `OrderBook`。
- `Add()`、`Executed()`、`Reduce()`、`Delete()` 等：这些函数用于处理订单和交易事件。

```c++
#pragma once

#include "HashMap.h"
#include <boost/container/flat_map.hpp>
#include <iostream>


```



## Backtrader

### 1 利用元编程为创建类时提供额外功能

如何隐式地让子类扩展父类某个属性，而非子类显式拼接或直接覆盖。

```python
class ParamMeta(type):
    def __new__(mcs, name, bases, dct):
        # 合并 params
        params = dct.get('params', ()) + sum(
            (getattr(b, 'params', ()) for b in bases), ()
        )
        # 更新 dct
        dct['params'] = params
        return super().__new__(mcs, name, bases, dct)


class Base(metaclass=ParamMeta):
    params = (('param1', 1), ('param2', 2))


class Child(Base):
    params = (('param3', 3),)


# 测试
print(Base.params)
print(Child.params)

```



```python
class MetaAbstractDataFeed(ABCMeta):

    def __new__(cls, name, bases, dct):
        # 合并 params
        params = sum(
            (getattr(b, 'params', ()) for b in bases), ()
        ) + dct.get('params', ())
        # 更新 dct
        dct['params'] = params
        return super().__new__(cls, name, bases, dct)


class AbstractDataFeed(metaclass=MetaAbstractDataFeed):
    ...
```

为什么需要元类

1. 类的构建过程:
   - 当定义一个类时，Python 实际上是在执行代码来构建这个类。这个过程本身就是由另一个称为“元类”的类来控制的。默认情况下，所有的 Python 类都是由 `type` 这个内置元类来构建的。
2. 自定义类的创建逻辑:
   - 元类允许你拦截并自定义类的创建过程。通过定义一个自定义元类，你可以控制类的定义，比如修改类的属性、方法，或者在类创建时自动执行某些操作。
3. 参数继承的特殊情况:
   - 在我们的例子中，我们想要自动地将基类的 `params` 与子类的 `params` 合并。这个过程需要在类被创建之前发生，因为一旦类被创建，它的属性（包括 `params`）就已经固定下来了。元类允许我们在类正式创建之前介入，并修改这些属性。

为什么不能直接在 `Base` 类中实现：

- 如果我们尝试在 `Base` 类的定义中合并 `params`，这只会在 `Base` 类被创建时执行一次。当创建 `Base` 的子类时，这个合并过程不会自动发生，因为它不是类创建的标准部分。
- 另外，当你在 `Base` 类中定义一些逻辑时，这些逻辑只有在运行类的方法或实例化类的对象时才会执行。而我们需要的是在类定义时就执行参数合并的逻辑。



使用元类可以在更底层、更早的阶段介入类的创建过程，这对于实现一些高级的、自动化的行为（比如自动参数继承）是必需的。尽管使用元类可能会使代码变得更复杂，但它们提供了极大的灵活性和强大的控制能力，特别是在设计需要自动化元数据处理的框架或库时。



元类在 Python 中是一个高级特性，它主要用于创建或修改类定义。元类提供了对类的创建过程的深度控制，允许程序员在类最终被创建之前，拦截并修改类的定义。这种功能可以用于多种用途：

1. **动态修改类属性和方法**：
   - 元类可以在类定义时动态地添加、修改或删除属性和方法。这使得元类成为创建高度动态和灵活的 API 或框架的有力工具。
2. **实现高级抽象**：
   - 元类可以用于构建复杂的抽象，例如，自动化某些模式（如注册模式）、实现接口检查、自动代理方法调用等。
3. **强制类级别的规则**：
   - 通过元类，可以实现对类定义的规则强制，例如，确保子类实现了特定的方法，或者强制命名约定等。
4. **优化类创建过程**：
   - 元类可以被用来优化类的创建过程，例如，通过预先计算和存储类的某些属性来减少运行时开销。
5. **类注册和跟踪**：
   - 元类可以用于在类创建时自动注册类，便于后续的类查找或管理。这在某些需要跟踪类定义的场景中非常有用。
6. **自定义数据模型和描述符**：
   - 元类可以用于创建自定义的数据模型和描述符，例如，修改类属性的访问、赋值或删除行为。
7. **实现单例模式**：
   - 通过元类，可以很容易地实现单例模式，确保类只有一个实例。
8. **类型检查和验证**：
   - 元类还可以用于类型检查和验证，确保类定义符合特定的约束。





## LightningTrader

事件处理的逻辑链：

继承链：注意区分理解`market_api`和`context`的组装关系，前者是后者的一部分，见第5点。所以是先有runtime对象，然后其内部才会创建marketapi对象。而runtime是在engine中创建的。

1. `actual_market`继承自`market_api`，后者是行情解析模块。

2. `asyn_actual_market`继承自`actual_market`和**`queue_event_source`**.

3. `ctp_api_market`继承自`asyn_actual_market`.

4. interface脚本中的`create_actual_market`方法会返回一个`ctp_api_market`对象的指针。

5. runtime类中`init_from_file`方法会调用`create_actual_market`方法以给类成员指针_market赋值（该指针以前向声明的方式所声明），也就是说 `_market`将指向一个`ctp_api_market`对象

6. runtime继承自context。

7. C风格的lightning脚本中的`lt_create_context`负责创建`runtime`对象。

8. engine类的构造函数中调用了`lt_create_context`创建了runtime对象为`ltobj _lt;`进行了赋值

9. engine也继承了**`queue_event_source`**.

10. Runtime_engine继承自engine，最终用户层将创建这个runtime_engine作为app进行策略注册以及`app->start_trading(strategys)`等等

11. 某一类策略比如做市策略`marketing_strategy`继承自`lt::strategy`和 `lt::tick_receiver`。它的初始化要求传入一个engine对象，example脚本中正是通过shared_ptr获取了管理的对象的指针进行了传递。

12. `app->start_trading(strategys)`这个方法是在runtime_engine中定义的。其内部调用了`lt_start_service(_lt)`, 这个方法通过宏的方式，本质是调用了context的`start_service`,即绑定监听事件，创建子线程，和死循环。

13. context的这个`start_service`方法中的绑定动作通过`get_market`, 它在context.h中是一个纯虚的，由子类runtime实现。返回第5点中提到的_market指针。

    ```c++
    get_market().bind_event(market_event_type::MET_TickReceived, std::bind(&context::handle_tick, this, std::placeholders::_1));
    ```

    这其中的`bind_event`方法在`async_market_api`类中（除了这个方法，还有subscribe，unsubscribe，update），内部调用了该类由于继承**`queue_event_source`**而获得的`add_handle`方法。该方法将会把事件类型和该类型对应的处理方法关系保存：

    ```c++
    _handle_map.insert(std::make_pair(type, handle));
    ```

14. `ctp_market_api`的方法`OnRtnDepthMarketData`中进行了行情数据的解析，包装成了一个数据对象tick，然后：

    ```c++
    this->fire_event(market_event_type::MET_TickReceived, tick);
    ```

    ```c++
    template<typename... Types>
    void fire_event(T type, const Types&... args) {
      event_data<T> data;
      data.type = type;
      fire_event(data, args...);
    }
    
    void fire_event(event_data<T>& data)
    {
      while (!_event_queue.insert(data));
    }
    ```

    这个`fire_event`方法就是**`queue_event_source`**中的，用于触发事件并将事件数据插入到事件队列中。fire的时候只需要fire指定的类型名。而不需要管handle。<mark>因为handle方法是在context注册给market对象或者trader对象。同理trader也是一样。fire函数内部自然会处理。也就是说market对象和trader对象各自都维护自己的handlemap和eventqueue。fire负责将新数据插入到eventqueue，由下面一点完成新数据的对应handle</mark>

15. `context`类的`start_service`中的死循环部分，每次循环都会调用它自己定义的update方法，而这个方法update方法调用的是`get_market().update()`（和`get_trader().update()`），这个update方法也就是`asyn_actual_market`中的：

    ```c++
    virtual void update() override
    	{
    		this->process();
    	}
    ```

    这个`process`就是**`queue_event_source`**中的方法负责将事件队列中的的事件取出来，然后进行trigger。

    ```c++
    void process()
    	{
    		event_data<T> data;
    		while (_event_queue.remove(data))
    		{
    			this->trigger(data.type, data.params);
    		}
    	}
    ```

    而trigger就是同类中的方法，从handlemap中找到事件类型对应的handle方法，进行调用。

    ```c++
    	void trigger(T type,const std::vector<std::any>& params)
    	{
    		auto it = _handle_map.equal_range(type);
    		while(it.first != it.second)
    		{
    			it.first->second(params);
    			it.first++;
    		}
    	}
    ```

    比如说，`context`的`start_service`方法绑定了事件`MET_TickReceived`和对应的handle方法`&context::handle_tick`，此时就会调用这个handle_tick. 这个方法将处理新数据对应id和时间戳的比较和更新到容器中。

16. 上述的context类的update方法除了调用`get_market().update()` ：

    ```c++
    void context::update()
    {
    	get_market().update(); // 调用对应handle方法以解析fire插入到market事件队列的新tick数据
      // Cont：这一步handle_tick后_tick_callback，产生了新的策略信号状态值
    	if(is_in_trading())
    	{
    		get_trader().update(); // 调用对应handle方法以解析fire插入到trader事件队列的新order数据
    		this->on_update(); // 空函数 不知作何用
    		if (this->_update_callback) 
    		{
    			this->_update_callback(); // 这一步来遍历strategy_map中的每个策略调用其on_update方法以检查策略信号状态来决定是否需要下单。
    		}
    	}
    }
    ```

    注意下这里的update_callback是在engine的构造函数中创建了runtime对象之后，然后给runtime对象绑定了多个engine中与realtime相关的函数，并不是context原本就有的函数。这也是为什么此处会做了个if判断的原因。

    ~~其中这个`this->on_update()`~~~~就是策略实现类比如套利策略的核心逻辑了。~~ 并不是, 这个方法在context中是纯虚的，而在runtime中的实现是一个空函数（？why）。~~真正的策略核心逻辑调用是，`_update_callback`~~ ：

    <mark>都不是，作者分拆了逻辑判断和执行操作的逻辑。比如套利策略中，`on_tick`负责判断有没有套利机会，如有则修改某个状态值，而`on_update`是根据状态值看要不要下单。</mark> 

    因此第15点所说的context启动时绑定的事件`MET_TickReceived`和对应的handle方法`&context::handle_tick`，这个handle_tick内部调用了`_tick_callback`，这个函数本来也是engine的，被绑定给了runtime对象。这个函数内部调用了`tkrc->on_tick(tick);`

    

    ```c++
    		static inline void _update_callback()
    		{
    			if (_self)
    			{
    				_self->process();
    				for(auto& it: _self->_strategy_map)
    				{
    					it.second->update();
    				}
    				_self->check_condition();
    			}
    		};
    ```

    我的理解这里的`it.second->update();` 就是某个策略的update方法，这个方法将调用on_update方法，后者是具体策略真正要实现的<u>**得到信号后要下单的逻辑**。</u>

    ```c++
    void strategy::update()
    {
    	this->on_update();
    }
    ```

    这个`_self`则是在engine的构造函数中创建完runtime之后，进行了`engine::_self = this;` self指向的这个strategy_map则是example去绑定策略到app的时候，去register了。

17. 综上所述，就事件驱动而言，涉及到三个部分，engine和engine包含两个关键的对象，market和trader。这三个都是**`queue_event_source`**的子类，因此他们内部都有独立的handlemap和eventqueue。他们描述的是交易过程中的三种事件：策略信号事件，市场数据更新事件和订单状态更新事件。

    对于engine，它在启动的时候调用register策略，将策略放入strategymap。见16点，每次市场数据更新后，必然会进行一次ontick策略信号判断，来更新状态值。然后再进行onupdate来决定下单。没有信号的时候会设置为INVALID，所以onupdate也不会去下单。

    我现在的理解是，策略需要编写自己的on_init，由基类的init函数调用，而init函数由engine的init_callback调用。只会初始化执行一次，suber就是engine貌似。策略对象把自己注册成某个品种id或者说code的recevier，实际上是一种字典。

    ```c++
    void arbitrage_strategy::on_init(subscriber& suber)
    {
    	suber.regist_tick_receiver(_code1,this);
    	suber.regist_tick_receiver(_code2, this);
      ...
    ```

    ```c++
    void subscriber::regist_tick_receiver(const code_t& code, tick_receiver* receiver)
    {
    	auto it = _engine._tick_receiver.find(code);
    	if (it == _engine._tick_receiver.end())
    	{
    		_engine._tick_receiver[code].insert(receiver);
    	}
    	else
    	{
    		it->second.insert(receiver);
    	}
    	_engine._tick_reference_count[code]++;
    }
    ```

    每次市场数据更新后，_tick_callback方法会去找receivers容器中code对应的所有策略，然后去调用这些策略的on_tick。这样也解决了多个策略需要同一种品种的情况。

    ```c++
    		static inline void _tick_callback(const tick_info& tick)
    		{
    			if (_self)
    			{
    				auto tk_it = _self->_tick_receiver.find(tick.id);
    				if (tk_it != _self->_tick_receiver.end())
    				{
    					for (auto tkrc : tk_it->second)
    					{
    						if (tkrc)
    						{
    							PROFILE_DEBUG(tick.id.get_id());
    							tkrc->on_tick(tick);
    							PROFILE_DEBUG(tick.id.get_id());
    						}
    					}
    				}
    ```

18. 注意realtime子线程创建的代码部分除了设置线程绑核和优先级等，while部分也在该代码块内部，所以。。此外，由于使用时先创建runtime_engine，这个对象的构造函数中创建了runtime对象，并调用了其init_from_file以创建了market对象和trader对象。这些结束了之后，才是client调用了app.start，也就是context的start_service。因此market和trader对象与交易所的交互以及OnRtnDepthMarketData（其中包含fire）并不在高速线程上，因为高速线程的任务代码部分只包含while部分，也就是procee market对象和trader对象的事件队列里的新数据的时候。

ltobj的定义：

```c++
enum context_type
{
	CT_INVALID,
	CT_RUNTIME,
	CT_EVALUATE
};

struct ltobj
{
	void* obj_ptr;

	context_type obj_type;

	//ltobj(context_type type):obj_type(type), obj_ptr(nullptr) {}
};
```

