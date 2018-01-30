CENTOS 6.5 ELK 6.1.2 安装文档
===

安装包
---


jdk8
**[http://www.oracle.com/technetwork/indexes/downloads/index.html](http://www.oracle.com/technetwork/indexes/downloads/index.html)**

elasticsearch 6.1.2
**[https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.2.rpm](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.2.rpm)**

logstash-6.1.2
**[https://artifacts.elastic.co/downloads/elasticsearch/logstash-6.1.2.rpm](https://artifacts.elastic.co/downloads/elasticsearch/logstash-6.1.2.rpm)**

kibana-6.1.2
**[https://artifacts.elastic.co/downloads/elasticsearch/kibana-6.1.2.rpm](https://artifacts.elastic.co/downloads/elasticsearch/kibana-6.1.2.rpm)**
        
jdk8
---

1. 下载安装：
```bash
cd /opt/
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.tar.gz"
tar xzf jdk-8u161-linux-x64.tar.gz
```

2. 修改首选项：
```bash
cd /opt/jdk1.8.0_161/
alternatives --install /usr/bin/java java /opt/jdk1.8.0_161/bin/java 2
alternatives --config java

alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_161/bin/jar 2
alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_161/bin/javac 2
alternatives --set jar /opt/jdk1.8.0_161/bin/jar
alternatives --set javac /opt/jdk1.8.0_161/bin/javac
```
    
3. 检查版本：
```bash
java -version
```

4. 设置环境变量：
```bash
export JAVA_HOME=/opt/jdk1.8.0_161
export JRE_HOME=/opt/jdk1.8.0_161/jre
export PATH=$PATH:/opt/jdk1.8.0_161/bin:/opt/jdk1.8.0_161/jre/bin
``` 
> 可以将上述变量加在/etc/environment 或者 ~/.bashrc 等配置文件中，以便重启自动生效

## elasticsearch 

1. 安装：
```bash
rpm -ivh <path>/elasticsearch-6.1.2.rpm
```

2. 启动：
```bash
service elasticsearch start 
```

3. 启动日志：
```bash
tail -f /var/log/elasticsearch/elasticsearch.log
```

4. 测试：
```bash
curl http://127.0.0.1:9200
```
返回类似代码是表示成功
```json
{
  "name" : "ZXbuBLL",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "-naVvEvKQ--kg-l53U0SAw",
  "version" : {
    "number" : "6.1.2",
    "build_hash" : "5b1fea5",
    "build_date" : "2018-01-10T02:35:59.208Z",
    "build_snapshot" : false,
    "lucene_version" : "7.1.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

> 需要内网访问或者公网访问
```bash
vi /etc/elasticsearch/elasticsearch.yml
修改
network.host: {实际地址}
```
        
> 修改端口
```bash
vi /etc/elasticsearch/elasticsearch.yml
修改
http.port: {实际端口}
```      

####错误1：java.lang.UnsupportedOperationException: seccomp unavailable: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in
```bash
vi /etc/elasticsearch/elasticsearch.yml
添加
bootstrap.system_call_filter: false
```

####错误2：flood stage disk watermark [95%] exceeded on

指定data文件路径
```bash
mkdir -p /mnt/xml/elasticsearchdata //{实际数据盘}
chown -R elasticsearch:elasticsearch /mnt/xml/elasticsearchdata/ //{实际数据盘}
vim /etc/elasticsearch/elasticsearch.yml
path.data: /mnt/xml/elasticsearchdata //{实际数据盘}
```

####错误3：max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
```bash
vi /etc/security/limits.conf 
添加或修改为如下内容:
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```

####错误4：max number of threads [2048] for user [elasticsearch] is too low, increase to at least [4096]
```bash
vi /etc/security/limits.d/90-nproc.conf
添加或修改为如下内容:
* soft nproc 4096
```   

## logstash 

1. 安装：
```bash
rpm -ivh <path>/logstash-6.1.2.rpm    
/usr/share/logstash/bin/system-install /etc/logstash/startup.options sysv
```

2. 启动：
```bash
service logstash start 
```

3. 启动日志：
```bash
tail -f /var/log/logstash/logstash.log
```

4. 测试：
  
  准备测试文件：logstash-test.conf 
```bash
  input { stdin { } 
  }
  output {
    elasticsearch { hosts => ["localhost:9200"] }
    stdout { codec => rubydebug }
  }
```
  测试文件内容是否正确：
  ```bash
  /usr/share/logstash/bin/logstash -f logstash-test.conf -t
  ```
  
  启动：
  ```bash
  /usr/share/logstash/bin/logstash -f logstash-test.conf 
  ```
   输入：
   ```bash
   test 
   ```
   打印：
```bash
{
   "@timestamp" => 2018-01-30T02:52:07.605Z,
     "@version" => "1",
         "host" => "iZ25vlqew7eZ",
      "message" => "test"
}
```

5. 配置文件路径：
```bash
/etc/logstash/conf.d/*.conf
``` 
    
6. 注意事项：
```text
只有一个管道，写在多个conf文件里的input和output使用跟写在同一个文件中效果相同，如果想不同的输入有不同的输出，可以在input和output上指定类型
```

## kibana 

1. 安装：
```bash
rpm -ivh <path>/kibana-6.1.2-x86_64.rpm   
```     

2. 修改配置文件:
```bash
vim /etc/kibana/kibana.yml
配置或添加如下参数
server.port: 5601
server.host: "{实际地址}"
server.name: "{显示名称}"
elasticsearch.url: "http://localhost:9200" //如果上面修改了地址，这里需要相应改动
```

3. 启动：
```bash
service kibana start
```

4. 日志：
```bash
tail -f /var/log/kibana/kibana.stdout
``` 

5. 测试：
```bash
http://{实际地址}:5601    
``` 
    