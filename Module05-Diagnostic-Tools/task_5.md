Refs:
* https://github.com/dotnet/runtime/blob/main/src/coreclr/vm/ClrEtwAll.man
* https://randomascii.wordpress.com/2015/09/24/etw-central/
* https://docs.microsoft.com/en-us/archive/blogs/vancem/
* https://github.com/dotnet/diagnostics/blob/main/documentation/design-docs/ipc-protocol.md
* https://lowleveldesign.org/2021/01/20/snooping-on-net-eventpipes/
* https://docs.microsoft.com/en-us/dotnet/core/diagnostics/diagnostics-client-library
* https://mattwarren.org/2016/06/20/Visualising-the-dotNET-Garbage-Collector/

# HT

## Testing server gc
After load testing open the trace file in PerfView, there is GCStats report, under Memory Group folder.  
Some stats that we are interested in:
* Total GC Pause: 19.1 msec
* Max GC Heap Size: 153.580 Mb

How does GC Rollup By Generation table look? Do you see Induced GCs?  
It looks that all data were allocated on gen0 and gen1 while gen2 is empty (there are all zeros). And there is no induced GCs.  

What were the reasons for the GCs happening? Look at GC Events by Time table and its Trigger Reason column.
The table shows that GCs were triggering by AllocSmall reason, which means GC happens in response to an allocation of an ordinary objects.

## Testing workstation gc
What's the Total GC Pause? What's the Max GC Heap Size? How do they compare to the Server GC results?  
* Total GC Pause: 63.9 msec
* Max GC Heap Size: 12.877 Mb

Workstation GC makes more pauses, basically 3 times more. However, the memory footprint is significantly less, it less 10 times.  

how does GC Rollup By Generation table look? Do you see Induced GCs?  
There is no induced GCs, but the gen2 was triggered once. In the same way, gen1 was triggered. On the contrary, workstation gc introduced 32 gen0 GCs, while server GC had only 1.  

look at GC Events by Time table for super important information:  
* Peak MB - "observed" maximum memory usage before the GC
* After MB - total memory usage after the GC
* Ratio Peak/After - the ratio, saying simply how "productive" was the GC (higher is better)

Overall, the ratio peak/after fluctuates between 0.88 and 0.9, though there are several slightly above 2. I may assume that this efficiency if this GC is not good sufficient.  

Use Background Workstation GC with the same app and load test, record a session and try to find Background GCs happening (hint: you may need to increase the load to observe them)! They will be listed in GC Events by time table with a letter B in Gen column. What are their pause times?  

????



