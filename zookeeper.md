# 分布式服务治理、分布式协调服务Zookeeper深入学习笔记
## 第一部分：Zookeeper环境搭建和基本使用
### 1.1 基本概念
#### ①集群角色
zookeeper集群引入了Leader、Follower、Observer三种角色，所有机器通过Leader选举来选定一台被称为Leader的机器，Leader服务器提供读和写服务，除Leader外，其他机器包括Follower和Observer，Follower和Observer都能提供读服务，唯一的区别在于<B>Observer不参与Leader选举过程，不参与写操作的过半写成功策略</B>，因此Observer可以在不影响写性能的情况下提升集群的性能。

#### ②会话（session）
zookeeper对外的服务端口默认是2181，客户端启动的时候，首先会与服务器建立一个TCP连接，从第一次连接建立开始，客户端会话的生命周期就开始了，通过这个连接，客户端能够心跳检测与服务器保持会话，也能够向zookeeper服务器发送请求并接受响应，同时还能够通过该链接接收来自服务器的Watch事件通知。

#### ③数据节点（Znode）
在zookeeper中，构成集群的机器，称为机器节点；数据模型中的数据单元，称为数据节点ZNode。zookeeper将所有数据存在内存中，数据模型是一棵树ZNode Tree，由斜杠（/）进行分割的路径，就是一个Znode。每个ZNode上都会保存自己的数据内容，同时还会保存一系列属性信息。

#### ④版本
对于每个ZNode，zookeeper都会为其维护一个叫作stat的数据结构，stat记录了这个ZNode的三个数据版本，分别是version（当前ZNode的版本）、cversion（当前ZNode子节点的版本）、aversion（当前ZNode的ACL版本）。

#### ⑤Watcher（事件监听器）
Watcher，是zookeeper的一个很重要的特性，zookeeper允许用户在指定节点上注册一些Watcher，并且有特定事件处罚的时候，zookeeper服务端会把事件通知到感兴趣的客户端，该机制是zookeeper实现分布式协调服务的重要特性

#### ⑥ACL
zookeeper采用ACL（Access Control Lists）策略来进行权限控制

- CREATE：创建子节点的权限
- READ：获取节点数据和子节点列表的权限
- WRITE：更新节点数据的权限
- DELETE：删除子节点的权限
- ADMIN：设置节点ACL的权限

注意：CREATE和DELETE针对的都是子节点

### 1.2 ZNode
节点类型分为三大类：持久性节点、临时性节点、顺序性节点

在开发中上述节点组合成一下四种节点类型：

#### 持久节点：
最常见的节点类型，一旦创建就一直存在，直到删除操作主动删除
#### 持久顺序节点：
有顺序的持久节点，创建节点的时候，会在节点后面加上一个数字后缀，表示顺序
#### 临时节点：
会被自动清理的节点，它的生命周期和客户会话绑在一起，客户端会话结束，节点被删除。与持久节点不同的是，临时节点不能创建子节点

#### 临时顺序节点：
有顺序的临时节点，节点名称后加数字后缀

### 1.3 事务ID
zk的事务一般指数据节点的创建、删除、更新等操作，对于每一个事务请求，zk都会为其分配一个全局唯一的事务ID，用ZXID来表示，通常是一个64位的数字。每一个ZXID对应一次更新操作，从这些ZXID中可以间接的识别出zk处理这些更新操作请求的全局顺序


### 1.4 Watcher--数据变更通知
在zk中，引入Watcher机制来实现分布式通知。zk允许客户端向服务器注册一个Watcher监听，当服务端的一些指定事件出发了这个Watcher，那么就会向指定客户端发送一个事件通知来实现分布式的通知功能。

客户端的流程：客户端在向服务器注册的同时，会将Watcher对象存储在客户端的WatcherManager当中。当zk服务器触发Watcher事件后，会向客户端发送通知，客户端线程从WatcherManager中取出对应的Watcher对象来执行回调逻辑

### 1.5 ACL--保障数据的安全
从三个方面理解ACL机制：<B>权限模式（Schema）、授权对象（ID）、权限（Permission）</B>，通常使用<B>scheme : id : permission</B>来标识一个有效的ACL信息。

####  权限模式：scheme
用来确定权限验证过程中使用的检验策略

