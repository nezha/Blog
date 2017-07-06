---
title: "TBSchedule使用实战"
date: 2016-08-03 00:00:00
category: 开源项目
tags: [TBSchedule,淘宝任务调度,zookeeper]
---

# tbschedule的部署与使用

## 1. zookeeper的集群部署与使用

假设我们的设想是在三台机子上建立zookeeper集群，分别是192.168.8.28 31 32的目标主机中部署集群环境

> 全程的部署及运行都是基于centos系统，并且需要保证有java环境

### 1.1 下载zookeeper的压缩包

下面的操作在三台主机中都是同样的操作创建同样的目录结构

```bash
#1.下载源文件
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.3.6/zookeeper-3.3.6.tar.gz
#2.解压到当前的目录
tar -zxvf zookeeper-3.3.6.tar.gz
#3.重命名
mv zookeeper-3.3.6 zk
#4.创建数据文件和日志文件目录，此时与zk目录是同级的
mkdir data
mkdir log
```

### 1.2 zookeeper配置

1.2.1 *进入zk目录，在`conf`目录下复制 `zoo_sample.cfg`文件至`zoo.cfg`*

```bash
cp zoo_sample.cfg zoo.cfg
```

1.2.2 *vim进入修改配置文件至如下：*

- [ ] `ClientPort`端口：此时我们是多机集群部署，所以都是指定为2181端口,如果在1台机器上部署多个server，那么每台机器都要不同的 clientPort，比如 server1是2181,server2是2182，server3是2183 

- [ ] `dataDir`和`dataLogDir`：`dataDir`和`dataLogDir`也需要区分下，将数据文件和日志文件分开存放，同时每个server的这两变量所对应的路径都是不同的

- [ ] `server.X`和`myid`： `server.X` 这个数字就是对应，`data/myid`中的数字。在3个server的myid文件中分别写入了1，2，3那么每个server中的`zoo.cfg`都配 server.1 server.2,server.3就行了。后面连着的2个端口是zookeeper间的通信端口可以一致

> **注意:**这里的`server.1=192.168.8.28:2287:3387`其中`data/myid`的值和`server.1`中1的值都要对应到后面的IP地址的主机

```
# The number of milliseconds of each tick
tickTime=2000

# The number of ticks that the initial
# synchronization phase can take
initLimit=10

# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5

# the directory where the snapshot is stored.
dataDir=/home/zhangyi/zookeeper/data

# the port at which the clients will connect
clientPort=2181

#the location of the log file
dataLogDir=/home/zhangyi/zookeeper/log

server.1=192.168.8.28:2287:3387
server.2=192.168.8.31:2288:3388
server.3=192.168.8.32:2289:3389
```

### 1.3 zookeeper的启动与测试

1.3.1 *启动zookeeper*

进入`zookeeper`的`bin`目录下，使用如下命令

> 分别进入三台目标主机的zookeeper下的bin目录启动下面命令

```bash
./zkServer.sh start
```

1.3.2 *测试运行状态*

> 执行下面命令就会进入该IP主机`zookeeper`的工作空间，其中启动过程不能出现任何的错误信息报出，如果出错查看`myid`有没有正确写入，并且使用命令`jps`查看是否已经启动了`zookeeper`服务

```bash
./zkCli.sh -server 192.168.8.28
```

## 2. tbschedule的部署与配置

### 2.1 下载tbschedule的源码，并且在tbschedule源码的基础上创建自己的maven项目

> tbschedule下载下来源码是放在`trunk/`的目录下的

### 2.2 tbschedule控制台部署

tbSchedule就是个用servlet/JSP 写的web项目，我们可以直接把war包部署到tomcat中，然后在浏览器访问

