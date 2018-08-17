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
![](2)

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

![](3)

clientPort：Zookeeper 服务器监听端口，用来接受客户端的访问请求
------


