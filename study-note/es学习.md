# linux安装es步骤

## es安装步骤

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

### （一）、页面无法访问127.0.0.1:9200

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

### （二）、es设置用户名密码

1. 修改elasticsearch.yml

~~~shell
增加配置：
# 配置X-Pack
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
重启es
~~~

2. 执行设置用户名和密码的命令，需要为4个用户分别设置密码：elastic、kibana、logstash_system、beats_system

~~~shell
#进入bin目录，执行命令
./elasticsearch-setup-passwords interactive
#设置密码
~~~

3. 修改kibana配置文件

   ~~~shell
   #进入kibana安装目录
   cd /usr/local/kibana-7.2.0-linux-x86_64/config
   
   #修改配置文件
   vim kibana.yml
   
   #添加配置
   elasticsearch.username: "elastic"
   elasticsearch.password: "xxx"
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

**启动**

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



## logstash安装

1. 下载安装包：wget https://artifacts.elastic.co/downloads/logstash/logstash-7.1.1.tar.gz
2. 解压 tar -zxvf  logstash-7.1.1.tar.gz
3. 修改文件夹名称





# es 命令

## 基础命令

~~~sehll
GET _cat/indices

GET yangben/_search

GET yangben/_count

GET yangben/_doc/16061

POST user/_doc
{
  "firstName":"d",
  "secondName":"w"
}

GET user/_search

POST user/_doc/1
{
  "firstName":"jack",
  "secondName":"ma"
}

POST user/_create/3
{
  "firstName":"tom3",
  "secondName":"smith",
  "sex":"1"
}


~~~

## 进阶



### 滤重分页

~~~shell
GET employee/_search
{

    "from":0,
    "size":1,
    "query":{
        "match_all":{
        }
    },
    "aggregations":{
        "agg":{
            "terms":{
                "field":"job",
                "size":10
            },
            "aggregations":{
                "top":{
                    "top_hits":{
                        "from":0,
                        "size":1,
                        
                        "seq_no_primary_term":false,
                        "explain":false,
                        "_source":{
                            "includes":[
                                "name","id","job","age","sal","gender"
                            ],
                            "excludes":[
                            ]
                        }
                    }
                },
                "bucket_field":{
                    "bucket_sort":{
                        "sort":[
                        ],
                        "from":0,
                        "size":10
                    }
                }
            }
        }
    }
}
~~~

### 短语查询：match_phrase

~~~shell
当一个字符串被分析时，分析器不仅只返回一个词条列表，它同时也返回原始字符串的每个词条的位置、或者顺序信息。
位置信息可以被保存在倒排索引(Inverted Index)中，像match_phrase这样位置感知(Position-aware)的查询能够使用位置信息来匹配那些含有正确单词出现顺序的文档，且在这些单词之间没有插入别的单词。

短语是什么
对于匹配了短语"quick brown fox"的文档，下面的条件必须为true：

quick、brown和fox必须全部出现在某个字段中。
brown的位置必须比quick的位置大1。
fox的位置必须比quick的位置大2。
如果以上的任何一个条件没有被满足，那么文档就不能被匹配。

GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": "quick brown fox"
            ，"slop" : 1  （可调节因子，可选。）
        }
}
相当于
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": {
                "query": "quick brown fox",
                "type":  "phrase"
            }
     }
}

~~~



# 常见报错

### （一）、内存限制：FORBIDDEN/12/index read-only / allow delete (api)]; 

