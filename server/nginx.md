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

## CentOs中使用源码安装nginx
  - 新建一个文件夹
  - [下载源码](http://nginx.org/en/download.html)
  ```shell
  #根据对应的版本进行修改
  wget http://nginx.org/download/nginx-1.8.0.tar.gz
  ```
  
  - nginx rtmp模块编译模板
    ```shell
    wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
    ```
  - 解压两个下载文件
    ```
    tar -zxvf nginx-1.8.0.tar.gz
  
    unzip master.zip
    ```
  - 添加rtmp编译模板到nginx
    进入到nginx** 目录下，执行以下命令
    ```shell
    ./configure  --add-module=../nginx-rtmp-module-master
    ```
    在这一步可能会包一些依赖库找不到的问题：error: the HTTP rewrite module requires the PCRE library.
    ```shell
    yum -y install pcre-devel
    ```
    error: the HTTP cache module requires md5 functions from OpenSSL library
    ```shell
    yum -y install openssl openssl-devel
    ```
    
  - 编译安装
    ```shell
    make
    make install
    ```
    在make时有可能报错：error this statement may fall through [-Werror=implicit-fallthrough=]
    ```shell
    make CFLAGS='-Wno-implicit-fallthrough'
    ```
    
  - 运行nginx
    ```shell
    nginx
    ```
    查看nginx配置是否正确
    ```shell
    nginx -t
    ```
  
    查看是否成功跑起
    ```shell
    ps -ef|grep nginx
    ```
  - 验证
  在浏览器中输入对应ip看能否显示nginx欢迎页
  注意：阿里云有端口安全组的限制，如果没有正确的配置，是无法访问的
  需要手动添加一个安全组规则，把对应的端口配到端口范围之中
  
