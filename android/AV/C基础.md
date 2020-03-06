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

## 结构体
结构体是一种自定义构造数据类型,把多个其他数据类型整合组合成一个数据类型。
- 关键字 struct
```c
  //声明结构体时不能初始化数据的
  struct People {
    int age;
    char name [20];
  }
  
  void main(){
    //1
    struct People people1 = {18, "james"};
    
    //
    struct People people2;
    people2.age = 28;
    //people2.name = "james"; //注意：当我们定义数据为字符数组的时候不能直接这样赋值；使用这种方式赋值，可以使用字符指针来声明数据类型；
    //sprintf(people2.name,"james");//可以使用这种方法赋值；还有string.h中的strcpy;
    printf("%d\n",people1.name)
  }
```

- 结构体的另外几种写法
```c
  //在结构体后面声明结构体的变量
  struct People {
    int age;
  }one,two={18};
  
  //匿名结构体，可以限制结构体变量的数量
  struct {
    int age;
    char* name;
  }singleInstance
  
  //结构体嵌套
  struct Teacher {
    char* programe;
    int age;
  }
  
  struct Student {
    int age;
    struct Teacher teacher;
  }
  
  // 也可以直接在Student结构体种申明老师的结构体
  struct Student {
    int age;
    struct Teacher {
      int age;
      Char* programe;
    } teacher;
  }
  //访问方式
  struct Student student;
  student.teacher.age = 18;
```

- 结构体与指针
```c
  //通过指针访问结构体的变量
  struct Student {
    int age;
    char* name;
  }
  
  void main(){
    struct Student student = {18,"james"}
    //指针指向这个变量
    struce Student* p = &student;
    printf("%d,%s\n", *p.age, p->name) //可以通过这两种方式拿到变量的值
  }
  
  //指针和结构体数组
  struct Student students[] = {{20,"james"},{18,"zion"}}
  //声明Student类型的指针
  struct Student* student_p;
  student_p = &student;
  
  //遍历1
  struct Student* loop_p = student_p;
  for (; loop_p < student_p + 2; ++loop_p) {
    printf("%d,%s\n",loop_p->age,loop_p->name);
  }

  //遍历2
  for (int i = 0; i < sizeof(students) / sizeof(struct Student); ++i) {
    printf("%d,%s\n",students[i].age,students[i].name);
  }
  
```

- typedef
给类型取别名
```
  struct Student {
    int age;
    char* name;
  }
  
  //取别名Student,以后声明可不用写struct
  typedef struct Student Student;
  
  也可以直接在定义时取别名
  typedef struct Student{
    int age;
  }Student,*P;
  
  //使用方式
  Student student = {18};
  P p = &student;
```

- 结构体函数指针
```
struct People {
  int age;
  void (*say)();
}

void sayHello(){
  printf("hello");
}

void main(){
  struct People people;
  people.say = sayHello; // 这里两种方式都可以
  people.say = &sayHello;
  people.say();
}
```
  
## 联合体
联合体内的数据只能有一个生效。给联合体中的一个属性赋值，另一个值就会置空。
```c
  union Value{
    int num;
    
    float fnum;
    
    char* str;
  }
  
  void main(){
    union Value value;
    value.num = 1;
    value.fnum = 2.0;
    value.str = "hello";
    //此时只要str的值是有效的
    printf("%d,%f,%s\n",value.num,value.funm,value.str);
  }
```

## 枚举
关键字 - enum
```c
  enum Week{
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday
  }
  
  void main(){
    enum Week week;
    week = Sunday;
    printf("%d\n",week)  //6
  }
```

## IO
文件流打开mode
  "r" 以只读方式打开文件，该文件必须存在。  
  "w" 打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。  
  "w+" 打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。  
  "a" 以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留）  
  "a+" 以附加方式打开可读写的文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾后，即文件原先的内容会被保留。（原来的EOF符不保留）  
  "wb" 只写打开或新建一个二进制文件，只允许写数据。  
  "wb+" 读写打开或建立一个二进制文件，允许读和写。  
  "ab" 追加打开一个二进制文件，并在文件末尾写数据。  
  "ab+"读写打开一个二进制文件，允许读，或在文件末追加数据。  
  
  
- 读文件
```c
  //读文件
void main(){
    char* filePath = "D:\\CLion 2018.1.5\\workspace\\vectordemo\\test_io.txt";
    FILE* stream = fopen(filePath,"w+");
    if (stream == NULL){
        printf("FILE NOT FOUND");
    }
    char buffer[50];
    while(fgets(buffer,50,stream)) {
        printf("%s", buffer);
    }
    fclose(stream);
}
```
- 写文件
```c
//写文件
void main(){
    char* filePath = "D:\\CLion 2018.1.5\\workspace\\vectordemo\\test_new.txt";
    FILE* stream = fopen(filePath,"w");

    char* content = "write content";
    fputs(content,stream);
    fclose(stream);
}

//附加写文件
void main(){
    char* filePath = "D:\\CLion 2018.1.5\\workspace\\vectordemo\\test_new.txt";
    FILE* stream = fopen(filePath,"a");

    char* content = " add content";
    fputs(content,stream);
    fclose(stream);
} 

//文件复制
void main(){
    char* filePath = "D:\\CLion 2018.1.5\\workspace\\vectordemo\\test_new.txt";
    char* fileCopyPath = "D:\\CLion 2018.1.5\\workspace\\vectordemo\\test_copy.txt";
    FILE* stream = fopen(filePath,"rb");

    FILE* write_stream = fopen(fileCopyPath,"wb");
    char buffer [50];
    int len = 0;
    while( (len = fread(buffer, sizeof(char),50,stream)) != 0){
        fwrite(buffer, sizeof(char),len,write_stream);
    };
    fclose(write_stream);
    fclose(stream);
}

//文件大小
void main(){
    char* filePath = "D:\\CLion 2018.1.5\\workspace\\vectordemo\\test_new.txt";
    char* fileCopyPath = "D:\\CLion 2018.1.5\\workspace\\vectordemo\\test_copy.txt";

    FILE* stream = fopen(filePath,"rb");
    fseek(stream,0,SEEK_END); //把游标指向末尾的偏移为0的地方
    long length = ftell(stream);//返回游标位置与文件开始位置的偏移量
    printf("%d\n",length);
}
```
