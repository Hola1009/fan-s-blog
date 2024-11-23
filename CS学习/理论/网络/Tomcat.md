# 认识Tomcat
Tomcat??? 汤姆猫!?爷青回!!!!😽
nonono~~ 不是这家伙
![[Pasted image 20231209215328.png|400]]
Tomcat其实是一个HTTP服务器
前面我们已经学习了 HTTP 协议, 知道了 HTTP 协议就是 HTTP 客户端和 HTTP 服务器之间的交互数据
的格式.
要使用HTTP进行通信, 就要实现HTTP服务端的程序
我们当然可以自己取实现, 但使用大佬写好的岂不更香🥰
咱么只需要用大佬写好的代码去二次加工即可大大提高了开发效率👍

# 下载安装
咱就不介绍了, 看看这位大佬的
[Tomcat安装及配置]([Tomcat安装及配置-CSDN博客](https://blog.csdn.net/what_how_why2020/article/details/114100385))

# 目录结构
对Tomcat解压后, 点开文件夹, 映入眼帘的是这样的目录结构
```
apache-tomcat-8.5.47\
bin\ 存放各种启动、停止脚本的。*.sh 是以后在 linux 上用的，*.bat 是在 windows
上用的
startup.bat 启动服务，双击即可使用
conf\ 相关的配置文件，目前我们不用关心
lib\ 运行 tomcat 需要的类库，我们不关心
logs\ 运行时的日志文件，我们有时需要查看日志，来发现定位一些问题
temp\ 临时文件夹，不关心
webapps\ 存放我们要运行的 web application 的文件夹，对于我们最常用的一个文件夹
work\ Tomcat 内部进行预编译的文件夹，我们不关心
下面都是一些文档，有兴趣的同学可以自行阅读
BUIDING.txt
CONTRIBUTING.md
LICENSE
NOTICE
README.md
RELEASE-NOTES
RUNNING.txt
```
其中我们最需要关注的目录就是 webapps 目录.👽 web applications , 就是说 web 应用存在这
这里保存的是咱们写的程序

# 启动服务器
在bin目录中找到`startup.bat `双击即可启动服务
在浏览器中输入 127.0.0.1:8080 即可看到 Tomcat 的默认欢迎页面.