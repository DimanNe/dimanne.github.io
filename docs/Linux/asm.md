title: asm

# **ASM**


## **Registers**


* **rip**: shows the memory address of the instruction that the CPU is executing.
* **rbx** is often used as a base pointer for various data structures or objects.

Calling conventions:

* Passing parameters:
    * **rdi**, **rsi**, **rdx**, **rcx**, **r8**, and **r9**: used to pass first six integer or pointer arguments
    * **xmm0** through **xmm7**: used to pass the first eight floating-point arguments
    * Additional arguments are passed on the stack, pushed onto the stack in reverse order (i.e., the last argument
      is pushed first).

* Return value:
    * **rax**: used to pass integer and pointer return values
    * **xmm0**: used to pass floating-point return values
    * **RDI** is used to return large structures or other data types that don't fit in the RAX register. The caller
      allocates space for this memory and passes its address in the **RDI** register as a hidden first argument;
      thus, functions returning these types appear to the callee as if they have an additional argument.

* Caller-saved and Callee-saved Registers:

    * **rax**, **rcx**, **rdx**, **rdi**, **rsi**, **r8**, **r9**, **r10**, **r11**, and the **xmm**: are caller-saved
      registers. If a calling function relies on the value of these registers to be the same after it calls another
      function, it has to save them (typically on the stack) before the call and restore them after the call.
    * **rbx**, **rsp**, **rbp**, **r12**, **r13**, **r14**, **r15**: callee-saved registers. If a called function
      (callee) wants to update/change these registers, it has to save the original values (again, typically on
      the stack) and restore them before returning.

* Stack Frame:

    * **rbp**: The register is often used as the frame pointer, which points to the base of the current function's
      stack frame. This convention makes it easier to reference local variables and function arguments, especially
      in debugging. However, modern compilers often use frame pointer omission (FPO) optimization in which case the
      RBP register is freed up for general use, and the stack pointer (**rsp**) is used directly to reference local
      variables.

    * **rsp**: The register always points to the top of the stack (keep in mind the stack grows downwards on x86_64).
