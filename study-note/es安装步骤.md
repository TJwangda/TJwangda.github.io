# linux安装es步骤

## es步骤

1. wget下载安装包  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz

2. tar包解压  tar -zxvf elasticsearch-7.1.1.tar.gz

3. 修改  mv elasticsearch-7.1.1 elasticsearch

4. 修改配置文件 vim elasticsearch.yml

5. ES 版本> = 5.0.0 时，是不能用超级管理员运行的，此时需要切换到普通账号或者新建 ES 账号

   添加用户组 groupadd elasticsearch

   添加用户指定用户名密码和分组 useradd -g elasticsearch -p elasticsearch elasticsearch

   进入bin目录启动 （-d 后台启动，不带就前台启动） $ ./elasticsearch -d

6. curl 127.0.0.1:9200测试是否启动成功。



## es安装问题

若页面无法访问127.0.0.1:9200

~~~shell
只能使用127.0.01或者localhost访问，使用ip地址无法访问？
解决办法：

① 修改 elasticsearch.yml 中的「network.host」

network.host: 0.0.0.0
② 重启 ES 出现如果如下报错，请依次按下面的步骤解决

ERROR: [3] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max number of threads [3818] for user [elasticsearch] is too low, increase to at least [4096]
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

每个进程最大同时打开文件数太小

修改 /etc/security/limits.conf 文件，增加如下配置，用户退出后重新登录生效

* soft nofile 65536
* hard nofile 65536
[2]: max number of threads [3818] for user [es] is too low, increase to at least [4096]

最大线程个数太低

同上修改 /etc/security/limits.conf 文件，增加如下配置，用户退出后重新登录生效

* soft nproc 4096
* hard nproc 4096
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

一个进程能拥有的最多的内存区域

修改 /etc/sysctl.conf 文件，增加如下配置，执行命令「 sysctl -p 」生效

vm.max_map_count=262144
③ 切换到 elasticsearch 用户并重启， curl 测试成功

ERROR: [1] bootstrap checks failed
[1]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
修改
elasticsearch.yml
取消注释保留一个节点
cluster.initial_master_nodes: ["node-1"]
~~~



## es添加ik分词器

找到对应es的ik分词器版本下载，解压到es目录的plugins下的ik文件夹

~~~shell
一、对应关系
https://github.com/medcl/elasticsearch-analysis-ik/blob/master/README.md

二.下载与之对应的版本
https://github.com/medcl/elasticsearch-analysis-ik/releases
~~~





## 安装kibana

路径usr/local/ 新建kibana文件夹

wget https://artifacts.elastic.co/downloads/kibana/kibana-7.1.1-linux-x86_64.tar.gz

启动

bin目录下kibana脚本前台启动 ./kibana

后台启动：nohup ../bin/kibana &    关闭：netstat -anltp|grep 5601（查出服务id关闭或者 ps -ef | grep '.*node/bin/node.*src/cli'）

ip访问失败，项目未启动

~~~shell
#端口被占用，修改port端口
curl 10.10.111.82:5601
curl: (7) Failed to connect to 10.10.111.82 port 5601: Connection refused

#修改conf中的配置文件的serverhost为0.0.0.0
Error: listen EADDRNOTAVAIL: address not available 10.10.111.82:5601
~~~