![](http://code.taobao.org/p/tbschedule/file/2216/Zookeeper%E9%85%8D%E7%BD%AE.JPG)

```
//向注册中心注册配置
http://{server}:{port}/ScheduleConsole/schedule/config.jsp
//配置调度任务
http://{server}:{port}/ScheduleConsole/schedule/index.jsp
```

### 2.3 修改代码中的zookeeper配置

修改`src/test/resources/schedule.xml`中的配置信息指向已经启动的`zookeeper`服务器。为了避免不同应用任务类型间冲突，`rootPath`尽量全局唯一

> 由于tbschedule是基于spring框架的，所以此处的`src/test/resources/schedule.xml`是需要spring中ApplicationContext来解析xml文件的，由于maven项目结构和spring结构不太一致一般情况下spring是解析不到`.xml`文件的，指定这个`.xml`目录的文件是项目根目录下一个以`项目名＋.imp`中`<sourceFolder url="file://$MODULE_DIR$/src/main/resources" type="java-resource" />`指定的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans default-autowire="byName"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="demoTaskBean" class="com.intsig.task.DemoTaskBean"/>
    <bean id="scheduleManagerFactory" class="com.taobao.pamirs.schedule.strategy.TBScheduleManagerFactory"
        init-method="init">
        <property name="zkConfig">
           <map>
              <entry key="zkConnectString" value="192.168.8.31:2181" />
              <entry key="rootPath" value="/root/zhangyi/zookeeper/data" />
              <entry key="zkSessionTimeout" value="10000" />
              <entry key="userName" value="root" />
              <entry key="password" value="admin" />
              <entry key="isCheckParentPath" value="true" />
           </map>
       </property>  
    </bean>
</beans>
```


## 3. tbschedule的实例分析

上面把所有的基本的配置及环境的搭建都讲好了，结下来的就是最接近实战的内容了，当然了关于spring的知识还是需要有部分的积累的，主要这里用到的是spring的依赖注入的思想，然后最主要的就是用maven导入spring-framework的框架。

### 3.1 任务和调度策略的初始化,启动

首先，这一部分的配置可以说有三种方式的配置

1. 通过java代码的初始化

> 这种方式主要参考的方式是源码中`InitialDemoConfigData.java`中变成方式，其中需要注意的是创建任务调度时`setDealBeanName()`中设置的名字需要和`schedule.xml`中执行任务的bean的id一致。这种方式先执行`InitialDemoConfigData`类，然后执行`StartDemoSchedule`

2. 通过xml文件的初始化

> 这种方式在我认为可扩展性，更强一点，就是可以将策略和任务的配置通过xml的形式写入, 如果对spring有一定的理解后可以尝试一下，这部分不做详细介绍

3. 通过tbschedule控制台的初始化

> 这种方式应该说是最为简单方便的初始化飞方式，配置策略和任务时需要注意的是`任务处理Bean`的名称必须和`schedule.xml`中执行任务的bean的id一致，如果要运行的话可以直接运行`StartDemoSchedule`

![](http://code.taobao.org/p/tbschedule/file/2218/%E6%8E%A7%E5%88%B6%E5%8F%B0.JPG)

### 3.2 具体逻辑代码的实现

> 在这部分中我们最为主要的是实现`IScheduleTaskDealSingle<T>`或`IScheduleTaskDealMulti<T>`中的`selectTasks`和`execute`。

3.2.1 **情景假设**

假设我们现在向姓名首字母a-z的所有用户发送一封邮件，我们需要以最快的方式发出去，这个时候就可以考虑使用分布式的方式用多台服务器不重复的同时发送目标邮件。

1. 创建调度策略，其中“最大线程组数量”设置为13，表示在机器上的通过4个线程组并行执行数据同步任务。

2. 创建调度任务，需要注意下面设置

**任务名称：**对应调度策略中的任务名称，标识任务和策略的关联关系；

**任务处理的SpringBean：**对应Demo TaskDeal服务Spring容器中的任务对象ID；

**每次获取数据量：**对应于bean任务类selectTasks方法参数 eachFetchDataNum

**执行开始时间：**“0 * * * * ?” 表示每分钟的0秒开始，表达式同Quartz设置的Crontab格式，有工具可以生成，详细解释参照[这里](http://dogstar.iteye.com/blog/116130)

**任务项：**在上面的a-z中有26个字母，我们可以让一个县城组处理两个首字母的人名，向这些人发邮件，我们就可以将任务划分为a-z的任务片。26个任务碎片被分配到13个线程组，那么每个线程组对应2个任务碎片，运行时任务项参数又被传递到bean任务类`selectTasks`方法的`List<TaskItemDefine> queryCondition`参数，例如第1个线程组调用`selectTasks`方法是`queryCondition`参数条件为a,b ，第2个线程组执行参数条件为c,d，bean任务类中根据参数生成对应的字母条件去数据库中找到响应的人名的邮箱信息，并将结果提交到`execute`方法执行，从而实现并行计算。

3.2.2 **demo演示**

```java
package Task;

import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

import com.taobao.pamirs.schedule.IScheduleTaskDealSingle;
import com.taobao.pamirs.schedule.TaskItemDefine;

import DBHelper.*;

public class DataSyncABean implements IScheduleTaskDealSingle<OrderInfo> {

    public List<CustomInfo> selectTasks(String taskParameter, String ownSign,
            int taskItemNum, List<TaskItemDefine> queryCondition,
            int eachFetchDataNum) throws Exception {

        List<OrderInfo> result = new ArrayList<OrderInfo>();
        if (queryCondition.size() == 0) {
            return result;
        }

        StringBuffer condition = new StringBuffer();
        for (int i = 0; i < queryCondition.size(); i++) {
            if (i > 0) {
                condition.append(",");
            }
            //获取到被分配到执行主机上的名字首字母
            condition.append(queryCondition.get(i).getTaskItemId());
        }

        String sql = "select * from custom " + "where "
                + " firstchar in (" + condition + ") " + "limit "
                + eachFetchDataNum;

        System.out.println("开始执行SQL：" + sql);

        ResultSet rs = MySQLHelper.executeQuery(sql);
        while (rs.next()) {
            CustomInfo custom = new CustomInfo();
            custom.name = rs.getString("name");
            custom.email = rs.getString("email");
            result.add(custom);

            if (rs.isLast()) {
                break;
            }
        }
        MySQLHelper.free(rs, rs.getStatement(), rs.getStatement()
                .getConnection());

        return result;
    }

    public Comparator<OrderInfo> getComparator() {

        return null;
    }

    public boolean execute(CustomInfo task, String ownSign) throws Exception {
        //TODO 执行发邮件的操作，
        //这里的task是一个单个的数据对象，即只对应一个人的信息
        return true;
    }
}
```


## 参考文献

1.http://code.taobao.org/p/tbschedule/wiki/tbschedule-quick-start/

2.http://www.cnblogs.com/sunddenly/p/4018459.html

3.http://blog.csdn.net/neosmith/article/details/46535853

4.http://geek.csdn.net/news/detail/65738?utm_source=tuicool&utm_medium=referral

5.http://blog.csdn.net/taosir_zhang/article/details/50728362

6.http://www.cnblogs.com/lengfo/p/4146797.html
