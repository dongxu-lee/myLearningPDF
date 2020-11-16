# 分布式理论、架构设计学习笔记
## 第一部分：基础理论及一致性算法
### 1.1 一致性概念
- 强一致性
- 弱一致性
- 最终一致性

### 1.2 CAP
 - C一致性：数据副本一致
 - A可用性：系统一直可用
 - P分区容错性：遭遇故障时仍能提供服务

三个中只能同时满足两个

### 1.3 BASE理论
- Basically Avaliable基本可用
- Soft state软状态
- Eventually consistent最终一致性

Base理论是对CAP中一致性和可用性权衡的结果

核心思想：放弃强一致性，采用适当方式实现最终一致性

### 1.4 一致性协议：2PC（两阶段提交）
- 阶段一：提交事务请求<B>Prepare</B>
1. 事务询问，协调者询问参与者是否可以执行事务提交操作
2. 执行事务：参与者看是执行事务，并将Redo和Undo信息记入事务日志
3. 参与者向协调者反馈事务询问的响应：Yes or No

- 阶段二：执行事务提交<B>Commit</B>
根据阶段一的参与者的响应结果决定事务提交或中止

<B>优点</B>
&emsp;原理简单，实现方便

<B>缺点</B>
&emsp;同步阻塞、单点问题、数据不一致、过于保守

- 同步阻塞：各阶段参与者都要等待其他参与者的响应
- 单点问题：协调者出问题会使参与者都堵塞
- 数据不一致：只有部分参与者收到了commit请求，会导致数据不一致
- 过于保守：没有容错机制


### 1.5 一致性协议：3PC（三阶段提交）
&emsp;2PC的改进版，把2PC中阶段一的提交事物请求一分为二，变为CanCommit、PreCommit、doCommit三阶段


&emsp;<B>优点</B>
相比2PC，降低了参与者的阻塞范围（第一阶段不阻塞），其次在单点故障后继续达成一致（2PC在提交阶段会出现此问题，3PC会根据协调者状态进行回滚或提交）

&emsp;<B>缺点</B>
如果参与者收到preCommit后出现网络分区，协调者和参与者无法通信，参与者等待超时后，会自动提交事务，这会导致数据不一致


### 1.6 一致性算法：Paxos算法
#### Paxos解决了什么问题？

&emsp;在分布式系统中，总会出现机器宕机或网络异常（包括消息延迟、对视、重复、乱序，网络分区等）等情况，Paxos算法就是解决如何在一个可能发生上述问题的分布式系统中，快速且正确的在集群内部对<B>某个数据的值</B>达成一致，并且保证无论发生任何异常，都不会破坏整个系统的一致性

### 1.7 一致性算法：Raft算法
心跳机制

日志复制

## 第二部分：分布式系统设计策略
### 2.1 心跳检测
固定频率向其他节点发送当前节点状态
### 2.2 高可用设计
主备、互备、集群
### 2.3 容错性

### 2.4 负载均衡

## 第三部分：分布式架构网络通信及自定义RPC

### 3.1 RPC
remote procedure call 远程过程调用

#### RPC架构
一个完整的RPC包含四个核心组件

- 客户端Client，服务的调用方
- 客户端存根Client Stub，存放服务端的地址信息，再将客户端的请求参数打包成网络消息，然后通过网络远程发送给服务方
- 服务端Server，服务提供者
- 服务端存根Server Stub，接收客户端发送过来的消息，将消息解包，并调用本地方法

在java中RPC框架很多，RMI、Hessian、Dubbo等


### 3.2 RMI
remote method invocation远程方法调用，是java原生支持的远程调用

#### 案例步骤

1. 创建远程接口，并且继承java.rmi.Remote接口
2. 实现远程接口，并且继承UnicastRemoteObject
3. 创建服务器程序：createRegistry()方法注册远程对象
4. 创建客户端程序（获取注册信息，调用接口方法）

#### 代码实现
服务端：

实体类User
~~~java
package com.ldx.pojo;

import java.io.Serializable;

/**
 * 引用对象应该实现序列化接口，这样才能在网络中传输
 */
public class User implements Serializable {

    private String name;
    private int age;

	// getter and setter
	// toString
}
~~~
service接口
~~~java
package com.ldx.demo;

import com.ldx.pojo.User;

import java.rmi.Remote;
import java.rmi.RemoteException;

/**
 * 远程服务对象接口必须要集成Remote接口，
 * 同时方法必须抛出RemoteException
 */
public interface IHelloService extends Remote {

    public String sayHello(User user) throws RemoteException;

}
~~~
service实现类
~~~java
package com.ldx.demo;

import com.ldx.pojo.User;

import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

/**
 * 远程服务实现类：必须继承UnicastRemoteObject
 */
public class HelloServiceImpl extends UnicastRemoteObject implements IHelloService {

    public HelloServiceImpl() throws RemoteException {
    }

    public String sayHello(User user) throws RemoteException {
        System.out.println("this is server, hello:" + user.getName());
        return "success";
    }
}
~~~
服务端main程序
~~~java
package com.ldx.server;

import com.ldx.demo.HelloServiceImpl;
import com.ldx.demo.IHelloService;

