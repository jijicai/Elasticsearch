## Linux环境（CentOS7） JDK8 安装
	下载页面：http://openjdk.java.net/install/
	执行命令，即可开始安装：[root@localhost ~]# yum install java-1.8.0-openjdk

## CentOS7环境-Elasticsearch集群、Kibana安装
    在CentOS7服务器中，文本使用账号：elsearch，去运行es服务、Kibana服务。
    
### 规划
    主机名称	IP					node.name(es)		集群名称
    es-138		192.168.110.138		es_server_001		jijicai_cluster
    es-139		192.168.110.139		es_server_002		jijicai_cluster
    es-140		192.168.110.140		es_server_003		jijicai_cluster
    
    Kibana：单节点，部署在 es-140机器上。
    
    先部署es-138机器。

### Elasticsearch下载
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.0.tar.gz
    若上面的方式下载比较慢，推荐使用迅雷下载，然后通过工具（WinSCP）传到Linux服务器（CentOS7）。
    把文件elasticsearch-6.8.0.tar.gz放到目录：/home/elsearch/softwares

### Elasticsearch配置
    解压：tar -xzf elasticsearch-6.8.0.tar.gz
    
    进入es主目录：cd elasticsearch-6.8.0/

    OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
    
***不能使用root账号启动 es服务，否则会报如下错误：***

    [2019-07-12T05:29:52,163][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [unknown] uncaught exception in thread [main]
    org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
            at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:163) ~[elasticsearch-6.8.0.jar:6.8.0]
            at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) ~[elasticsearch-6.8.0.jar:6.8.0]
            at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-6.8.0.jar:6.8.0]
            at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) ~[elasticsearch-cli-6.8.0.jar:6.8.0]
            at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-6.8.0.jar:6.8.0]
            at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:116) ~[elasticsearch-6.8.0.jar:6.8.0]
            at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:93) ~[elasticsearch-6.8.0.jar:6.8.0]
    Caused by: java.lang.RuntimeException: can not run elasticsearch as root
            at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:103) ~[elasticsearch-6.8.0.jar:6.8.0]
            at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:170) ~[elasticsearch-6.8.0.jar:6.8.0]
            at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:333) ~[elasticsearch-6.8.0.jar:6.8.0]
            at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) ~[elasticsearch-6.8.0.jar:6.8.0]
            
            
#### 配置max file descriptors、配置max number of threads
    打开文件：vi /etc/security/limits.conf
    在文件结尾添加：elsearch        -       nofile          65535
                    elsearch        -       nproc           4096
    退出账号elsearch重新登录即可生效。
    
#### 配置max virtual memory
    打开文件：vi /etc/sysctl.conf
    在文件结尾添加：vm.max_map_count = 262144
    执行命令：sysctl -p，即可生效。
    
    ulimit -n 65535
    /etc/security/limits.conf
    
    ulimit -u 4096
    /etc/security/limits.conf
    
    sysctl -w vm.max_map_count=262144
    /etc/sysctl.conf
    
    把参数network.host配置为非默认值，切上面的参数没有配置的话，启动时控制台会报如下错误：
    ERROR: [3] bootstrap checks failed
    [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
    [2]: max number of threads [3795] for user [hongka] is too low, increase to at least [4096]
    [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.8/file-descriptors.html
    参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.8/max-number-of-threads.html
    参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.8/vm-max-map-count.html
    
#### 配置防火墙
    执行命令：firewall-cmd --state（或： systemctl status firewalld.service），查看防火墙的状态。
    若防火墙正在运行，请关闭它，或者设置9200端口可以穿过防火墙。这样局域网内的其他机器才能访问该es服务。
    关闭服务：systemctl stop firewalld.service	
    在开机时禁用服务：systemctl disable firewalld.service
    
    
#### 配置文件config/elasticsearch.yml
    在文件结尾添加：
    cluster.name: jijicai_cluster
    node.name: es_server_001
    network.host: _site_
    #network.bind_host: _site_
    #network.publish_host: _site_
    #http.port: 9200
    discovery.zen.ping.unicast.hosts: ["192.168.110.138","192.168.110.139","192.168.110.140"]
    #discovery.zen.minimum_master_nodes: 1
    
    其中 _site_，一般情况下，表示局域网IP。
    参数network.host 可以同时设置network.bind_host、network.publish_host。
    但有时候需要分别设置，例如：es服务运行在一个代理服务器背后时，需要把这两个参数设置为不同的值。
    
    把配置好的es复制到另外两台机器上：
    cd /home/elsearch/softwares/elasticsearch-6.8.0
    scp -r ./elasticsearch-6.8.0 192.168.110.139:/home/elsearch/softwares/
    scp -r ./elasticsearch-6.8.0 192.168.110.140:/home/elsearch/softwares/
    
    并把yml配置文件中的参数node.name分别修改为：es_server_002、es_server_003
    修改系统配置、防火墙。
    
    es复制到其他服务器后，应当删除 data、log目录下的数据：
    cd /home/elsearch/softwares/elasticsearch-6.8.0
    rm -rf data/*
    rm -rf log/* 

### Elasticsearch运行
    进入es主目录：cd /home/elsearch/softwares/elasticsearch-6.8.0
    依次后台启动各个es服务：bin/elasticsearch -d
    通过浏览器访问：http://192.168.110.138:9200/_cat/nodes?v，即可查看3个几点组成的集群情况，如下：
    ip              heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
    192.168.110.138           23          95   1    0.09    0.13     0.09 mdi       *      es_server_001
    192.168.110.140            9          93  90    1.06    0.30     0.14 mdi       -      es_server_003
    192.168.110.139           11          93   2    0.60    0.26     0.13 mdi       -      es_server_002
    
    停止命令，其中结尾的数字为es服务的进程ID：
    kill -SIGTERM 17416
    
    通过命令：ps -aux|grep elasticsearch，可以查找es服务的进程ID。
 
### Kibana下载
    wget https://artifacts.elastic.co/downloads/kibana/kibana-6.8.0-linux-x86_64.tar.gz
    若上面的方式下载比较慢，推荐使用迅雷下载，然后通过工具（WinSCP）传到Linux服务器（CentOS7）。
    把文件elasticsearch-6.8.0.tar.gz放到目录：/home/elsearch/softwares

### Kibana配置
    解压：tar -xzvf kibana-6.8.0-linux-x86_64.tar.gz
    
    进入Kibana主目录：cd kibana-6.8.0-linux-x86_64/
    
    配置文件config/kibana.yml
    在文件结尾添加：
    server.host: "192.168.110.140"
    elasticsearch.hosts: ["http://192.168.110.138:9200","http://192.168.110.139:9200","http://192.168.110.140:9200"]
    
    参考：https://www.elastic.co/guide/en/kibana/6.8/settings.html

### Kibana运行
    执行命令：bin/kibana（后台启动：nohup bin/kibana &），启动Kibana服务。
    通过浏览器访问：http://192.168.110.140:5601，即可进入Kibana页面。
    打开Kibana的菜单[Monitoring]，就可以看到这三个es服务组成的集群。
