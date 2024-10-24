---
date: '2019-05-28'
title: Couchbase集群搭建

---



* [couchbase 集群搭建](#couchbase-%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA)
+ [集群配置过程](#%E9%9B%86%E7%BE%A4%E9%85%8D%E7%BD%AE%E8%BF%87%E7%A8%8B)
- [安装 couchbase 7\.2\.4](#%E5%AE%89%E8%A3%85-couchbase-724)
- [创建一个集群](#%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E9%9B%86%E7%BE%A4)
- [将新节点加入老集群](#%E5%B0%86%E6%96%B0%E8%8A%82%E7%82%B9%E5%8A%A0%E5%85%A5%E8%80%81%E9%9B%86%E7%BE%A4)
- [创建bucket及user](#%E5%88%9B%E5%BB%BAbucket%E5%8F%8Auser)

+ [遇到的问题](#%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98)
- [场景1：集群添加新节点报错](#%E5%9C%BA%E6%99%AF1%EF%BC%9A%E9%9B%86%E7%BE%A4%E6%B7%BB%E5%8A%A0%E6%96%B0%E8%8A%82%E7%82%B9%E6%8A%A5%E9%94%99)
- [场景2：找回web登录密码](#%E5%9C%BA%E6%99%AF2%EF%BC%9A%E6%89%BE%E5%9B%9Eweb%E7%99%BB%E5%BD%95%E5%AF%86%E7%A0%81)
- [场景3：以特定账号启动couchbase服务](#%E5%9C%BA%E6%99%AF3%EF%BC%9A%E4%BB%A5%E7%89%B9%E5%AE%9A%E8%B4%A6%E5%8F%B7%E5%90%AF%E5%8A%A8couchbase%E6%9C%8D%E5%8A%A1)
- [场景4：利用 couchbase\-cli 管理数据库](#%E5%9C%BA%E6%99%AF4%EF%BC%9A%E5%88%A9%E7%94%A8-couchbase-cli-%E7%AE%A1%E7%90%86%E6%95%B0%E6%8D%AE%E5%BA%93)



couchbase 集群搭建
==============


CouchBase是一款开源的、分布式的、面向文档的NoSQL数据库，主要用于分布式缓存和数据存储领域，它是Apache CouchDB和MemBase这两个NoSQL数据库的合并的产物，而MemBase是基于Memcached的。因此CouchBase联合了CouchDB的简单可靠和Memcached的高性能，以及membase的可扩展性。


\[TOC]


集群配置过程
------


### 安装 couchbase 7\.2\.4


* 所有节点：安装 couchbase\-server\-community 7\.2\.4



```
#.下载并安装
cd /d d:\
wget -c http://iso.sqlfans.cn/couchbase/wins/couchbase-server-community_7.2.4-windows_amd64.msi
start /wait couchbase-server-community_7.2.4-windows_amd64.msi /quiet

#.设置防火墙，示例内网为 10.30.0.0/255.255.0.0
netsh advfirewall firewall add rule name="permit_couchbase_unencrypt_8091" protocol=TCP dir=in remoteip=10.30.0.0/255.255.0.0 localport=4369,6060,8091-8094,9100-9105,9998-9999,11209-11215,18091-18093,21100-21299 action=allow

```

### 创建一个集群


* 第1步，在主节点上（示例`10.30.3.144`）用 chrome浏览器 打开 `http://10.30.3.144:8091/ui/index.html`，点击 `Setup New Cluster`


[![](https://static.sqlfans.cn/file/20240709/cb-setup-newcluster.png)](https://static.sqlfans.cn/file/20240709/cb-setup-newcluster.png)


* 第2步，配置集群信息，示例集群名为 `sqlfans` 管理员账号为 `Administrator` 密码为 `Admin_147`


[![](https://static.sqlfans.cn/file/20240709/cb-setup-cluster-name.png)](https://static.sqlfans.cn/file/20240709/cb-setup-cluster-name.png)


* 第3步，勾选 我接受，然后点击 `Configure Disk, Memory, Services`


[![](https://static.sqlfans.cn/file/20240709/cb-setup-i-accept.png)](https://static.sqlfans.cn/file/20240709/cb-setup-i-accept.png)


* 第4步，配置当前节点的ip地址（示例`10.30.3.144`）及数据目录（示例`d:/couchbase/data`），并完成安装


[![](https://static.sqlfans.cn/file/20240709/cb-setup-ip-datadir.png)](https://static.sqlfans.cn/file/20240709/cb-setup-ip-datadir.png)


### 将新节点加入老集群


* 第1步，用 chrome 浏览器打开集群节点的控制台，示例 `http://10.30.3.144:8091/ui/index.html`，在 `Servers` 标签页点击 ADD SERVER 按钮，输入新节点的ip地址（示例`10.30.3.145`），将其加入该集群


[![](https://static.sqlfans.cn/file/20240709/cb-cluster-node-add.png)](https://static.sqlfans.cn/file/20240709/cb-cluster-node-add.png)


* 第2步，添加新节点之后，再点击右上角的 Rebalance 按钮完成新节点的数据同步


[![](https://static.sqlfans.cn/file/20240709/cb-cluster-node-rebalance.png)](https://static.sqlfans.cn/file/20240709/cb-cluster-node-rebalance.png)


### 创建bucket及user


* 第1步，用 chrome 浏览器打开集群节点的控制台，示例 `http://10.30.3.144:8091/ui/index.html`，在 `Bucket` 标签页点击 ADD BUCKET 按钮，输入新bucket的名称（示例`web_prd`）并分配内存（示例`2048`MB）


[![](https://static.sqlfans.cn/file/20240709/cb-config-bucket-add.png)](https://static.sqlfans.cn/file/20240709/cb-config-bucket-add.png)


* 第2步，创建bucket之后，在 `Security` 标签页点击 ADD USER 按钮，输入新user的名称（示例`user_prd`）及密码，并勾选对指定bucket（示例`web_prd`）的权限


[![](https://static.sqlfans.cn/file/20240709/cb-config-user-add.png)](https://static.sqlfans.cn/file/20240709/cb-config-user-add.png)


遇到的问题
-----


### 场景1：集群添加新节点报错


* 报错1：集群添加新节点之后，执行 Rebalance 之后报错 CPU with SSE4\.2 extensions is required，启用新节点的 SSE 4\.2 指令集（pve请修改cpu的Type为host）即可



```
Service 'memcached' exited with status 1. Restarting. Messages:
2024-07-10T10:10:16.610950+08:00 CRITICAL Failed to initialise - CPU with SSE4.2 extensions is required - terminating.

```

* 报错2：集群新节点启用SSE 4\.2指令集并重启之后，报错 invalid UTF\-8 byte at index 38: 0xDA，将 couchbase 版本由 7\.6\.0 更换为 7\.2\.4



```
Service 'memcached' exited with status 255. Restarting. Messages:
2024-07-10T10:16:11.272519+08:00 CRITICAL Failed to create required listening socket:
"{"family":"inet","host":"*","port":11210,"system":false,"tag":"","tls":false,"type":"mcbp"}". 
Errors: [json.exception.type_error.316] invalid UTF-8 byte at index 38: 0xDA. Terminating.

```

### 场景2：找回web登录密码


* 不知道密码的情况下重置，示例 新密码为 `Admin_147`



```
cd /d C:\"Program Files"\Couchbase\Server\bin
couchbase-cli reset-admin-password -c 127.0.0.1:8091 -u Administrator -p Admin_147 --new-password Admin_147

```

* 已知密码的情况下重置，示例 新密码为 `Admin_258`



```
cd /d C:\"Program Files"\Couchbase\Server\bin
couchbase-cli reset-admin-password -c 127.0.0.1:8091 -u Administrator -p Admin_147 --new-password Admin_258

```

### 场景3：以特定账号启动couchbase服务


* 以特定账号启动couchbase服务，示例账号 `couchbase` 密码 `AdMin_147`



```
net user couchbase AdMin_147 /add /Y /expires:never
net localgroup administrators couchbase /add

takeown /f "%ProgramFiles%\Couchbase" /a /r
icacls "%ProgramFiles%\Couchbase" /t /grant:r couchbase:f

#.自动脚本会启动失败,手动修改服务的启动账号就好了
net stop CouchbaseServer
sc config CouchbaseServer obj= ".\couchbase" password= "AdmMin_147"
net start CouchbaseServer
sc config CouchbaseServer start= auto

```

### 场景4：利用 couchbase\-cli 管理数据库


* couchbase\-cli server\-list 查看 节点状态



```
couchbase-cli server-list -c 127.0.0.1:8091 -u Administrator -p Admin_147

```

* couchbase\-cli user\-manage 管理 只读账号



```
#.只能创建1个只读账号，示例名称 user_readonly 密码 123456
couchbase-cli user-manage -c 127.0.0.1:8091 --set --ro-username=user_readonly --ro-password=123456 -u Administrator -p Admin_147

#.查询只读账号
couchbase-cli user-manage -c 127.0.0.1:8091 --list -u Administrator -p Admin_147

#.删除只读账号
couchbase-cli user-manage -c 127.0.0.1:8091 --delete --ro-username=user_readonly -u Administrator -p Admin_147

```

Copyright © www.baidd.cn 2023 All Right Reserved更新时间：
2024\-09\-07 22:07:13


