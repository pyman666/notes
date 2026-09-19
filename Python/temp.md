
# 部署Flask 项目

生产模式部署：

- 部署 Flask 应用时，通常都是使用一种 WSG1 应用服务器搭配 Nginx 作为反向代理；

- 常用的 WSG1 服务器：gunicorn、uwsgi；

- 安装 Nginx: `yum install nginx`（红帽系列Linux发行版上的指令）；

- 安装 Gunicorn: `pip install gunicorn`；

- 启动 Gunicorn: `gunicorn -b 127.0.0.1:8080-D my_app:app`；

- 编辑 Nginx 配置文件 `vim -ziginx/nginx.conf`；

- 启动 Nginx : `/usr/sbin/nginx`

# 码农高天

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
    return self.x == other.x  && self.y == other.y
```
