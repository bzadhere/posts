---
title: linux多线程服务端编程
date: 2020-01-06 14:08:00
tags:
categories: 读书笔记
---



# 线程安全的对象生命管理周期

weak_ptr 与 shared_ptr 搭档
RAII 资源获取即初始化
写个对象池

```c++
template <class T, class L>
class TObjectPool
{
private:
    int m_capacity = 1024;	// 容量，默认1024
    int m_offset;	// 偏移量，栈顶为0
    T* m_objects;	// 对象存储空间
    L m_lock;	// 锁对象，压栈出栈时加锁
    
public:
    TObjectPool()
    {
        m_offset = 0;
        m_objects = new T[m_capacity];
    }
    
	TObjectPool(int capacity)
    {
        m_capacity = capacity;
		m_offset = 0;
        m_objects = new T[m_capacity];
    }
    
    ~TObjectPool()
    {
        while(m_offset > 0)
            m_objects[--m_offset].~T();
        delete[] m_objects;
        m_capacity = 0;
    }
    
    /*int Length()
    {
        ScopeLock lock(m_lock);
        return m_offset;
    }
    int Capacity()
    {
        ScopeLock lock(m_lock);
        return m_capacity;
    }
    bool Empty()
    {
        ScopeLock lock(m_lock);
        return m_capacity == 0;
    }
    bool Full()
    {
        ScopeLock lock(m_lock);
        return m_capacity == m_offset;
    }*/
    bool Push(T &obj)
    {
        ScopeLock lock(m_lock);
        if(m_capacity > m_offset)
        {
            new (&(m_objects[m_offset++])) T(obj); // 拷贝构造
            return true;
        }
        return false;
    }
    bool Pop(T &obj)
    {
        ScopeLock lock(m_lock);
        if(m_offset > 0)
        {
            obj = m_objects[--m_offset];
            m_objects[m_offset].~T();   // 析构掉
            return true;
        }
        return false;
    }
    void Clear()
    {
        
    }
    bool Reserve(int newCap)
    {
        if(m_capacity < newCap)
        {
            T * objs = new T[nNewCap];
            // 逐个复制
            if(m_objects != nullptr)
            {
                while(m_offset > 0)
                {
                    --m_offset;
                    objs[m_offset] = m_objects[m_offset];
                    m_objects[m_offset].~T();
                }
                delete[] m_objects;
            }
            m_objects = objs;
        }
        else if(m_offset > newCap)
        {
        	while(m_offset > newCap)
            {
                m_objects[--m_offset].~T(); // 析构多余的
            }
        }
        m_capacity = newCap;
    }
};
```



# 线程同步精要

尽量使用高层同步设施（线程池，队列，）
使用普通互斥锁和条件变量来同步，采用RAII和Scoped Locking
写个安全的Singleton, 用shared_ptr来实现copy-on-write

```c++
template <class T>
class Singleton
{
public:
    T* Get()
    {
        phtread_once(&once, &init);
        return m_pObj;
    }
    void init()
    {
        m_pObj = new T;
    }
    T* m_pObj;
	pthread_once_t once;
};
/* example */
class Test
{
    static Test* Instance()
    {
        static Singleton<Test> test;
        return test.Get();
    }
}
```



# 常用变成模型

**单线程服务器模型**，“non-blocking IO + IO multiplexing” ， 即Reactor模式。程序基本结构一个事件驱动，以事件驱动和回调方式实现。

```c++
// 缺点：回调函数必须非阻塞，割裂了业务逻辑不易于维护
while(!bStop)
{
  switch(poll())
  {
  	case 0:
  		break; // 超时
  	case -1:
  		break; // 出错
  	default:
  		break; // IO事件回调 accept或read
  }
}
```

**多线程服务器模型:**

> 1 阻塞IO, 一个请求一个连接（V4查询代理）
> 2 阻塞IO，线程池，是对1的优化
> 3 non-blocking IO + one loop per thread， 即JAVA NIO
> 4 Leader/Flower等高级模式

one loop per thread优点:

> 1 线程数据固定，启动时创建好，不会频繁创建和销毁
> 2 方便线程间调配负载，Eventloop代表线程主循环，把IOChannel(如TCP连接)注册到需要干活的线程。
> 3 IO事件的发生是固定的，同一个TCP链接不需要考虑并发

同步，调用者主动等待返回结果；异步，调用发出之后，调用就直接返回了，所以没有返回结果。
但是在处理IO的时候，阻塞/非阻塞/IO复用(select/epoll)，都属于同步IO，异步IO有专门的API接口。

**消息队列和线程池**，参考Java的Arrary和Linked，线程分类(IO线程，计算线程，第三方库如log，数据库连接线程)

适用场景，具体分析，IO密集型和计算密集型，链接数



# 多线程系统编程精要

__thread 只能修饰非POD类型，不能修饰class，可以修饰全局或函数内的静态变量
epoll增删改fd需要在同一个线程中进行，所以Eventloop需要wakeup()
Socket对象包装文件描述符fd，便于读写不串话及生命周期管理(shared_ptr)

















