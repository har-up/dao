# C PLUS PLUS
## c++ 与 c区别
- c++ 可以混编 c
- c++ 是面向对象编程， c是面向过程编程

## using namespace
c++中有命名空间的定义，可以在不同命名空间中定义相同的数据，从而进行区分；
```c++
namespace A{
  int a = 10
}

namespace B{
  int a = 20;
}

void main(){
  cout << A::a << endl; //10
  cout << B::a << endl; //20
}
```

## class
```c++
class Person{
  private:
  int age;
  char* name;
  
  public:
  void setAge(int age){
    this -> age = age;
  }
  
  int getAge(){
    return this -> age;
  }
}
```

## 结构体
c++中的结构体在使用时不需要在前边写struct,因此会和class会有混淆；

## 引用
c++中多了引用的概念，引用变量与原变量是同一内存
```c++
int a = 10;
int &b = a;
printf("%#x,%#x\n",&a,&b);//两个地址是一个
```

## 可变参数
```c++
//第一个参数必须指定
void args(int x,...){
    va_list args_p;
    va_start(args_p,x);
    cout << va_arg(args_p,int) << endl;
    cout << va_arg(args_p,int) << endl;
    cout << va_arg(args_p,int) << endl;
    va_end(args_p);
}
```

## 构造函数/析构函数/拷贝构造函数
c++中写类时一般把类写一个头文件，然后有个实现的c++文件，用的时候直接include头文件
```c++
//c++的构造函数和java中一致
class People{
private:
    int age;
    char *name;

public:
    //有参构造函数，默认构造函数被覆盖
    People(int age){
      this->age = age;
      this->name = (char*)malloc(sizeof(name));
      cout << "有参构造函数" << endl;
    }

    //析构函数
    ~People(){
        free(name);
        cout << "析构函数" << endl;
    }

    //拷贝构造函数 浅拷贝写法（默认）,浅拷贝后析构函数中释放内存会出现问题，因为重复释放了同一内存空间
//    People(const People &obj){
//        this->name = obj.name;
//        this->age = obj.age;
//    }

    //拷贝构造函数  深拷贝写法
    People(const People &obj){
        this->age = obj.age;
        this->name = (char*)malloc(sizeof(name) + 1);
        strcpy(this->name,obj.name);
    }

};

void main(){
    args(0,10,20,50);
    People people(30);
    People people1 = people; //这里会调用拷贝构造函数
}

```

## 有参构造函数属性初始化
```c++
//当类中属性有有参构造函数的类对象时，该类需要定义有参构造从而给属性赋值
class Teacher{
private:
    char *name;
    int age;

public:
    Teacher(char *name){
        this->name = name;
        cout << "有参构造函数" << endl;
    }

};

class Student{
private:
    char *name;
    int age;
    Teacher teacher;

public:
    Student(char *teacharName) : teacher(teacharName){
        cout << "有参构造函数" << endl;
    }

};
```

## new/delete
c++中可以通过new来动态分配内存，delete来释放内存
```c++
void main(){
    Student *student1 = new Student("new");  //与c的区别是 这样会调用构造函数
    delete(student1);  // 会调用析构函数
    
    //数组和java类似
    int *arr = new int[10]
    delete [] arr; 释放数组
}
```

## 静态属性和方法
```c++
class Student{
public:
    static int age;

    static void setAge(int age){  //静态函数才能访问静态属性  且静态函数不能访问非静态属性
        Student::age = age;
    }
}



int Student::age = 20;//静态属性在类外边赋值
```

## 类的内存分配
- 类的内存大小不包含其静态属性和函数
- C/C++ 内存分区：栈、堆、全局（静态、全局）、常量区（字符串）、程序代码区。类的函数放在程序代码区，静态属性放在全局区。

## 常函数
常函数定义
```c++
void getAge() contst{ //不能修改指针指向的值，又不能修改指针的值
  cout << this.age << endl;
}


void main(){
  //常量对象只能调用常量函数，不能调用非常量函数
  const Student student;
  student.getAge();
}
```

## 友元函数
```c++
class A{
	//友元函数
	friend void modify_i(A *p, int a);
private:
	int i;
public:
	A(int i){
		this->i = i;
	}
	void myprint(){
		cout << i << endl;
	}
	
};

	//友元函数的实现，在友元函数中可以访问私有的属性
	void modify_i(A *p, int a){
		p->i = a;
	}

	void main(){
		A* a = new A(10);
		a->myprint();

		modify_i(a,20);
		a->myprint();


## 友元类
```c++
//友元类
class A{		
	//友元类
	friend class B;
private:
	int i;
public:
	A(int i){
		this->i = i;
	}
	void myprint(){
		cout << i << endl;
	}	
};

