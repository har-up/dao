# Fmod开发
## 集成Fmod
- [fmod](https://www.fmod.com/download)相关代码
- 其中core/examples有一些示例项目，可以用AS直接打开，如果是在现有项目集成需要做如下配置
  - 把fmod.jar导入项目并依赖
  - 项目app下新建一个文件夹jniLibs,把fomd core中的so库拷贝到该目录
  - 拷贝实例项目中的Application.mk和Anroid.mk到app下，其中需要改一些内容：
  ```xml
  //Android.mk   
  LOCAL_PATH := $(call my-dir)  //该文件相对路径
  FMOD_API_ROOT := $(LOCAL_PATH)

  include $(CLEAR_VARS)

   LOCAL_MODULE            := fmodL //本地加载的库名
  LOCAL_SRC_FILES         := ${FMOD_API_ROOT}/jniLibs/$(TARGET_ARCH_ABI)/libfmod$(FMOD_LIB_SUFFIX).so //这里指定到拷贝到项目的so库 FMOD_LIB_SUFFIX是从build.gradle中传来
  LOCAL_EXPORT_C_INCLUDES := src/main/cpp/inc

  include $(PREBUILT_SHARED_LIBRARY)

  include $(CLEAR_VARS)

  LOCAL_MODULE            := fmod
  LOCAL_SRC_FILES         := ${FMOD_API_ROOT}/jniLibs/$(TARGET_ARCH_ABI)/libfmod.so
  LOCAL_EXPORT_C_INCLUDES := src/main/cpp/inc



  include $(PREBUILT_SHARED_LIBRARY)

  include $(CLEAR_VARS)

  LOCAL_MODULE            := example //本地c/c++文件编译后的so库名
  LOCAL_SRC_FILES         := src/main/cpp/common_platform.cpp src/main/cpp/common.cpp src/main/cpp/play_sound.cpp //需要编译的本地c/c++文件
  LOCAL_C_INCLUDES        := src/main/cpp
  LOCAL_SHARED_LIBRARIES  := fmod fmodL //指定依赖上两个so模块

  include $(BUILD_SHARED_LIBRARY)
  ```
  - 把对应的本地c/c++文件放到上边配置的src/main/cpp/下，其中还依赖一些头文件需要处理好
  - 在app/build.gradle文件中加构建配置：
  ```xml
  android{
     debug {
         buildConfigField("String[]", "FMOD_LIBS", "{ \"fmodL\" }")
         externalNativeBuild {
          ndkBuild {
            arguments 'FMOD_CONFIG=ReleaseWithLogging', 'FMOD_LIB_SUFFIX=L'
          }
         }
     }
     release {
        buildConfigField("String[]", "FMOD_LIBS", "{ \"fmod\" }")
        externalNativeBuild {
            ndkBuild {
                arguments 'FMOD_CONFIG=Release', 'FMOD_LIB_SUFFIX='
            }
        }
     }
  
    externalNativeBuild { 
        ndkBuild {
            path "Android.mk"
        }
    }
  }
  ```
  - 注意
    - 现在AS中有直接支持链接编译c,而不需要Android.mk,只需把jniLibs建在src/main 目录下，这种情况下会与Android.mk冲突。不过具体怎么用还需要探索
    - 新版本fmod中的jar包中的org.fmod.FMOD类新增了一个静态方法，如果导入的是旧版本的jar包就会报错
  
  
## 变声
- 正常的播放
```c++
Fmod:: System *system //Fmod系统对象
Fmod:: Channel *channel //一个通道
FMOD::Sound *sound;
bool playing = true;

//播放系统初始化
//32代表最大的播放通道
//第二个参数代表系统初始化模式
//第三个参数待认识
system -> init(32,FMOD_INIT_NORMAL,NULL)；

//创建声音
//1 播放的声音文件
//2 FMOD_MODE
//3 ..
//4 sound二级指针
system -> createSound(Common_MediaPath(str_path), FMOD_DEFAULT, 0, &sound)

//播放声音
system->playSound(sound,0,false,&channel);

//判断是否还在播放
channel.isPlaying(&playing);

//播放完毕或退出时释放资源
system.close();
system.release();

```
- 变声
Fmod中的变声主要是通过DSP
```c++
Fmod:: DSP *dsp = 0;
//创建一个DSP
system->createDSPByType(FMOD_DSP_TYPE_PITCHSHIFT, &dsp)；其中的type有很多中，用于实现不同的声音效果
//给该类型的dsp设定一些值
dsp->setParameterFloat(FMOD_DSP_PITCHSHIFT_PITCH,3.0);
//给指定一个通道添加音效，playSound后添加
channel.addDsp(0，dsp);

//更新
system -> update();

//播放完后释放资源
dsp.release();

```

 

