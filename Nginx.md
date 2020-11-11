# Nginx学习笔记

### 代码及文档地址 

文档地址：
1. github文档地址：[https://github.com/dongxu-lee/myLearningPDF/blob/main/Nginx.md](https://github.com/dongxu-lee/myLearningPDF/blob/main/Nginx.md)
2. 语雀文档地址：[https://www.yuque.com/bailihang-3fszp/bkgbrq/kd4nip](https://www.yuque.com/bailihang-3fszp/bkgbrq/kd4nip)


## 1. 基础回顾
Nginx到底是什么？

	Nginx是一个高性能的HTTP和翻箱底阿里web服务器，占用内存少，并发强

Nginx能做什么？

	Http服务器
	反向代理服务器
	动静分离

Nginx特点？

	跨平台
	简单
	高并发
	稳定性强


## 2. 核心配置文件
全局块、events块、http块

## 3. 反向代理
~~~xml
proxy_pass 127.0.0.1:8080
~~~

## 4. 负载均衡
- 轮询

~~~
upstream myserver {
	server 127.0.0.1:8080;
	server 127.0.0.1:8081;
}

location /abc {
	proxy_pass http://myserver/;
}
~~~

 - weight

~~~
upstream myserver {
	server 127.0.0.1:8080 weight=1;
	server 127.0.0.1:8081 weight=2;
}
~~~

- ip_hash

~~~
upstream myserver {
	ip_hash;
	server 127.0.0.1:8080 weight=1;
	server 127.0.0.1:8081 weight=2;
}
~~~

## 5. 动静分离
~~~
location /static/ {
	root staticData;
}
~~~

























