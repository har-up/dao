## 抓包工具
- charles  付费软件 
- fiddler 比较老牌的抓包工具，界面稍显简陋
- AnyProxy  阿里巴巴的产品
- whistle 基于Node.js的全能抓包工具

## Whistle
- 安装和更新
基于Node.js,在node.js环境下直接安装
```
npm install -g whistle
```
- 安装Whistle的Chrome插件-avwo/whistle-for-chrome
whistle 要支持https的话需要在pc上下载证书，然后根据文档中的配置进行安装

## 查看request和response
任何抓包工具都需要手机配置代理，只有连接代理才能在电脑上看到所有的请求信息

- 过滤
settings -> filter

- 替换域名

- 修改请求参数

- 修改返回值 
  - 可以用本地file来mock
  - 修改响应码
  - 使用values来mock
  - 替换返回的字段
  - 安装mock plugin
  -
- 模拟低网速的情形

- 查看webview的console 
