# nginx
nginx是一个免费开源的http服务器，邮件代理服务器，通用的支持负载均衡的tcp/udp代理服务器   
[niginx安装](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#stable_vs_mainline)

## 反向代理和正向代理
- 正向代理
  内部网络用户向internet网络中的服务器发起请求，用户请求的代理就叫做正向代理服务器。正向代理隐藏了真实客户端信息。
- 反向代理
  外部网络（internet网络)用户向内部网络服务器发起请求，这个服务器的代理就是反向代理服务器。 
  反向代理隐藏了真实服务器的信息。
  
  
## 基本功能
- 可以在nginx.conf文件中配置一些参数，通过重启或发送信号的方式去加载配置


## 配置负载均衡
  - 可配置http tcp/udp的负载均衡
  - 检测http/tcp/udp连接健康状态
  
## 响应缓存

## 安全控制

## 监听

## 高可用

## 动态模块
