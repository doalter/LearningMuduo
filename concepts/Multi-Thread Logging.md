## Multi-Thread Logging

### Requirements of Logging
***Contents***  
For key processes, normally we should log the following info:  
1. id of every received internal message;
2. contents of every received eternal message;
3. contents of every message sent out;
4. key interal status changes

***Format***
1. time stamp;
2. thread id;
3. file name;
4. line number;
5. log level

***Function Requirements***
1. muliple producers(produce log messages) and single consumer(write messages to files);
2. rolling (delete old log files)

***Performance Requirments***
1. one million messages per second;
2. not block the business process
3. not lead to contention in multiple threads

### Logging Implemetation in Muduo
First, take a look at the code.
```C++
#define LOG_INFO if (muduo::Logger::logLevel() <= muduo::Logger::INFO) \
  muduo::Logger(__FILE__, __LINE__).stream()
```
So ``LOG_INFO << "log info!";``expans to
```C++
if (muduo::Logger::logLevel() <= muduo::Logger::INFO)
    muduo::Logger(__FILE__, __LINE__).stream() << "log info!";
```
In the code above, ``logLevel()`` is a static function which returns a global variable that indicates the log level. Log level is initialized as INFO by default, and can be also set by ``setLogLevel()``.
```C++
Logger::Logger(SourceFile file, int line)
  : impl_(INFO, 0, file, line)
{
}
```
Logger constructor constructs a SourceFile object and an Impl object. SourceFile stores the file name and the length (of the file name).  
The Impl object is the one who do the specific log thing. In its construcor, it outputs the timestamp, thread id and log level.  
And then ``stream()`` returns a LogStream object used to output the log info.
```C++
LogStream& stream() { return impl_.stream_; }
```
LogSteam is a class to output the log messages to buffer. (See chapter 11.6 in the book)

### Async Logging in Muduo

