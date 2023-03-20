# Operating System System Call : Sleep
First we define the system call Sleep, and we record its number and behavior in ```suerprog/syscall.h```  
Then we make the system call ```Sleep``` into kernal.  
```
        .globl  Sleep
        .ent    Sleep
Sleep:
        addiu   $2,$0,SC_Sleep
        syscall
        j       $31
        .end    Sleep
```
We go into file ```exception.cc```, and define the behavior of the system call ```SC_Sleep```which NachOS is responsible for.  
```c++
//userprog/exception.cc

case SC_Sleep:
        val=kernel->machine->ReadRegister(4);
        cout << "Sleep Time:" << val << "(ms)" << endl;
        kernel->alarm->WaitUntil(val);
        return;
```