~~~shell
[2021-07-09T07:49:06,554][WARN ][o.e.c.r.a.DiskThresholdMonitor] [node-1] high disk watermark [90%] exceeded on [MkxoUOysRAy0mnrw5DGmDA][node-1][/usr/local/elasticsearch/elasticsearch/data/nodes/0] free: 5.6gb[5.6%], shards will be relocated away from this node
[2021-07-09T07:49:36,555][WARN ][o.e.c.r.a.DiskThresholdMonitor] [node-1] high disk watermark [90%] exceeded on [MkxoUOysRAy0mnrw5DGmDA][node-1][/usr/local/elasticsearch/elasticsearch/data/nodes/0] free: 5.6gb[5.6%], shards will be relocated away from this node
[2021-07-09T07:49:36,556][INFO ][o.e.c.r.a.DiskThresholdMonitor] [node-1] rerouting shards: [high disk watermark exceeded on one or more nodes]
[2021-07-09T07:50:06,557][WARN ][o.e.c.r.a.DiskThresholdMonitor] [node-1] high disk watermark [90%] exceeded on [MkxoUOysRAy0mnrw5DGmDA][node-1][/usr/local/elasticsearch/elasticsearch/data/nodes/0] free: 5.6gb[5.6%], shards will be relocated away from this node
[2021-07-09T07:50:36,559][WARN ][o.e.c.r.a.DiskThresholdMonitor] [node-1] high disk watermark [90%] exceeded on [MkxoUOysRAy0mnrw5DGmDA][node-1][/usr/local/elasticsearch/elasticsearch/data/nodes/0] free: 5.6gb[5.6%], shards will be relocated away from this node
[2021-07-09T07:50:36,560][INFO ][o.e.c.r.a.DiskThresholdMonitor] [node-1] rerouting shards: [high disk watermark exceeded on one or more nodes]
[2021-07-09T07:51:06,561][WARN ][o.e.c.r.a.DiskThresholdMonitor] [node-1] high disk watermark [90%] exceeded on [MkxoUOysRAy0mnrw5DGmDA][node-1][/usr/local/elasticsearch/elasticsearch/data/nodes/0] free: 5.6gb[5.6%], shards will be relocated away from this node

~~~

#### 方案：在config/elasticsearch.yml中增加配置：cluster.routing.allocation.disk.threshold_enabled: false



### （二）、elasticsearch 启动报错

~~~shell
ps -ef | grep elastics2021-07-28 01:57:57,232 main ERROR RollingFileManager (/usr/local/elasticsearch/elasticsearch/logs/my-application_server.json) java.io.FileNotFoundException: /usr/local/elasticsearch/elasticsearch/logs/my-application_server.json (Permission denied) java.io.FileNotFoundException: /usr/local/elasticsearch/elasticsearch/logs/my-application_server.json (Permission denied)
	at java.io.FileOutputStream.open0(Native Method)
	at java.io.FileOutputStream.open(FileOutputStream.java:270)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:133)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory.createManager(RollingFileManager.java:640)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory.createManager(RollingFileManager.java:608)
	at org.apache.logging.log4j.core.appender.AbstractManager.getManager(AbstractManager.java:113)
	at org.apache.logging.log4j.core.appender.OutputStreamManager.getManager(OutputStreamManager.java:114)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager.getFileManager(RollingFileManager.java:188)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:145)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:61)
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.build(PluginBuilder.java:123)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createPluginObject(AbstractConfiguration.java:959)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:899)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:891)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.doConfigure(AbstractConfiguration.java:514)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.initialize(AbstractConfiguration.java:238)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.start(AbstractConfiguration.java:250)
	at org.apache.logging.log4j.core.LoggerContext.setConfiguration(LoggerContext.java:547)
	at org.apache.logging.log4j.core.LoggerContext.start(LoggerContext.java:263)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:234)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:127)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:294)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124)
	at org.elasticsearch.cli.Command.main(Command.java:90)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92)

