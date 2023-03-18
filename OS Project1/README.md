# Operating-System Thread Management 
## Motivation 
Two processes run at the same time.  
In mutiprogramming system, the two processes run concurrently.  
Therefore, the result may be out of control.  
Now we have the source code from nachos-4.0/code/test/test1.c and nachos-4.0/code/test/test2.c.  
We should adjust the source code to guarantee the result to be accurate.  
## Implementation 
The two processes may execute the same code segment.  
Therefore we modify nachos-4.0/code/userprog/addrspace.cc and nachos-4.0/code/userprog/addrspace.h  
First, we add an array to record the status of the physical page table in addrspace.h and addrspace.cc  
Then creat pagetable in addrspace.cc  
Modify the function AddrSpace::Load() in addrspace.cc  
We calculate the entry point from the virtual memory address.  
## Result 
./nachos -e ../test/test1 -e ../test/test2  
  
Output  
![image](https://github.com/lillianone02/Operating-System/blob/1b6856ea00fc5e7c3633d1ac13986b3200023ad0/OS%20Project1/MutiThread.png)  
