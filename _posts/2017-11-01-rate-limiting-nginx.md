---
layout: post
title: Nginx限流模块limit_req_zone
author: magic
date:   2017-11-01
categories: Nginx
tags: nginx limit_req
permalink: /archivers/rate-limiting-nginx
---
* 目录
{:toc}

原文地址：[Rate Limiting with NGINX and NGINX Plus](https://www.nginx.com/blog/rate-limiting-nginx/)  

模块文档：[Module ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)  

## 前言  

NGINX最有用但经常被误解和配置错误的特征之一就是速率限制。 它允许您限制用户在给定时间段内可以执行的HTTP请求数量。 请求可以像访问网站的首页的GET请求一样简单，也可以是登录表单上的POST请求。
<!--more-->
速率限制可以用于安全目的，例如减慢暴力密码猜测攻击。 它可以通过将传入请求速率限制为真实用户的典型值，并且（通过日志记录）来识别目标URL，可以帮助防止DDoS攻击。 更一般地说，它用于保护上游应用服务器免受同时太多用户请求的淹没。

在本博客中，我们将介绍使用NGINX进行速率限制的基础知识以及更高级的配置。 速率限制在NGINX Plus中的工作方式相同。

要了解有关NGINX速率限制的更多信息，请观看我们的 [网络需求讲座](https://www.nginx.com/resources/webinars/rate-limiting-nginx/)。
  
## NGINX速率限制的工作原理

NGINX速率限制使用泄漏桶算法，其在电信和分组交换计算机网络中广泛使用以在带宽受限时处理突发性。类比是一个桶，水倒在顶部，从底部泄漏;如果浇水速度超过其泄漏速率，则桶溢出。在请求处理方面，水表示来自客户端的请求，桶表示根据先进先出（FIFO）调度算法请求等待处理的队列。泄漏的水代表离开缓冲区的请求，由服务器进行处理，溢出表示被丢弃和从不服务的请求。
![image](https://cdn-1.wp.nginx.com/wp-content/uploads/2017/06/leaky-bucket-featured-image-300x180.jpg)
  
## 配置基本速率限制

速率限制配置有两个主要的指令：limit_req_zone和limit_req，如下例所示：
```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

server {  
    location /login/ {  
        limit_req zone=mylimit;  
        proxy_pass http://my_upstream;  
    }  
}
```
[limit_req_zone](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html?&_ga=2.164848063.1021539605.1509515973-1887652234.1507531106#limit_req_zone)指令定义了速率限制的参数，而[limit_req](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html?&_ga=2.117465225.1021539605.1509515973-1887652234.1507531106#limit_req)在指定出现的上下文中启用速率限制（在该示例中，是对于/login/的所有请求）。

limit_req_zone指令通常在http块中定义，使其可在多个上下文中使用。它需要以下三个参数：

+ key——定义应用限制的请求特性。在该示例中，它是NGINX变量$binary_remote_addr，它是保存客户端IP地址的二进制表示形式。这意味着我们将每个唯一IP地址限制为由第三个参数定义的请求速率。 （我们使用这个变量，因为它占用比客户端IP地址$remote_addr的字符串表示少的空间。
  
+ zone——定义用于存储每个IP地址状态的共享内存区域以及访问请求限制URL的频率。将信息保存在共享内存中意味着可以在NGINX工作进程之间共享。该定义有两个部分：区域名称由zone=keyword标识，以及冒号后面的内存大小。大约16000个IP地址的状态信息需要1M内存，所以我们的区域可以存储大约16万个IP地址。

如果NGINX需要添加新条目时存储空间不足，则删除最旧的条目。如果释放的空间仍然不足以容纳新记录，则NGINX返回状态码503（服务临时不可用）。另外，为了防止内存耗尽，每次NGINX创建一个新的条目时，最多可以删除在前60秒内没有使用的两个条目。
  
+ rate——设置最大请求速率。在该示例中，速率不能超过每秒10个请求。NGINX实际上以毫秒的粒度跟踪请求，所以这个限制对应于每100毫秒1个请求。因为我们不允许突发（见下一节），这意味着如果在先前允许的时间到达不到100毫秒，则拒绝该请求。  

limit_req_zone指令设置速率限制和共享内存区域的参数，但实际上并不限制请求速率。  
为此，您需要通过在那里添加一个limit_req指令来将限制应用于特定位置或服务器块。  
在这个例子中，我们是对/login/的限制了请求速率。  
因此，现在每个唯一的IP地址访问/login/请求速率限制在每秒10个请求 - 更准确地说，是每100ms一个请求。
  
## 处理突发请求
  
如果我们在100毫秒内得到2个请求怎么办？ 对于第二个请求，NGINX向客户端返回状态码503。 这可能不是我们想要的，因为应用程序本质上是突发的。 相反，我们希望缓冲任何超量的请求并及时为他们提供服务。这是我们将burst参数用于limit_req的位置，如下面更新的配置中所示：
 ```
location /login/ {  
    limit_req zone=mylimit burst=20;  
    proxy_pass http://my_upstream;  
}  
```
burst参数定义了在超出zone指定的速率后，客户端可以保留多少请求。   （这里的zone是mylimit，速率限制为每秒10个请求，或每100毫秒1个请求）  在上一个请求之后的100ms内到达的请求将被放入队列中，这里我们将队列大小设置为20。  

这意味着如果21个请求同时从给定的IP地址到达，则NGINX会立即将第一个请求转发到上游服务器组，并将剩余的20个队列放入队列中。然后它每100毫秒转发一个排队的请求，只有当一个传入的请求使队列请求的数量超过20时，才返回503。  

## 无延迟队列  

burst的配置可以使请求处理很平滑，但是不太实用，因为它可能使您的站点看起来很慢。   在我们的示例中，队列中的第20个数据包需要等待2秒钟才会被转发，此时对客户端的响应可能不再有用。   要解决这种情况，请添加nodelay参数以及burst参数：  
```
location /login/ {  
    limit_req zone=mylimit burst=20 nodelay;  
    proxy_pass http://my_upstream;  
}  
```
使用nodelay参数，NGINX仍然根据burst参数分配队列中的时隙，并强制配置速率限制，但不排除排队请求的转发。 相反，当请求到达“太早”时，只要在队列中有可用的插槽，NGINX将立即转发。 它将该插槽标记为“已占用”，并且不会将其释放以供另一个请求使用，直到适当的时间过去（在我们的示例中，在100毫秒之后）。  

假设如前面所述，20个插槽队列是空的，21个请求从给定的IP地址同时到达。 NGINX立即转发所有21个请求，并在队列中标记20个插槽，然后每100毫秒释放1个插槽。 （如果有25个请求，NGINX将立即转发其中21个，标记20个插槽，并拒绝4个状态为503的请求）。  

现在假设在第一组请求被转发之后的101ms，另外20个请求同时到达。 而此时队列中只有1个插槽已被释放，所以NGINX转发1个请求，并拒绝其他19个，状态为503。如果在20个新请求到达之前已经过去了501毫秒，则5个插槽是空的，所以NGINX立即转发5个请求，并拒绝其他15个。  

这样效果相当于每秒10个请求的速率。 如果您想要限制请求速率，而不限制请求之间允许的间距，则nodelay选项将非常有用。  

**Note: 对于大多数的部署，我们建议将burst和nodelay参数包含在limit_req指令中。**   


## 高级配置示例  

通过将基本速率限制与其他NGINX功能相结合，您可以实现更细微的流量限制。  

### 白名单

此示例显示如何对不在“白名单”的任何人的请求强制使用率限制。  
```
geo $limit {
    default 1;
    10.0.0.0/8 0;
    192.168.0.0/24 0;
}

map $limit $limit_key {
    0 "";
    1 $binary_remote_addr;
}

limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;

server {
    location / {
        limit_req zone=req_zone burst=10 nodelay;

        # ...
    }
}
```

此示例使用[geo](http://nginx.org/en/docs/http/ngx_http_geo_module.html?&_ga=2.169034913.1021539605.1509515973-1887652234.1507531106#geo)和[map](http://nginx.org/en/docs/http/ngx_http_map_module.html?&_ga=2.42137317.1931673330.1509515938-1887652234.1507531106#map)指令。 geo块为白名单中的IP地址分配0到$limit，对所有其他IP地址分配1。 然后，我们使用地图将这些值转换成一个键，像这样：

+ If $limit is 0, $limit_key is set to the empty string
+ If $limit is 1, $limit_key is set to the client’s IP address in binary format

将两个配置合起来，白名单IP地址对应的$limit_key为空字符串，其他IP地址为其他值。 当limit_req_zone字典（对应的key）的第一个参数为空字符串时，限流不会应用，因此白名单IP地址（10.0.0.0/8和192.168.0.0/24子网）不受限制。同时其他IP地址限制每秒5个请求。  

limit_req指令对location "/" 应用限制，配置限制其允许突发最多10个请求，转发不延迟。  

### 同一个location包含多个limit_req配置  

您可以在单个location包含多个limit_req配置。所有请求将受到这些限制的作用，这意味着使用最严格的限制。  
例如，如果多个指令施加延迟，则使用最长的延迟。类似地，如果这是任何配置的作用，即使其他指令允许它们通过，请求也被拒绝。  

扩展前面的例子，我们可以对白名单上的IP地址应用速率限制：
```
http {
    # ...

    limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;
    limit_req_zone $binary_remote_addr zone=req_zone_wl:10m rate=15r/s;

    server {
        # ...
        location / {
            limit_req zone=req_zone burst=10 nodelay;
            limit_req zone=req_zone_wl burst=20 nodelay;
            # ...
        }
    }
}
```

白名单上的IP地址与第一个速率限制（req_zone）不匹配，但匹配第二个（req_zone_wl），因此受到每秒15个请求的限制。不在白名单上的IP地址与两个速率限制匹配，因此受到更多的限制：极每秒最多5个请求。  

## 配置相关选项  

### 日志级别

默认情况下，NGINX记录由于速率限制而延迟或丢弃的请求，如本例所示：
> 2015/06/13 04:20:00 [error] 120315#0: *32086 limiting requests, excess: 1.000 by zone "mylimit", client: 192.168.1.2, server: nginx.com, request: "GET / HTTP/1.0", host: "nginx.com"

日志条目中的字段包括::

+ limiting requests ——  Indicator that the log entry records a rate limit(显示此请求日志条目受到了速率限制)
+ excess —— Number of requests per millisecond over the configured rate that this request represents(此速率限制每100ms的最大请求数)
+ zone —— Zone that defines the imposed rate limit(此速率限制的zone名称)
+ client —— IP address of the client making the request(客户端IP)
+ server —— IP address or hostname of the server(服务器的主机名或IP地址)
+ request —— Actual HTTP request made by the client(HTTP协议版本)
+ host —— Value of the Host HTTP header(HTTP协议里面的host)


默认情况下，NGINX在错误级别记录拒绝请求，如上例中的[error]所示。 （它将延迟请求记录在一个级别以下，默认情况下为info。）要更改日志记录级别，请使用limit_req_log_level指令。 在这里我们设置拒绝请求登录在警告级别：  
```
location /login/ {
    limit_req zone=mylimit burst=20 nodelay;
    limit_req_log_level warn;

    proxy_pass http://my_upstream;
}
```

### 发送给客户端的错误代码

默认情况下，当客户端超过其速率限制时，NGINX响应状态码503（服务暂时不可用）。 使用limit_req_status指令设置不同的状态代码（在本示例中为444）：  
```
location /login/ {
    limit_req zone=mylimit burst=20 nodelay;
    limit_req_status 444;
}
```

### 拒绝特定location的所有请求

如果您要拒绝所有特定网址的请求，而不是限制它们，请在location块配置，包含deny all指令：
```
location /foo.php {
    deny all;
}
```

## 总结

我们已经涵盖了NGINX和NGINX Plus提供的许多速率限制功能，包括为HTTP请求设置不同位置的请求率，以及配置其他功能来限制速率，如突发和节点参数。 我们还涵盖了对白名单和黑名单客户端IP地址应用不同限制的高级配置，并解释了如何记录拒绝和延迟的请求。  

想自己在NGINX Plus尝试配置速率限制——可以今天就开始学习[30天免费培训](https://www.nginx.com/blog/rate-limiting-nginx/#free-trial)或者[联系我们](https://www.nginx.com/blog/rate-limiting-nginx/#contact-us)做个示例。