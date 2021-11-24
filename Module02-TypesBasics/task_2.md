Refs:
* https://retroscience.net/x64-assembly.html
* https://docs.microsoft.com/en-us/cpp/build/x64-calling-convention?view=msvc-160
* https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/jit/object-stack-allocation.md
* https://sharplab.io/

# Explore code 
Explore JITed result of the following code, both for Debug and Release.
```
using System;

namespace Samples
{
    public class Echoer   {
        public int Echo(string message){
            SomeType c = new SomeType() { Data = message };
            return Helper(c);      
        }
        
        private int Helper(SomeType c) => c.Data.Length;   
    }
    
    public class SomeType   {
        public string Data;   
    }   
}
```


|C#|JITed Debug|JITed Release|
|---|---|---|
|no changes|JITed Debug code is much bigger. There are a lot of calls and extra instructions.|Release version is better, very concise.|
|Change SomeType to be struct. Do you see signicant changes in the JITted code? |SomeType ctor is gone. |SomeType ctor is gone. Many over instructions as well, only `mov` and `ret` are left.|
|What if SomeType is a record? |all `object` methods are explicitly written in SomeType (e.g. Samples.SomeType.ToString()). Two ctors, system methods are appeared as well. |All those methods from debug version are just optimized.|
|What if it is a record struct (you may need to switch it to the C# Next: Record structs branch touse it). |Here we have less methods compare to the previous example. system methods are gone. Some methods are still explicitly written in SomeType (e.g. Samples.SomeType.ToString()). However, other classes and methods that are using SomeType look like they are using struct (value type)|same methods - just optimized.|
|What if you disable inlining of Helper|It simplifies code. Got rid of one method.|It is very simple, especially, when there is a struct and this inlining.|

