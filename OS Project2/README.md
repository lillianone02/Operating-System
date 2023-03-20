# Operating System System Call : Sleep

## Implementation
First we define the system call Sleep, and we record its number and behavior in ```uerprog/syscall.h```  
Then we make the system call ```Sleep``` into kernal.  
```
uerprog/syscall.h

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
userprog/exception.cc

case SC_Sleep:
        val=kernel->machine->ReadRegister(4);
        cout << "Sleep Time:" << val << "(ms)" << endl;
        kernel->alarm->WaitUntil(val);
        return;
```
```Alarm```represent interrupt vector in NachOS.  
We adjust ```WaitUntil``` in ```Alarm```to practice Sleep.  
We count the number of the interrupt.  
When the number equal to the sleep time, OS should put the certain Thread back to ```Ready Queue```.  
```c++
threads/alarm.h

#ifndef ALARM_H
#define ALARM_H

#include "copyright.h"
#include "utility.h"
#include "callback.h"
#include "timer.h"
#include <list>
#include "thread.h"
class sleepList {
    public:
        sleepList():_current_interrupt(0) {};
        void PutToSleep(Thread *t, int x);
    bool PutToReady();
    bool IsEmpty();
    private:
        class sleepThread {
            public:
                sleepThread(Thread* t, int x):
                    sleeper(t), when(x) {};
                Thread* sleeper;
                int when;
        };

    int _current_interrupt;
    std::list<sleepThread> _threadlist;
};
// The following class defines a software alarm clock. 
class Alarm : public CallBackObj {
  public:
    Alarm(bool doRandomYield);  // Initialize the timer, and callback 
                // to "toCall" every time slice.
    ~Alarm() { delete timer; }

    void WaitUntil(int x);      // suspend execution until time > now + x
  private:
    Timer *timer;               // the hardware timer device
    sleepList _sleepList;
    void CallBack();            // called when the hardware
                // timer generates an interrupt
};
#endif // ALARM_H
```
