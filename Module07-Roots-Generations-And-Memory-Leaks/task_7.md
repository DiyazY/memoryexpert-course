# Refs:
* https://tooslowexception.com/pinned-object-heap-in-net-5/
* https://devblogs.microsoft.com/dotnet/provisional-mode/
* https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector
* https://devblogs.microsoft.com/dotnet/the-updated-getgcmemoryinfo-api-in-net-5-0-and-how-it-can-help-you/

## HT
# IMPORTANT: I guess on arm chip it doesn't really work.
TODO: try it on Parallel VM
### LOH profiling
* I was not able to trigger AllocLarge GC by this: `dotnet SuperBenchmarker.dll -y 100 -n 10000000 -c 64 -u http://localhost:5000/api/loh`
* Interesting fact that many times the `Working Set` was passed by `Allocated` and the GC was not triggered. 
  * This strange behavior could be caused by the usage of ARM processor. Moreover, the original apps didn't support .net6 which is compatible with arm processor. Thus I forced them to be .net6 applications.
* this `dotnet SuperBenchmarker.dll -y 10 -n 10000000 -c 64 -u http://localhost:5000/api/loh` doesn't triggered `AllocLarge` as well.
* Here is the report from `dotnet-gcmon`
```
Monitoring process with name: MemoryLeak and pid: 8702
GC#     index |            type |   gen | pause (ms) |                reason | gen0 size (mb) | LOH size (mb) | peak/after | gen2 size (mb) |
---------------------------------------------------------------------------------------------------------------------------------------------
GC#         1 | NonConcurrentGC |     0 |      45.88 |            AllocSmall |          7.379 |         0.419 |       5.01 |          0.000 |
GC#         2 | NonConcurrentGC |     1 |      94.28 |            AllocSmall |         34.139 |         0.419 |       4.13 |          4.037 |
GC#         3 | NonConcurrentGC |     0 |      61.33 |            AllocSmall |          0.000 |         0.419 |      17.24 |          4.037 |
------------------------------------------------------------------------------
Heap Stats as of 2023-02-26 13:38:34Z (Run 3 for gen 0):
  Heaps: 10
  Handles: 15,644
  Pinned Obj Count: 0
  Last Run Stats:
    Total Heap: 8,251,360 Bytes
      Gen 0:               240 Bytes
      Gen 1:         3,243,832 Bytes
      Gen 2:         4,037,352 Bytes
      Gen 3:           419,392 Bytes
      Gen 4:           550,544 Bytes
------------------------------------------------------------------------------
GC#         4 | NonConcurrentGC |     2 |      55.01 |            AllocSmall |          0.000 |         0.419 |      14.71 |          5.004 |
GC#         5 | NonConcurrentGC |     0 |      58.55 |            AllocSmall |          0.000 |         0.419 |      11.68 |          5.004 |
GC#         6 | NonConcurrentGC |     0 |      42.26 |            AllocSmall |         30.432 |         0.419 |       2.50 |          5.004 |
GC#         7 | NonConcurrentGC |     1 |      52.03 |            AllocSmall |         26.559 |         0.419 |       3.41 |          7.341 |
GC#         8 | NonConcurrentGC |     0 |      46.05 |            AllocSmall |         10.762 |         0.419 |       4.98 |          7.341 |
------------------------------------------------------------------------------
Heap Stats as of 2023-02-26 13:50:35Z (Run 8 for gen 0):
  Heaps: 10
  Handles: 38,057
  Pinned Obj Count: 1
  Last Run Stats:
    Total Heap: 23,782,344 Bytes
      Gen 0:        10,761,792 Bytes
      Gen 1:         2,344,896 Bytes
      Gen 2:         7,340,840 Bytes
      Gen 3:           419,392 Bytes
      Gen 4:         2,915,424 Bytes
------------------------------------------------------------------------------
GC#         9 | NonConcurrentGC |     0 |      54.50 |            AllocSmall |          9.965 |         0.419 |       4.30 |          7.341 |
GC#        10 | NonConcurrentGC |     2 |      35.51 |            AllocSmall |          9.832 |         0.419 |       5.27 |          4.397 |
```
* what are the result? What GC are triggered and why? What are the generation sizes in time?
  * All GCs were triggered by `AllocSmall` and on gen0, gen1, gen2.