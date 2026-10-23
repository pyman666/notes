
##  码农高天

list内存不同

```python
sys.getsizeof([0] * 3);  # 80
sys.getsizeof([0, 0, 0]);  # 120
sys.getsizeof([0 for _ in range(3)]);  # 88
```

为什么None一定用is比较

```python
class A:
    def __bool__(self):
        return True

    def __eq__(self, other):
        return True

a = A()
if a:
    print("a")

if a == None:
    print("aa")
```

自定义类型的对象作字典的键

```python
class Position:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __hash__(self):
        return hash((self.x, self,y))

    def __eq__(self, other):
        return self.x == other.x & self.y == other.y
```
