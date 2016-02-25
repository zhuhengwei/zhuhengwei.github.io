---
title: Shell脚本启动Storm集群
date: 2015-12-06 20:35:45
categories: [大数据,系统运维]
tags: [shell,大数据,storm,集群]
---

最近因为学习需要，搭建了storm集群，装了比较旧的版本storm-0.8.1,也安装了ZMQ，中间安装过程之曲折不属本文范围，以后再整理。本文先把storm集群启动的脚本放出,其中包含了启动zookeeper。其他集群，spark，hadoop启动、停止脚本可参考本脚本。代码不完善之处依照具体业务修改。
##### 启动之前的准备工作
1）机器配置,ip-机器名 在hosts中配置好
2）配置各节点storm 的STORM_HOME和ZOOKEEPER_HOME
3）各节点SSH免密码访问
Done

##### 脚本代码
```bash
	#!/bin/bash
	##准备工作：
	#机器配置,ip-机器名 在hosts中配置好
	#配置各节点storm 的STORM_HOME和ZOOKEEPER_HOME
	#SSH免密码访问
	#
	#脚本在主节点下运行
	#
	
	#zk的节点，3台，不在主节点启动
	zookeeperServers='Server58 Server60 Server62'
	
	#工作节点，不含主节点
	supervisorServers='Server58 Server60 Server62'
	
	#nimbus和ui节点
	nimbusServer='Server76'
	
	#启动各个zookeeper server
	for i in $zookeeperServers;
	do
	ssh -T $i <<EOF
		source /etc/profile
		cd \$ZOOKEEPER_HOME
		./bin/zkServer.sh start
	EOF
	echo 启动zookeeper$i...[ done ]
	sleep 1
	done
	
	#启动主节点的nimbus/ui
	cd $STORM_HOME
	python bin/storm nimbus >/dev/null 2>&1 &
	python bin/storm supervisor >/dev/null 2>&1 &
	python bin/storm ui >/dev/null 2>&1 &
	
	sleep 1
	
	echo 启动主节点$nimbusServer nimbus...[ done ]
	echo 启动主节点$nimbusServer supervisor...[ done ]
	echo 启动主节点$nimbusServer UI...[ done ]
	
	#启动其余所有的supervisor
	for j in $supervisorServers;
	do
	ssh -T $j <<EOF
		source /etc/profile
		cd \$STORM_HOME
		python bin/storm supervisor >/dev/null 2>&1 &
	EOF
	echo 启动从节点$j supervisor...[ done ]
	sleep 1
	done
	
	#检查nimbus是否启动
	count=`jps |grep nimbus |wc -l`
	if [ $count = 0 ]
	then
		cd $STORM_HOME
		python bin/storm nimbus >/dev/null 2>&1 &
	fi
	
	#检查UI是否启动
	count=`jps |grep core |wc -l`
	if [ $count = 0 ]
	then
		cd $STORM_HOME
		python bin/storm ui >/dev/null 2>&1 &
	fi
	
	#检查其余各个supervisor是否启动
	#如果刚才没有正常启动，则启动
	for k in $supervisorServers;
	do
	ssh -T $k <<EOF
	source /etc/profile
	count3=`jps |grep supervisor |wc -l`
	if [ \$count3 = 0 ]
	then
		cd \$STORM_HOME 
		python bin/storm supervisor >/dev/null 2>&1 &
	fi
	EOF
	done
	
	
	#检查主节点supervisor是否启动
	count2=`jps |grep supervisor |wc -l`
	if [ $count2 = 0 ]
	then
		cd $STORM_HOME
		python bin/storm supervisor >/dev/null 2>&1 &
	fi
	
	sleep 2
	echo storm集群启动成功....[ done ] 
```
附CSDN代码块链接。[来自 CODE的代码片。](https://code.csdn.net/snippets/1437227/master/download)