##### 1.IP
可以针对具体IP地址或者某个网段进行权限控制，如ip:192.168.0.1或者ip:192.168.0.1/24（标识针对192.168.0.*网段）

##### 2.Digest
最常用的权限控制模式，要更符合我们对权限控制的认识，其使用“username:password”形式的权限标识来进行权限配置，便于区分不同应用来进行权限控制

当通过“username:password”配置权限标识后，zk会先后对其进行SHA-1加密和BASE64编码

##### 3.World
最开放的权限控制模式，这种权限控制几乎没有任何作用，可以看做是特殊的Digest模式，它只有一个权限标识“world:anyone”

##### 4.Super
超级用户，也是一种特殊的Digest模式，超级用户可以对任意zk上的数据节点进行任何操作

#### 授权对象：ID
一般是IP地址或者机器等等

#### 权限
- CREATE：创建子节点的权限
- READ：获取节点数据和子节点列表的权限
- WRITE：更新节点数据的权限
- DELETE：删除子节点的权限
- ADMIN：设置节点ACL的权限

### 1.6 命令行操作
#### 创建节点
~~~
create [-s][-e] path data acl
其中，-s指定创建顺序节点，-e指定创建临时节点，如果不指定，就是创建持久节点；acl进行权限控制
~~~
①创建顺序节点
~~~
create -s /zk-test 123
~~~
②创建临时节点
~~~
create -e /zk-temp 123
~~~

#### 读取节点
~~~
ls path
查看指定节点下的第一级的所有子节点
~~~

~~~
get path
获取指定节点的数据内容和信息
~~~

~~~
ls2 path
获取当前节点的信息和直系子节点列表
~~~

#### 更新信息和删除信息
~~~
set path data
更新信息
~~~

~~~
delete path
删除节点：但要注意，如果当前节点有子节点，当前节点是无法删除的，需要先删除子节点，再删除当前节点
~~~

### 1.7 API操作
~~~java
package com.ldx.api;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.io.IOException;
import java.util.concurrent.CountDownLatch;

public class CreateSession implements Watcher {

    private static CountDownLatch countDownLatch = new CountDownLatch(1);

    private static ZooKeeper zooKeeper;

    public static void main(String[] args) throws IOException, InterruptedException {
        zooKeeper = new ZooKeeper("127.0.0.1:2181", 5000, new CreateSession());
        System.out.println(zooKeeper.getState());

        //countDownLatch.await();
        System.out.println("会话真正建立");
        //阻塞主进程，让process执行
        Thread.sleep(Integer.MAX_VALUE);

    }

