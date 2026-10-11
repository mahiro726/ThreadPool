# ThreadPool

基于 C++ 和 POSIX pthread 实现的模板线程池。线程池以单例方式获取，默认创建 5 个工作线程；任务通过队列分发，工作线程在队列为空时休眠，并在有新任务时被唤醒。停止线程池时会先处理队列中尚未完成的任务，再退出工作线程。

## 功能

- 使用 `ThreadPool<T>` 管理可调用任务，并通过 `Enqueue()` 提交任务。
- 使用互斥锁和条件变量保护任务队列、休眠和唤醒流程。
- 通过 `Stop()` 通知工作线程结束，通过 `Wait()` 等待工作线程退出。
- 提供 pthread 线程、互斥锁、作用域锁和条件变量的封装。
- 提供带时间、日志级别、进程号和源码位置的日志，支持控制台和文件输出。

## 环境要求

- Linux/POSIX 环境及支持 pthread 的 C++ 编译器。
- C++17 或更高版本（日志文件策略使用 `std::filesystem`）。
- 链接 pthread。

## 快速开始

```cpp
#include <functional>
#include "ThreadPool.hpp"

using namespace NS_LOG_MODULE;

int main()
{
	ENABLE_CONSOLE_LOG_STRATEGY();

	auto *pool = ThreadPool<std::function<void()>>::GetInstance();
	pool->Enqueue([] {
		LOG(LogLevel::INFO) << "task is running";
	});

	pool->Stop();
	pool->Wait();
	return 0;
}
```

例如将代码保存为 `main.cpp`，可使用以下命令编译：

```sh
g++ -std=c++17 -pthread -I. main.cpp -o thread_pool_demo
```

`GetInstance()` 首次调用时会创建并启动线程池。上例中的 `Stop()` 会唤醒工作线程并让它们处理完已排队任务；随后 `Wait()` 等待所有工作线程结束。

## 日志

默认不会自动选择日志输出策略。可在提交任务前启用控制台输出：

```cpp
ENABLE_CONSOLE_LOG_STRATEGY();
```

或启用文件输出：

```cpp
ENABLE_FILE_LOG_STRATEGY();
```

文件日志默认写入 `./log/log.txt`。使用 `LOG(level)` 可构造日志，例如 `LOG(LogLevel::INFO) << "message";`。
