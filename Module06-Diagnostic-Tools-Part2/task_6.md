# Refs:
* Counters
  * https://github.com/dotnet/diagnostics/blob/main/documentation/design-docs/eventcounters.md
  * https://docs.microsoft.com/en-us/dotnet/core/diagnostics/available-counters
  * https://github.com/dotnet/runtime/blob/main/src/libraries/System.Diagnostics.Tracing/documentation/EventCounterTutorial.md
  * https://github.com/dotnet/diagnostics/blob/main/documentation/dotnet-trace-instructions.md#using-dotnet-trace-to-collect-counter-values-over-time
* Dumps
  * https://kevingosse.medium.com/analyze-your-memory-dumps-in-c-with-dynamd-8e4b110b9d3a
  * https://devblogs.microsoft.com/visualstudio/managed-memory-dump-analyzers/
  * https://docs.microsoft.com/en-us/dotnet/core/diagnostics/debug-linux-dumps
  * https://www.poppastring.com/blog/collecting-managed-crash-dumps-on-app-services-for-linux
  * https://devblogs.microsoft.com/dotnet/gc-perf-infrastructure-part-1/
  * https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/botr/xplat-minidump-generation.md
  * https://github.com/dotnet/diagnostics/blob/main/documentation/dotnet-dump-instructions.md
  * https://www.tessferrandez.com/blog/2021/03/18/debugging-a-netcore-memory-issue-with-dotnet-dump.html
  * https://www.tessferrandez.com/blog/2008/02/04/debugging-demos-setup-instructions.html
* Profiles and application performance monitoring
  * https://github.com/dotnet/crank
    * https://github.com/dotnet/crank/blob/main/docs/README.md
    * https://docs.microsoft.com/en-us/events/dotnetconf-2021/benchmarking-aspnet-applications-with-net-crank
    * https://developpaper.com/usage-of-net-performance-testing-framework-crank/
    * https://docs.microsoft.com/en-us/aspnet/signalr/overview/performance/signalr-connection-density-testing-with-crank
  * https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/botr/profiling.md
  * https://chnasarre.medium.com/start-a-journey-into-the-net-profiling-apis-40c76e2e36cc
  * https://chnasarre.medium.com/dealing-with-modules-assemblies-and-types-with-clr-profiling-apis-a7522a5abaa9
  * https://blog.elmah.io/debugging-system-outofmemoryexception-using-net-tools/
* Cloud and containers
  * https://github.com/dotnet/diagnostics/blob/main/documentation/design-docs/ipc-protocol.md
  * https://github.com/dotnet/runtime/blob/main/docs/project/linux-performance-tracing.md
  * https://docs.microsoft.com/en-us/dotnet/core/diagnostics/diagnostics-in-containers#using-net-core-cli-tools-in-a-container
  * https://github.com/dotnet/dotnet-docker
  * https://devblogs.microsoft.com/dotnet/introducing-dotnet-monitor/
  * https://docs.microsoft.com/en-us/dotnet/core/diagnostics/trace-perfcollect-lttng
  * https://devblogs.microsoft.com/dotnet/running-with-server-gc-in-a-small-container-scenario-part-0/
  * https://devblogs.microsoft.com/dotnet/running-with-server-gc-in-a-small-container-scenario-part-1-hard-limit-for-the-gc-heap/

# HT
## app counters
* how does "% Time in GC since last GC (%)" look?
  * In my case, it is just steady and precisely 0.
* how does "Allocation Rate (B / 1 sec)" look? Is it stable?
  * Allocation Rate fluctuates between close to 0 M and slightly under 4.5 M drastically. Hence, it doesn't look level off. (M - MB)
    * Corrections: Actually, we can consider it as a constant, something ~4MB/second.
* how does "ThreadPool Thread Count" look? Is the number stable?
  * In the beginning, the thread count reached its peak that was slightly above 35, and then it plumed to 28 and then to 20. After that, it was fluctuating between 17 and 21 for the rest of the time. Those are peaks' fluctuations, however, throughout a given span the bottom was 16, and during that, those peaks were fluctuating between 16 and themself.
    * Conclusion: the app was busy!
* how does "ThreadPool Queue Length" look? Is there any excessive queuing?
  * most of the time it was about 0, however in the middle of testing, there are two spikes with 2 and 1 numbers, respectively.
  * I can't say that there is any excessive queuing.
    * Conclusion: it looks ok, the app is able to "catch up" 
    * Note: this is **very important** when load testing. Always check that if you are not overloading a testing app that is not capable to cope with that amount of load, and you just start to measure some artificial, overloaded state of application.
