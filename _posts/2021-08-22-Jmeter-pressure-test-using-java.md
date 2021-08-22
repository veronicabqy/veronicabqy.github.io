

# **一、** 确定目的

本次联想词搜索功能调用了企业服务，所以需要评估接口的性能，需要压测企业字典服务。

# **二、** 准备数据

压测数据：抓取线上公司名称，进行分词生成25W条数据。

命令为：

tail -n 100000 response.log | awk '{print $7}' | awk '/^name=/' | awk '{split($0,a,"=");print a[2]}'| awk '{split($0,b,",");print b[1]}' | awk '{if($0!="null") print }'

截取效果：

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F43.tmp.jpg) 

由于进行的是联想词搜索，还需要对文本进行分词处理。

# **三、** 根据需求创建Java工程并实现JavaSamplerClient的重写

1. 将如下依赖导入到java工程中

   <img src="C:\Users\bianqinyu\AppData\Roaming\Typora\typora-user-images\image-20210822103127914.png" alt="image-20210822103127914" style="zoom: 50%;" />

但是需要注意的是<scope>的范围是provided,原因是当范围是provided的时候，maven项目在进行打包的时候并不会将该jar包打包进去。

如果使用的是complie参数的话，会将jar包打包进去，这样会导致的后果是\jmeter\lib\ext\下的所有jar包和项目中的jar包重复。会产生冲突。

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F53.tmp.jpg) 

2. 将新建的类extends AbstractJavaSamplerClient 类。

3. 需要重写"getDefaultParameters","setupTest","runTest"和"teardownTest"四个方法 

public Arguments getDefaultParameters();设置可用参数及的默认值；

public void setupTest(JavaSamplerContext arg0)：每个线程测试前执行一次，做一些初始化工作；

public SampleResult runTest(JavaSamplerContext arg0)：开始测试，从arg0参数可以获得参数值，执行多次依赖于设置方式；

public void teardownTest(JavaSamplerContext arg0)：测试结束时调用,只执行一次；

<img src="C:\Users\bianqinyu\AppData\Roaming\Typora\typora-user-images\image-20210822113417185.png" alt="image-20210822113417185" style="zoom:67%;" />

4. java项目要打包成jar包放置在jmeter的安装目录/bin中，本文是/ apache-jmeter-modify-2.13/bin。

 

# **四、** Jmeter的安装及运行

## 1. Jmeter概念

jmeter是基于java的压力测试工具，用于对软件做压力测试，最初被用来设计用于web应用测试，后来扩展至其他测试静态和动态资源，例如静态文件、java小服务程序等，Jmeter可以用于对服务器、网络或对象模拟巨大的负载，来自不同压力类别下测试他们的强度和分析整体性能

## 2. **本地jmeter环境准备及运行**

1） 解压apache-jmeter-modify-2.13.zip

2） 点击/ apache-jmeter-modify-2.13/bin/jmeter.bat打开软件

3） Java项目生成的jar包放置在/ apache-jmeter-modify-2.13/lib/ext中

4） 打开jmeter之后，添加线程组

测试计划（右键）添加Threads（Users）线程组

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F55.tmp.jpg) 

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F66.tmp.jpg) 

线程数：虚拟用户数，一个虚拟用户占用一个进程或线程，设置多少虚拟用户数在这里也就是设置多少线程数。

准备时长：设置的虚拟用户数需要多长时间全部启动，如果线程数为5，准备时长为10，那么需要10秒启动5个线程，也就是每2秒钟启动1个线程。

循环次数：每个线程发送请求的次数，如果线程数为100，循环次数为10，那么每个线程发送10次请求，总请求数为100*10=1000.如果勾选了“永远”，那么所有线程会一直发送请求。

4）添加java请求

线程组（右键）添加Sampler->Java请求

对于jmeter来说，取样器（Sampler）是与服务器进行交互的单元，一个取样器通常进行三部分工作：1.向服务器发送请求 2.记录服务器的响应数据 3.记录相应的时间信息

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F67.tmp.jpg) 

需要注意的是java请求的属性主要有两个：

a) 类名称：选定需要测试的java类，该类需要继承Jmeter的AbstractJavaSamplerClient。

b) 设置请求参数：需要将java类中的参数设置在表中，需要对其进行参数化。该参数需要与java类中的getDefaultParameters()中的参数一一对应。

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F68.tmp.jpg) ![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F79.tmp.jpg)

5） Jmeter之参数化

参数化顾名思义就是将所需要测试的数据上传至程序入口进行压力测试。

具体方式如下：

a) 添加配置元件 线程组（右键）->添加->配置元件-> CSV Data Set Config

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F7A.tmp.jpg) 

b) CSV Data Set Config配置

需要配置要参数化数据的文件目录，以及参数名称

 

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F7B.tmp.jpg) 

注：

