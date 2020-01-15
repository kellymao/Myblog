---
layout: post
title: "prometheus 安装"
date: 2019-11-14 
description: "prometheus 安装"
tag: prometheus 监控
---   
  


## 安装prometheus

<br> 

### 下载软件包

对于prometheus，假设我们需要监控MySQL，那么我们需要下载至少3个组件，如下：

prometheus程序包

node_exporter：监控主机磁盘、内存、CPU等硬件性能指标的采集程序包

mysql_exporter： 监控mysql各种性能指标的采集程序包

下载链接（该页面始终只有一个最新版本）：https://prometheus.io/download/


安装步骤参考：[https://mp.weixin.qq.com/s?__biz=MzU0MTczNzA1OA==&mid=2247484205&idx=1&sn=7f2dc686036e22efebed8934e624d74c&chksm=fb242a20cc53a336f5c3c901d335ffb12cd1b6d3ae17850cdeba0dad052045ae2791892df56f&mpshare=1&scene=1&srcid=1115bHEC5YovvkwWAMkQkKPr&sharer_sharetime=1573810398781&sharer_shareid=9b64bf3a2476b2c5d499db84b8628fde&rd2werd=1#wechat_redirect](https://mp.weixin.qq.com/s?__biz=MzU0MTczNzA1OA==&mid=2247484205&idx=1&sn=7f2dc686036e22efebed8934e624d74c&chksm=fb242a20cc53a336f5c3c901d335ffb12cd1b6d3ae17850cdeba0dad052045ae2791892df56f&mpshare=1&scene=1&srcid=1115bHEC5YovvkwWAMkQkKPr&sharer_sharetime=1573810398781&sharer_shareid=9b64bf3a2476b2c5d499db84b8628fde&rd2werd=1#wechat_redirect)





 

## 启动prometheus

<br> 

	/data/prometheus/prometheus  --config.file="/data/conf/prometheus.yml" \
		--web.listen-address=":9090" \
		--web.external-url="http://10.18.8.178:9090/" \
		--web.enable-admin-api \
		--log.level="info" \
		--storage.tsdb.path="/data/prometheus2.0.0.data.metrics" \
		--storage.tsdb.retention="30d" 

	
## 安装 granfa 

<br> 	

### 下载：

grafana程序包：https://grafana.com/grafana/download 

grafana-dashboards包：https://github.com/percona/grafana-dashboards/releases 
	
### 配置数据源

参考上文：




### 导入dashboard json文件 

参考上文： 

### plugin画图插件的导入


（1） 手动导入

在 grafana setting界面里面查看plugins 的位置。 解压dashboards将plugins的pane包拷贝到该位置



	
	
	# 本例子中plugins目录
	
	root@pts/8 # ll /data/opt/grafana/plugins
	总用量 0
	drwxrwxr-x 5 root root 179 11月 13 12:58 digiapulssi-grafana-breadcrumb-panel
	drwxrwxr-x 6 root root 295 3月  22 2019 grafana-grafana-polystat-panel
	drwxrwxr-x 6 root root 239 9月   5 22:31 grafana-piechart-panel
	drwxrwxr-x 5 root root 196 7月   9 14:27 jdbranham-grafana-diagram
	drwxrwxr-x 5 root root 319 12月 20 2018 NatelEnergy-grafana-discrete-panel
	drwxrwxr-x 5 root root 167 12月 17 2017 petrslavotinek-grafana-carpetplot
	drwxrwxr-x 5 root root 271 8月  12 18:32 Vertamedia-clickhouse-grafana
	drwxrwxr-x 7 root root 310 7月  16 23:48 yesoreyeram-yesoreyeram-boomtable-panel
 
（2） 命令行安装导入 ,默认安装在 /var/lib/grafana/plugins

   /data/opt/grafana/bin/grafana-cli  plugins list-remote  
   /data/opt/grafana/bin/grafana-cli plugins install digiapulssi-breadcrumb-panel 
 
参考： https://grafana.com/docs/plugins/installation/

### 客户端安装 mysql_exporter 


	#cat /etc/sysconfig/mysqld_exporter_3306 
	# See https://github.com/prometheus/mysqld_exporter
	ARGS="--config.my-cnf /etc/.mysqld_exporter_3306.cnf \
	--collect.binlog_size \
	--collect.info_schema.processlist \
	--web.listen-address=0.0.0.0:9104"

	#cat  /etc/.mysqld_exporter_3306.cnf
	[client]
	user=zabbix_monitor
	password=zabbix_monitor
	host=127.0.0.1
	port=3306

	#启动：
	/usr/local/bin/mysqld_exporter --config.my-cnf /etc/.mysqld_exporter_3306.cnf \
	--collect.binlog_size \
	--collect.info_schema.processlist \
	--web.listen-address=0.0.0.0:9104"

参考： 

https://www.cnblogs.com/mafeng/p/7680451.html 

http://c.biancheng.net/view/4476.html


