# code: from source to execution

do you know how your code

**int f(){**

**return 42;**

**}**

turns into

**mov eax, 42
ret**

and then into

**B8 2A 00 00 00 C3** 

this?

<aside>
💡

Here’s the short thing:

**source code** → something happens, **Intermediate code** forms → something happens again, **Machine code** is formed.

We’ll clear the ‘something’ in this article.

</aside>

Life would be very simple if we humans could write 1s and 0s and directly give machine it’s prefered machine code. But since we don’t have 1000 hands per person and the outputs that we’re expecting out of computers have evolved to complexities unimaginable, we need another simpler way to talk to the machines. And that is why we have different programming languages and their compilation processes.

There are **compiled languages** and **interpreted languages** divided on the basis of when the code is executed.

Compiled languages are programming languages that are converted into machine code by the compiler and only then, is there an *executable file.*

Interpreted languages are not converted into compiled code, rather the source code is directly executed line-by-line by the interpretor.

CAVEAT: most of the lanugages today use a mixture of the two ideas. Ex. Initially, the **JVM interprets the Java bytecode** produced by the Java compiler, executing it instruction by instruction until it identifies frequently executed (“hot”) code paths, which are then compiled into native machine code by the JIT compiler for faster execution.

# Different representations of the code

## Source code

>> This is the code that the programmer sees. It’s in programming languages. 

Ex: .py files for Python code and .c files for C code.

>> it’s human-readable.

>> code has semantics (meaning) and a form. 

>> features like comments, indentation and format.

```cpp
int add(int a, int b){
	return a+b;
}
```

**Details on Structure**:

- **Declarations**: Define types and variables (e.g., int a).
- **Expressions**: Computations like a + b.
- **Statements**: Control flow like return.
- **Modules/Files**: Organized into files with includes/imports for modularity.

## IR: Intermediate Representation

>> It is the intermediate code that is formed.

>> it is a little bit more lower-level. Hence it forms a key phase in compiler’s front-end.

>> It is platform independent. Which enhances portability. 

>> For n languages and m targets, you need n front-ends + m back-ends instead of n*m full compilers.

>> Intermediate code has several levels before the code turns into machine code:

- high-level - ex: Syntax Tree
    - close to source code, can be used to trace back to the source code
    - used for early optimizations
    - example:
        
        ![Abstract Syntax Tree (AST) - type of a mid-level IR 
        credits: GeeksForGeeks](code%20from%20source%20to%20execution/gfg-IR-w1.png)
        
        Abstract Syntax Tree (AST) - type of a mid-level IR 
        credits: GeeksForGeeks
        
- mid-level - ex: TAC Three address code
    - Ex. T1 = T2 op A
    - has maximum of three operands
    - uses temprories (eg: T1)
    - The typical form of a three address statement is expressed as *x = y op z*, where *x, y*, and *z* represent memory addresses.
    - ex: x = (a + b * c) / (a - b * c)
        
        t1 = b * c
        t2 = a + t1
        t3 = a - t1
        x  = t2 / t3
        
        Did you notice the reusable t1? Yes, optimization.
        
- low-level - ex: Register Transfer Language or LLVM
    - stack based - ex: Java bytecode or CPython bytecode
        - it is also mid-lower level IR
        - stack based
        - replicates a stack
        - features:
            - no named registers
            - everything flows through stack
    - closer to machine-code
    - register based
    - has memory accesses and registers
    - unlimited temp (Ex. T1…) values
    - made for heavy optimizations
    - does not contain irrelevant syntaxes
    - example
        
        → code : C++
        
        ```cpp
        int sum (int a, int b){
        	return a + b;
        }
        ```
        
        → IR : LLVM format
        
        ```llvm
        define i32 @sum (i32 %a, i32 %b){
        	%1 = add i32 %a, %b
        	ret i32 %1
        }
        ```
        
        1. we can see that even though there was no variable defined in the actual code, in LLVM format, there is a `%1` temp created and that is returned. 
        2. other thing that we can notice is how we add the two i32 integers. 
            1. first we write **add** and then the two references.
- Many compilers use multiple levels: high-level (tree-like) → mid (SSA) → low (register-based).
****

One question often asked is this. Why IR? Why not just take the source code and execute that directly?

Glad you asked.

