# Share

Rust structure:

- Crates
- Modules
- Items

:: is used as a separator

String::new ??

Ownership
Memory

- Stack
- Heap
- Function call stack

Stack & Heap memore allocation
<https://stackoverflow.com/questions/23981391/how-exactly-does-the-callstack-work>

It's called stack memory allocation coz it happens in the Function call stack
Call stack is responsible for maintaining the local variables and parameters during function execution.
Function prologue & function epilogue

Stack vs Heap
The stack is the memory set aside as scratch space for a thread of execution.
The heap is memory set aside for dynamic allocation.

<https://doc.rust-lang.org/book/ch16-01-threads.html#using-threads-to-run-code-simultaneously>  
Process -> Thread
Thread is lightweight process coz they share the same shared memore(heap)
Registers and
Rust Thread
Each process has a separate memory address space, which means that a process runs independently and is isolated from other processes.
Address space

Runtime

Ownership rules
- Each value has its variable that is his owner
- One owner at a time
- As soon as variable is out of scope the value gets dropped off
