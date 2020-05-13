```shell

PLATFORM=armv7a
TARGET=$PLATFORM-linux-androideabi
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/windows-x86_64/bin
SYSROOT=$NDK/sysroot
PREFIX=`dirname $0`/Android/$PLATFORM
 
CFLAG="-D__ANDROID_API__=$API -U_FILE_OFFSET_BITS -DBIONIC_IOCTL_NO_SIGNEDNESS_OVERLOAD -Os -fPIC -DANDROID -D__thumb__ -mthumb -Wfatal-errors -Wno-deprecated -mfloat-abi=softfp -marm"
 
mkdir -p $TMPDIR
 
build_one()
{
./configure \
--ln_s="cp -rf" \
--prefix=$PREFIX \
--cc=$TOOLCHAIN/$TARGET$API-clang \
--cxx=$TOOLCHAIN/$TARGET$API-clang++ \
--ld=$TOOLCHAIN/$TARGET$API-clang \
--target-os=android \
--arch=$ARCH \
--cpu=$PLATFORM \
--cross-prefix=$TOOLCHAIN/$ARCH-linux-androideabi- \
--disable-asm \
--enable-cross-compile \
--enable-shared \
--disable-static \
--enable-runtime-cpudetect \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-doc \
--disable-symver \
--enable-small \
--enable-gpl --enable-nonfree --enable-version3 --disable-iconv \
--enable-jni \
--enable-mediacodec \
--disable-decoders --enable-decoder=vp9 --enable-decoder=h264 --enable-decoder=mpeg4 --enable-decoder=aac \
--disable-encoders --enable-encoder=vp9_vaapi --enable-encoder=h264_nvenc --enable-encoder=h264_v4l2m2m --enable-encoder=hevc_nvenc \
--disable-demuxers --enable-demuxer=rtsp --enable-demuxer=rtp --enable-demuxer=flv --enable-demuxer=h264 \
--disable-muxers --enable-muxer=rtsp --enable-muxer=rtp --enable-muxer=flv --enable-muxer=h264 \
--disable-parsers --enable-parser=mpeg4video --enable-parser=aac --enable-parser=h264 --enable-parser=vp9 \
--disable-protocols --enable-protocol=rtmp --enable-protocol=rtp --enable-protocol=tcp --enable-protocol=udp \
--disable-bsfs \
--disable-indevs --enable-indev=v4l2 \
--disable-outdevs \
--disable-filters \
--disable-postproc \
--extra-cflags="$CFLAG" \
--extra-ldflags="-marm"
}
 
build_one
 
make clean
 
make -j4
 
make install
```
