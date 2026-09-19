
- [tomcat](#tomcat)
- [目录结构](#目录结构)
  - [虚拟目录](#虚拟目录)

## tomcat

## 目录结构

目录结构：
```
bin/               启动和关闭tomcat的bat文件；
conf/              配置文件；
lib/               tomcat运行所需jar包；
logs/              存放日志；
webapps/           存放web应用；
work/              工作目录，存放jsp被访问后生成的server、class文件；
server.xml         配置和server相关的信息，如启动端口、host等；
web.xml            配置web应用；
tomcat-user.xml    配置tomcat用户密码权限等；
```
web应用目录：

### 虚拟目录

把web应用放到webapps目录，tomcat就会自动管理。如果希望tomcat管理其它目录下的web应用，可以建立虚拟目录。

在conf目录下server.xml文件的Host节点添加如下内容：
```xml
<Host>
  <!--http://localhost:8080/other_web/index.html-->
  <Context path="/other_web" docBase="D:/web_folder" />
</Host>
```
添加之后重启才能生效。
