---
title: "完美解决：IntelliJ IDEA通过maven构建的项目默认是Java5,改成Java8编译会出错"
date: 2017-06-17 00:00:00
category: 疑难解惑
tags: [IntelliJ IDEA, maven]
---

## 问题是什么

当我使用maven构建项目的时候，系统给项目分配的api自动会设置成Java5.0 的版本，这个版本下Java8的好多功能都不支持的，甚至连`@override`都不支持。

![](http://wx3.sinaimg.cn/mw1024/9d2c4511gy1fgo8nj4r8aj21d60jgaea.jpg)

## 那我改成Java8不就行了

于是我在上图中改成了Java8,于是乎就出现了下面的问题....连编译都不能通过了。

![](http://wx2.sinaimg.cn/mw1024/9d2c4511gy1fgo8nhmo86j216m0cs76t.jpg)

## 完美解决在这里

最后参考了<https://stackoverflow.com/>上的方法，出入已经找不到了，如有同学发现我来加上引用。

主要在`pom.xml`中置顶`build`的版本
如下`xml`代码段

```xml
<build>
  <plugins>
	<plugin>
	  <groupId>org.apache.maven.plugins</groupId>
	  <artifactId>maven-compiler-plugin</artifactId>
	  <version>3.5.1</version>
	  <configuration>
		<source>1.8</source>
		<target>1.8</target>
	  </configuration>
	</plugin>
  </plugins>
</build>
```

即如下图：
![](http://wx2.sinaimg.cn/mw1024/9d2c4511gy1fgo8niyw92j20w20qq78d.jpg)