2021-07-28 01:57:57,237 main ERROR Could not create plugin of type class org.apache.logging.log4j.core.appender.RollingFileAppender for element RollingFile: java.lang.IllegalStateException: ManagerFactory [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory@723e2d08] unable to create manager for [/usr/local/elasticsearch/elasticsearch/logs/my-application_server.json] with data [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$FactoryData@6d4a82[pattern=/usr/local/elasticsearch/elasticsearch/logs/my-application-%d{yyyy-MM-dd}-%i.json.gz, append=true, bufferedIO=true, bufferSize=8192, policy=CompositeTriggeringPolicy(policies=[TimeBasedTriggeringPolicy(nextRolloverMillis=0, interval=1, modulate=true), SizeBasedTriggeringPolicy(size=134217728)]), strategy=DefaultRolloverStrategy(min=-2147483648, max=2147483647, useMax=false), advertiseURI=null, layout=ESJsonLayout{patternLayout={"type": "server", "timestamp": "%d{yyyy-MM-dd'T'HH:mm:ss,SSSZ}", "level": "%p", "component": "%c{1.}", "cluster.name": "${sys:es.logs.cluster_name}", "node.name": "%node_name", %notEmpty{%node_and_cluster_id, } "message": "%notEmpty{%enc{%marker}{JSON} }%enc{%.-10000m}{JSON}" %exceptionAsJson }%n}, filePermissions=null, fileOwner=null]] java.lang.IllegalStateException: ManagerFactory [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory@723e2d08] unable to create manager for [/usr/local/elasticsearch/elasticsearch/logs/my-application_server.json] with data [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$FactoryData@6d4a82[pattern=/usr/local/elasticsearch/elasticsearch/logs/my-application-%d{yyyy-MM-dd}-%i.json.gz, append=true, bufferedIO=true, bufferSize=8192, policy=CompositeTriggeringPolicy(policies=[TimeBasedTriggeringPolicy(nextRolloverMillis=0, interval=1, modulate=true), SizeBasedTriggeringPolicy(size=134217728)]), strategy=DefaultRolloverStrategy(min=-2147483648, max=2147483647, useMax=false), advertiseURI=null, layout=ESJsonLayout{patternLayout={"type": "server", "timestamp": "%d{yyyy-MM-dd'T'HH:mm:ss,SSSZ}", "level": "%p", "component": "%c{1.}", "cluster.name": "${sys:es.logs.cluster_name}", "node.name": "%node_name", %notEmpty{%node_and_cluster_id, } "message": "%notEmpty{%enc{%marker}{JSON} }%enc{%.-10000m}{JSON}" %exceptionAsJson }%n}, filePermissions=null, fileOwner=null]]
	at org.apache.logging.log4j.core.appender.AbstractManager.getManager(AbstractManager.java:115)
	at org.apache.logging.log4j.core.appender.OutputStreamManager.getManager(OutputStreamManager.java:114)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager.getFileManager(RollingFileManager.java:188)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:145)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:61)
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.build(PluginBuilder.java:123)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createPluginObject(AbstractConfiguration.java:959)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:899)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:891)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.doConfigure(AbstractConfiguration.java:514)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.initialize(AbstractConfiguration.java:238)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.start(AbstractConfiguration.java:250)
	at org.apache.logging.log4j.core.LoggerContext.setConfiguration(LoggerContext.java:547)
	at org.apache.logging.log4j.core.LoggerContext.start(LoggerContext.java:263)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:234)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:127)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:294)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124)
	at org.elasticsearch.cli.Command.main(Command.java:90)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92)

2021-07-28 01:57:57,238 main ERROR Unable to invoke factory method in class org.apache.logging.log4j.core.appender.RollingFileAppender for element RollingFile: java.lang.IllegalStateException: No factory method found for class org.apache.logging.log4j.core.appender.RollingFileAppender java.lang.IllegalStateException: No factory method found for class org.apache.logging.log4j.core.appender.RollingFileAppender
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.findFactoryMethod(PluginBuilder.java:235)
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.build(PluginBuilder.java:135)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createPluginObject(AbstractConfiguration.java:959)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:899)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:891)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.doConfigure(AbstractConfiguration.java:514)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.initialize(AbstractConfiguration.java:238)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.start(AbstractConfiguration.java:250)
	at org.apache.logging.log4j.core.LoggerContext.setConfiguration(LoggerContext.java:547)
	at org.apache.logging.log4j.core.LoggerContext.start(LoggerContext.java:263)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:234)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:127)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:294)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124)
	at org.elasticsearch.cli.Command.main(Command.java:90)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92)

