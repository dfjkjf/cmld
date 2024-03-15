---
sort: 1
---

# C++多线程

## 智能锁和保险箱

想象这样一个场景，Danny和Peter在黑板上写名字，"My name is Danny."和"My name is Peter."。两人同时写字，最终黑板上呈现"My name My name is Peter. is Danny."。

**1. 标准的多线程共享资源竞争冲突**

```c++
#include <mutex>
#include <thread>
#include <iostream>
#include <functional>
using namespace std;

void DannyWrite(string &blackboard)
{
    blackboard=+"My";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " name";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+=" is";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " Danny\n";
}
void PeterWrite(string& blackboard)
{
    blackboard+="My";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " name";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+=" is";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " Peter\n";
}

int main()
{
    string blackboard;
    thread DannyThread(DannyWrite, std::ref(blackboard));
    thread PeterThread(PeterWrite,std::ref(blackboard));
    DannyThread.join();
    PeterThread.join();
    cout<<blackboard<<endl;
    
    return 0;
}
```
造成上述问题的原因是Danny和Peter都有“一支笔”，我们让笔只有一个，只有拿到笔的人才能写字。

**2. 增加临界资源**

```c++
#include <mutex>
#include <thread>
#include <iostream>
#include <functional>
using namespace std;

void DannyWrite(string &blackboard)
{
    blackboard=+"My";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " name";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+=" is";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " Danny\n";
}
void PeterWrite(string& blackboard)
{
    blackboard+="My";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " name";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+=" is";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " Peter\n";
}
void PeterWriteWithMutex(mutex& amutex, string& blackboard )
{
    /* raii的设计预防因Exception未unlock */
    unique_lock<std::mutex> lk(amutex);
    PeterWrite(blackboard);
}
void DannywriteWithMutex(mutex& amutex, string& blackboard)
{
    unique_lock<std::mutex> lk(amutex);
    DannyWrite(blackboard);
}

int main()
{
    string blackboard;
    std::mutex amutex;
    thread DannyThread(DannywriteWithMutex, std::ref(amutex), std::ref(blackboard));
    thread PeterThread(PeterWriteWithMutex,std::ref(amutex), std::ref(blackboard));
    DannyThread.join();
    PeterThread.join();
    cout<<blackboard<<endl;
    
    return 0;
}
```
这样的设计还有一个隐患，未lock也能访问共享资源，此时我们需要把共享资源提前锁上。

**3. 保险箱设计，获取共享资源前已加锁**


```c++
#include <mutex>
#include <thread>
#include <iostream>
#include <functional>
using namespace std;

template <typename T>
class MutexSafe
{
private:
    mutex _mutex;
    T* _resource;
    T* operator ->(){}
    T& operator &(){}
public:
    MutexSafe(T* resource):_resource(resource){}
    ~MutexSafe(){delete _resource;}
    void lock()
    {
        _mutex.lock();
    }
    void unlock()
    {
        _mutex.unlock();
    }
    bool try_lock()
    {
        return _mutex.try_lock();
    }
    mutex& Mutex()
    {
        return _mutex;
    }
    
    // this is version 3 of Acquire
    //  it requires the passed in lock parameter is the safe object itself
    // thus it avoids the issue that an arbitrary lock can access the resource.
    // Also, it now requires the lock to be the same type of Safe that Acquire is.
     T& Acquire (unique_lock<MutexSafe<T>>& lock)
    {
        MutexSafe<T> *_safe = lock.mutex();
        if(&_safe->Mutex()!=&_mutex)
        {
            throw "wrong lock object passed to Acquire function.\n";
        }
        return *_resource;
    }
    // This overloaded Acquire allows unique_lock<mutex> to be passed in as a parameter
    T& Acquire (unique_lock<mutex>& lock)
    {
        if(lock.mutex()!=&_mutex)
        {
            throw "wrong lock object passed to Acquire function.\n";
        }
        return *_resource;
    }
};
void DannyWrite(string &blackboard)
{
    blackboard=+"My";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " name";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+=" is";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " Danny\n";
}
void PeterWrite(string& blackboard)
{
    blackboard+="My";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " name";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+=" is";
    this_thread::sleep_for(std::chrono::milliseconds(rand()%3));
    blackboard+= " Peter\n";
}

typedef MutexSafe<string>  Safe;
void SafeDannyWrite(Safe& safe)
{
    unique_lock<Safe> lock(safe);
    string& blackboard = safe.Acquire(lock);
    DannyWrite(blackboard);
}
void SafePeterWrite(Safe& safe)
{
    unique_lock<Safe> lock(safe);
    string& blackboard = safe.Acquire(lock);
    PeterWrite(blackboard);
}


int main()
{
    Safe safe(new string());
    thread DannyThread(SafeDannyWrite,ref(safe));
    thread PeterThread(SafePeterWrite, ref(safe));
    DannyThread.join();
    PeterThread.join();
    unique_lock<Safe> lock(safe);
    string& blackboard = safe.Acquire(lock);
    cout<<blackboard<<endl;
    
    return 0;
}
```

