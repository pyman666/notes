# contextlib

[contextmanager](#contextmanager)
[closing](#closing)

标准库，主要用来简化上下文管理器的编写和使用，也就是 `with` 语句相关的资源管理。

##  contextmanager

`contextmanager`是一个装饰器，不用专门写一个带 `__enter__`、`__exit__` 方法的类，就能创建上下文管理器。

1. 被装饰的函数必须是生成器函数：通常通过 `yield` 产生生成器，调用函数后得到生成器对象。
2. `yield` 前的代码在进入 `with` 时执行；`yield` 后的代码在退出 `with` 时执行。异常处理也有对应机制。
3. 如果 `with` 内部发生异常，异常会在 `yield` 处重新抛入生成器，因此可以用 `try/except/finally` 处理异常或确保清理逻辑执行。

```python
from contextlib import contextmanager

@contextmanager
def foo():
    try:
        yield
    except:
        print("error")

with foo() as f:
    1 / 0

# ----------------

try:
    1 / 0
except:
    print("error")
```

##  closing

将有`close`方法的对象快速变成上下文管理器，并把对象变为`as`后的变量。其等价于

```python
@contextmanager
def closing(thing):
    try:
        yield thing
    finally:
        thing.close()
```

具体用法

```python
class A:
    def foo(self):
        print("A")

    def close(self):
        print("close")

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

with A() as a:
    a.foo()

# --------------------------------
from contextlib import closing

class A:
    def foo(self):
        print("A")

    def close(self):
        print("close")

with closing(A()) as a:
    a.foo()
```
