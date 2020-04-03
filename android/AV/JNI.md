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

- 访问构造方法构造一个类的实例
```c
//访问构造方法
//使用java.util.Date产生一个当前的时间戳
JNIEXPORT jobject JNICALL Java_com_jni_JniTest_accessConstructor
(JNIEnv *env, jobject jobj){
	jclass cls = (*env)->FindClass(env, "java/util/Date");
	//jmethodID
	jmethodID constructor_mid = (*env)->GetMethodID(env, cls, "<init>", "()V");
	//实例化一个Date对象
	jobject date_obj = (*env)->NewObject(env, cls, constructor_mid);
	//调用getTime方法
	jmethodID mid = (*env)->GetMethodID(env, cls, "getTime", "()J");
	jlong time = (*env)->CallLongMethod(env, date_obj, mid);

	printf("\ntime:%lld\n",time);

	return date_obj;
}
```

- 调用父类的构造方法（java中实现不了的技术）
```c
//调用父类的方法
JNIEXPORT void JNICALL Java_com_jni_JniTest_accessNonvirtualMethod
(JNIEnv *env, jobject jobj){
	jclass cls = (*env)->GetObjectClass(env, jobj);
	//获取man属性（对象）
	jfieldID fid = (*env)->GetFieldID(env, cls, "human", "Lcom/dongnaoedu/jni/Human;");
	//获取
	jobject human_obj = (*env)->GetObjectField(env, jobj, fid);

	//执行sayHi方法
	jclass human_cls = (*env)->FindClass(env, "com/dongnaoedu/jni/Human"); //注意：传父类的名称
	jmethodID mid = (*env)->GetMethodID(env, human_cls, "sayHi", "()V");

	//执行
	//(*env)->CallObjectMethod(env, human_obj, mid);
	//调用的父类的方法
	(*env)->CallNonvirtualObjectMethod(env, human_obj, human_cls, mid);
}
```

- 中文乱码问题
```c
//中文问题
JNIEXPORT jstring JNICALL Java_com_jni_JniTest_chineseChars
(JNIEnv *env, jobject jobj, jstring in){
	char *c_str = "马蓉与宋江";
	//jstring jstr = (*env)->NewStringUTF(env, c_str);
	//执行String(byte bytes[], String charsetName)构造方法需要的条件
	//1.jmethodID
	//2.byte数组
	//3.字符编码jstring

	jclass str_cls = (*env)->FindClass(env, "java/lang/String");
	jmethodID constructor_mid = (*env)->GetMethodID(env, str_cls, "<init>", "([BLjava/lang/String;)V");

	jbyteArray bytes = (*env)->NewByteArray(env, strlen(c_str));
	//byte数组赋值
	//0->strlen(c_str)，从头到尾
	//对等于，从c_str这个字符数组，复制到bytes这个字符数组
	(*env)->SetByteArrayRegion(env, bytes, 0, strlen(c_str), c_str);

	//字符编码jstring
	jstring charsetName = (*env)->NewStringUTF(env, "GB2312");

	//调用构造函数，返回编码之后的jstring
	return (*env)->NewObject(env,str_cls,constructor_mid,bytes,charsetName);
}
```

- 接受数组参数并排序
```c
//传入
JNIEXPORT void JNICALL Java_com_jni_JniTest_giveArray
(JNIEnv *env, jobject jobj, jintArray arr){
	//jintArray -> jint指针 -> c int 数组
	jint *elems = (*env)->GetIntArrayElements(env, arr, NULL);
	//printf("%#x,%#x\n", &elems, &arr);

	//数组的长度
	int len = (*env)->GetArrayLength(env, arr);
	//排序
	qsort(elems, len, sizeof(jint), compare);	

	//同步
	//mode
	//0, Java数组进行更新，并且释放C/C++数组
	//JNI_ABORT, Java数组不进行更新，但是释放C/C++数组
	//JNI_COMMIT，Java数组进行更新，不释放C/C++数组（函数执行完，数组还是会释放）
	(*env)->ReleaseIntArrayElements(env, arr, elems, JNI_COMMIT);
}
```

- 返回数组到java
```c
//返回数组
JNIEXPORT jintArray JNICALL Java_com_jni_JniTest_getArray(JNIEnv *env, jobject jobj, jint len){
	//创建一个指定大小的数组
	jintArray jint_arr = (*env)->NewIntArray(env, len);
	jint *elems = (*env)->GetIntArrayElements(env, jint_arr, NULL);	
	int i = 0;
	for (; i < len; i++){
		elems[i] = i;
	}

	//同步
	(*env)->ReleaseIntArrayElements(env, jint_arr, elems, 0);	

	return jint_arr;
}
```

