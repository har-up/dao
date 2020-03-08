# JNI
## JNI开发流程
- java工程中编写Native方法
- 使用javah 生成对应native类文件的.h头文件
- 把生成的头文件复制到c/c++工程中，导入生成的.h头文件所需的一些jdk中的头文件
- 实现生成头文件中声明的函数
- 生成动态库文件.dll文件
- 配置动态库文件路径到系统环境变量
- java工程加载动态库

## 动态库和静态库
- 动态库后缀名为.dll,静态库为.a
- 动态库不被包含在执行程序.exe中，静态库会被编译放到执行程序.exe中


## JNIENV
JNIENV在c与在c++中是不同的
在c中是结构体指针，
在c++中是结构体的别名，该结构体就是封装调用的c中结构体指针结构体的函数。


## JNI调用访问JAVA
- 获取属性
```c
JNIEXPORT jstring JNICALL Java_com_test_getFiled(JNIEnv *env, jobject jobject){
    //获取类
    jclass cls = (*env) -> GetObjectClass(env,jobject);
    //获取属性ID
    jfieldID  id = (*env) -> GetFieldID(env,cls,"key","Ljava/lang/String;");
    //通过属性ID获取对应属性
    jstring jstr = (*env) -> GetObjectField(env,jobject,id);
    //获取到属性值后需要转换成对应的c数据类型
    char *str = (*env) -> GetStringUTFChars(env,jstr,JNI_FALSE);   
    //数据进行处理后  如果需要返回时还需要把c数据类型转成jni类型进行返回
    ...
}
```

- 获取静态属性
```c
  JNIEXPORT void JNICALL Java_com_test_getStaticFiled
        (JNIEnv *env, jclass cls){
    jfieldID  id = (*env) -> GetStaticFieldID(env,cls,"key","Ljava/lang/String;");
    jstring jstr = (*env) -> GetStaticObjectField(env,cls,id);
}
```

- 获取方法及调用
```c
JNIEXPORT void JNICALL Java_com_test_getMethod
        (JNIEnv *env, jobject jobj){
    jclass cls = (*env) -> GetObjectClass(env,jobj);
    //获取方法ID,其中的最后一个参数签名  可以通过“javap -s -p java文件全路径”的方式来获取
    jmethodID  methodId =(*env) -> GetMethodID(env,cls,"getInt","(I)I");
    jint value = (*env) -> CallIntMethod(env,jobj,methodId);
}
```

- 获取静态方法及调用
```c
//函数实现
JNIEXPORT void JNICALL Java_com_test_getStaticMethod
        (JNIEnv *env, jclass cls){
    jmethodID  methodId =(*env) -> GetStaticMethodID(env,cls,"getInt","(I)I");
    jint value = (*env) -> CallIntMethod(env,cls,methodId);
}
```
