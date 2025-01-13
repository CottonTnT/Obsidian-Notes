- Table 18.1 summarizes the datatypes natively supported by `RISC-V` C programs
![[Pasted image 20240513133034.png]]

# 1 Register

![[Pasted image 20240513133113.png]]


- The RISC-V calling convention passes arguments in registers when possible. 
- Up to eight integer registers, a0–a7, and up to eight floating-point registers, fa0–fa7, are used for this purpose
-  if a function argument is more than a pointer-word(word means 64bits here),we can put that in a register pair, so the same convention holds true for return address register a0-a1

 - Saver 为 `caller(调用者)` 是在函数调用中不会保存的寄存器。Saver 为 `callee(被调用者)` 是在函数调用中会保存的寄存器。


# 2 stack

![[Pasted image 20240513135832.png]]

 In the standard `RISC-V` calling convention, the stack grows downward and the `stackpointer` is *always kept 16-byte aligned*.





