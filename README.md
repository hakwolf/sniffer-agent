# Sniffer-Agent

> Sniffer TCP package, parsed with mysql protocol, optional you can just print on screen or send query info to Kafka.
> 抓取tcp包解析出mysql语句，将查询信息打印在屏幕上或者发送到Kafka

### Architecture

架构设计：

本项目采用模块化设计，主要分为四大模块：抓包模块，协议解析模块，输出模块，心跳模块
![架构设计图](https://github.com/zr-hebo/sniffer-agent/blob/master/images/arch.png)

### Parse Protocol

sniffer-agent采用模块化结构，支持用户添加自己的解析模块，只要实现了统一的接口即可
- [x] MySQL
- [ ] PostgreSQL
- [ ] Redis
- [ ] Mongodb

目前输出的内容都是解析结果组成的json。
MySQL协议的解析结果示例如下：
```
{"sid":"10.XX.XX.XX:54656","sip":"192.168.XX.XX","sport":3306,"user":"root","db":"sniffer","sql":"show tables","bt":"2019-08-05 18:23:09","cms":15}
```
其中sid代表客户端ip：port组成的session标识，sip代表server ip，sport代表server port，user代表查询用户，db代表当前连接的库名，sql代表查询语句，bt代表查询开始时间，cms代表查询消耗的时间，单位是毫秒

### Exporter

输出模块主要负责，将解析的结果对外输出。默认情况下输出到命令行，可以通过指定export_type参数选择kafka，这时候会直接将解析结果发送到kafka。
同样只要实现了export接口，用户可以自定义自己的输出方式。

### Install:

环境：

golang：1.12

libpcap包

测试脚本运行在python3环境下


1.安装依赖，目前自测支持Linux系列操作系统，其他版本的系统有待验证

CentOS:
```
yum install libpcap-devel
```

Ubuntu:
```
apt-get install libpcap-dev
```
2.执行编译命令 go build

### Demo

目前只支持MySQL协议的抓取，需要将编译后的二进制文件上传到MySQL服务器上
1.最简单的使用

`./sniffer-agent`

2.指定log级别，可以指定的值为debug、info、warn、error，默认是info

`./sniffer-agent --log_level=debug`

默认会监听 网卡：eth0，端口3306

3.指定网卡和监听端口

`./sniffer-agent --interface=eth0 --port=3358`

4.指定输出到kafka，为了将ddl和select、dml区分处理，这里使用了两个topic来生产消息

`./sniffer-agent --export_type=kafka --kafka-server=$kafka_server:$kafka_server --kafka-group-id=sniffer --kafka-async-topic=non_ddl_sql_collector --kafka-sync-topic=ddl_sql_collector`

5.指定严格模式，通过查询获取长连接的用户名和数据库

`./sniffer-agent --strict_mode=true --admin_user=root --admin_passwd=123456`

#### 题外话
在做这个功能之前，项目组调研过类似功能的产品，最有名的是 [mysql-sniffer](https://github.com/Qihoo360/mysql-sniffer) 和 [go-sniffer](https://github.com/40t/go-sniffer)，这两个产品都很优秀，不过我们的业务场景要求更多。
我们需要将提取的SQL信息发送到kafka进行处理，之前的两个产品输出的结果需要进行一些处理然后自己发送，在QPS比较高的情况下，这些处理会消耗较多的CPU；
另外mysql-sniffer使用c++开发，平台的适用性较差，后期扩展较难。
开发的过程中也借鉴了这些产品的思想，另外在MySQL包解析的时候，参考了一些 [TiDB](https://github.com/pingcap/tidb) 的内容，部分私有变量和函数直接复制使用，这里向这些优秀的产品致敬，如有侵权请随时联系。

### License
[MIT](https://opensource.org/licenses/MIT)