It’s because IR’s help in optimizations. They form the last part of the front-end of compilation process. You can make any sort of optimizations: constant folding, loop unrolling, inlining, etc.

What we’ve seen so far is how your code goes from source code level, to an IR (that is close to being a foreign language to us).

Ex: Java (via javac) → JVM Bytecode, C++ → via Clang compiler → LLVM IR 

- (gcc has its own IR of c++ code that is RTL)

Let’s see now how that IR is translated into machine code.

But before that,

- Little case study on JVM bytecode and LLVM IR (open toggle)
    
    we will see how this function below
    
    ```cpp
    int add (int x, int y){
    	return x + y;
    }
    ```
    
    turns into IR of Java and C++ (via Clang)
    
    ![image.png](code%20from%20source%20to%20execution/image.png)
    

a lot of things happen when we move to the final step from IR to machine code!

### From IR to Machine code: pipeline (in rough phases)

1. IR optimization:
    1. remove unnecessary functions/variables/ops
    2. do all the (2+3)s to 5s
    3. **inline:** expand functions
2. convert to lower level, architecture aware form
    - GCC GIMPLE (IR) → RTL
    - LLVM (IR) → MachineIR
    1. Example: Stack-based IR (JVM bytecode) gets "de-stacked" into register form for JIT.
3. now this is my favourite part:
    
    mapping of IR functions to CPU operations happen here.
    
    and different architectures have different forms of the same instructions.
    
    Here’s what I mean:
    
    - say you have a *add i32.* this is basically *add int.*
    - in x86 CPUs → ADD reg, reg
    - in ARM CPUs → ADD rd, rn, rm
    - here’s a small comparison table (highly recommended to check out!)
        
        
        | IR Op | x86 Instruction | ARM Instruction |
        | --- | --- | --- |
        | add i32 | ADD reg, reg | ADD rd, rn, rm |
        | load i32 | MOV reg, [mem] | LDR rd, [rn] |
        | branch | JMP / Jcc | B / B.cond |
4. Register allocation: uses algorithm “**Graph Coloring**”
    - registers - fast, memory - slow.
    - if there are extra variables then spill one to stack, use it later.
5. Scheduling and tweaks
    - schedule instructions. meaning the instructions used majorly are called first
    - small tweaks like **add i32 %a, 1** → **inc i32 %a**
6. Code Emission & Linking
****Output assembly or object file is formed (e.g., .o file), then link to executable.
    - Assemble: Text asm → binary (opcodes + operands).
    - Link: Resolve externals, add runtime (e.g., libc).
    Example: Final x86 machine code for simple **add**: **55 48 89 E5 89 7D FC 89 75 F8 8B 55 FC 8B 45 F8 01 D0 5D C3** (prologue + add + epilogue).

## Machine Code

>> it is the lowest level representation of code.

>> it is **architecture dependent**. Put simply, to make a compiler, you’d need to know the type of CPU the code will be executed on.

>> everything in machine code is “out there”. Meaning, the registers information, jumps, memory addresses etc. No abstractions.

>> it has 

- opcode - ex: add, jump
- registers - ex: EAX on x86 architecture
- memory operands - addresses for load/store
- immediates - constants embedded in instructions.

>> this has “unlimited” IR virtual registers (ex %1, %2..)

And there we have it.

here’s something for a review.

![Picture source: geeksforgeeks.org](code%20from%20source%20to%20execution/gfg-compiler-phases.png)

Picture source: geeksforgeeks.org

From language specific details, check out the https://medium.com/javarevisited/code-compilation-from-source-to-machine-code-1375e49d00b6 article.

~ Aayushya Tiwari

# References

https://www.geeksforgeeks.org/compiler-design/intermediate-code-generation-in-compiler-design/ - GFG on Intermediate Code Generation

https://www.geeksforgeeks.org/compiler-design/three-address-code-compiler/ - GFG on TAC

https://medium.com/javarevisited/code-compilation-from-source-to-machine-code-1375e49d00b6 - Brilliant essay on how Java, C++ and Python compile code

https://net-informations.com/python/iq/linking.htm - article on how python code is compiled

https://en.wikipedia.org/wiki/Abstract_syntax_tree - wiki on ASTs

LLMs: grok.com, chatgpt.com