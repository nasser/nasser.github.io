---
title: Using Mono.Cecil for code generation
layout: page
---

For a long time I had incorrectly assumed that the fantastic [Mono.Cecil library](https://cecil.pe/) could *only* operate on CLR bytecode on disk. This was a problem for because I wanted to write languages that had REPLs and live coding systems that depended on the ability to generate code in memory at runtime, so I assumed Cecil was not an option. My earlier compilers used the [System.Reflection.Emit API](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit?view=netframework-4.8) to do code generation, with the conspicuous [`AppDomain.DefineDynamicAssembly`](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain.definedynamicassembly?view=netframework-4.8) and [`TypeBuilder.CreateType`](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.typebuilder.createtype?view=netframework-4.8) producing runnable code in memory, figuring this was the only way to do it.

It turns out I was *way wrong*. Cecil does not have a similarly direct API but you can still pull it off. The trick is to write the `ModuleDefinition` to a [`MemoryStream`](https://docs.microsoft.com/en-us/dotnet/api/system.io.memorystream?view=netframework-4.8) and then load those bytes via the [`Assembly.Load`](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.assembly.load?view=netframework-4.8#System_Reflection_Assembly_Load_System_Byte___) overload that expects a byte array. Here's the F# implementation from a compiler I am working on.

```fsharp
open System.Reflection
open Mono.Cecil

let writeToMemory (modDef : ModuleDefinition) : Assembly =
    use stream = new MemoryStream()
    modDef.Write(stream)
    Assembly.Load(stream.GetBuffer())
```

Boom, just like that you have a usable Assembly loaded into your AppDomain. It seems obvious in hindsight but it really kept me away from Cecil for a while — which is a shame because their API is really wonderful!