class B{
public:
	//B这个友元类可以访问A类的任何成员
	void accessAny(){
		a.i = 30;		
	}
private:
	A a;
};
```
		system("pause");
	}
```

## 运算符重载
可以对运算符进行重载
```c++
class Point{
public:
    int x;
    int y;

    Point(int xx,int yy){
        this->x = xx;
        this->y = yy;
    }
};

Point operator+(Point &point1,Point &point2){
    return Point(point1.x+point2.x,point1.y + point2.y);
}
```

## 继承
c++的继承基本和java类似
- 继承的父类无默认构造函数时子类需要处理
- c++可以多继承
- c++在继承时也有访问修饰符
- 继承的二义性 //虚继承解决（不同路径继承来的同名属性只有一份拷贝）
- 继承的重写多态，默认是没有重载的，需要加关键字virtual


## 抽象类
具有纯虚函数的类
```
class people{
  virtual void say() = 0; // =0 代表纯虚函数
}
```

## 接口
c++ 中的接口只是逻辑上的区分，语法上和抽象类一样

## 模板函数
函数的业务逻辑一致，只是数据类型不一致（泛型）
```c++
template <typename T> //定义泛型

void log(T t){
    cout << t << endl;
}
```

## 模板类
- 类的某个属性不确定时，可以定义模板类。这种情况下定义模板时的关键字class和typename作用一样
```c++
template <class T>
class B{
public:
    B(T t){
        this->t = t;
    }

    int getA(){
        return t;
    }
private:
    T t;
};

void main(){
  B<int> b = B<Int>(10);
}

```

- 继承模板类
```c++
class M: public B<int>{ //继承时指定模板类中的泛型类型
public:
    M(int a,int b): B<int>(b){
        this-> a = a;
    }

    int getA(){
        return a;
    }
private:
    int a;
};

void main(){
    M m = M(10,20);
    cout << m.getA() << endl;
    cout << m.getB() << endl;
}
```
- 模板类继承模板类
```c++
template <class T>
class B{
public:
    B(T t){
        this->t = t;
    }

    int getB(){
        return t;
    }
private:
    T t;
};

template <class T>
class N: public B<T>{
public:
    N(T n,T t):B<T>(t){
        this->n = n;
    }

    int getN(){
        return n;
    }

private:
    T n;
};

void main(){
    N<int> n = N<int>(100,200);
    cout << n.getB() << n.getN() << endl;
}
```

## 异常处理
- 异常抛出捕获
```c++
// 可以直接抛出任何对象
void count(int num){
    if (num > 10){
        throw "数字过大";
    }
}

class MyException{

};
void count(int num) throw (MyException,int){   //定义抛出的异常类型
    throw int
}

void main(){
    try {
        count(30);
    }catch (const char* msg){  //可以根据throw的任何对象类型进行捕获
        cout << msg << endl;
    }catch (...){  //捕获类型不确定时用 ...；
        cout << "类型不确定" << endl;
    }
}
```

- c++中的标准异常
```c++
void count(int num) throw (MyException){
    throw out_of_range("标准异常之超出范围");
}

void main(){
    try {
        count(3);
    }catch (out_of_range e){
        cout << e.what() << endl;
    }
}
```

## c++的类型转换
为了可读性高，减少代码潜在风险，c++中有四个标准转换函数。
- static_cast 用于普遍转换
- const_cast  可以把常量转成一个指针从而进行修改
- dynamic_cast  两个子类转型时会多态检测，如果给定的对象是另一个子类对象时，会转型失败
- reIntercepter_cast 函数指针转换

## IO
```c++
    char* filename = "C:\\Users\\Administrator\\Desktop\\iotest.txt";
    char* filename2 = "C:\\Users\\Administrator\\Desktop\\iotest2.txt";
    ofstream out(filename2,ios::binary);
    ifstream in(filename,ios::binary);
    if (out.bad()){
        cout << "打开文件异常"  << endl;
    }

    while (!in.eof()){
        char  buffer [1024] = {"0"};
        in.read(buffer, 1024);
        out.write(buffer, sizeof(buffer));
    }

    out.close();
    in.close();
```
