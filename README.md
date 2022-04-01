# 实验2

## 实验内容

1. 阅读理解示例程序。
	1. 理解并发
	2. 理解调度
	3. 理解同步
2. 说明示例程序是否能适合解决N个生产者和1个消费者问题，并设计实验验证
	1. 理解临界资源，临界区
	2. 理解互斥
	3. 一个消费者时
3. 参照教材修改为N个生产者和1个消费者问题
4. 按照教材代码写出N个生产者和M个消费者问题的解决方案
	N:M时
5. 利用信号量解决同步问题（书上知识点：前趋图）。
	1. 直接运行3.3的实验代码，结果如何？为何是这样的结果？
	2. 在3.3代码的for循环内部加入usleep，结果又是怎样？
	3. 加入信号量，使得有序输出ABC
	4.  AAABBBCC... 可以有多种方式实现，自己完成后，参考其他同学的代码，扩展思路

## 实验目的

* 通过程序模拟及验证生产者消费者问题等经典问题，深入理解并发中的同步和互斥的概念

## 实验总结

* 并发
	//
* 调度
	//
* 同步
	//
* 临界资源，临界区
	//
* 互斥
	//
* 2.3
	//
* //********

## 代码

### N个生产者和1个消费者
```c
#include <stdio.h>   
#include <stdlib.h>   
#include <pthread.h>   
#include <unistd.h>   
#include <sys/types.h>   
#include <semaphore.h> 
int n=10, in=0,out=0,buffer[10];
sem_t empty,full,mutex;
void* producer(void *arg){
    int tag= pthread_self()%100;
    int nextPro;
    srand(time(NULL)+tag);
    while(1){
        nextPro = rand()%97;
        sem_wait(&empty);
        sem_wait(&mutex);
        buffer[in] = nextPro;
        printf("production:%d %d\n",nextPro,in); 
        usleep(1000*1000);
        in = (in+1)%n;
        sem_post(&mutex);
        sem_post(&full);
        usleep(1000*1000);
    }
} 
void* consumer(void *arg)  {
    int item;
    while(1){
        sem_wait(&full);
        item = buffer[out];
        sem_post(&empty);
        printf("consume:%d %d\n",item, out);
        out = (out+1)%n;
        usleep(1000*1000);
    }
}
int main()  {  
	pthread_t tid[7];
    //初始有10个缓冲区
    sem_init( &empty, 0,10);
    sem_init( &full, 0,0);
    sem_init( &mutex, 0, 1);
    for(int i = 0; i < 6; i++) 
        pthread_create(&tid[i], NULL, producer, NULL);
    pthread_create(&tid[6], NULL, consumer, NULL);  
    for(int i = 0; i < 7; i++)
	pthread_join(tid[i], NULL);      
    printf("main is over\n");
}
```

### N个生产者和M个消费者问题
```c
#include <stdio.h>   
#include <stdlib.h>   
#include <pthread.h>   
#include <unistd.h>   
#include <sys/types.h>   
#include <semaphore.h> 
int n=10, in=0,out=0,buffer[10];
sem_t empty,full,mutex;
void* producer(void *arg){
    int tag= pthread_self()%100;
    int nextPro;
    srand(time(NULL)+tag);
    while(1){
        nextPro = rand()%97;
        sem_wait(&empty);
        sem_wait(&mutex);
        buffer[in] = nextPro;
        printf("production:%d %d\n",nextPro,in); 
        usleep(1000*1000);
        in = (in+1)%n;
        sem_post(&mutex);
        sem_post(&full);
        usleep(1000*1000);
    }
} 
void* consumer(void *arg)  {
    int item;
    while(1){
        sem_wait(&mutex);
        sem_wait(&full);
        item = buffer[out];
        sem_post(&empty);
        printf("consume:%d %d\n",item, out);
        out = (out+1)%n;
        sem_post(&mutex);
        usleep(1000*1000);
    }
}
int main()  {  
	pthread_t tid[10];
    //初始有10个缓冲区
    sem_init( &empty, 0,10);
    sem_init( &full, 0,0);
    sem_init( &mutex, 0, 1);
    for(int i = 0; i < 6; i++) 
        pthread_create(&tid[i], NULL, producer, NULL); 
    for(int i = 6; i < 10; i++) 
	pthread_create(&tid[6], NULL, consumer, NULL);  
	for(int i = 0; i < 10; i++)  
		pthread_join(tid[i], NULL);      
	printf("main is over\n");
}
```

### 有序输出ABC
```c
#include <stdio.h>   
#include <stdlib.h>   
#include <pthread.h>   
#include <unistd.h>   
#include <sys/types.h>   
#include <semaphore.h>
sem_t s_A, s_B, s_C;
void* p1(void *arg)
{
	for(int i=0;i<10;i++)
    {
        sem_wait(&s_C);
	    printf("A\n");
        sem_post(&s_A);
	}
} 
void* p2(void *arg)
{
	for(int i=0;i<10;i++)
    {
        sem_wait(&s_A);
	    printf("B\n");
        sem_post(&s_B);
	}
} 
void* p3(void *arg)
{
	for(int i=0;i<10;i++)
    {
        sem_wait(&s_B);
	    printf("C\n");
        sem_post(&s_C);
	}
} 

int main()  
{
    sem_init(&s_A, 0, 0);
    sem_init(&s_B, 0, 0);
    sem_init(&s_C, 0, 1);
	pthread_t tid[3];
	pthread_create(&tid[0], NULL, p1, NULL);
	pthread_create(&tid[1], NULL, p2, NULL);
	pthread_create(&tid[2], NULL, p3, NULL);
	for (int i = 0; i < 3; i++)
		pthread_join(tid[i], NULL);
	printf("main is over\n");
}
```

### 有序输出AAABBBBCCC
```c
#include <stdio.h>

```
