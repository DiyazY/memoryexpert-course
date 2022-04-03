Refs:
* https://github.com/dotnet/runtime/blob/main/src/coreclr/vm/ClrEtwAll.man
* https://randomascii.wordpress.com/2015/09/24/etw-central/
* https://docs.microsoft.com/en-us/archive/blogs/vancem/
* https://github.com/dotnet/diagnostics/blob/main/documentation/design-docs/ipc-protocol.md
* https://lowleveldesign.org/2021/01/20/snooping-on-net-eventpipes/
* https://docs.microsoft.com/en-us/dotnet/core/diagnostics/diagnostics-client-library
* https://mattwarren.org/2016/06/20/Visualising-the-dotNET-Garbage-Collector/

# HT

After load testing open the trace file in PerfView, there is GCStats report, under Memory Group folder.  
Some stats that we are interested in:
* Total GC Pause: 19.1 msec
* Max GC Heap Size: 153.580 Mb

How does GC Rollup By Generation table look? Do you see Induced GCs?  
It looks that all data were allocated on gen0 and gen1 while gen2 is empty (there are all zeros). And there is no induced GCs.  

What were the reasons for the GCs happening? Look at GC Events by Time table and its Trigger Reason column.
The table shows that GCs were triggering by AllocSmall reason, which means GC happens in response to an allocation of an ordinary objects.



