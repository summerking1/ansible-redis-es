## 使用注意事项

1. 基于ansible环境，ansible安装略
2. redis我没有采用免编译各节点需要提前准备gcc-c++环境
3. elasticsearch各节点需要提前执行java角色部署java环境
4. 注意关闭防火墙

## 使用方法
1. 按实际情况更改hosts文件
2. 修改work_yml中对应的nodeipX（集群实际ip）
3. 安装redis集群：`ansible-playbook java.yml`
4. 安装redis集群：`ansible-playbook redis.yml`
5. 安装elasticsearch`ansible-playbook elasticsearch.yml`
6. 集群会自动开机启动
   


### 1.首先验证各个节点能ping通
```shell
[root@localhost work_yml]# ansible all -m ping 
192.168.0.177 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
192.168.0.179 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
192.168.0.178 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
[root@localhost work_yml]# 
```

### 2.关闭防火墙
```shell
[root@localhost work_yml]# ansible all -m service -a 'name=firewalld state=stopped'
```

### 3.部署java环境
```shell
[root@localhost work_yml]# ll
总用量 12
-rw-r--r--. 1 root root 340 1月  26 09:42 elasticsearch.yml
-rw-r--r--. 1 root root 315 1月  26 09:42 java.yml
-rw-r--r--. 1 root root 295 1月  26 09:42 redis.yml
[root@localhost work_yml]# pwd
/etc/ansible/work_yml
[root@localhost work_yml]# ansible-playbook java.yml 
```

- 验证
  
```shell
[root@localhost work_yml]# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
[root@localhost work_yml]# 
```

### 4.部署redis

**注意需要提前准备gcc-c++环境**

`yum install -y gcc-c++`

```shell
[root@localhost work_yml]# ansible-playbook redis.yml
...
PLAY RECAP ******************************************************************************************************************************************************************************************************
192.168.0.177              : ok=25   changed=17   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.0.178              : ok=24   changed=16   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.0.179              : ok=24   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- 验证

```shell
[root@localhost smartmining]# ps -ef |grep redis
root     121639      1  0 09:56 ?        00:00:00 /home/smartmining/redis-5.0.5/src/redis-server 192.168.0.178:27000 [cluster]
root     121641      1  0 09:56 ?        00:00:00 /home/smartmining/redis-5.0.5/src/redis-server 192.168.0.178:27001 [cluster]
root     125286   1947  0 09:58 pts/0    00:00:00 grep --color=auto redis
[root@localhost smartmining]# redis-cli -h 192.168.0.178 -p 27000
192.168.0.178:27000> cluster nodes
9f6933b649782aebe995020394d7c4fa2ae5dd74 192.168.0.178:27001@37001 slave e52490b0f7b8902e964355d5c049f963e51f9ee3 0 1611626301811 1 connected
e52490b0f7b8902e964355d5c049f963e51f9ee3 192.168.0.177:27000@37000 master - 0 1611626303816 1 connected 0-5460
dbe8ee65dd5ace34f5595d8eebecafd70418991c 192.168.0.179:27001@37001 slave 62268baec35e3ab7fcc1dda2e84cc5950076d95a 0 1611626302814 6 connected
661a33b46ee8d80a41d77d6e4941afce60347264 192.168.0.179:27000@37000 master - 0 1611626301000 5 connected 10923-16383
96fec91a514b80771f646ed3e7ebae5506959cf0 192.168.0.177:27001@37001 slave 661a33b46ee8d80a41d77d6e4941afce60347264 0 1611626301000 5 connected
62268baec35e3ab7fcc1dda2e84cc5950076d95a 192.168.0.178:27000@37000 myself,master - 0 1611626302000 3 connected 5461-10922
192.168.0.178:27000> 
```
### 5.部署elasticsearch
```shell
[root@localhost work_yml]# ansible-playbook elasticsearch.yml
···
PLAY RECAP ******************************************************************************************************************************************************************************************************
192.168.0.177              : ok=22   changed=20   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.0.178              : ok=22   changed=20   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.0.179              : ok=22   changed=20   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- 验证

```shell
[root@localhost work_yml]# jps
33716 Jps
31770 Elasticsearch
[root@localhost work_yml]# curl 192.168.0.177:29200
{
  "name" : "es1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "ZlwLGXLYSrCM9bvShPdOmw",
  "version" : {
    "number" : "6.3.1",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "eb782d0",
    "build_date" : "2018-06-29T21:59:26.107521Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
[root@localhost work_yml]# curl 192.168.0.177:29200/_cat/nodes
192.168.0.177 34 24 3 0.79 0.64 0.37 mdi * es1
192.168.0.179 30 22 3 0.32 0.48 0.26 mdi - es3
192.168.0.178 32 22 3 0.21 0.33 0.23 mdi - es2
[root@localhost work_yml]# 
```

1. 手动关机机命令：
   ansible redis -m shell -a "sh /home/smartmining/redis_cluster/script/shutdown.sh"
   ansible elasticsearch -m shell -a "sh /home/smartmining/elasticsearch_cluster/script/shutdown.sh"

2. 手动开机命令：
   ansible redis -m shell -a "sh /home/smartmining/redis_cluster/script/start.sh"
   ansible elasticsearch -m shell -a "sh /home/smartmining/elasticsearch_cluster/script/start.sh"

