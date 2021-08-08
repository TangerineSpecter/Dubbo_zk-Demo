# Dubbo_zk-Demo
Dubbo微服务+Zookeeper分布式演示

#### Zookeeper介绍

- 中间件，提供协调服务
- 作用于分布式系统，可以为大数据服务
- 支持java和C语言

#### Zookeeper特性

- 一致性：数据一致性，数据按照顺序分批入库
- 原子性：事务要么成功要么失败，不会局部化
- 单一视图：客户端连接集群中的任一zk节点，数据都是一致的
- 可靠性：每次对zk的操作状态都会保存在服务端
- 实时性：客户端可以读取到zk服务端的最新数据

#### 单机Zookeeper安装

1.下载完成后，解压。

> 下载地址：https://downloads.apache.org/zookeeper/

2.配置环境变量

```
sudo vim ~/.bash_profile
打开后写入：
# zookeeper
export ZK_HOME=解压zk目录路径
PATH=${ZK_HOME}/bin
```

3.复制一份zoo_sample.cfg改名为zoo.cfg

> 配置说明
```yaml
# 服务器与客户端之间交互的基本时间单元（ms）
tickTime=2000
# zookeeper所能接受的客户端数量
initLimit=10
# 服务器和客户端之间请求和应答之间的时间间隔（心跳机制）
syncLimit=5
# zookeeper中使用的基本时间单位, 毫秒值.
tickTime=2000
# 数据目录. 可以是任意目录.（必须配置）
dataDir=zk根目录/data
# log目录, 同样可以是任意目录. 如果没有设置该参数, 将使用和#dataDir相同的设置.
dataLogDir=zk根目录/log
# t监听client连接的端口号.
clientPort=2181
```
如果根目录没有data和log目录，则进行创建

4.启动zk

```bash
#启动
zkServer.sh start 
#查看状态
zkServer.sh status
```

5.通过zkCli.sh进入控制台

#### Zookeeper目录结构

- bin：主要的运行命令
- conf：存放配置文件
- contrib：附加功能
- dist-maven：mvn编译后目录
- docs：文档
- lib：需要依赖的jar包
- recipes：案例demo代码
- src：源码

#### Zookeeper基本数据模型介绍

- 树形结构
- 每一个节点称为znode，可以有子节点，也可以有数据
- 每个节点分为临时节点和永久节点，临时节点客户端断开会丢失
- 每个zk节点都有各自的版本号
- 每个节点数据发生变化，该节点的版本号会累加（乐观锁）
- 删除/修改过时的节点，版本号不匹配则会报错
- 每个zk节点存储数据不宜过大，几kb就好
- 节点可以设置权限acl，通过权限来限制用户的访问

#### Zookeeper数据模型基本操作

```bash
# 查看某个路径下目录列表
ls [path]
# 查看目录更详细的信息
ls2 [path]
# 获取节点数据和状态信息,watch对节点进行事件监听
get [path] watch
# 查看当前节点状态信息
stat [path] watch
# 创建节点并赋值,s表示顺序节点，e表示临时节点，path路径，data数据，acl权限，默认world全世界
create [-s][-e] [path] data acl
# 修改节点存储的数据,version表示版本号，查看详情里面的dataVersion
set [path] data [version]
# 删除节点
delete [path] [version]
```

> 详情字段说明

```bash
# 创建节点id
cZxid = 0x0
# 创建节点时间 
ctime = Thu Jan 01 08:00:00 CST 1970
# 修改节点id
mZxid = 0x0
# 修改节点时间
mtime = Thu Jan 01 08:00:00 CST 1970
# 子节点id
pZxid = 0x3
# 子节点version
cversion = 1
# 当前节点数据版本号，变化累加1
dataVersion = 0
# 当前节点权限版本号，变化累加1
aclVersion = 0
# 
ephemeralOwner = 0x0
# 数据长度
dataLength = 0
# 子节点数量
numChildren = 1
```

#### Zookeeper的作用

- master节点选举，主节点挂了，从节点会接受工作，保证这个节点是唯一的，保证集群高可用
- 统一配置文件管理，只需要部署一台服务器，则可以把相同的配置文件同步更新到其他服务器。
- 发布与订阅，类型消息队列，dubbo发布者把数据存储在znode上，订阅者会读取到这个数据
- 提供分布式锁
- 集群管理，保证集群中数据的强一致性

#### Zookeeper的watcher机制

- 针对每个节点的操作，都会有一个监督者watcher(一个事件)
- 当监控某个对象（znode）发生变化，就会触发watcher事件
- 一次性的，触发后立刻销毁
- 父节点，子节点增删改都能够触发其watcher
- 针对不同类型的操作，触发的watcher事件也不同：
    - 节点创建事件
    - 节点删除事件
    - 节点数据变化事件
    - 这样就可以根据节点的变化类型进行相应的操作
    
#### 