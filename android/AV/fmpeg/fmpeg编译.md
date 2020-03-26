# 在Linux中编译FFmpeg
- 使用xshell/xftp连接Linux操作系统

- 准备好ndk到Linux系统中，解压
  解压命令:直接./压缩包

- 配置环境变量
  在profile或profile.d中添加ndk的环境变量，不同的linux版本配置可能不同

- 准备好FFmpeg源码并解压
  把FFmpeg源码放到一个目录下然后解压：./

- 编写shell编译脚本
```xml
#!/bin/bash
make clean
export NDK=/usr/ndk/android-ndk-r10e
export SYSROOT=$NDK/platforms/android-9/arch-arm/
export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64
export CPU=arm
export PREFIX=$(pwd)/android/$CPU
export ADDI_CFLAGS="-marm"

./configure --target-os=linux \
--prefix=$PREFIX --arch=arm \
--disable-doc \
--enable-shared \
--disable-static \
--disable-yasm \
--disable-symver \
--enable-gpl \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-doc \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install

```

- 编译
  可能有些FFmpeg下的文件有权限读所以会报找不到的错误，需要给目录下的所有文件赋予权限 chmod 777 -R ./
