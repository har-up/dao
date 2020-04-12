#LINUX下的POSIX
posix是linux下的标准线程操作库
## 引入
```c
#include<pthread.h>
```

## 基本使用
```c
#include<pthread.h>

void* callback(void* arg){
  int i=0;
  for(;i<100;i++){
    if(i==50){
      pthread_exit((void*)result); //线程主动退出
    }
    printf("%s:%d",arg,i);
  }
}

void main(){
  pthread_t thread;
  void* result;
  pthread_create(&thread,NULL,callback,"thread 1");
  
  pthread_join(thread,&result); //主线程等待创建的线程执行完毕；
  
  pthread_cancel(thread); //终止某个线程
}
```

## 互斥锁
```c
#include<stdio.h>
#include<stdlib.h>
#include<pthread.h>
#include<unistd.h>

int i=0;
pthread_mutex_t mutex;
void* callback(void* arg){
  pthread_mutex_lock(&mutex); //锁住，只让一个线程访问
  for(;i<10;i++){
    printf("%s: %d\n",arg,i);
    sleep(1);
  }
  i=0;
  pthread_mutex_unlock(&mutex);//解锁，释放资源的访问
}

void main(){
  pthread_t thread1;
  pthread_t thread2;
  pthread_t thread3; 
  pthread_mutex_init(&mutex,NULL);
  void* result1;
  void* result2;
  pthread_create(&thread1,NULL,callback,"thread1");
  pthread_create(&thread2,NULL,callback,"thread2");
  pthread_create(&thread3,NULL,callback,"thread3");
             
  pthread_join(thread1,NULL);
  pthread_join(thread2,NULL);
  pthread_join(thread3,NULL);
  pthread_mutex_destroy(&mutex);
}   

```

## 简单的生产者消费者
```c
#include<stdio.h>
#include<stdlib.h>
#include<pthread.h>
#include<unistd.h>

int i=0;
pthread_mutex_t mutex;

void* callback(void* arg){
  pthread_mutex_lock(&mutex);
  for(;i<10;i++){
    printf("%s: %d\n",arg,i);
    sleep(1);
  }
  i=0;
  pthread_mutex_unlock(&mutex);
}

void main(){
  pthread_t thread1;
  pthread_t thread2;
  pthread_t thread3; 
  pthread_mutex_init(&mutex,NULL);
  void* result1;
  void* result2;
  pthread_create(&thread1,NULL,callback,"thread1");
  pthread_create(&thread2,NULL,callback,"thread2");
  pthread_create(&thread3,NULL,callback,"thread3");
             
  pthread_join(thread1,NULL);
  pthread_join(thread2,NULL);
  pthread_join(thread3,NULL);
  pthread_mutex_destroy(&mutex);
}   

```

## 多线程生产及消费
```c
#include<stdio.h>
#include<stdlib.h>
#include<pthread.h>
#include<unistd.h>

#define PRODUCER_NUM 3
#define CONSUMER_NUM 2

int product = 0;
pthread_mutex_t mutex;
pthread_cond_t has_product;

void* fun_product(void* arg){
  printf("producer %s start\n",(int*)arg);
  while(1){
    pthread_mutex_lock(&mutex);
    product++;
    printf("produce product+1:product=%d\n",product);
    pthread_mutex_unlock(&mutex);
    sleep(1);
    pthread_cond_signal(&has_product);
  }
}


void* fun_consume(void* arg){
  printf("consumer %s start\n",(int*)arg);
  while(1){
    pthread_mutex_lock(&mutex);
    while(product > 0){
      product--;
      printf("consume product-1: product=%d\n",product);
      pthread_mutex_unlock(&mutex);
      sleep(0.2);
    }
    printf("no product wait\n");
    pthread_cond_wait(&has_product,&mutex);
    pthread_mutex_unlock(&mutex);
  }
}

void main(){


  pthread_t threads[PRODUCER_NUM+CONSUMER_NUM];

  pthread_t producer;
  pthread_t consumer;
  pthread_mutex_init(&mutex,NULL);
  pthread_cond_init(&has_product,NULL);

  //pthread_create(&producer,NULL,fun_product,"product start");
  //pthread_create(&consumer,NULL,fun_consume,"consume start");

  int i=0;
  for(;i<PRODUCER_NUM;i++){
    pthread_create(&threads[i],NULL,fun_product,i+"");
  }

  for(;i<sizeof(threads)/sizeof(pthread_t);i++){
    pthread_create(&threads[i],NULL,fun_consume,i+"");
  }

  //pthread_join(producer,NULL);
  //pthread_join(consumer,NULL);
  int n=0;
  for(;n<sizeof(threads)/sizeof(pthread_t);n++){
    pthread_join(threads[n],NULL);
  }

  pthread_mutex_destroy(&mutex);
  pthread_cond_destroy(&has_product);
}

```

