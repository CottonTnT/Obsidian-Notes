- Table 18.1 summarizes the datatypes natively supported by RISC-V C programs
![[Pasted image 20240513133034.png]]

- The RISC-V calling convention passes arguments in registers when possible. Up to eight integer registers, a0–a7, and up to eight floating-point registers, fa0–fa7, are used for this purpose
- In the standard RISC-V calling convention, the stack grows downward and the stackpointer is always kept 16-byte aligned.
![[Pasted image 20240513133113.png]]

-  if a function argument is more than a pointer-word(word means 64bits here),we can put that in a register pair, so the same convention holds true for return address register a0-a1


![[Pasted image 20240513135832.png]]