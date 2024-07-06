# async用法
> `async` 是一个用于异步执行函数的模板函数,返回一个`std::future`对象，该对象用于获取函数的返回值


## aysnc的启动策略

`std::async`函数可以接受几个不同的启动策略，这些策略在`std::launch`枚举中定义。除了`std::launch::async`之外，还有以下启动策略：

1. `std::launch::deferred`：这种策略意味着任务将在调用`std::future::get()`或`std::future::wait()`函数时延迟执行。换句话说，任务将在需要结果时同步执行。
2. `std::launch::async | std::launch::deferred`：这种策略是上面两个策略的组合。任务可以在一个单独的线程上异步执行，也可以延迟执行，具体取决于实现。

默认情况下，`std::async`使用`std::launch::async | std::launch::deferred`策略。这意味着任务可能异步执行，也可能延迟执行，具体取决于实现。需要注意的是，不同的编译器和操作系统可能会有不同的默认行为。