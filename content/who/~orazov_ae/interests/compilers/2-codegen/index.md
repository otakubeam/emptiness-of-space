---
title: 2. Vm Instruction Generation 
customdate: 2022 Sep 09 (金) 02:46PM 
---

## _start:

The last week I've been doing some work on codegeneration (as in converting AST
to linear sequence of custom assembly-like instructions).

Perhaps, the most exciting, I can now compile programs such as the one below.

```
{
  fun f(b: Bool) Int {                           
      var res = if b { 100 } else { 101 };       
      res + res                                  
  }                                              
                                                 
  f(true)                                        
}                                                
```

### What was new

The difficulty here being compiling and 'linking' function's invocations with
the correct compiled chunks.

The translation of funtions also called for a new entity called
'FrameTranslator' which is responsible for simulating the runtime state of the
stack and assigning the right offsets.

Also I needed some time to think about how to handle direct and indirect calls.
This is something I have never encountered before so it was interestring.

### The above program produces the following debug output

- First, it prints out the result of codegeneration:

  It consists of two chunks:

  - One for f()

    In the 'Value' section we can see the constants that are used in the chunk.
    f() indeed uses 100 and 101.
  
  - One for main() (which is an implicit top-level function)

    It uses constans true (1) and 0. The last one is the number of the chunk
    that is the target of indirect_call.  

    You might be wondering why the call is indirect. Indeed, in this case the
    target could be inlined into the 'call' instruciton. Deciding on the right
    mechanism for direct/indirect calls will be done during the semantic
    analysis phase on the AST. For now, indirect_call "just works".


  ```
  [!] Executable chunk:
  	[.] Instruction: get_arg, 2 0 0
  	[.] Instruction: jump_if_false, 0 0 4
  	[.] Instruction: push_stack, 0 0 0
  	[.] Instruction: jump, 0 0 5
  	[.] Instruction: push_stack, 1 0 0
  	[.] Instruction: get_local, 1 0 0
  	[.] Instruction: get_local, 1 0 0
  	[.] Instruction: add, 0 0 0
  	[.] Instruction: ret_fn, 0 0 0
  
  
  	[-] Value: 100
  	[-] Value: 101
  [!] Executable chunk:
  	[.] Instruction: push_stack, 0 0 0
  	[.] Instruction: push_stack, 1 0 0
  	[.] Instruction: get_local, 1 0 0
  	[.] Instruction: indirect_call, 0 0 0
  	[.] Instruction: fin_call, 1 0 0
  
  
  	[-] Value: 0
  	[-] Value: 1
  ```

- Second, it prints out the internal state and just executed instruction for
  every step of the machine.


1. Encounter funtion declaration f(), place it on the stack as local

   ```
   [^] Instr: push_stack, 0 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	0	0	0	0	0	0	0	0	0	0	0	0	0	0	
   fp	  	sp	  	  	  	  	  	  	  	  	  	  	  	  	  	
   ```

2. Encounter invocation of f(true), place argument true on the stack.

   ```
   [^] Instr: push_stack, 1 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	0	0	0	0	0	0	0	0	0	0	0	0	0	
   fp	  	  	sp	  	  	  	  	  	  	  	  	  	  	  	  	
   ```

3. Push f()'s chunk number and consume it with indirect_call

   ```
   [^] Instr: get_local, 1 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	0	0	0	0	0	0	0	0	0	0	0	0	0	
   fp	  	  	  	sp	  	  	  	  	  	  	  	  	  	  	  	

   [^] Instr: indirect_call, 0 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	0	0	0	0	0	0	0	0	0	0	
     	  	  	  	  	fp	sp	  	  	  	  	  	  	  	  	  	
   ```

   Notice the change in stack layout (4 and 1 is return address):
   - 4 is for the 4-th instruction, 
   - 1 is for number of the main() chunk.

4. [!] Now we execute in the body of f()

   ```
   [^] Instr: get_arg, 2 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	1	0	0	0	0	0	0	0	0	0	
     	  	  	  	  	fp	  	sp	  	  	  	  	  	  	  	  	
   ```

5. Fall through to the push stack
   
   A bit of detail: here we push the constant with the number 0 offset in the
   constant table of the chunk, that is 100.

   ```
   [^] Instr: jump_if_false, 0 0 4
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	1	0	0	0	0	0	0	0	0	0	
     	  	  	  	  	fp	sp	  	  	  	  	  	  	  	  	  	
   
   [^] Instr: push_stack, 0 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	100	0	0	0	0	0	0	0	0	0	
     	  	  	  	  	fp	  	sp	  	  	  	  	  	  	  	  	

   [^] Instr: jump, 0 0 5
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	100	0	0	0	0	0	0	0	0	0	
     	  	  	  	  	fp	  	sp	  	  	  	  	  	  	  	  	
   ```


6. Get the first local value and duplicate it

   ```
   [^] Instr: get_local, 1 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	100	100	0	0	0	0	0	0	0	0	
     	  	  	  	  	fp	  	  	sp	  	  	  	  	  	  	  	
   
   [^] Instr: get_local, 1 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	100	100	100	0	0	0	0	0	0	0	
     	  	  	  	  	fp	  	  	  	sp	  	  	  	  	  	  	
   
   [^] Instr: add, 0 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	100	200	100	0	0	0	0	0	0	0	
     	  	  	  	  	fp	  	  	sp	  	  	  	  	  	  	  	
   ```

7. Return from the function, namely:

   - Restore the state of the stack (sp, fp, remove locals)
   - Jump back to the previous chunk

   A bit of detail: the result of the function is saved on ret_fn in an
   internal register 'rax'. It's then placed on the stack again after fin_call
   finishes popping arguments from the stack.

   This vm is caller-clean-up, accordingly.

   ```
   [^] Instr: ret_fn, 0 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	1	4	1	0	100	200	100	0	0	0	0	0	0	0	
   fp	  	  	sp	  	  	  	  	  	  	  	  	  	  	  	  	
   
   [^] Instr: fin_call, 1 0 0
   [!] Stack:
   0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	
   0	0	200	4	1	0	100	200	100	0	0	0	0	0	0	0	
   fp	  	  	sp	  	  	  	  	  	  	  	  	  	  	  	  	
   ```

## Thankfully, the test passes and we are greeted with the friendly message

```
===============================================================================
All tests passed (166 assertions in 100 test cases)
```
