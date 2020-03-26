# 在Linux中编译FFmpeg
- 使用xshell/xftp连接Linux操作系统

- 准备好ndk到Linux系统中，解压
  解压命令:直接./压缩包

- 配置环境变量
  在profile或profile.d中添加ndk的环境变量，不同的linux版本配置可能不同

- 准备好FFmpeg源码并解压
  把FFmpeg源码放到一个目录下然后解压：./

- 编译脚本
  编写shell编译脚本

- 编译
  可能有些FFmpeg下的文件有权限读所以会报找不到的错误，需要给目录下的所有文件赋予权限 chmod 777 -R ./