Filename：指保存参数化数据的文件目录，也就是参数文件（远程的bin目录下）

File encoding：UTF-8，新建参数文件的编码格式

Variable Names（comma-delimited）：分隔符的定义，本测试只有一个参数，当有好几个参数的时候，需要写明几个参数的名称，每个名称中间需要有分隔符分隔，分隔符的定义需要在“Delimitet”中定义

Delimitet：表示分隔符

Allow quote data：选项为“true”的时候对全角字符的处理出现乱码。

Recycle on EOF：是否循环读入，因为CSV Data Set Config一次读入一行，如果线程数超过文本的记录行数，那么可以从头再次读入。

c) 将参数在java请求中以${} 的形式进行参数化，类似于传参。

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F7C.tmp.jpg) 

6）添加吞吐量定时器

线程组（右键）->添加->定时器-> Constant Throughput Timer

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F8C.tmp.jpg) 

Constant Throughput Timer 的主要属性介绍：

Target throughput（in samples per minute）：目标吞吐量。注意这里是每分钟发送的请求数，因此，对应测试需求中所要求的20 QPS ，这里的值应该是1200 。

Calculate Throughput based on：有5个选项，分别是：

This thread only：控制每个线程的吞吐量，选择这种模式时，总的吞吐量为设置的 target Throughput 乘以矣线程的数量。

All active threads：设置的target Throughput 将分配在每个活跃线程上，每个活跃线程在上一次运行结束后等待合理的时间后再次运行。活跃线程指同一时刻同时运行的线程。

All active threads in current thread group：设置的target Throughput将分配在当前线程组的每一个活跃线程上，当测试计划中只有一个线程组时，该选项和All active threads选项的效果完全相同。

All active threads （shared ）：与All active threads 的选项基本相同，唯一的区别是，每个活跃线程都会在所有活跃线程上一次运行结束后等待合理的时间后再次运行。

All cative threads in current thread group （shared ）：与All active threads in current thread group 基本相同，唯一的区别是，每个活跃线程都会在所有活跃线程的上一次运行结束后等待合理的时间后再次运行。

注意：

Constant Throughput Timer只有在线程组中的线程产生足够多的request 的情况下才有意义，因此，即使设置了Constant Throughput Timer的值，也可能由于线程组中的线程数量不够，或是定时器设置不合理等原因导致总体的QPS不能达到预期目标。

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F8D.tmp.jpg) 

7）添加聚合报告

线程组（右键）->添加->监听器->聚合报告

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F8E.tmp.jpg) 

8）添加察看结果树

线程组（右键）->添加->监听器->察看结果树

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9F9F.tmp.jpg) 

9）点击运行

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9FA0.tmp.jpg) 

10）查看聚合报告

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9FA1.tmp.jpg) 

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9FA2.tmp.jpg) 

11）查看察看结果树

察看结果树的可以显示每一个线程的运行结果。即java代码的返回值。

![img](file:///C:\Users\BIANQI~1\AppData\Local\Temp\ksohtml\wps9FB2.tmp.jpg) 

**最终目的是为了在本地生成.Jxm文件，以至于在服务端运行，测试集群运行效果。运行即可生成.jxm文件**

## 3. **服务端jmeter环境准备及运行**

1）将apache-jmeter-modify-2.13.zip上传至所申请服务器，

（公司的服务器中一般已经上传了该文件夹，位置为/opt/soft/apache-jmeter-modify-2.13）

**注**：1.上传大文件时为防止超时导致的传输终端，可使用 rz –bey。其中，

​		-b ，-binary用binary的方式上传或下载，默认为ascii

-e 强制escape 所有控制字符，比如Ctrl+x，DEL等，

2.使用work权限即可

2）上传本地生成的.Jxm文件及数据至服务器

注：需要修改.jxm文件的数据地址，改为在服务器中的地址

3）启动Jmeter

命令为nohup sh jmeter-server &

4）运行.jmx文件

命令为./jmeter -n -t  /opt/soft/apache-jmeter-modify-2.13/bin/entictionary.jmx

 

注意及时清理jmeter，命令为

ps -ef|grep -E 'jmeter|JMeter' |grep -v grep|awk '{print $2}'|xargs kill -9

# **五、** 服务端进行压测

通过修改服务器端.jmx文件中的不同参数压测，在服务调用平台查看具体的运行情况。

本次以压测主要是分别500、1000、2000qps进行压测。

# **六、** 压测结果

 

方式一：vmstat或者 top命令

方式二：公司的监控平台监测效果 

 

# ***\*附录：学习链接\****

https://www.cnblogs.com/yanghj010/p/9574655.html

http://www.docin.com/p-934344068.html

https://www.cnblogs.com/onmyway20xx/p/4195177.html

https://www.cnblogs.com/imyalost/p/11762359.html

 

 

 