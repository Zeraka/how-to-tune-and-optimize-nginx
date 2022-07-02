---


---
# How to tune and optimize performance of NGINX/network-workload

There are three aspects for tuning nginx:

```
1. kernel parameters
2. nginx configuration
3. caching and compression
```

## Kernel parameters


Commonly, we can modify some formal parameters for tuning nginx. Refer to [nginx.com-tuning-nginx](https://www.nginx.com/blog/tuning-nginx/)

what is somaxconn?

Each socket in status of LISTEN, has its own listening queue, the length of the queue is **related to** the following two aspects: somaxconn and listen().

somaxconn parameter defines the maximum listening queue length of each port in the system, which is a global parameter and its default value is 128.

The default value is too small, we normally modify it to 1024 at least.

```
echo 1024 > /proc/sys/net/core/somaxconn
```

backlog

what is backlog?

backlog is a parameter for limiting socket connection numbers.

Shake hands three times.

A server has two types of status, SYN_RCVD and ESTABLISHED.

If a client wants to established a connection with a server, the client will sent a package that includes key information like SYN=1,ACK=0. Then the server receives a package whose SYN=1, ACK=1, which the package is also called SYN+ACK package. At this time the connection is not established so it is put in the half-connection queue. Half-connection queue is also called SYN queue, in which the connections are all still not established although the server has sent SYN+ACK. If the server received ACK from client again, the connection will be established and then the connection will be moved to full-connection queue also called ACCEPT queue. 

**backlog is the max number of established connections.**

Why shake hands three times?

Prevent time-out SYN package from being received by the server. Commonly, two times is enough,  but it may lead to bug. If a client sends a SYN package but the package is timed out, the client sends a SYN package again and the server establishes a connection with the client.

After the connection ends, the timed-out SYN package arrives at the server, so the connection will be established again. But at this time the client will not send any packages to the server, which is a waste of resources. If using three times hand-shake, the server will send a SYN+ACK package to the client, if the client indeed want to connect then it will return a ACK package to the server. Three times hand-shake.


[backlog是什么](https://blog.csdn.net/qq_45691748/article/details/111343509?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-111343509-blog-78369343.pc_relevant_multi_platform_whitelistv1&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

[TCP参数之backlog](https://blog.csdn.net/u010597819/article/details/108687671?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108687671-blog-78369343.pc_relevant_multi_platform_whitelistv1&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108687671-blog-78369343.pc_relevant_multi_platform_whitelistv1&utm_relevant_index=1)

backlog provided

How about status of socket?

TCP_CLOSE

How about status of TCP?

half-connection and full-connection

What is a TCP queue?

SYN Package?

What is SYN Package?

[详解linux中的backlog](https://blog.csdn.net/daocaokafei/article/details/115336575)

https://github.com/denji/nginx-tuning

[TCP之三：TCP/IP协议中backlog参数(队列参数)](https://www.cnblogs.com/duanxz/p/5231673.html)

 Important :  [TCP 的backlog详解及半连接队列和全连接队列](https://blog.csdn.net/u010039418/article/details/78369343)
