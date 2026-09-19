
- [Flask](#flask)
- [路由](#路由)
- [配置](#配置)
- [请求扩展](#请求扩展)
- [Blueprint](#blueprint)
- [request](#request)
- [session](#session)
- [g](#g)
- [Response](#response)
  - [signal](#signal)
  - [flash](#flash)
  - [render\_template](#render_template)
  - [redirect](#redirect)
  - [jsonify](#jsonify)
  - [url\_for](#url_for)
- [flask-session](#flask-session)
- [wtforms](#wtforms)
- [变量](#变量)
- [语句](#语句)
- [过滤器](#过滤器)
- [继承](#继承)

# Flask

## 路由

`@app.route('/index/<int:arg>', methods=['GET'], endpoint='ind')`

本质：

`self.add_url_rule`

参数：

`<int:arg>`：转换器，`DEFAULT_CONVERTERS`

`methods`：请求方式

`endpoint`：别名，默认为被装饰函数的函数名

`defaults`：

`strict_slashes`：是否对`/`严格

`redirect_to`：重定向

`subdomain`：允许访问的域名

## 配置

- 默认值：

`app.config.items()`

- 直接配置

`app.config["DEBUG"] = True`

`app.debug = True`

- 文件配置

`app.config.from_pyfile("settings.py")`

*settings.py*：`DEBUG=True...`

- 类配置

`app.config.from_object(setting.Config)`
```python
class Config:
    DEBUG = False
    TESTING = False

class TestConfig(Config):
    DEBUG = True
```
- 其他配置

`app.config.from_json("config.json")`：json文件

`app.config.from_mapping({"DEBUG": True})`：字典

`app.config.from_envvar("环境变量名称")`：环境变量

## 请求扩展

类似django中间件

- `@app.before_request`

请求来了就会触发，类似于django的`process_request`。如有多个，执行顺序**从上往下**。可应用于登陆认证重定向

- `@app.after_request`

请求走了就会触发，类似django的`process_response`。如有多个，执行顺序**从下往上**。路由函数第一个参数为response，且返回response

- `@app.before_first_request`

项目启动后第一次请求服务器时触发，跟浏览器、用户无关，可用于搞一些初始化

- `@app.teardown_request`

每一次请求之后执行，即使遇到异常也会执行。可用于记录日志

- `@app.errorhandler(500)`

用于捕获异常

- `@app.template_global`

模板全局标签

- `@app.template_filter`

模板全局过滤器（多参数时调用方式较蛋疼）

- `@app.context_processor`

todo

## Blueprint

没有蓝图之前分文件app要在文件之间导来导去，有了蓝图之后可以分文件分app，请求扩展啥的也都有，只是在当前蓝图对象管理下的有效。

`user = Blueprint(“user”, __name__)`

参数：

`name`：blueprint的名称，将添加到每个`endpoint`之前

`import_name`：blueprint包名称

`url_prefix`：附加到所有蓝图url的路径，以使其与app的其余路由区分开

`url_defaults`：蓝图路由默认接收的默认值的字典

`root_path`：默认情况下，会自动基于import\_name计算蓝图根目录

`cli_group`：控制flask命令下的组名（蓝图有一个cli组来注册嵌套的CLI命令）

其他与Flask参数基本相同。
```python
"""app.py"""

from flask import Flask
from user import user_blue

app = Flask(__name__)
app.register_blueprint(user_blue)

if __name__ == '__main__':
    app.run(port=5678)

"""user.py"""

from flask import Blueprint

user_blue = Blueprint('user', __name__, url_prefix='/user')

@user_blue.route('/login')
def login():
    return 'login'

@user_blue.route('/modify_password')
def modify_password():
    return 'modify_password'
```
---

# 接口

## request

属性：

`request.url`：带域名带参数请求路径

`request.base_url`：带域名请求路径

`request.path`：不带域名，请求路径

`request.full_path`：不带域名，带参数的请求路径

`request.url_root`：域名，https://127.0.0.1:5000/

`request. host_root`：域名，https://127.0.0.1:5000/

`request.host`：127.0.0.1:5000

`request.method`：提交的方法

`request.form`：post请求提交的数据

`request.args`：get请求提交的数据，封装过的字典

`request.query_string`：get请求提交的数据，query字符串

`request.values`：get和post提交的数据总和

`request.cookies`：客户端所带的cookie

`request.headers`：请求头

`request.files`：上传的文件

## session

除了请求对象还有session对象（app.session\_interface对象），他允许在不同请求间存储用户的信息，他是在cookie的基础上实现的，并对cookie进行密钥签名。

flask内置session实现：响应时，通过`SECRET_KEY`（使用session必须配置）生成密钥写入cookie返回给浏览器（save\_session）。下次发送请求携带cookie过来，通过cookie反解赋值给session（open\_session）。

而django实现：生成随机字符串、存入数据库、写入cookie返回给浏览器。

cookie：存放在客户端对键值对

session：存放在客户端对键值对

token：存放在客户端，通过算法来校验

设置：session["username"] = "xxx"

删除：session.pop("username", None)

## g

全局变量，在当次请求中一直有效，在当次请求中放值取值。为了不让在请求中往request中放东西。

session跨request，只要session还未失效，不同request请求都会获得同一个session。g对象不需要考虑过期时间，每请求一次，g对象就改变一次或者重新赋值一次。

## Response

flask.wrapper.Response

make\_response

res = make\_response("Hello World")

res.set\_cookie("xxx", "lqz")

res.delete\_cookie("key")

res.headers["xxx"] = "xxx"

# 其他

### signal

flask中的信号基于blinker，让开发者可以在请求过程中定制一些用户行为。当程序执行到某个位置，自动触发某些操作，异步执行。
```python
from flask import Flask, signals

app = Flask(__name__)

def call(*args, ""kwargs):
    print("触发信号")

signals.request_started.connect(call)
```
### flash

闪现，闪现系统使得在一个请求结束的时候记录一个信息供下次获取。flask闪现是基于flask内置的session的，利用浏览器的session缓存闪现信息，所以必须设置secret\_key。

在模板中在且仅仅在下一个请求中访问这个数据；在视图中获取闪现信息不必非得是两次连续的请求，只要保证是第一次取相应的闪现信息就可以取得到。

应用：如在a页面注册成功，跳转到b页面，在b页面显示a页面注册成功信息

设置：flash("xxx", category="error")，如果没有第二个参数，默认分类为message

取值：get\_flashed\_messages(category\_filter=["error"])

模版中取值：{% with messages = get\_flashed\_messages(with\_categories=true) %} 。。。{% endwith %}

### render\_template

渲染html模板，后面可加参数传入模板中

### redirect

重定向

### jsonify

### url\_for

根据endpoint参数提供的路由的别名反向解析url

---

# 扩展

## flask-session

替换flask内置session，将默认保存的签名cookie中的值保存到redis、数据库、file等

配置：

- `SESSION_TYPE`

设置session保存位置，null/redis/memcached/filesystem/sqlalchemy/mongodb；

- `SESSION_REDIS`

如果SESSION\_TYPE = ‘redis’，那么设置该参数连接哪个redis，其是一个连接对象；如果不设置的话，默认连接127.0.0.1:6379/0。

- `PERMANENT_SESSION_LIFETIME`

设置session的有效期，即cookie的失效时间，单位是s，默认会话是永久性的；

- `SESSION_PERMANENT`

是否使用永久会话，默认True，但是如果设置了`PERMANENT_SESSION_LIFETIME`，则这个失效；

- `SESSION_USE_SIGNER`

是否为cookie设置签名来保护数据不被更改，默认False；如果设置True，那么必须设置flask的secret\_key参数；

- `SESSION_KEY_PREFIX`

在所有的会话键之前添加前缀，对于不同的应用程序可以使用不同的前缀；默认“session:”，即保存在redis中的键的名称前都是以“session:”开头；

- `SESSION_COOKIE_NAME`

设置返回给客户端的cookie的名称，默认是“session”；

- `SESSION_COOKIE_DOMAIN`

设置会话的域，默认是当前的服务器；

- `SESSION_COOKIE_PATH`

设置会话的路径，即哪些路由下应该设置cookie，如果不设置，默认为`/`，所有的路由都会设置cookie；

- `SESSION_COOKIE_HTTPONLY`

cookie应该和httponly标志一起设置，默认为True，一般采用默认；

- `SESSION_COOKIE_SECURE`

cookie是否和安全标志一起设置，默认为False，一般采用默认。
```python
from flask import Flask
import redis
from flask_session import Session, RedisSessionInterface

app = Flask(__name__)
conn = redis.StrictRedis(host="127.0.0.1", port=6390, db=4)

# 法一
app.session_interface = RedisSessionInterface(conn, key_prefix="hz")

# 法二
app.config["SESSION_TYPE"] = "redis"
app.config["SESSION_REDIS"] = conn
app.config["SESSION_KEY_PREFIX"] = "hn"
Session(app)
```
## wtforms

功能：校验数据、渲染标签

# 模板语言

flask默认使用jinjia2模版引擎，支持函数直接调用

## 变量

- 值

`{{ v }}`

- 列表

`{{l.1}}`、`{{l[4]}}`

- 字典

`{{d.key}}`、`{{d['key']}}`、`{{d.get('key')}}`

## 语句

- 注释

`{# 注释 #}`

- if

`{% if True %}`

`{% elif True %}`

`{% else %}`

`{% endif %}`

- for

`{% for i in lst %}`

`{% endfor %}`

## 过滤器

- TODO

`{{ v|length }}`

- 处理xss攻击

模板：`{{ html|safe }}`

后端：`Markup()`

## 继承
