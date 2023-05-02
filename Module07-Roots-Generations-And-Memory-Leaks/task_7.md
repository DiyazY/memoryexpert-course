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
  * This strange behavior could be caused by the usage of ARM processor. Moreover, the original apps didn't support .net5 which is compatible with arm processor. Thus I forced them to be .net6 applications.
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


## on proper Windows machine
### TASK 1.1
- what are the result? What GC are triggered and why? What are the generation sizes in time?
  - The test showed a lot of GC Gen2 which were triggered by `AllocLarge` and done by `BackgroundGC`. Rarerly, the GC Gen0 were triggered by `AllocSmall` and done by `NonConcurrentGC`.  As we can see, `BacgroundGC` for Gen2 takes signifinltly more time than for Gen0(3 times more). While Gen2 GC was triggered a lot, it is clear that the pressure was on LOH, on a table, we can see how the size of LOH flactuates drastically while gen2 size is the same and gen0 is going up.
```
Monitoring process with name: MemoryLeak and pid: 17056  
| GC#     index |            type |   gen | pause (ms) |                reason | gen0 size (mb) | LOH size (mb) | peak/after | gen2 size (mb) |
---------------------------------------------------------------------------------------------------------------------------------------------
GC#       132 |    BackgroundGC |     2 |       5,84 |            AllocLarge |          1,712 |         5,970 |       0,00 |          6,360 |
...
GC#       311 | NonConcurrentGC |     0 |       1,84 |            AllocSmall |          0,006 |        12,774 |       1,39 |          6,360 |
GC#       312 |    BackgroundGC |     2 |       6,23 |            AllocLarge |          0,643 |         2,908 |       1,74 |          6,360 |
...
GC#       313 |    BackgroundGC |     2 |       4,70 |            AllocLarge |          1,478 |         9,457 |       0,90 |          6,360 |
GC#       314 |    BackgroundGC |     2 |       8,35 |            AllocLarge |          2,603 |         4,099 |       1,23 |          6,360 |
GC#       315 |    BackgroundGC |     2 |       5,37 |            AllocLarge |          1,786 |         4,949 |       0,98 |          6,360 |
GC#       316 |    BackgroundGC |     2 |       5,13 |            AllocLarge |          2,663 |         9,202 |       0,90 |          6,360 |
GC#       317 |    BackgroundGC |     2 |       5,14 |            AllocLarge |          3,332 |         4,269 |       1,21 |          6,360 |
```
Over time, on a printed info page we can see that Gen 0 and Gen 3 changes their sizes frequently. 
  ```
  Heap Stats as of 2023-04-16 12:50:37Z (Run 557 for gen 2):  
  Heaps: 8  
  Handles: 1 064  
  Pinned Obj Count: 0  
  Last Run Stats:  
    Total Heap: 12 966 584 Bytes  
      Gen 0:         4 342 784 Bytes  
      Gen 1:           195 416 Bytes  
      Gen 2:         6 360 336 Bytes  
      Gen 3:         1 036 528 Bytes  
      Gen 4:         1 031 520 Bytes  

  Heap Stats as of 2023-04-16 12:50:40Z (Run 562 for gen 2):
  Heaps: 8
  Handles: 1 064
  Pinned Obj Count: 0
  Last Run Stats:
    Total Heap: 20 036 512 Bytes
      Gen 0:         8 095 528 Bytes
      Gen 1:           195 416 Bytes
      Gen 2:         6 360 336 Bytes
      Gen 3:         4 353 712 Bytes
      Gen 4:         1 031 520 Bytes
  
  Heap Stats as of 2023-04-16 12:50:42Z (Run 567 for gen 0):
  Heaps: 8
  Handles: 1 073
  Pinned Obj Count: 0
  Last Run Stats:
    Total Heap: 11 369 256 Bytes
      Gen 0:               192 Bytes
      Gen 1:           218 888 Bytes
      Gen 2:         6 360 336 Bytes
      Gen 3:         3 758 320 Bytes
      Gen 4:         1 031 520 Bytes
  
  ```
### TASK 1.2
```
Monitoring process with name: MemoryLeak and pid: 15128
GC#     index |            type |   gen |                reason | peak size (mb) | promoted (mb) | LOH size (mb) | LOH survival rate | LOH frag ratio |
-------------------------------------------------------------------------------------------------------------------------------------------------------
GC#         1 |    BackgroundGC |     2 |            AllocLarge |          0,000 |         1,194 |        13,630 |                 0 |              0 |
GC#         2 |    BackgroundGC |     2 |            AllocLarge |          0,000 |         0,854 |        11,656 |                 0 |              0 |
GC#         3 |    BackgroundGC |     2 |            AllocLarge |         29,850 |         1,704 |        17,525 |                 0 |              0 |
GC#         4 |    BackgroundGC |     2 |            AllocLarge |         29,850 |         0,769 |         8,339 |                 0 |              0 |
GC#         5 |    BackgroundGC |     2 |            AllocLarge |         27,808 |         1,109 |        15,484 |                 0 |              0 |
GC#         6 |    BackgroundGC |     2 |            AllocLarge |         27,808 |         0,854 |         8,254 |                 0 |              0 |
GC#         7 |    BackgroundGC |     2 |            AllocLarge |         20,579 |         0,684 |         6,553 |                 0 |              0 |
GC#         8 |    BackgroundGC |     2 |            AllocLarge |         18,877 |         0,684 |         2,980 |                 0 |              0 |
GC#         9 |    BackgroundGC |     2 |            AllocLarge |         23,300 |         0,939 |        10,976 |                 0 |              0 |
GC#        10 |    BackgroundGC |     2 |            AllocLarge |         23,300 |         0,514 |         0,939 |                 0 |              0 |
GC#        11 |    BackgroundGC |     2 |            AllocLarge |         24,746 |         1,109 |        12,422 |                 0 |              0 |
GC#        12 |    BackgroundGC |     2 |            AllocLarge |         24,746 |         0,599 |         2,640 |                 0 |              0 |
GC#        13 | NonConcurrentGC |     0 |            AllocSmall |         24,302 |         4,954 |        11,486 |           epäluku |              4 |
GC#        14 |    BackgroundGC |     2 |            AllocLarge |         17,695 |         5,553 |         3,576 |                 0 |              0 |
```
- what are the result? What's LOH fragmentation and survival rate?
  - well, for all gen2 `BackgroundGC`s the resons are `AllocLarge` fragmentation and survival rates are zero. Actually, in PerfView, you can see survival rate which is between 2-4-5, and fragmentation % which are mostly between 70-90, and rarely 20-60.
- we know that those FullGCs are happening because of AllocLarge, which means - most probably - the LOH allocation budget has been exceeded. 
Let's confirm that by recording dotnet-trace session and dig in into GCStats. If so, what allocation budgets are calculated for LOH?
I checked recorded *.nettrace file and in section "Condemned reasons for GCs", the main reason for GCs is Generation Budget Exceeded. Hence, the assumption is Confirmed.

### TASK 2
- open the dump with the help of dotnet-dump analyze and look what's keeping InternalIdentifier instance(s) alive. In which generations those instance(s) live?
  - First of all, it has 2 different instances that GC dump treats as two separate classes. Secondly, I was not able to understand from which generation.
- also use dumpalc (use help) for looking what information you can get about the corresponding assembly context
  - nothing. The only thing is that there are two ` WebWorkerApp.PluginAssemblyLoadContext`.

  This part is well described in Solution document. Use it as reference!
