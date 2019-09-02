## 执行nginx重启命令时，报找不到nginx.pid的解决方法

当nginx被停止（nginx -s stop）或者直接杀掉了进程（kill -9 nginx的进程号）或者意外重启后，调用命令（nginx -s reload 或 nginx -s reopen)会报错：无法找到“url/local/logs/nginx.pid"

这句话中，有好几个知识点，也包含了一些错误，错误的是把reload或者reopen当作了启动的命令。来依次总结一下：

#### 1.nginx的常用命令

停止：

直接杀nginx进程。ps aux | grep nginx查看nginx的主进程号，调用kill -9 nginx的进程号来强制停止nginx。（还有”kill -quit nginx的进程号“来从容停止nginx，“kill -term nginx的进程号”来快速的停止nginx）

启动：

进入nginx安装目录/sbin/下执行

nginx或者nginx -c 特定位置的nginx.conf（一般默认是 ./nginx -t -c .url/local/nginx/nginx.conf)

重启：

nginx -s reload 平滑的重启。配置重载。



nginx工作中，包括一个master进程，多个worker进程。worker进程负责具体的http等相关工作，master进程主要是进行控制等操作。

nginx -s reload命令加载修改后的配置文件，命令下达后发生如下事件。

1.nginx的master进程检查配置文件的正确性，若是错误则返回错误信息，nginx继续采用原配置文件进行工作（因为worker未受到影响）

2.nginx启动新的worker进程，采用新的配置文件

3.nginx将新的请求分配新的worker进程

4.nginx等待以前的worker进程的全部请求都返回后，关闭相关worker进程

5.重复上面过程，直到全部旧的worker进程都被关闭掉。

所以，重启之后，master的进程号不变，worker的进程号会改变

日志分割：

nginx -s reopen 重新打开日志文件

为什么要切割日志？一般nginx安装好后有些人会打开日志记录，有些人会关闭日志记录，打开日志记录的人一般都会把架设再nginx上的所有网站日志都存在同一个文件里（比如我存在access.log日志文件里），

这样日积月累所有网站的访问记录就会把日志文件越积越大，当需要查看日志文件的时候一看就是一大串，不方便查找。现在，如果我把每天的日志文件分割开来用相应的日期标识出来这样就大大方便查找了。

我是建议打开日志记录，日志记录里面存放着很多有用的东西。比如：浏览器名称，可以方便你对网站的排版做出调整；IP地址，如果网站受到攻击，你就可以查到那个IP地址。

Linux下我们可以简单的把日志文件mv走，但是你会发现mv走后新的日志文件没有重新生成，一般Linux下用的文件句柄，文件被打开情况下你mv走文件，但是原来操作这个文件的进程还是有这个文件的inode等信息。

原进程还是读写原来的文件，因此简单的mv是无法生效的。

因此建议过程如下

1.mv原文件到新文件目录中，这个时候nginx还写这个文件（写入新位置文件中了）

2.调用nginx -s reopen用来打开日志文件，这样nginx会把新日志信息写入这个新的文件中

这样完成了日志的切割工作，同时切割过程中没有日志文件的丢失。

测试当前配置文件是否正确：nginx -t

测试指定配置文件是否正确：nginx -t 指定的配置文件路径



#### 2.var/run/nginx.pid文件

首先/var/run这个目录是干嘛用的？

此文件夹包含描述系统启动以来系统信息的数据。此文件夹下的文件必须在启动过程初期清除（删除或归零）。程序可以在/var/run下有自己的子文件夹。原先放在/etc下的进程标识（PID）文件必须放在/var/run里面。PID文件的命名管理的<program-name>.pid。所以，nginx的PID文件名为/var/run/nginx.pid。

nginx.pid存放的是nginx的master进程的进程号。

#### 3.为什么会报错

 nginx被停止时，var/run/nginx.pid被删除了。而reopen和reload命令需要通过nginx.pid获取进程号，会去找var/run/nginx.pid，如果不存在，就报错了。

#### 4.总结

reopen是在nginx启动的情况给做分割日志用的，reload也是在nginx启动的情况下做平滑重启的，他们都依赖nginx进程存在的情况下。并不是字面上启动或打开的意思。



# 真正的启动命令是：nginx或者nginx -c 指定目录的配置文件nginx.conf。查看进程存在即表明启动成功，之后再调用reload和reopen就不会报错了。









































