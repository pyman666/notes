# **contextmanager**

`contextmanager`是一个装饰器**，**被装饰的函数在被调用时，必须返回一个生成器。

> 生成器`yield`的值就是`with`语句`as`后面的变量，`yield`前的代码相当于`enter`函数，`yield`后的代码相当于`exit`函数（存疑？）
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
# closing

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
from contextlib import closing

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

class A:
    def foo(self):
        print("A")

    def close(self):
        print("close")

with closing(A()) as a:
    a.foo()
```
