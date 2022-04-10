# 实验2

## 实验要求

1. 阅读理解示例程序。
	1. 理解并发:并发是什么？程序中有没有并发？如何并发运行？
	2. 理解调度:调度是什么意思？调度发生的时机？
	3. 理解同步:生产者和消费者哪个快？为何不会出错?
2. 说明示例程序是否能适合解决N个生产者和1个消费者问题，并设计实验验证
	1. 理解临界资源，临界区:有没有？在哪？为什么是？
	2. 理解互斥:什么是互斥？如何互斥？生产者之间是否需要互斥?
3. 参照教材修改为N个生产者和1个消费者问题
4. 按照教材代码写出N个生产者和M个消费者问题的解决方案
5. 利用信号量解决同步问题（书上知识点：前趋图）。
	1. 直接运行3.3的实验代码，结果如何？为何是这样的结果？
	2. 在3.3代码的for循环内部加入usleep，结果又是怎样？
	3. 加入信号量，使得有序输出ABC
	4. 有序输出AAABBBCC...

## 实验目的

* 通过程序模拟及验证生产者消费者问题等经典问题，深入理解并发中的同步和互斥的概念

## 实验内容

1.  
   * 并发：是指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。示例程序中存在并发。
   * 调度就是从就绪队列中按照一定的算法选择一个进程并将处理机分配给它运行，以实现进程的并发执行。
        >将代码中producer的两个usleep语句注释掉,运行效果见图一,解释：由于生产者的时间片没有结束,所以一开始生产者直接生产了10个产品,之后由于缓存区满了,生产者进入等待阶段,消费者才开始运行,拿出一个产品后,由于存在usleep语句使consumer进程挂起一段时间，从而切换到producer进程，在生产一个产品后，又使缓存区满，从而进入等待，等待consumer进程挂起结束后又执行consumer进程，这样往返重复执行。
   * 生产者和消费者不会出错的原因是存在着信号量使其不能同时访问临界资源。
        >程序不会发生错误,运行效果见图二,解释：当缓冲区为空，生产者生产的产品可以放进入，但消费者无法从缓冲区获取产品，所以消费者处于等待状态，当生产者向缓冲区放入一个产品之后，对信号量full进行了V增加操作，此时消费者就可以从缓冲区获取产品了；
        当生产者连续生产了四个产品放入缓冲区，但消费者都还没消费时，此时信号量empey=0，生产者无法继续向缓冲区存放产品，处于等待状态，当消费者从缓冲区取出一个产品，并对信号量empty进行V操作后，生产者可以继续向缓冲区存放产品。

2. 示例程序不能适合解决N个生产者和1个消费者问题,运行效果见图三
   * 临界资源：是一种系统资源，是必须互斥访问的资源，这种资源同时只能被一个进程所使用，而临界区：则是每个进程中访问临界资源的一段代码，是属于对应进程的，临界区前后需要设置进入区和退出区以进行检查和恢复。
         >//
    * 互斥：是指散步在不同任务之间的若干程序片断，当某个任务运行其中一个程序片段时，其它任务就不能运行它们之中的任一程序片段，只能等到该任务运行完这个程序片段后才可以运行。增添互斥量使不同程序之间互斥。生产者间需要互斥：若生产者间不进行互斥处理，生产者间可能同时使用缓冲区，从而导致临界资源的不一致。
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
#include <stdlib.h>   
#include <pthread.h>   
#include <unistd.h>   
#include <sys/types.h>   
#include <semaphore.h>
sem_t s_A, s_B, s_C, s;
void* p1(void *arg)
{
	for(int i=0;i<30;i++)
    {
        sem_wait(&s_C);
	    printf("A\n");
        sem_wait(&s);
        int x;
        sem_getvalue(&s_C, &x);
        if (x == 0)
            for (int j = 0; j < 3; j++)
                sem_post(&s_A);
        sem_post(&s);
	}
} 
void* p2(void *arg)
{
	for(int i=0;i<30;i++)
    {
        sem_wait(&s_A);
	    printf("B\n");
        sem_wait(&s);
        int x;
        sem_getvalue(&s_A, &x);
        if (x == 0)
            for (int j = 0; j < 3; j++)
                sem_post(&s_B);
        sem_post(&s);
	}
} 
void* p3(void *arg)
{
	for(int i=0;i<30;i++)
    {
        sem_wait(&s_B);
	    printf("C\n");
        sem_wait(&s);
        int x;
        sem_getvalue(&s_B, &x);
        if (x == 0)
            for (int j = 0; j < 3; j++)
                sem_post(&s_C);
        sem_post(&s);
	}
} 

int main()  
{
    sem_init(&s_A, 0, 0);
    sem_init(&s_B, 0, 0);
    sem_init(&s_C, 0, 3);
    sem_init(&s, 0, 1);
	pthread_t tid[3];
	pthread_create(&tid[0], NULL, p1, NULL);
	pthread_create(&tid[1], NULL, p2, NULL);
	pthread_create(&tid[2], NULL, p3, NULL);
	for (int i = 0; i < 3; i++)
		pthread_join(tid[i], NULL);
	printf("main is over\n");
}
```
