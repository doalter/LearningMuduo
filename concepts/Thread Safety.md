## Thread Safety

### Definition of thread safety
A thread-safe class should satisfy three conditions:
+ It behaves correctly when accessed from multiple threads,
+ regardless of the scheduling or interleaving of the execution of those threads by the runtime environment,
+ and with no additional synchronization or other coordination on the part of the calling code.

Briefly speaking, if the calling code behaves correctly ***without using lock***, then it's thread safe.

### Thread safety for constructor
The only restriction for thread safe constructor is not leaking ``this pointer`` while constructing.
That means:
+ Do NOT register any callback function in constructor;
+ Do NOT transfer this pointer to any cross-thread object;
Why? This is because other objects may access to the object that incompletely constructed.

### Thread safety for destructor
A thread safe destructor is much more complicated.  
Normally we use mutex to protect the critical section, but destructor will delete the mutex. It sucks!  
Let's say we use mutex to protect a destruct process (free resources etc).  
Consider this scenario when thread A destruct an object and automatically delete the mutex, while thread B is waiting for this mutex. It's a disaster! Thread B would never get the mutex!

Mutex is not the way to assure the thread safety of destructor.

### Why shared_ptr / weak_ptr?
Take Observer model as example. What would happen if the observer object is about to destruct and the observable object need to notity observer a update? Core dump!
So we need a mechanism to let us know whether an object is still alive or not.
Then we got weak_ptr. If the object pointed by a weak_ptr is dead, the weak_ptr will be aware of that and the promoted shard_ptr will point to null. Here in the observable calss we don't use shared_ptr because shared_ptr can lead to the immortality of the observer objects.

shared_ptr/weak_ptr solved the problem that we can destruct an object safely, because shared_ptr makes sure the object is not in use before destructing, and we don't need a mutex in destructor.

But the thread safty problem still exists when multiple threads read/write a shared_ptr pointed object simultaneously, and here we can use mutex safely.

Note that shared_ptr itself is not a thread_safe object as it is just a normal object which has two members(pointer and count), which means shared_ptr needs be procted by mutex in multi-threads.

We can do it this way:
```C++
shared_ptr<Foo> globalPtr;

void read()
{
    shared_ptr<Foo> localPtr;
    {
        MutexLockGuard lock(mutex);
        localPtr = globalPtr;
    }
    doRead(localPtr); //doRead and doWrite also needs a mutex to protect the data pointed by ptr
}

void write()
{
    shared_ptr<Foo> newPtr(new Foo);
    {
        MutexLockGuard lock(mutex);
        globalPtr = newPtr;
    }
    doWrite(newPtr); //doRead and doWrite also needs a mutex to protect the data pointed by ptr
}
```