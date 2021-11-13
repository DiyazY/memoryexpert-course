# 1.1 
Basically, I see the pattern how dotnet publishes application.  
```
> create process
    > create thread
        > load image
    > create thread
        > load image
    > create thread
        > load image
    ................
    > thread exits
    > thread exits
    > thread exits
> process exits
....
> create process
    > create thread
        > load image
    > create thread
        > load image
    > create thread
        > load image
    ................
    > thread exits
    > thread exits
    > thread exits
> process exits

```

Self-contained publishing generates more events. Probably, it does more operations and definitely loads more libs into memory. 
I didn't dig so deep, but I get the concept!  
`Note: Process Monitor util is a handy tool for diagnosing and exploring events`  
Refs: 
* https://www.youtube.com/watch?v=pjKNx41Ubxw
# 1.2
- See how it is compiled to IL. 
- Does it use tail. call or not? ==> It doesn't contain any tails
- Try to understand the generated code. ==> interestingly, IL code doesn't have any recursions. it looks like a loop which is looks optimized.
- See how it looks under C#. ==> C# code is kind of similar to what i have seen in IL code. No recursion, instead of it, while loop.
- In the end, see the JIT result. ==> JITed code looks good. All things happen at native code level. 

Refs:
* https://www.felixcloutier.com/x86/
* https://en.m.wikibooks.org/wiki/X86_Disassembly/Functions_and_Stack_Frames
* https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture
* https://mattwarren.org/2018/04/06/Taking-a-look-at-the-ECMA-335-Standard-for-.NET/
* https://natemcmaster.com/blog/2017/12/21/netcore-primitives/
* https://github.com/dotnet/runtime/blob/main/docs/design/features/tiered-compilation.md