- 创建和释放全局引用变量
```c
//全局引用
//共享(可以跨多个线程)，手动控制内存使用
jstring global_str;

//创建
JNIEXPORT void JNICALL Java_com_jni_JniTest_createGlobalRef(JNIEnv *env, jobject jobj){
	jstring obj = (*env)->NewStringUTF(env, "jni development is powerful!");
	global_str = (*env)->NewGlobalRef(env, obj);
}


//释放
JNIEXPORT void JNICALL Java_com_jni_JniTest_deleteGlobalRef(JNIEnv *env, jobject jobj){
	(*env)->DeleteGlobalRef(env, global_str);
}
```

- 创建和销毁弱全局引用变量
```c
//弱全局引用
//节省内存，在内存不足时可以是释放所引用的对象
//创建：NewWeakGlobalRef,销毁：DeleteGlobalWeakRef
//创建
JNIEXPORT void JNICALL Java_com_jni_JniTest_createGlobalRef(JNIEnv *env, jobject jobj){
	jstring obj = (*env)->NewStringUTF(env, "jni development is powerful!");
	global_str = (*env)->NewWeakGlobalRef(env, obj);
}


//释放
JNIEXPORT void JNICALL Java_com_jni_JniTest_deleteGlobalRef(JNIEnv *env, jobject jobj){
	(*env)->DeleteGlobalWeakRef(env, global_str);
}
```

- 异常处理
1.保证Java代码可以运行
2.补救措施保证C代码继续运行
```c
//JNI自己抛出的异常，可以通过Throwable捕获到
//用户通过ThrowNew抛出的异常，可以在Java层捕捉
JNIEXPORT void JNICALL Java_com_jni_JniTest_exeception(JNIEnv *env, jobject jobj){
	jclass cls = (*env)->GetObjectClass(env, jobj);
	jfieldID fid = (*env)->GetFieldID(env, cls, "key2", "Ljava/lang/String;");
	//检测是否发生Java异常
	jthrowable exception = (*env)->ExceptionOccurred(env);
	if (exception != NULL){
		//让Java代码可以继续运行
		//清空异常信息
		(*env)->ExceptionClear(env);

		//补救措施
		fid = (*env)->GetFieldID(env, cls, "key", "Ljava/lang/String;");
	}

	//获取属性的值
	jstring jstr = (*env)->GetObjectField(env, jobj, fid);
	char *str = (*env)->GetStringUTFChars(env, jstr, NULL);

	//对比属性值是否合法
	if (_stricmp(str, "super jason") != 0){
		//认为抛出异常，给Java层处理
		jclass newExcCls = (*env)->FindClass(env, "java/lang/IllegalArgumentException");
		(*env)->ThrowNew(env,newExcCls,"key's value is invalid!");
	}
}
```

- JNI的缓存策略
关键字 static
```c
//缓存策略

//static jfieldID key_id 
JNIEXPORT void JNICALL Java_com_jni_JniTest_cached(JNIEnv *env, jobject jobj){
	jclass cls = (*env)->GetObjectClass(env, jobj);	
	//java端不断调用 获取jfieldID也只获取一次
	//局部静态变量
	static jfieldID key_id = NULL;
	if (key_id == NULL){
		key_id = (*env)->GetFieldID(env, cls, "key", "Ljava/lang/String;");
		printf("--------GetFieldID-------\n");
	}
}

//初始化全局变量，动态库加载完成之后，立刻缓存起来，这样可以提升代码执行速率
jfieldID key_fid;
jmethodID random_mid;
JNIEXPORT void JNICALL Java_com_jni_JniTest_initIds(JNIEnv *env, jclass jcls){	
	key_fid = (*env)->GetFieldID(env, jcls, "key", "Ljava/lang/String;");
	random_mid = (*env)->GetMethodID(env, jcls, "genRandomInt", "(I)I");
}
```

## Local Reference
  当线程从java执行切换到Native上下文时，都会针对一个Native method调用创建一个Local Reference表，在这个Native method执行过程中
  每次引用到java对象都会在表中插入一条local reference。虽然在Native method执行结束后会自动删除这些引用，但当一个Native method
  中引用很多java对象时，就会英文超出Local Reference表内存的限制而报错，所以在引用后不需要时需要释放掉这个引用。
