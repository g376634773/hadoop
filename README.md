# hadoop HA 部署

参考文档<https://www.cnblogs.com/linjiqin/p/12444927.html>

部署版本：hadoop-3.3.0

zk: 3.4.8

1、报错
```
部署完成后，监听hadoop日志可能会发现如下报错：
2021-04-16 17:37:09,477 WARN org.apache.hadoop.hdfs.server.namenode.ha.EditLogTailer: Unable to trigger a roll of the active NN
java.util.concurrent.ExecutionException: java.io.IOException: Cannot find any valid remote NN to service request!
	at java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.util.concurrent.FutureTask.get(FutureTask.java:206)
	at org.apache.hadoop.hdfs.server.namenode.ha.EditLogTailer.triggerActiveLogRoll(EditLogTailer.java:425)
	at org.apache.hadoop.hdfs.server.namenode.ha.EditLogTailer$EditLogTailerThread.doWork(EditLogTailer.java:484)
	at org.apache.hadoop.hdfs.server.namenode.ha.EditLogTailer$EditLogTailerThread.access$400(EditLogTailer.java:450)
	at org.apache.hadoop.hdfs.server.namenode.ha.EditLogTailer$EditLogTailerThread$1.run(EditLogTailer.java:467)
	at org.apache.hadoop.security.SecurityUtil.doAsLoginUserOrFatal(SecurityUtil.java:485)
	at org.apache.hadoop.hdfs.server.namenode.ha.EditLogTailer$EditLogTailerThread.run(EditLogTailer.java:463)
Caused by: java.io.IOException: Cannot find any valid remote NN to service request!
	at org.apache.hadoop.hdfs.server.namenode.ha.EditLogTailer$MultipleNameNodeProxy.call(EditLogTailer.java:582)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)

解决办法：
hdfs写的那台机器是待机状态的，所以不支持，要在active 机器中写才行

强制切换active，执行bin/hdfs haadmin -transitionToActive --forcemanual nn1
[root@master hadoop-3.3.0]# bin/hdfs haadmin -transitionToActive --forcemanual nn1
You have specified the --forcemanual flag. This flag is dangerous, as it can induce a split-brain scenario that WILL CORRUPT your HDFS namespace, possibly irrecoverably.

It is recommended not to use this flag, but instead to shut down the cluster and disable automatic failover if you prefer to manually manage your HA state.

You may abort safely by answering 'n' or hitting ^C now.

Are you sure you want to continue? (Y or N) y
```
此时由standby变为active了
![image](https://user-images.githubusercontent.com/51428270/115007184-4efb0700-9edc-11eb-81fb-f7e777ed8d97.png)

也可以不使用强制切换，一般情况是谁先启动DFSZKFailoverController，谁就是active

./bin/hadoop-daemons.sh start zkfc
