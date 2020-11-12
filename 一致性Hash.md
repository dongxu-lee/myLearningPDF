# 一致性Hash问题及解决方案学习笔记

## 1. Hash算法
### 1.1 直接寻址法
把数据和数组下标绑定在一起，查找的时候，直接array[n]

	优点：速度快，O(1)
	缺点：浪费空间

### 1.2 取模
取模容易产生hash冲突
#### 开放寻址法
向前或向后找空闲位置
#### 链地址法
在当前位置使用链表存储

## 2. Hash算法应用场景
### 2.1 请求的负载均衡
Nginx的ip_hash
#### 2.1.1 Nginx的ip_hash策略
把ip的前三段作为求hash的参数，也就是说如果请求的ip前三段一样（同一个局域网），请求会被转发到同一台服务器
### 2.2 分布式存储
Redis、Hadoop、ElasticSearch、MySQL分库分表

## 3.  代码实现一致性Hash算法
### 普通Hash算法
~~~java
package consistentHash;

/**
 * 普通hash算法实现
 */
public class GeneralHash {

    public static void main(String[] args) {
        // 定义客户端IP
        String[] clients = new String[]{"10.78.12.3", "113.25.63.1", "126.12.3.8"};

        // 定义服务器数量
        int serverCount = 3; // 编号对应0,1,2

        // hash(ip)%node_counts=index
        // 根据index锁定应该路由到的tomcay服务器
        for (String client : clients) {
            int hash = Math.abs(client.hashCode());
            int index = hash%serverCount;
            System.out.println("客户端： " + client + " 被路由到服务器编号为： " + index);
        }
    }
}
~~~
### 一致性Hash算法（不使用虚拟节点）
~~~java
package consistentHash;

import java.util.SortedMap;
import java.util.TreeMap;

/**
 * 一致性hash算法，不使用虚拟节点
 */
public class ConsistentHashNoVirtual {

    public static void main(String[] args) {
        // 一、服务器IP对应到hash环上
        String[] tomcatServers = new String[]{"123.111.3.1", "123.101.35.2", "111.20.35.2", "123.98.26.3"};

        // 使用有序map
        SortedMap<Integer, String> hashServerMap = new TreeMap<>();

        for (String tomcatServer : tomcatServers) {
            // 求服务器IP的hash
            int serverHash = Math.abs(tomcatServer.hashCode());
            // 存储hash和服务器ip的kv关系，存储后的key都是已经从小到大排好序的
            hashServerMap.put(serverHash, tomcatServer);
        }

        // 二、计算客户端IP的hash值
        String[] clients = new String[]{"10.78.12.3", "113.25.63.1", "126.12.3.8"};
        for (String client : clients) {
            int clientHash = Math.abs(client.hashCode());
            // 三、hash环顺时针找客户端放置的位置
            // tailMap(int x)的作用：返回所有的key值比x值大的kv集合
            // 返回serverHash比clientHash大的kv集合
            SortedMap<Integer, String> integerStringSortedMap = hashServerMap.tailMap(clientHash);
            if(integerStringSortedMap.isEmpty()) {
                // clientHash比最大的serverHash还大，取整个hash环的第一个服务器
                Integer firstkey = hashServerMap.firstKey();
                System.out.println("=======>>>>客户端： " + client + " 被路由到服务器： " + hashServerMap.get(firstkey));
            }else {
                // 取tailMap集合里的第一个
                Integer firstkey = integerStringSortedMap.firstKey();
                System.out.println("=======>>>>客户端： " + client + " 被路由到服务器： " + hashServerMap.get(firstkey));
            }

        }
    }
}
~~~
### 一致性Hash算法（使用虚拟节点）
~~~java
package consistentHash;

import java.util.SortedMap;
import java.util.TreeMap;

/**
 * 一致性hash算法，使用虚拟节点
 */
public class ConsistentHashWithVirtual {

    public static void main(String[] args) {
        // 一、服务器IP对应到hash环上
        String[] tomcatServers = new String[]{"123.111.3.1", "123.101.35.2", "111.20.35.2", "123.98.26.3"};

        // 使用有序map
        SortedMap<Integer, String> hashServerMap = new TreeMap<>();

        // 定义针对每个真实服务器虚拟出来几个节点
        int virtualCount = 3;

        for (String tomcatServer : tomcatServers) {
            // 求服务器IP的hash
            int serverHash = Math.abs(tomcatServer.hashCode());
            // 存储hash和服务器ip的kv关系，存储后的key都是已经从小到大排好序的
            hashServerMap.put(serverHash, tomcatServer);

            // 处理虚拟节点
            for (int i = 0; i < virtualCount; i++) {
                int virtualHash = Math.abs((tomcatServer + "#" + i).hashCode());
                hashServerMap.put(virtualHash, "---由虚拟节点" + i + "映射过来的请求: " + tomcatServer);
            }

        }

        // 二、计算客户端IP的hash值
        String[] clients = new String[]{"10.78.12.3", "113.25.63.1", "126.12.3.8"};
        for (String client : clients) {
            int clientHash = Math.abs(client.hashCode());
            // 三、hash环顺时针找客户端放置的位置
            // tailMap(int x)的作用：返回所有的key值比x值大的kv集合
            // 返回serverHash比clientHash大的kv集合
            SortedMap<Integer, String> integerStringSortedMap = hashServerMap.tailMap(clientHash);
            if(integerStringSortedMap.isEmpty()) {
                // clientHash比最大的serverHash还大，取整个hash环的第一个服务器
                Integer firstkey = hashServerMap.firstKey();
                System.out.println("=======>>>>客户端： " + client + " 被路由到服务器： " + hashServerMap.get(firstkey));
            }else {
                // 取tailMap集合里的第一个
                Integer firstkey = integerStringSortedMap.firstKey();
                System.out.println("=======>>>>客户端： " + client + " 被路由到服务器： " + hashServerMap.get(firstkey));
            }
        }
    }
}
~~~

## 4. Nginx实现一致性Hash策略