* Let's do some additional sanity checks:
  * check also "type loader"-related counters like "IL Bytes Jitted (B)", "Number of AssembliesLoaded" and "Number of Methods Jitted" - they should be stable to confirm there is no "dynamic type generation" leak
    * yes, they are stable ===> there **is not "dynamic type generation" leak**
  * check "CPU Usage (%)" to make sure the traffic is not too big and the app is able to handle it 
    * most of the time it was fluctuating between 1-3, but there are several spikes that went to 24, 39 and 41.
    * Conclusion: traffic is not too big and app is able to handle it!
  * also "ThreadPool Completed Work Item Count (Count / 1 sec)" should be pretty stable, showing that the application is constantly in a healthy state
    * the main trend fluctuated between 1000 and 1200 range. (nevertheless, there are some significant declines to literally to 0, which is strange);
    * Conclusion: app is constantly in a healthy state, it is stable ~1100.
  * the last but not the least, check "Exception Count (Count / 1 sec)" to make sure there is no excessive exception handling in the app
    * not at all, it is exactly at 0 level.
* select "GC Heap Size (MB)" and "Working Set (MB)" counters. What's the behavior? Is it aligned to what we've seen under https://localhost:5001/ graph?
  * I have not constantly looked at that graph, but it shows the same pattern.
  * it shows that cleaning Managed Heap doesn't reclaim memory to operating system. This is because GC internally prefers to **reuse** memory instead of constantly getting it from the operating system. 
* A bonus. Select "Gen 0 Size (B)", "Gen 1 Size (B)", "Gen 2 Size (B)", "LOH Size (B)" and "POH Size (B)" counters. Although we haven't touched the topic of generations, yet - just look what are the reported sizes? What's the maximum?
  * maximums (B):
    * Gen 0 Size: 18.7 M
    * Gen 1 Size: 4.5 M
    * Gen 2 Size: 4.2 M
    * LOH Size: 98.7 K
    * POH 1 Size: 769.4 K   
## app dumps
* Open recorded task2.csv in a tool like https://www.csvplot.com/. The moment of taking a dump should be clearly visible? How? 
  * I took 3 dumps and I don't see in a graph at what moment those were taken. I can guess that allocation rate jumps are the moments of collecting gc-dumps.
    * I guess I find this noticeable thing though I didn't notice it. Basically, here is the quote from docs:   
  [```To walk the GC heap, this route triggers a generation 2 (full) garbage collection, which can suspend the runtime for a long time, especially when the GC heap is large. Don't use this route in performance-sensitive environments when the GC heap is large.```](https://github.com/dotnet/dotnet-monitor/blob/main/documentation/api/gcdump.md)  
  Based on that information, I should have seen some collecting of gen2. 
    * Answer: we shouldn't take a dump at some random time, instead of it, we should take it when the GC happens in order to get as much information(what will be collected and what not.) ===> **gcdump is triggering GC** to make sure that we will get these data. ===> I should see Full GC!!!
* As a result from the dotnet-gcdump command we should have a  le like 20211126_173518_19000.gcdump. Open it in PerfView and open (the only one) Heap Stacks view. There, look around in the RefTree tab that allows to top-down analysis of the memory usage. What's taking up the most memory?
  * In my case,  `Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets!Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets.Internal.SocketConnection` was an object that has the most inclusive cost among others (it is from `[static vars]`). Note it contains `Microsoft.AspNetCore.Server.Kestrel.Core!Microsoft.AspNetCore.Server.Kestrel.Https.Internal.SslDuplexPipe` that costs the maximum of its child objects.
    * static vars that are used by ASP.NET hosting infrastructure to keep internal buffers
    * GC is able to clear the most of the managed objects.
    * memory usage is small
* Is there something "not reachable" in the Managed Heap? Clear [not reachable from roots] text from ExcPats input at the top of the Heap Stacks dialog.
  * I have not find any difference after clearing the text form `ExcPats`, which means there are probably no "not reachable" smt.
  * Conclusion:  GC cleaned everything and there was no "floating garbage" so apparently there were no not reachable objects in memory.
* Open .gcdump file in Visual Studio to see how it is presented there.
  * In VS I was able to reach some level of detalization, though it was not as detailed as in perfView. But I noticed one line that I was not able to find in my perfView  
  (Byte[] (Bytes > 100K)	21	1179864	1179864)




