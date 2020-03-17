# Fmod开发
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
  
  
  
