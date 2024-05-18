# JNI
开发步骤如下
## 安装NDK
打开Android Studio的SDK-Manager -> SDK-Tools -> 勾选NDK -> OK


## 编写native方法
在java类中编写一个native方法，完成后在类的com包名目录目录下通过javah生成头文件
例如Test.java
```shell
javah com.demo.test.Test
```

## 新建cpp目录
在src.main目录下新建一个cpp目录，用于存放c/c++代码。
将头文件拷贝到该目录下，同时新建一个.cpp文件，引用头文件，然后编写代码实现头文件中的函数。
> 注意：如果.cpp文件内容拷贝头文件，记得cpp文件中得删除如下代码：
    ```c++
    #ifndef {filename}
    #define {filename}
    ```

## 新建cmake文件
在cpp目录下新建一个CMakeLists.txt文件，一般的内容如下
```txt
cmake_minimum_required(VERSION 3.18.1)

#工程名
project("xx")

#设置so库输出目录
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/../jniLibs/${ANDROID_ABI})

add_library( # Sets the name of the library.
        xx

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        xx.cpp)

target_link_libraries( # Specifies the target library.
        xx

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})        
```

## build 构建
在build.gradle文件中添加构建任务
```gradle

android {

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }

    externalNativeBuild {
        cmake {
            path file('src/main/cpp/CMakeLists.txt')
            version '3.18.1'
        }
    }
}
```

点击构建，构建完毕会在对应目录生成so库文件


## java文件中加载so库
在java静态代码块中加入对so库的加载代码，把前缀lib去掉，如下：
```java
    //静态代码块
    static {
        System.loadLibrary("xx");
    }
```
完成后就可以运行项目看是否能正常调用native代码了

## 遇到的问题
* 构建so库报错
如果遇到如下问题,是cmake版本的问题，可以降低版本试试。
```shell
com.android.build.gradle.external.cmake.server.ServerProtocolV1.readExpected(ServerProtocolV1.java:519)
```


