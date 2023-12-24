# task 2

## 任务要求
This component is responsible for scheduling read and write operations on the DiskManager.  You will implement a new class called DiskScheduler in *src/include/storage/disk/disk_scheduler.h* and its corresponding implementation file in *src/storage/disk/disk_scheduler.cpp*.
该组件负责调度对DiskManager的读写操作。您将在src/include/storage/disk/disk_scheduler.h中实现一个名为DiskScheduler的新类，并在src/storage/disk/disk_scheduler.cpp中实现相应的实现文件。


The disk scheduler can be used by other components (in this case, your BufferPoolManager in Task #3) to *queue disk requests*, representedby a DiskRequest struct (already defined in *src/include/storage/disk/disk_scheduler.h*).  The disk scheduler will maintain a background worker thread which is responsible for processing scheduled requests.
磁盘调度器可以被其他组件(在本例中是任务#3中的BufferPoolManager)用于对磁盘请求进行排队，由DiskRequest结构(已经在src/include/storage/disk/disk_scheduler.h中定义)表示。磁盘调度器将维护一个负责处理调度请求的后台工作线程。


The disk scheduler will utilize a shared queue to schedule and process the DiskRequests.  One thread will add a request to the queue, and the disk scheduler's background worker will process the queued requests.  We have provided a Channel class in src/include/common/channel.h to facilitate the safe sharing of data between threads, but feel free to use your own implementation if you find it necessary.


磁盘调度器将利用`shared queue`来调度和处理`diskrequest`。一个线程`request`添加到队列中，磁盘调度器的后台工作程序将处理排队的请求。我们已经在src/include/common/Channel.h中提供了一个Channel类，以促进线程间数据的安全共享，但如果您觉得有必要，可以自由地使用您自己的实现。


The DiskScheduler constructor and destructor are already implemented and are responsible creating and joining the background worker thread.  You will only need to implement the following methods as defined in the header file (src/include/storage/disk/disk_scheduler.h) and in the source file (src/storage/disk/disk_scheduler.cpp):


DiskScheduler构造函数和析构函数已经实现，它们负责创建和加入后台工作线程。你只需要实现头文件(src/include/storage/disk/disk_scheduler.h)和源文件(src/storage/disk/disk_scheduler.cpp)中定义的以下方法:

- Schedule(DiskRequest r) : Schedules a request for the DiskManager to execute.  The DiskRequest struct specifies whether the request is for a read/write, where the data should be written into/from, and the page ID for the operation.  The DiskRequest also includes a std::promise whose value should be set to true once the request is processed.调度DiskManager执行的请求。DiskRequest结构指定请求是否为读/写，数据应该写入/从哪里写入，以及操作的页ID。DiskRequest还包括一个std::promise，一旦请求被处理，它的值应该被设置为true


- StartWorkerThread() : Start method for the background worker thread which processes the scheduled requests.  The worker thread is created in the DiskScheduler constructor and calls this method.  This method is responsible for getting queued requests and dispatching them to the DiskManager.  Remember to set the value on the DiskRequest's callback to signal to the request issuer that the request has been completed.  This should not return until the DiskScheduler's destructor is called.处理调度请求的后台工作线程的启动方法。工作线程在DiskScheduler构造函数中创建并调用此方法。该方法负责获取排队的请求并将其分配给DiskManager。记住要在DiskRequest的回调上设置值，以向请求发出请求已经完成的信号。在调用DiskScheduler的析构函数之前，这个函数不应该返回


Lastly, one of the fields of a DiskRequest is a std::promise.  If you are unfamiliar with C++ promises and futures, you can check out their documentation.  For the purposes of this project, they essentially provide a callback mechanism for a thread to know when their scheduled request is completed.  To see an example of how they might be used, check out disk_scheduler_test.cpp.

Again, the implementation details are up to you, but you must make sure that your implementation is thread-safe.