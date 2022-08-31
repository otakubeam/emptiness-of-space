---
title: Function calls in SBlang
customdate: 2022 Aug 31 (水) 07:38PM 
---

This quick post is intended as a guide-level introduction to the SBlang VM
assembly.

I recently got around to implementing jumps and function calls. Once you have
that the language becomes mich more expressive. If you have jumps you The
following is an example of using them.

## Invert bool and its instructions

Suppose we have a SBlang function that inverts the value of a Bool, like so:

```
fun invert(b: Bool) Bool {
    if b == false {
        return true;
    }

    false
}
```

Suppose also that it's invoked from main with the following command

```
invert(true)
```

### Let's see what assembly this code should generate

- The invocation from main generates the following:

  ```
    vm::ExecutableChunk chunk{
        .instructions =
            {
                vm::Instr{
                    .type = vm::InstrType::PUSH_STACK,
                    .arg1 = 0,  // push true
                },
                vm::Instr{
                    .type = vm::InstrType::CALL_FN,
                    .arg1 = 1,  // chunk 1 ~ compiled_fn
                    .arg2 = 0,  // ip is 0
                    .arg3 = 0,
                },
                vm::Instr{
                    .type = vm::InstrType::FIN_CALL,
                    .arg1 = 1,  // pop one arg from the stack
                },
            },
        .attached_vals{
            vm::rt::PrimitiveValue{.tag = vm::rt::ValueTag::Bool,  //
                                   .as_bool = true},               //
        },
    };
  ```

  - This is a stack-based VM, so first we should push all the arguments onto
    the stack.

  - Then the compiler generates the function call to the 1st chunk of compiled
    functions (the pieces of bytecode live in different chunks);  
    
    We use the symbol table to keep track of which chunks contain what and
    generate the right calls

  - From there the execution should start with ip of 0 (parameters $2 and $3
    are responsible for that)

  - The final instruction is actually executed after the call, as you might
    expect. Its duty is to clean up the callstack of the pushed arguments and
    push the return value.
  

- Let's see what the function generates

  ```
    vm::ExecutableChunk compiled_fn_invert_bool{
        .instructions =
            {
                vm::Instr{
                    .type = vm::InstrType::FROM_STACK,
                    .arg1 = 0,  // push argument i
                },
                vm::Instr{
                    .type = vm::InstrType::JUMP_IF_FALSE,
                    .arg2 = 0,
                    .arg3 = 4,  // maybe skip next 2 instrs
                },
                vm::Instr{
                    .type = vm::InstrType::PUSH_STACK,
                    .arg1 = 1,  // push constant false
                },
                vm::Instr{
                    .type = vm::InstrType::RET_FN,
                },
                vm::Instr{
                    .type = vm::InstrType::PUSH_STACK,
                    .arg1 = 0,  // push constant true
                },
                vm::Instr{
                    .type = vm::InstrType::RET_FN,
                },
            },
        .attached_vals{
            vm::rt::PrimitiveValue{.tag = vm::rt::ValueTag::Bool,  //
                                   .as_bool = true},               //
            vm::rt::PrimitiveValue{.tag = vm::rt::ValueTag::Bool,  //
                                   .as_bool = false},              //
        },
    };

    CHECK(vm::BytecodeInterpreter::InterpretStandalone(
              {chunk, compiled_fn_invert_bool}) == (int)false);
  }
  ```

  Simple enough!  We compare the argument with the value `false` and possibly
  jump. 

  FROM_STACK instruction is used to reference the argument with respect to the
  frame pointer of the stack.

  RET_FN restores the state of the stack registers and jumps back into the 0-th
  chunk (that is our main)

## This is meant as a quick introduction

- A: You are encouraged to look at the source code. It's really simple. The
  interpretation of RET CALL and FIN should become clear. The stack is the main
  charachter here so don't forget it either.

- B: You could also hand-write recursive functions or something. Was quite fun
  for me. (Your mileage may vary though...)
