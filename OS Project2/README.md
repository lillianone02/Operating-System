# Operating System System Call : Sleep
First we define the system call Sleep, and we record its number and behavior in ```suerprog/syscall.h```  
Then we make the system call Sleep into kernal.  
```
.globl  Sleep
        .ent    Sleep
Sleep:
        addiu   $2,$0,SC_Sleep
        syscall
        j       $31
        .end    Sleep
```