e2021-07-28 01:57:57,260 main ERROR RollingFileManager (/usr/local/elasticsearch/elasticsearch/logs/my-application.log) java.io.FileNotFoundException: /usr/local/elasticsearch/elasticsearch/logs/my-application.log (Permission denied) java.io.FileNotFoundException: /usr/local/elasticsearch/elasticsearch/logs/my-application.log (Permission denied)
	at java.io.FileOutputStream.open0(Native Method)
	at java.io.FileOutputStream.open(FileOutputStream.java:270)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:133)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory.createManager(RollingFileManager.java:640)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory.createManager(RollingFileManager.java:608)
	at org.apache.logging.log4j.core.appender.AbstractManager.getManager(AbstractManager.java:113)
	at org.apache.logging.log4j.core.appender.OutputStreamManager.getManager(OutputStreamManager.java:114)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager.getFileManager(RollingFileManager.java:188)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:145)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:61)
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.build(PluginBuilder.java:123)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createPluginObject(AbstractConfiguration.java:959)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:899)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:891)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.doConfigure(AbstractConfiguration.java:514)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.initialize(AbstractConfiguration.java:238)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.start(AbstractConfiguration.java:250)
	at org.apache.logging.log4j.core.LoggerContext.setConfiguration(LoggerContext.java:547)
	at org.apache.logging.log4j.core.LoggerContext.start(LoggerContext.java:263)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:234)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:127)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:294)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124)
	at org.elasticsearch.cli.Command.main(Command.java:90)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92)

2021-07-28 01:57:57,261 main ERROR Could not create plugin of type class org.apache.logging.log4j.core.appender.RollingFileAppender for element RollingFile: java.lang.IllegalStateException: ManagerFactory [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory@723e2d08] unable to create manager for [/usr/local/elasticsearch/elasticsearch/logs/my-application.log] with data [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$FactoryData@33e4b9c4[pattern=/usr/local/elasticsearch/elasticsearch/logs/my-application-%d{yyyy-MM-dd}-%i.log.gz, append=true, bufferedIO=true, bufferSize=8192, policy=CompositeTriggeringPolicy(policies=[TimeBasedTriggeringPolicy(nextRolloverMillis=0, interval=1, modulate=true), SizeBasedTriggeringPolicy(size=134217728)]), strategy=DefaultRolloverStrategy(min=-2147483648, max=2147483647, useMax=false), advertiseURI=null, layout=[%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n, filePermissions=null, fileOwner=null]] java.lang.IllegalStateException: ManagerFactory [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$RollingFileManagerFactory@723e2d08] unable to create manager for [/usr/local/elasticsearch/elasticsearch/logs/my-application.log] with data [org.apache.logging.log4j.core.appender.rolling.RollingFileManager$FactoryData@33e4b9c4[pattern=/usr/local/elasticsearch/elasticsearch/logs/my-application-%d{yyyy-MM-dd}-%i.log.gz, append=true, bufferedIO=true, bufferSize=8192, policy=CompositeTriggeringPolicy(policies=[TimeBasedTriggeringPolicy(nextRolloverMillis=0, interval=1, modulate=true), SizeBasedTriggeringPolicy(size=134217728)]), strategy=DefaultRolloverStrategy(min=-2147483648, max=2147483647, useMax=false), advertiseURI=null, layout=[%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n, filePermissions=null, fileOwner=null]]
	at org.apache.logging.log4j.core.appender.AbstractManager.getManager(AbstractManager.java:115)
	at org.apache.logging.log4j.core.appender.OutputStreamManager.getManager(OutputStreamManager.java:114)
	at org.apache.logging.log4j.core.appender.rolling.RollingFileManager.getFileManager(RollingFileManager.java:188)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:145)
	at org.apache.logging.log4j.core.appender.RollingFileAppender$Builder.build(RollingFileAppender.java:61)
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.build(PluginBuilder.java:123)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createPluginObject(AbstractConfiguration.java:959)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:899)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:891)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.doConfigure(AbstractConfiguration.java:514)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.initialize(AbstractConfiguration.java:238)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.start(AbstractConfiguration.java:250)
	at org.apache.logging.log4j.core.LoggerContext.setConfiguration(LoggerContext.java:547)
	at org.apache.logging.log4j.core.LoggerContext.start(LoggerContext.java:263)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:234)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:127)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:294)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124)
	at org.elasticsearch.cli.Command.main(Command.java:90)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92)