    public void process(WatchedEvent watchedEvent) {
        if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
            System.out.println("执行process...");
            //countDownLatch.countDown();
        }
        try {
            //createNodeSync();
            //getNodeData();
            //getChildrens();
            deleteNodeSync();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 1.创建同步节点
    private void createNodeSync() throws KeeperException, InterruptedException {
        String persistent = zooKeeper.create("/persistent", "持久性节点内容".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        String ephemeral = zooKeeper.create("/ephemeral", "临时节点内容".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        String persistent_sequential = zooKeeper.create("/persistent_sequential", "持久性顺序节点内容".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);

        System.out.println("持久性节点:" + persistent);
        System.out.println("临时节点:" + ephemeral);
        System.out.println("持久性顺序节点:" + persistent_sequential);
    }

    // 2.获取节点内容
    private void getNodeData() throws KeeperException, InterruptedException {
        byte[] data = zooKeeper.getData("/persistent", false, null);
        System.out.println(new String(data));
    }

    // 获取节点的子节点列表
    private static void getChildrens() throws KeeperException, InterruptedException {
        zooKeeper.getChildren("/persistent", true);
    }

    // 3.删除节点
    private void deleteNodeSync() throws KeeperException, InterruptedException {
        Stat stat = zooKeeper.exists("/per", false);
        System.out.println(stat == null ? "节点不存在" : "节点存在");

        if (stat != null) {
            zooKeeper.delete("/per", -1);
        }

        Stat stat2 = zooKeeper.exists("/per", false);
        System.out.println(stat2 == null ? "节点不存在" : "节点存在");


    }

}
~~~

### 1.8 开源客户端
zkClient：Github上的开源客户端

Curator：Netflix开源的优秀客户端

## 第二部分：Zookeeper应用场景和深入进阶
### 2.1 应用场景
#### 数据发布/订阅
zookeeper采用推拉结合的方式：客户端向服务端注册自己需要关注的节点，一旦该节点的数据发生变化，那么服务端就会向相应的客户端发送Watcher事件通知，客户端接收到这个消息通知之后，需要主动到服务端获取最新的数据。
#### 命名服务

#### 集群管理


#### Master选举
zookeeper强一致性，保证客户端无法重复创建一个已经存在的数据节点。也就是多个客户端同时创建一个相同节点，最终只有一个成功。利用这个特性，可以在分布式环境中进行Master选举。

#### 分布式锁
##### 排它锁
竞争创建一个临时节点，并注册监听Watcher，当获取到锁的客户端宕机或任务完成执行删除操作时，临时节点都会被删除，其他客户端继续执行竞争create操作
##### 共享锁
读写请求都到节点下创建临时顺序节点，如读请求：READ-000001，写请求：WRITE-000002。

判断顺序：

1. 创建完节点后，获取父节点下的所有子节点，并注册监听节点变更
2. 确定自己的节点序号在所有子节点中的顺序
3. 对于读请求：若没有比自己序号小的节点或比自己小的节点全是读请求，则表示自己获取到共享锁，执行读请求。对于写请求L若自己不是序号最小的节点，需要等待。
4. 接收到Watcher通知后，重复步骤1。

上述方法的问题：每次Watcher通知后，其实只有序号最小的那个节点需要操作，其他的仍然要等待，但他们也接收到了Watcher通知，所以也重新获取了一遍子节点列表，所以很浪费。

改进：Watcher不要监听父节点，而是监听离自己最近的且序号比自己小的节点的状态就可以。因为只有前一个节点操作完了，才轮到自己，这样可以减少Watcher通知。


#### 分布式队列
##### FIFO：先入先出
和共享锁相似
##### Barrier：分布式屏障，元素聚集后统一安排
设置一个阈值，当子节点个数达到阈值时，一次性读取所有子节点进行处理，具体处理过程和共享锁相似。子节点个数达不到阈值就等待。

### 2.2 深入进阶
#### ZAB协议
ZAB协议是zookeeper的数据一致性的核心算法

支持崩溃恢复的原子广播协议

ZAB核心：对会改变zookeeper服务器数据状态的事务请求的处理方式

> 所有请求都由一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，余下的服务器被称为Follower服务器，Leader服务器负责将一个客户端事务请求转换成一个事务proposal，并将该proposal转发给所有的Follower服务器，之后等待Follower服务器反馈，一旦超过半数的Follower服务器进行了正确的反馈后，Leader再次向所有Follower服务器发送commit消息，要求将proposal提交

ZAB协议包括两种最基本的模式：崩溃恢复和消息广播

##### 崩溃恢复：

1. 超过半数Follower和Leader数据同步，就退出崩溃恢复。
2. ZAB需要确保那些Leader在发出commit之前就崩溃的时候，还没提交的事务能够最终被提交。
3. ZAB需要确保丢弃那些只在Leader上被提出的事务。如Leader提出了一个事务之后就崩溃了，导致Follower都没收到这个事务，当Leader恢复后，这个事务需要被丢弃。

ZAB对崩溃恢复的实现：确保新选出来的Leader服务器拥有集群中所有机器最高编号（即ZXID最大）的事务proposal，那么可以保证新选出来的Leader一定具有所有已经提交的提案。且可以省去检查proposal的提交和丢弃工作这一步骤。



##### 消息广播：

1. 只有Leader可以消息广播，广播过程类似二阶段提交。
2. 不能解决Leader崩溃带来的数据不一致问题。


#### leader选举
##### 服务器启动时期的leader选举
（1）每个server发出一个投票，包含ZXID、myid等

（2）接收来自各个服务器的投票，并判断有效性

（3）处理投票

- 优先检查ZXID。ZXID大的当选leader
- ZXID相同，比较myid，myid大的当选leader

（4）统计投票，某个投票信息被超过半数的机器接受，则这个投票信息标识的机器当选leader

（5）改变服务状态，leader --> leading、follower --> following

##### 服务器运行时期的leader选举
（1）变更状态，follower --> looking

（2）每个server进行投票

（3）剩余步骤和启动时期的步骤相同




