## 条件变量

想象这样一个场景，Peter在黑板上写股票价格，Danny盯着黑板，当价格大于$90时卖出。

```c++
#include <mutex>
#include <condition_variable>
#include <thread>
#include <iostream>
#include <functional>
using namespace std;

template <typename T>
class MutexSafe
{
private:
    mutex _mutex;
    T* _resource;
    T* operator ->(){}
    T& operator &(){}
public:
    MutexSafe(T* resource):_resource(resource){}
    ~MutexSafe(){delete _resource;}
    void lock()
    {
        _mutex.lock();
    }
    void unlock()
    {
        _mutex.unlock();
    }
    bool try_lock()
    {
        return _mutex.try_lock();
    }
    mutex& Mutex()
    {
        return _mutex;
    }
    T& Acquire (unique_lock<MutexSafe<T>>& lock)
    {
        MutexSafe<T> *_safe = lock.mutex();
        if(&_safe->Mutex()!=&_mutex)
        {
            throw "wrong lock object passed to Acquire function.\n";
        }
        return *_resource;
    }
    T& Acquire (unique_lock<mutex>& lock)
    {
        if(lock.mutex()!=&_mutex)
        {
            throw "wrong lock object passed to Acquire function.\n";
        }
        return *_resource;
    }
};

struct StockBlackboard
{
    float price;
    const string name;
    StockBlackboard(const string stockName, float stockPrice=0):name(stockName),price(stockPrice){}
};

typedef MutexSafe<StockBlackboard> StockSafe;

void PeterUpdateStock_Notify (StockSafe & safe, condition_variable& condition)
{
    for (int i=0;i<10;i++)
    {
        {
            unique_lock<StockSafe> lock(safe);
            StockBlackboard& stock= safe.Acquire(lock);
            stock.price = abs(rand()%10)*10;
            cout<<"Peter udpated the price to $"<<stock.price<<endl;
            if(stock.price >= 90)
            {
                lock.unlock();
                cout<<"Peter notified Danny at price $"<<stock.price<<", ";
                condition.notify_one();
                this_thread::sleep_for(std::chrono::milliseconds(10));
                lock.lock();
            }
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(rand()%10));
    }
}
void DannyWait_ReadStock(StockSafe & safe, condition_variable& priceCondition)
{
    unique_lock<mutex> lock(safe.Mutex());
    cout<<"Danny is waiting for the right price to sell..."<<endl;
    priceCondition.wait(lock); /* 在等待期间，线程会释放与之关联的互斥锁，以便其他线程可以访问共享资源 */
    StockBlackboard& stock= safe.Acquire(lock);
    cout<<"Danny sell at: $"<<stock.price<<endl;
}

int main()
{
    StockSafe safe (new StockBlackboard("APPL",30));
    condition_variable priceCondition;
    thread DannyThread(DannyWait_ReadStock, std::ref(safe), std::ref(priceCondition));
    thread PeterThread( PeterUpdateStock_Notify, std::ref(safe), std::ref(priceCondition));
    PeterThread.join();
    DannyThread.join();
    return 0;
}
```

## 线程安全消息队列

当Danny要关注多只股票时，他希望能收到符合要求的消息，于是有了消息队列：
1. 至少一个生产者线程和一个消费者线程，任何时候只有一个线程能访问消息队列，意味着我们必须有唯一的锁保护这个队列。
2. 当消息队列空时，消费线程阻塞；当消息队列满时，生产线程阻塞

```c++
#include <mutex>
#include <condition_variable>
#include <thread>
#include <iostream>
#include <queue>
#include <functional>
using namespace std;

template <typename MsgType>
class MsgQueue
{
private:
    queue<MsgType> _queue; 
    mutex _mutex;
    condition_variable _enqCv;
    condition_variable _deqCv;
    int _limit;
public:
    MsgQueue(int limit=3):_limit(limit){}
    void Enqueue(MsgType & msg)
    {
        unique_lock<mutex> lock(_mutex);
        if(_queue.size()>=_limit)
        {
            cout<<"queue is full, wait()..."<<endl;
            _enqCv.wait(lock,[this]{return _queue.size()<_limit;});
        }
        _queue.push(msg);
        _deqCv.notify_one();
    }
    MsgType Dequeue()
    {
        unique_lock<mutex> lock(_mutex);
        if(_queue.size()<=0)
        {
            cout<<"queue is empty, wait()..."<<endl;
            _deqCv.wait(lock, [this]{return _queue.size()>0;});
        }
        MsgType& msg = _queue.front();
        _queue.pop();
        _enqCv.notify_one();
        return msg;
    }
    int Size()
    {
        unique_lock<mutex> lock(_mutex);
        return _queue.size();
    }
};

struct StockCommand
{
    string command;
    float price;
    StockCommand(){}
    StockCommand(const StockCommand& cp):command(cp.command),price(cp.price){}
    void ExecuteCommand()
    {
        if(price>0)
            cout<<"Command "<<command<<" is executed at price $"<<price<<endl;
        else
            cout<<"market closed because the price is $"<<price<<endl;
    }
};

typedef MsgQueue<StockCommand> TaskQueueType;

void Danny_ReadStockMsgQueue(TaskQueueType &taskQueue)
{
    while(1)
    {
        StockCommand task=taskQueue.Dequeue();
        task.ExecuteCommand();
        if(task.price<0)
        {
            cout<< "thread finshed!"<<endl;
            break;
        }
        // the sleep function simulates that the task takes a while to finish 
        std::this_thread::sleep_for(std::chrono::milliseconds(rand()%100));
    }
}

int main()
{
    TaskQueueType taskQueue(2);
    thread DannyThread(Danny_ReadStockMsgQueue, std::ref(taskQueue));

    for(int i=0;i<10;i++)
    {
        StockCommand task;
        task.price= i+1;
        if(task.price >5)
            task.command = "sell at price $";
        else
            task.command = "buy at price $";    
        taskQueue.Enqueue(task);
    }
    StockCommand marketCloseCommand;
    marketCloseCommand.command="Market Closed!";
    marketCloseCommand.price=-1;
    taskQueue.Enqueue(marketCloseCommand);

    PeterThread.join();
    return 0;
}
```