2021-07-28 01:57:57,262 main ERROR Unable to invoke factory method in class org.apache.logging.log4j.core.appender.RollingFileAppender for element RollingFile: java.lang.IllegalStateException: No factory method found for class org.apache.logging.log4j.core.appender.RollingFileAppender java.lang.IllegalStateException: No factory method found for class org.apache.logging.log4j.core.appender.RollingFileAppender
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.findFactoryMethod(PluginBuilder.java:235)
	at org.apache.logging.log4j.core.config.plugins.util.PluginBuilder.build(PluginBuilder.java:135)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createPluginObject(AbstractConfiguration.java:959)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:899)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.createConfiguration(AbstractConfiguration.java:891)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.doConfigure(AbstractConfiguration.java:514)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.initialize(AbstractConfiguration.java:238)
	at org.apache.logging.log4j.core.config.AbstractConfiguration.start(AbstractConfiguration.java:250)
	at org.apache.logging.log4j.core.LoggerContext.setConfiguration(LoggerContext.java:547)
	at org.apache.logging.log4j.core.LoggerContext.start(LoggerContext.java:263)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:234)
	at org.elasticsearch.common.logging.LogConfigurator.configure(LogConfigurator.java:127)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:294)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124)
	at org.elasticsearch.cli.Command.main(Command.java:90)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)

~~~

#### 解决方案

文件权限问题

~~~shell
ll查看logs目录
[root@ysp-t logs]# ll
total 11884
-rw-rw-r-- 1 elasticsearch elasticsearch    2286 Jul 21 02:34 my-application-2021-07-20-1.json.gz
-rw-rw-r-- 1 elasticsearch elasticsearch    2129 Jul 21 02:34 my-application-2021-07-20-1.log.gz
-rw-rw-r-- 1 elasticsearch elasticsearch    5437 Jul 23 01:42 my-application-2021-07-21-1.json.gz
-rw-rw-r-- 1 elasticsearch elasticsearch    5197 Jul 23 01:42 my-application-2021-07-21-1.log.gz
-rw-r--r-- 1 root          root            10961 Jul 27 08:32 my-application-2021-07-23-1.json.gz
-rw-r--r-- 1 root          root            10437 Jul 27 08:32 my-application-2021-07-23-1.log.gz
-rw-rw-r-- 1 elasticsearch elasticsearch       0 Jul  1 07:29 my-application_audit.json
-rw-rw-r-- 1 elasticsearch elasticsearch    7067 Jul 28 01:11 my-application_deprecation.json
-rw-rw-r-- 1 elasticsearch elasticsearch    3251 Jul 28 01:11 my-application_deprecation.log
-rw-rw-r-- 1 elasticsearch elasticsearch       0 Jul  1 07:29 my-application_index_indexing_slowlog.json
-rw-rw-r-- 1 elasticsearch elasticsearch       0 Jul  1 07:29 my-application_index_indexing_slowlog.log
-rw-rw-r-- 1 elasticsearch elasticsearch       0 Jul  1 07:29 my-application_index_search_slowlog.json
-rw-rw-r-- 1 elasticsearch elasticsearch       0 Jul  1 07:29 my-application_index_search_slowlog.log
-rw-r--r-- 1 root          root             2715 Jul 27 08:32 my-application.log
-rw-r--r-- 1 root          root             3049 Jul 27 08:32 my-application_server.json

# 修改文件权限
[root@ysp-t logs]# chown  -R elasticsearch:elasticsearch my-application-2021-07-23-1.json.gz
[root@ysp-t logs]# chown  -R elasticsearch:elasticsearch my-application-2021-07-23-1.log.gz
[root@ysp-t logs]# chown  -R elasticsearch:elasticsearch my-application.log

# 重启es
~~~









