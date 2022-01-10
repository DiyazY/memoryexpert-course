Refs:
* https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals
* https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/
* https://devblogs.microsoft.com/premier-developer/understanding-different-gc-modes-with-concurrency-visualizer/
* https://www.red-gate.com/products/dotnet-development/ants-memory-profiler/learning-memory-management/memory-management-fundamentals
* https://devblogs.microsoft.com/dotnet/the-history-of-the-gc-configs/
* https://docs.microsoft.com/en-us/dotnet/core/run-time-config/garbage-collector


## HT
|GC Mode|GC concurrency|delays between calls|concurrency|Allocation Budget|Response time, median(ms)|Description|
|---|---|---|---|---|---|---|
|Server|Concurrent|64|100|~250MB|1|Allocation Budget grown from 70MB to 250MB. Mostly, it handles all request in 1 ms and all GCs(mostly gen0) were predictable. the pattern was very repeatable, and GC happened around 150mb|
|Server|Concurrent|1000|10|~800MB|1-6|Allocation Budget grown from 70MB to 800MB. Notably, the app allocates a lot of memory and the allocated memory grows steadily. GCs happen very rarely (noticed only gen0), but when it happens the response time degrades drastically. |
|Workstation|Non-Concurrent|64|100|100MB|1|A lot of GCs happen(gen0), CPU usage is getting more extensive. allocation is fluctuating between 10 and 20MB, though the budget is 100MB.|
|Workstation|Non-Concurrent|1000|10|200mb|3000-10000|a lot of GCs happen(gen0, gen1, gen2). Responsiveness is downgraded drastically. There is no clear pattern in allocation, the memory consumption is always around 100mb| 