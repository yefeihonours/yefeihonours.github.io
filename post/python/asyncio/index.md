#### 1. Coroutines 协程

Python3 中协程使用标准库中 `asyncio` 中的 `async def` 方法声明或者使用装饰器 `@asyncio.coroutine` . 在Python3.5之后开始使用 `async def` 形式的协程, 并且如果不是需要兼容以前版本的话，尽量使用这种形式.

<!--more-->

直接调用协程代码并不会执行, 在调度器运行之前, 协程并不会返回任何东西. 有两种方式可以启动协程: 在一个已经启动的协程中使用`await coroutines  `或者`yield from coroutines `  ; 或者使用`ensure_future` 方法或者`AbstractEventLoop.create_task()` 方法进行调度执行协程. 

协程(coroutines)或者任务(tasks)只有在循环事件(event loop)运行的时候才能运行. 

**简单的 Hello World 协程**

```
import asyncio

async def hello_world():
    print("Hello World!")
loop = asyncio.get_event_loop()
# 锁定代码调用至 hello_world() 协程运行结束
loop.run_until_complete(hello_world())
loop.close()
```

**协程显示当前日期**

```
import asyncio
import datetime

async def display_date(loop):
    end_time = loop.time() + 5.0
    while True:
      print(datetime.datetime.now())
      if (loop.time() + 1.0) >= end_time:
          break
      await asyncio.sleep(1)

loop = asyncio.get_event_loop()
# 锁定代码调用至 display_date() 协程运行结束
loop.run_until_complete(display_date(loop))
loop.close()
```

**链式协程调用**

```
import asyncio

async def compute(x, y):
    print("Compute %s + %s ..." % (x, y))
    await asyncio.sleep(1.0)
    return x + y

async def print_sum(x, y):
    result = await compute(x, y)
    print("%s + %s = %s" % (x, y, result))

loop = asyncio.get_event_loop()
loop.run_until_complete(print_sum(1, 2))
loop.close()

"""
compute() 被链接在 print_sum() 方法后面:
print_sum() 方法会等待 compute() 方法执行结束返回结果后继续执行

Event Loop           Task             print_sum()            compute()	
    | loop is running  |                  |                     |
    | ---------------> | task is pending  |                     |
    |                  | ---------------> | await compute(1, 2) |
    |                  |                  | ----------------->  |print("Compute...
    |                  | <--------------- | ------------------  |await sleep(1.0)
    | <--------------- |                  | coro is suspended   |coro is suspended
    | sleep 1s         |                  |                     |
    | ---------------> |                  |                     |
    |                  | ---------------- | ----------------->  |coro is running
    |                  |                  | coro is running     |return 1 + 2
    |                  |                  | <-----------------  |coro is done
    |                  |                  | raise StopIteration |
    |                  | <--------------- | print("%s + ...     |
    | <--------------- | raise StopIter.. | coro is done        |
    | loop is done     | task is done     |                     |

"""
```

#### 2. Future

```
# Future类定义
class asyncio.Future(*, loop=None):
	def cancel():
		"""取消 Future 并调度 callbacks 回调"""
	def cancelled():
		"""Return True if the future was cancelled."""
	def done():
		"""Return True if the future is done.[ result | exception | cancelled ]"""
	def result():
		"""返回 Future 设置的 result."""
	def exception():
		"""返回 Future 设置的 exception."""
	def add_done_callback(fn):
		"""Future 完成时调用"""
	def set_result(result):
		"""设置 Future 返回值"""
	def set_exception(exception):
		"""设置 Future 返回异常"""
```

**在 run_until_complete() 使用 Future**

```
import asyncio

async def slow_operation(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')
    
loop = asyncio.get_event_loop()
future = asyncio.Future()
asyncio.ensure_future(slow_operation(future))
loop.run_until_complete(future)
print(future.result())
loop.close()

"""
协程方法进行计算(sleep(1)), 并在 future 中保存 result
run_until_complete() 会等待 future 结束.
"""
```

**在 run_forever() 中使用 Future**

```
import asyncio

async def slow_operation(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')
    
def got_result(future):
    print(future.result())
    loop.stop()
    
loop = asyncio.get_event_loop()
future = asyncio.Future()
asyncio.ensure_future(slow_operation(future))
future.add_done_callback(got_result)
try:
    loop.run_forever()
finally:
    loop.close()
    
"""
使用 Future.add_done_callback() 方法实现和 run_until_complete() 同样的功能
"""
```

#### 3. Task

```
class asyncio.Task(coro, *, loop=None):
    def all_tasks(loop=None):
        """classmethod: 返回 event loop 内所有 task 的集合"""
    def current_task(loop=None):
        """classmethod: 返回 event loop 内当前运行的 task """
    def cancel():
        """终止当前 task """
    def get_stack(*, limit=None):
        """"""
```