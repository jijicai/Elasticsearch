    Elasticsearch 的运行环境：jdk8+，Elasticsearch官方推荐版本为：Oracle JDK version 1.8.0_131。
    Window环境：win10 专业版
    Linux环境：CentOS-7-x86_64-Minimal-1810.iso


**下载页面：**

    elastic的网页在一些网络环境下加载比较慢，遇到页面加载失败的情况，请多刷新几次。
    最新软件包的下载页面：https://www.elastic.co/cn/start
    https://www.elastic.co/cn/downloads/
	
	GitHub上的源码包：https://github.com/elastic/elasticsearch/releases/tag/v6.8.0
	下载各个版本的软件包：https://www.elastic.co/cn/downloads/past-releases#elasticsearch
	
## win10环境-Elasticsearch、Kibana安装

### Elasticsearch下载 
    Elasticsearch6.8.0各个环境的下载页面：https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-0
    在win10环境中，本文所下载的版本是：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.0.zip
    直接使用浏览器下载可能网速比较慢，推荐使用迅雷下载。

### Elasticsearch配置
    把elasticsearch-6.8.0.zip解压到指定的目录C:\softwares，解压后es主目录为：C:\softwares\elasticsearch-6.8.0
    单节点es服务，默认情况下无需任何配置。
		
### Elasticsearch运行
    打开cmd命令行窗口，输入命令：bin\elasticsearch.bat，即可运行es服务了。
    通过浏览器访问：http://localhost:9200，若能正常返回es服务的基本信息，则说明es服务已经正常启动。
    
    后台启动：bin\elasticsearch.bat -d，在win环境下，这种后台启动只是不输出控制台日志，关闭当前cmd窗口，es服务会停止。
        在win环境中，可以把elasticsearch注册成为一个后台服务。
        
    带参数的启动：bin\elasticsearch.bat -E cluster.name=jijicai_cluster -E node.name=es_server_001
		
		
### Kibana下载
    Kibana6.8.0各个环境的下载页面：https://www.elastic.co/cn/downloads/past-releases/kibana-6-8-0
    在win10环境中，本文所下载的版本是：https://artifacts.elastic.co/downloads/kibana/kibana-6.8.0-windows-x86_64.zip

### Kibana配置
    把kibana-6.8.0-windows-x86_64.zip解压到指定的目录C:\softwares，解压后es主目录为：C:\softwares\kibana-6.8.0-windows-x86_64
    Kibana的默认es服务URI：http://localhost:9200
    默认情况下无需任何配置。
	
### Kibana运行
    打开cmd命令行窗口，输入命令：bin\kibana.bat，即可运行Kibana服务了。
    通过浏览器访问：http://localhost:5601，即可进入Kibana UI界面，左边是各个菜单栏。
    单击菜单[Monitoring],可以看到有一个es服务节点（Nodes: 1），单击这个链接，进入到节点列表页面。
    然后，再单击列表中Name列的es服务的名称超链接，会进入到一个动态图表页面。
    这个页面展现了es服务的内存（JVM、Index）/CPU使用情况等等。
	
### Elasticsearch集群配置
    单机3个es服务组成的集群，相同的es主目录（$ES_HOME），无需任何配置，使用默认配置即可。启动命令如下节。
    
    多机多节点集群配置，请参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.8/modules-network.html
	
### Elasticsearch集群运行
    单机3个es服务组成的集群，启动命令如下：
    bin\elasticsearch.bat -E cluster.name=jijicai_cluster -E node.name=es_server_001 -E http.port=9200 ^
    -E path.data=C:\softwares\elasticsearch-6.8.0\data -E path.logs=C:\softwares\elasticsearch-6.8.0\logs

    bin\elasticsearch.bat -E cluster.name=jijicai_cluster -E node.name=es_server_002 -E http.port=9202 ^
    -E path.data=C:\softwares\elasticsearch-6.8.0\data2 -E path.logs=C:\softwares\elasticsearch-6.8.0\logs2

    bin\elasticsearch.bat -E cluster.name=jijicai_cluster -E node.name=es_server_003 -E http.port=9203 ^
    -E path.data=C:\softwares\elasticsearch-6.8.0\data3 -E path.logs=C:\softwares\elasticsearch-6.8.0\logs3
    
    通过浏览器访问：http://localhost:9200/_cat/nodes?v，即可查看3个几点组成的集群情况，如下：
    
    ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
    127.0.0.1           49          62  13                          mdi       *      es_server_001
    127.0.0.1           41          62  11                          mdi       -      es_server_003
    127.0.0.1           51          62  12                          mdi       -      es_server_002
    
    1个主节点，2个从节点，其中带 * 号的表示主节点。这种集群方式不建议在生产环境中使用。
    对于这个集群，Kibana也无需任何配置，打开Kibana的菜单[Monitoring]，也可以看到这三个实例组成的集群。
    
    参数network.host是默认配置的话，es服务集群只能本机访问。