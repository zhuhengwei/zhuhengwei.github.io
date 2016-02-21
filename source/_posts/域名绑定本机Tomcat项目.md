---
title: 域名绑定本机Tomcat项目
date: 2014-12-02 20:14:45
categories: [服务器运维]
tags: [Tomcat,域名,IP]
---
##### 说明
+ 本文的前提是已经注册了域名，并且在域名管理里添加了A记录，且指向本机IP.
+ 服务器为tomcat,在自己电脑上.
+ 本文没有涉及动态域名绑定IP的问题.

##### 步骤
1. 启动服务器后，浏览器输入localhost:8080/xxx就可以访问到xxx项目，或许你还没有设置首页，那么需要输入localhost:8080/xxx/yyy.jsp。接下来要去掉8080，因为域名管理不能设置端口，而http默认都是80，因此需要修改tomcat的conf文件夹下的server.xml。我们打开这个文件，找到8080，把8080改成80即可。再次输入localhost/xxx就可以访问到本地项目，这一步应该都没有问题。
2. 绑定域名，修改server.xml找到，下面一行。
```xml
	<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
```
	**修改为**
```xml
	<Host name="www.zzz.zzz" debug="0" appBase="webapps" unpackWARs="true" autoDeploy="true" xmlValidation="false" xmlNamespaceAware="false">
```
	重启后浏览器输入www.zzz.zzz/xxx就可以访问项目，但是如何做到只输入www.zzz.zzz就访问项目呢？

3. 再次找到server.xml。只需要在</Host>前面加上如下代码(注意这个位置)一般都在xml文件最下面。
```xml
	<Context docBase="/xxx" path="" reloadable="true"/> // 这里的"/xxx"要有斜杠.
```
此时直接访问www.zzz.zzz就会直接访问到刚才设置的项目。