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
