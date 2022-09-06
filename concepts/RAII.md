## RAII: Resource Acquisition Is Initialization
### What is RAII?
A C++ programming technique which guarantees that the resource is available to any function that may access the object.

RAII can be summarized as follows:
>
> + encapsule each resource into a class, where
>   + the constructor acquires the resource and establishes all class invariants or throws an exception if that cannot be done,
>   + the destructor releases the resource and never throws exceptions;
> + always use the resource via an instance of a RAII-class that either
>    + has automatic storage duration or temporary lifetime itself, or 
>    + has lifetime that is bounded by the lifetime of an automatic or temporary object

### RAII example in Muduo
```C++
//Currently in c++, we use "delete" to delete the copy constructor
//noncopyable actually makes the copy constructor and the assignment operator private.
class MutexLockGuard : boost::noncopyable
{
public:
    explicit MutexLockGuard(MutexLock& mutex) : mutexlock_(mutex)
    {
        mutexlock_.lock();
    }

    ~MutexLockGuard()
    {
        mutexlock_.unlock()
    }
private:
    MutexLock& mutexlock_;
}

class MutexLock() : boost::noncopyable  
{
public:
    MutexLock() : holder(0) { pthread_mutex_init(&mutex_, NULL); }

    ~MutexLock()
    {
        assert(holder_ == 0);
        pthread_mutex_destroy(&mutex_);
    }

    void lock()
    {
        pthread_mutex_lock(&mutex_);
        holder_ = CurrentThread::tid(); //Muduo method. Actually gettid(), get unique tid of a thread.
    }

    void unlock()
    {
        holder_ = 0; //tid value 0 is an illegal value
        pthread_mutex_unlock(&mutex_);
    }

private:
    pthread_mutex_t mutex_;
    pid_t holder_;
}
```
Let's take a look at class MutexLockGuard. We use it as follows:
```C++
boo()
{
    MutexLockGuard lock(mutexlock); // Normally the parameter mutexlock is a private member of a Class.
    do sth; //Modify some sharing data
}
```
In the function boo(), we initialize ``lock`` which meanwhile get the mutex and lock it. When we leave the boo function, the destructor automatically unlock the mutex. This is useful when there is an exception in ``do sth`` and returns. The destructor will always call the unlock no matter where and when the function returns.

Actually, class MutexLock is also an example of RAII. It creates the mutex by constructor and destroy it by destructor. If an object holds the mutex releases, then the mutex is destroyed automatically.
