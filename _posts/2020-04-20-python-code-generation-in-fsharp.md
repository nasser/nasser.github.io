---
layout: page
title: Python Code Generation in F#
date: 2020-04-20 18:42 -0400
---

I am working on [a compiler](https://github.com/nasser/OSKAR/tree/v3) implemented in F# targeting Python (the ultimate goal is to generate scripts that run inside of [Houdini](https://www.sidefx.com/)). That means that at some point I am going to have to write strings of Python code out to a file.

I want to avoid the error-prone process of concatenating strings together myself. My ideal approach is to generate a Python AST and have *that* decompile into a valid Python source. This is possible in Python using the built in `ast` module and the [`astor` package](https://pypi.org/project/astor/).

```python
>>> import ast
>>> import astor
>>> code = ast.IfExp(
...         ast.Compare(ast.Name("a", None), [ast.Gt()], [ast.Name("b", None)]),
...         ast.Call(ast.Name("some_function", None), [ast.Num(100)], [], None, None),
...         ast.Call(ast.Name("some_other_function", None), [ast.Num(50)], [], None, None))
>>> astor.to_source(code)
'(some_function(100) if (a > b) else some_other_function(50))'
```

But I am not writing my compiler in Python, I am writing it in F#.[^1] It turns out using [IronPython](https://ironpython.net/) and some crimes one can get it to work if one was so motivated. 

IronPython does expose AST types, but they are not the same AST types that the Python `ast` module produces, and as a result they are not usable for source code generation. Instead, I use my own discriminated union as an idiomatic stand-in for the Python AST. A healthy dose of ~~crimes~~ runtime reflection allows us to bridge the type safe world of F# and the dynamic stringly typed world of IronPython with minimal effort.

Instead of astor I opted for the more lightweight [`codegen` project](https://github.com/CensoredUsername/codegen), a single python file that exclusively performs AST to Python source generation, but the approach is the same. First, we set up our IronPython using the standard procedure.

```fsharp
open System.IO
open IronPython.Hosting
open IronPython.Runtime
open Microsoft.FSharp.Reflection

let engine = Python.CreateEngine()
```

We follow that by setting up the search paths. This will differ on different systems, but IronPython just needs to be able to find its standard library.

```fsharp
let pythonPath = [|"/usr/lib/python2.7"
                   Directory.GetCurrentDirectory()|]
engine.SetSearchPaths(pythonPath)
```

Now the fun begins. We define our own `AST` type named and shaped *exactly* like [the Python 2.7 AST types](https://docs.python.org/2.7/library/ast.html). I've only included a few entries here, but the idea is that I would add entries as I encountered corners of the language that I needed.

```fsharp
// based on https://docs.python.org/2.7/library/ast.html
type AST =
    | Compare of AST * AST seq * AST seq
    | Call of AST * AST seq * AST seq * AST option * AST option
    | BinOp of AST * AST * AST
    | If of AST * AST seq * AST seq
    | IfExp of AST * AST * AST
    | Add | Sub | Mult | Div
    | Eq | NotEq | Lt | LtE | Gt | GtE | Is | IsNot | In | NotIn
    | Num of obj
    | Name of string
```

This is nice, idiomatic, mostly type safe[^2] F# code. Note that I reproduce some of the cruft that comes from the Python 2.7 AST types (`Call` takes two optional nodes) and omit others (`Name` should take an extra argument, but it is not in the type) to be dealt with later.  We can construct our node similar to how we did in Python above.

```fsharp
let code = IfExp(
            Compare(Name "a", [Gt], [Name "b"]), 
            Call (Name "some_function", [Num 100], [], None, None),
            Call (Name "some_other_function", [Num 50], [], None, None))
```

Converting this into a Python AST node takes a little bit of work, mostly type conversions

```fsharp
let ast = engine.ImportModule("ast")

/// Given a name and an array of arguments returns a Python AST node instance
/// 
/// This function handles some special casing and heuristics as well
let pythonAstNode name args =
    // look up ast type in ast module by name
    let astType = ast.GetVariable(name) :?> Types.PythonType
    match name with
    // Name takes an extra argument that we always want to be null for now
    | "Name" -> engine.Operations.Invoke(astType, Array.append args [|null|])
    // invoke the ast type on the provided arguments to return a new instance
    | _ -> engine.Operations.Invoke(astType, args)

/// Convert an F# sequence into a Python list
let toList<'a> (s:'a seq) =
    let l = List()
    for v in s do
        l.Add(v) |> ignore
    l    

/// Given an instance of our AST union (or sequences or options thereof)
/// return a Python AST node instance, Python list, or null
let rec toPythonAst (a:obj) =
    match a with
    | :? AST ->
        match FSharpValue.GetUnionFields(a, typeof<AST>) with
        | case, x -> pythonAstNode case.Name (Array.map toPythonAst x)
    | :? seq<AST> as s -> toList (Seq.map toPythonAst s) :> obj
    | :? option<AST> as o -> 
        match o with
        | Some v -> toPythonAst v
        | None -> null
    | _ -> a
```

We use reflection on our `AST` type (`FSharpValue.GetUnionFields`) to look up the corresponding Python AST type names in the `ast` module. That's why the type names have to match exactly! To avoid this kind of hack I would have to write code that walked the tree and did something like 

```fsharp
match ast with
| Call ... -> ast.GetVariable("Call")
| Add ... -> ast.GetVariable("Add")
| Sub ... -> ast.GetVariable("Sub")
...
```

Which is tedious unto maddening. You can report me to the functional language police. I will stand for my crimes.

Once that machinery is in place, converting to Python AST nodes and then into source is straightforward.

```fsharp
let codegen = engine.ImportModule("codegen")
let codegenToSource = codegen.GetVariable("to_source") :?> PythonFunction
let pythonCode = toPythonAst code
printfn "%A" (engine.Operations.Invoke(codegenToSource, pythonCode))
```

And that's it. It's a pretty small amount of code to build a pretty substantial bridge! The whole thing is in [this gist](https://gist.github.com/nasser/2ce039eca06710c4c4701b36d3b729cc).

[^1]: Early versions of the compiler *were* written in Python, specifically in anticipation of the code generation requirements, but I found the absence of F#'s type system and pattern matching (or maybe the presence of Python's dynamic mutability and commitment to class-based object oriented programming) to be enough of a deal-breaker to abandon that approach.
[^2]: I am not sure how numbers map between F# and Python, so the `Num` node takes an `obj` parameter for now. 