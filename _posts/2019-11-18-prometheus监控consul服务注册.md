---
layout: post
title: "consul 集群的搭建"
date: 2019-11-14 
description: "consul 集群的搭建"
tag: prometheus 监控
---   
  


## 安装Consul

<br> 

### 下载软件包

下载地址 https://www.consul.io/downloads.html

找到对应版本的安装包 

	wget https://releases.hashicorp.com/consul/1.6.1/consul_1.6.1_linux_amd64.zip 

	unzip consul_1.6.1_linux_amd64.zip 
	root@pts/4 # ll consul
	-rwxr-xr-x 1 root root 107695155 9月  13 03:46 consul


### 安装验证

	root@pts/4 # /opt/consul 
	Usage: consul [--version] [--help] <command> [<args>]

	Available commands are:
		acl            Interact with Consul's ACLs
		agent          Runs a Consul agent
		catalog        Interact with the catalog
		config         Interact with Consul's Centralized Configurations
		connect        Interact with Consul Connect
		debug          Records a debugging archive for operators 

## 启动Consul 

<br> 

### 命令行启动

<br> 

三节点consul server 启动：


	consul agent -server -bootstrap-expect 3 -data-dir /tmp/consul -node=s1 -bind=10.201.102.198 -ui-dir ./consul_ui/ -rejoin -config-dir=/etc/consul.d/ -client 0.0.0.0

	
对应的参数的解释：

[https://www.cnblogs.com/niejunlei/p/5982911.html](https://www.cnblogs.com/niejunlei/p/5982911.html)	


### 配置文件启动 

<br> 

编辑配置文件：

	 cat server.json 
	{
		"bind_addr":"10.18.8.53",
		"client_addr":"0.0.0.0",
		"datacenter":"dc1",
		"data_dir":"/opt/consul-data",
		"encrypt":"EXz7LFr8hpQ4id8EDYuFoQ==",                   #/opt/consul keygen 生成
		"log_level":"INFO",
		"enable_syslog":true,
		"enable_debug":true,
		"node_name":"ConsulServer1",
		"server":true,
		"ui":true,
		"bootstrap_expect":3,
		"leave_on_terminate":false,
		"skip_leave_on_interrupt":true,
		"rejoin_after_leave":true,
		 "recursors" : [ "10.168.0.12","10.168.2.127","10.106.196.118"],
	  "ports" : {
		"dns" : 53
	  },
		"retry_join":[
			"10.18.8.53:8301",
			"10.18.8.45:8301",
			"10.18.8.49:8301"
		]
	}

	
启动：

	consul agent -server -config-dir=/etc/consul.d/server/	 > /data/consul/consul.log & 

例子：

[https://www.cnblogs.com/EikiXu/p/10684769.html](https://www.cnblogs.com/EikiXu/p/10684769.html)

### 基本管理 

<br> 

	consul members 
	consul operator raft list-peers
	/opt/consul operator raft remove-peer -id=b1eb93fd-a576-2ca2-314d-0771af2d5427

kv 操作：


	root@pts/4 # /opt/consul kv put name zhangsan 
	Success! Data written to: name
	root@pts/4 # /opt/consul kv get name 
	zhangsan
	 
	 
	root@pts/4 # /opt/consul kv put db/mysql1/name m1
	Success! Data written to: db/mysql1/name
	root@pts/4 # /opt/consul kv get db/mysql1/name
	m1	
	
## 服务注册 

<br> 	
	
### 配置文件注册 ,后 consul reload 

<br> 


	cat /etc/consul.d/mysql.json 
	{
		"services":[
			{
			"id":"10.168.0.119-3306",
				"name":"mysqldexporter",
				"tags":[
					"prod"
				],

				"meta":{
			"region":"网速机房",
			"environment":"prod",
			"cluster":"db1",
			"replication_set":"bgslave",
					"service_name":"10.168.0.119:3306-db1"
				},
				"address":"10.168.0.119",
				"port":9104,
				"checks":[
					{
						"Tcp":"10.168.0.119:3306",
						"interval":"3s"
					}
				]
			},

					{
			"id":"10.168.0.65-33060",
				"name":"mysqldexporter",
				"tags":[
					"prod"
				],

				"meta":{
					"region":"网速机房",
					"environment":"prod",
					"cluster":"db2",
					"replication_set":"bgslave",
					"service_name":"10.168.0.65:33060-db2"
				},
				"address":"10.168.0.65",
				"port":9105,
				"checks":[
					{
						"Tcp":"10.168.0.65:33060",
						"interval":"3s"
					}
				]
			}  


		]
	}

### 命令行http api注册

<br> 

	curl http://10.18.8.178:8500/v1/agent/service/register -X PUT -i -H "Content-Type:application/json" -d '{
	  "ID": "10.168.0.161-3309",  
	  "Name": "mysqldexporter",
	  "Tags": [
		"prod",
		"dc1"
	  ],
	  "Meta":{
					"region":"网速机房",
					"environment":"prod",
					"cluster":"resource",
					"replication_set":"master",
					"service_name":"10.168.0.161:3309-resource"
	  
	  
	  },
	  "Address": "10.168.0.161",
	  "Port": 9104,
	  "EnableTagOverride": false,
	  "Check": {
		
		"Tcp": "10.168.0.161:3309",
		"Interval": "10s",
		"Timeout": "5s"
	  }
	}'


参考： 

https://www.cnblogs.com/mafeng/p/7680451.html 

http://c.biancheng.net/view/4476.html

https://mp.weixin.qq.com/s?__biz=MzU0MTczNzA1OA==&mid=2247484205&idx=1&sn=7f2dc686036e22efebed8934e624d74c&chksm=fb242a20cc53a336f5c3c901d335ffb12cd1b6d3ae17850cdeba0dad052045ae2791892df56f&mpshare=1&scene=1&srcid=1115bHEC5YovvkwWAMkQkKPr&sharer_sharetime=1573810398781&sharer_shareid=9b64bf3a2476b2c5d499db84b8628fde&rd2werd=1#wechat_redirect
