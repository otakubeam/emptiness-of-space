---
title: "Weekly update: 22-08-10"
customdate: "2022 Aug 10 (水) 08:49AM"
tags: ["status"]
---

## So, what did I manage to accomplish this week?

First thing I did was to add simple typing to the interpreter.
This includes primitive types as well as function types.

    fun factorial(n: Int) Int {
       if n == 0 {
          1
       } else {
          mul(n, factorial(n-1))
       }
    }

Otherwise I've added structs support. 
Now I can do something like this:

    struct Base {                       
      a: Int,                           
    };                                  
                                        
    struct Nested {                     
      b: Base,                          
    };                                  
                                        
    var inst = Nested:{Base:{123}};     
                                        
    fun takingStruct(str: Nested) Base {
      str.b                             
    }     

One thing I didn't consider was self-referential types.
I guess I'll just want add pointers, huh?

Otherwise I wanted to refactor things a bit, 
because the code started to kind of drift apart. For example:

1. I need to hookup locations with every AST node for better error messages.
2. Remove redundant fields in visitors with global symbol table.
3. Better handle accessing fields of a struct.

   Right now, you can not do fun().field for example

4. Decide what the rt/ directory actually represents

   It's not the most general runtime. The implementation of functions seems to
   be tied to the Interpreter. Otherwise I could reuse the same thing when
   emitting bytecode. Primitive and Struct objects seem to be general enough.

   Clearly, there is tension between inptepreter, intrinsics and bytecode vm.

Speaking of which. I looked into different kinds of IR. 

- Modern Compiler Implementation by Andrew Appel was the first thing I read and
  honestly I think the treatment is pretty horrible. I just don't understand
  the rationale behind his design. Maybe it's just me, though.

- "Crafting Interpreters" is good. Althought Robert builds a bytecode
  interpreter I feel this is close enough to real compilation. At least as the
  first step.

- Otherwise I read a bit of lcc book. The projects I've been reading up on are
  hare, zig and cproc.

- Yesterday I read chapters about types and IR from "Engineering a Compiler"
  and also several RCFs (mir and hir mostly) from Rust. As always Rust docs are
  amazing.

## Next week

Should I throw everything out of the window? Is codegen compatible with my
current implementation? How much would I need to duplicate? I would of course
like to keep both implementations with minimal maintenance cost.

I guess I need to look at my own work with a "hostile eye" and remove
everything I deem unworthy. Make some room for something better.
