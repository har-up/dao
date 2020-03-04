# C语言基础
## 基本数据类型
- char 
- short
- int
- long
- float
- double

## 指针
- 是什么   
  在内存中存储的数据都会有一个编号（地址），通过这个编号可以找到存储的数据内容，指针变量存放的是所指变量的地     址。

- 用法
```c
  //定义变量a
  int a = 100;
  //定义指针p指向a,p的值就是a的地址
  int* p = &a;
```
- 指针运算 指针指向的是数组时，可以使用指针变量来访问数组中的每一个数据
```c
  // 定义数组a
  int a = {1,2,3,4};
  // 定义指针指向数组a的首地址
  int* p = &a;
  for(int i=0;i<sizeof(a)/sizeof(int);i++){
    printf("%d\n",*p)
    p++; // 指针+1会根据定义的是int指针指向数组的下一数
  }
```

- 二级及多级指针
  定义二级指针指向另一个一级指针可以访问到一级指针指向的变量;多级指针类似
  ```c
    int a = 1;
    int* p = &a;
    //定义二级指针
    int** p2 = $p; 
    printf("%d",**p2); //指向a
    printf("%#x,%#x",*p2,p)//两个的地址值一样
  ```
  
 - 函数指针
  可以申明函数指针，用于指向一个函数
  ```c
    int add(int a,int b){
      return a + b;
    }
    
    // 通过函数指针指向一个函数
    int test_fun_p(){
      int *fun_p(int a,int b)
      fun_p = &add;  
      printf("%d",funp(1,3))
    }
    
    // 把函数指针作为其他函数的参数
    //1,定义函数指针类型
    typedef int *fun_p(int a,int b);
    //作为形参
    int test(fun_p p, int m, int n){
      return p();
    }
  ```
  
  ## 动态内存分配
  - 内存划分
    - 栈区 内存自动分配，会自动释放。一个应用分配的栈区内存较小，一般只有几m
    - 堆区 内存手动分配，需要手动释放。一个应用可分配的堆区一般可以达到系统内存的80%
      
  - 堆区的动态内存分配与释放
    - 分配： malloc()
    - 重新分配：realloc()
      重新分配有两种情况：
        - 当前地址块还有足够连续的内存空间，此时返回的指针和重新分配前一样
        - 当前地址块没有足够多的内存空间够开辟，此时返回的和之前不一样
    - 释放： free()
    ```c
    void test(){
      int* p = malloc(10 * sizeof(int))
      for(int i=0; i<10; i++){
        *p = i;
        printf("%d\n",*p)
        p++;
      }
        
      // 重新分配  重新分配传入的指针需要是初始没有偏移的，否则会报
      int* p2 = realloc(p - 10,20 * sizeof(int)){
      for(int i=0; i<20; i++){
        *p = i;
        printf("%d\n",*p)
        p++;
        }
      }
        
      // 使用完后记得释放,释放也需要是指针的初始地址
      free(p2-20);
      p2 = NULL;
      p = NULL;
    }
    ```
      
## 字符串
- 使用数组声明  可以修改字符串内容
```c
  //1
  char str [] = {'h','e','l','l','o',' ','w','o','r','l','d','\0'};
  //2
  char str [] = "hello world";
   
  //更改字符
  str[2] = 'w';
   
```
- 使用指针声明  不能修改字符串
```c
char* str = "hello world";
printf("%s\n",str) // hello world;
pringtf("%#x\b",str) //字符串首地址
```
  