## 线程池

Danny只是一个人，当消息队列有大量消息时，需要成千上万个Danny？

引入线程池

```c++
#include <mutex>
#include <condition_variable>
#include <thread>
#include <iostream>
#include <queue>
#include <functional>
using namespace std;

template <typename MsgType>
class MsgQueue
{
private:
    queue<MsgType> _queue; 
    mutex _mutex;
    condition_variable _enqCv;
    condition_variable _deqCv;
    int _limit;
public:
    MsgQueue(int limit=3):_limit(limit){}
    void Enqueue(MsgType & msg)
    {
        unique_lock<mutex> lock(_mutex);
        if(_queue.size()>=_limit)
        {
            cout<<"queue is full, wait()..."<<endl;
            _enqCv.wait(lock,[this]{return _queue.size()<_limit;});
        }
        _queue.push(msg);
        _deqCv.notify_one();
    }
    MsgType Dequeue()
    {
        unique_lock<mutex> lock(_mutex);
        if(_queue.size()<=0)
        {
            cout<<"queue is empty, wait()..."<<endl;
            _deqCv.wait(lock, [this]{return _queue.size()>0;});
        }
        MsgType& msg = _queue.front();
        _queue.pop();
        _enqCv.notify_one();
        return msg;
    }
    int Size()
    {
        unique_lock<mutex> lock(_mutex);
        return _queue.size();
    }
};

struct StockCommand
{
    string command;
    float price;
    StockCommand(){}
    StockCommand(const StockCommand& cp):command(cp.command),price(cp.price){}
    void ExecuteCommand()
    {
        if(price>0)
            cout<<"Command "<<command<<" is executed at price $"<<price<<endl;
        else
            cout<<"market closed because the price is $"<<price<<endl;
    }
};

typedef MsgQueue<StockCommand> TaskQueueType;

class ThreadPool
{
private:
    int _limit;
    vector<thread*> _workerThreads;
    TaskQueueType& _taskQueue;
    bool _threadPoolStop=false;
public:
  
    void ExecuteCommand()
    {
        while(1)
        {
            StockCommand task=_taskQueue.Dequeue();
            task.ExecuteCommand();
            if(task.price<0)
            {
                _threadPoolStop=true;
                _taskQueue.Enqueue(task);// tell other threads to stop
            }
            if(_threadPoolStop)
            {
                cout<< "thread finshed!"<<endl;
                return;
            }
            // the sleep function simulates that the task takes a while to finish 
            std::this_thread::sleep_for(std::chrono::milliseconds(rand()%100));
        }
    }

    ThreadPool(TaskQueueType& taskQueue,int limit=3):_limit(limit),_taskQueue(taskQueue){
        for(int i=0;i<_limit;++i)
        {
            _workerThreads.push_back(new thread(&ThreadPool::ExecuteCommand,this));
        }
    }
    ~ThreadPool(){
        for(auto threadObj: _workerThreads)
        {
            if(threadObj->joinable())
                {
                    threadObj->join();
                    delete threadObj;
                }
        }
    }
};

int main()
{
    TaskQueueType taskQueue(5);
    ThreadPool pool(taskQueue,3);
    // the sleep function simulates the situation that all  
    // worker threads are waiting for the empty message queue at the beginning
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    for(int i=0;i<10;i++)
    {
        StockCommand task;
        task.price= i+1;
        if(task.price >5)
            task.command = "sell at price $";
        else
            task.command = "buy at price $";    
        taskQueue.Enqueue(task);
    }
    StockCommand marketCloseCommand;
    marketCloseCommand.command="Market Closed!";
    marketCloseCommand.price=-1;
    taskQueue.Enqueue(marketCloseCommand);
    return 0;
}
```
