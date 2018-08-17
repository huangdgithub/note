## Zookeeper 伪集群 配置教程
------
> * 首先需要在你的Linux服务器上安装java环境（即JDK）
> * 下载Zookeeper包，地址：http://apache.claz.org/zookeeper/zookeeper-3.5.3-beta/zookeeper-3.5.3-beta.tar.gz
> * Zookeeper集群部署机器数量为奇数，这里不做解释。
> * 根据自我需要，部署多套zookeeper实例

------
### 实例配置目录如图：
![](https://github.com/huangdgithub/note/blob/master/img/1.png)

### 实例配置如图：
![](https://github.com/huangdgithub/note/blob/master/img/2.png)

###部分参数说明：
```python
    server.0=192.168.3.133:8080:7770
    server.1=192.168.3.133:8081:7771
    server.2=192.168.3.133:8082:7772
```
其中server中的 0，1，2 分别对应每个实例myid中的值

```python
dataDir=/tmp/zookeeper/data
dataLogDir=/tmp/zookeeper/logs
```
dataDir对应Zookeeper保存数据的目录
dataLogDir对应Zookeeper 将写数据的日志文件目录

![](https://github.com/huangdgithub/note/blob/master/img/3.png)

clientPort：Zookeeper 服务器监听端口，用来接受客户端的访问请求

------

### zookeeper 启动和查看状态

![](https://github.com/huangdgithub/note/blob/master/img/4.png)

![](https://github.com/huangdgithub/note/blob/master/img/5.png)


### 注意事项

在一台机器上部署了3个server，需要注意的是在集群为分布式模式下我们使用的每个配置文档模拟一台机器，也就是说单台机器及上运行多个Zookeeper实例。但是，必须保证每个配置文档的各个端口号不能冲突，除了clientPort不同之外，dataDir也不同。另外，还要在dataDir所对应的目录中创建myid文件来指定对应的Zookeeper服务器实例。

> * clientPort端口：如果在1台机器上部署多个server，那么每台机器都要不同的 clientPort，比如 server1是2181,server2是2182，server3是2183

> * dataDir和dataLogDir：dataDir和dataLogDir也需要区分下，将数据文件和日志文件分开存放，同时每个server的这两变量所对应的路径都是不同的

> * server.X和myid： server.X 这个数字就是对应，data/myid中的数字。在3个server的myid文件中分别写入了0，1，2，那么每个server中的zoo.cfg都配 server.0 server.1,server.2就行了。因为在同一台机器上，后面连着的2个端口，3个server都不要一样，否则端口冲突

------

### 部分其它异常记录：

Zookeep启动正常，报错：Error contacting service. It is probably not running，一般来说有如下几种情况。

> * 端口（clientPort）可能被占用，可以使用命令查看（netstat -anp | grep xxxx）
> * dataDir或者dataLogDir 对应目录未生成，或者路径写错
> * tmp目录下myid文件写错，不同的节点上面的myid里的数字不同，分别设置：0,1,2等等。
> * 防火墙状态  
        1：service iptables stop //关闭防火墙    
        2：service iptables status //查看状态  
        3：chkconfig iptables off //禁用防火墙  
    如果是ubuntu请使用如下命令：  
        1：查看防火墙状态：ufw status   
        2：关闭防火墙：ufw disable   
    Centos请使用如下命令：   
        1：查看防火墙状态：firewall-cmd --state   
        2：关闭防火墙：systemctl stop firewalld.service   