import java.net.MalformedURLException;
import java.rmi.AlreadyBoundException;
import java.rmi.Naming;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;

public class RMIServer {

    public static void main(String[] args) throws RemoteException, AlreadyBoundException, MalformedURLException {

        // 1.创建一个远程对象
        IHelloService helloService = new HelloServiceImpl();

        // 2.启动注册服务；创建了远程对象注册表Registry的实例，并指定端口为8888
        LocateRegistry.createRegistry(8888);

        //3.真正注册：绑定的URL的标准格式：rmi://host:port/name rmi可以省略
        Naming.bind("//127.0.0.1:8888/zm", helloService);
    }
}
~~~

客户端：

实体类User
~~~java
package com.ldx.pojo;

import java.io.Serializable;

/**
 * 引用对象应该实现序列化接口，这样才能在网络中传输
 */
public class User implements Serializable {

    private String name;
    private int age;

	// getter and setter
	// toString
}
~~~
service接口
~~~java
package com.ldx.demo;

import com.ldx.pojo.User;

import java.rmi.Remote;
import java.rmi.RemoteException;

/**
 * 远程服务对象接口必须要集成Remote接口，
 * 同时方法必须抛出RemoteException
 */
public interface IHelloService extends Remote {

    public String sayHello(User user) throws RemoteException;

}
~~~
客户端main程序
~~~java
package com.ldx.client;

import com.ldx.demo.IHelloService;
import com.ldx.pojo.User;

import java.net.MalformedURLException;
import java.rmi.Naming;
import java.rmi.NotBoundException;
import java.rmi.Remote;
import java.rmi.RemoteException;

public class RMIClient {

    public static void main(String[] args) throws RemoteException, NotBoundException, MalformedURLException {
        // 在RMI服务注册表中查找名称为zm的对象
        IHelloService helloService = (IHelloService) Naming.lookup("//127.0.0.1:8888/zm");

        // 调用方法
        User user = new User();
        user.setName("字母");
        user.setAge(13);
        System.out.println(helloService.sayHello(user));
    }
}
~~~

#### 实现原理
客户端对象 ---> 客户端stub（代理） ---> 服务端stub（代理） ---> 服务端对象


### 3.3 BIO、NIO、AIO
#### 同步和异步
同步：java自己处理IO的读写

异步：java将IO委托给OS处理

#### 阻塞和非阻塞
阻塞：调用结果返回之前，当前线程挂起。线程挂起，不会分配CPU资源，但挂起的线程优先级仍然比其他线程高，所以是阻塞的

非阻塞：在不能立刻得到结果之前，该调用不会阻塞当前线程

#### BIO同步阻塞IO
一个连接一个线程

#### NIO同步非阻塞IO
一个请求一个线程，客户端发送的连接请求会注册到多路复用器上，多路复用轮询到连接有IO请求时，才会启动线程处理

#### AIO异步非阻塞IO
一个有效请求一个线程，客户端的IO请求都是OS先处理完了再通知服务器启动线程进行处理



### 3.4 NIO实现代码
服务端：
~~~java
package netty;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

public class NettyServer {

    public static void main(String[] args) throws InterruptedException {

        //1.创建NioEventLoopGroup的两个实例对象
        //它们就是两个线程池，磨人的线程数就是CPU核心数*2
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        //2.创建服务启动辅助类：装配一些组件的
        ServerBootstrap bootstrap = new ServerBootstrap();
        bootstrap.group(bossGroup, workerGroup)
                //指定服务器端监听套接字通道
                .channel(NioServerSocketChannel.class)
                //设置业务职责链
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ChannelPipeline pipeline = ch.pipeline();
                        //将一个一个的channelHandler添加到责任链上
                        //在请求进来和响应出去的时候，都需要channelHandler处理
                        pipeline.addLast(new StringEncoder());
                        pipeline.addLast(new StringDecoder());
                        pipeline.addLast(new SimpleChannelInboundHandler<String>() {
                            @Override
                            protected void channelRead0(ChannelHandlerContext channelHandlerContext, String s) throws Exception {
                                System.out.println(s);
                            }
                        });
                    }
                });

        // 监听端口
        // sync：用于阻塞当前的thread，一直到端口绑定操作完成
        ChannelFuture f = bootstrap.bind(8123).sync();
        System.out.println("tcp server start success...");
        // 应用程序会阻塞等待直到服务器的channel关闭
        f.channel().closeFuture().sync();
    }

}

~~~


客户端：
~~~java
package netty;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;

import java.util.Date;

public class NettyClient {

    public static void main(String[] args) throws InterruptedException {

        //1.创建NioEventLoopGroup的实例对象
        NioEventLoopGroup group = new NioEventLoopGroup();

        // 2.创建bootstrap对象
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel channel) throws Exception {
                        ChannelPipeline pipeline = channel.pipeline();
                        pipeline.addLast(new StringEncoder());
                    }
                });

        Channel channel = bootstrap.connect("127.0.0.1", 8123).channel();

        while (true) {
            channel.writeAndFlush(new Date() + ": hello world");
            Thread.sleep(2000);
        }


    }
}

~~~

### 3.5 基于Netty自定义RPC






