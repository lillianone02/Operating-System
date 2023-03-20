# Operating System System Call : Sleep

## Motivation 
We want to add a system call ```Sleep```.  
At the same time, we have to comfigure some techniques to meature when to wake up the sleeping ```Thread```.  

## Implementation
First we define the system call Sleep, and we record its number and behavior in ```uerprog/syscall.h```  
Then we make the system call ```Sleep``` into kernal.  
```
// uerprog/syscall.h

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
// userprog/exception.cc

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
// threads/alarm.h

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
OS have to check whether the Thread has to be waken up by ```Alarm```or not.  
```c++
// threads/alarm.cc 

void Alarm::CallBack() {
    Interrupt *interrupt = kernel->interrupt;
    MachineStatus status = interrupt->getStatus();
    bool woken = _sleepList.PutToReady();
    //如果沒有程式需要計數了，就把時脈中斷遮蔽掉
    if (status == IdleMode && !woken && _sleepList.IsEmpty()) {// is it time to quit?
        if (!interrupt->AnyFutureInterrupts()) {
            timer->Disable();   // turn off the timer
        }
    } else {                    // there's someone to preempt
        interrupt->YieldOnReturn();
    }
}

void Alarm::WaitUntil(int x) {
    //關中斷
    IntStatus oldLevel = kernel->interrupt->SetLevel(IntOff);
    Thread* t = kernel->currentThread;
    cout << "Alarm::WaitUntil go sleep" << endl;
    _sleepList.PutToSleep(t, x);
    //開中斷
    kernel->interrupt->SetLevel(oldLevel);
}

bool sleepList::IsEmpty() {
    return _threadlist.size() == 0;
}

void sleepList::PutToSleep(Thread*t, int x) {
    ASSERT(kernel->interrupt->getLevel() == IntOff);
    _threadlist.push_back(sleepThread(t, _current_interrupt + x));
    t->Sleep(false);
}

bool sleepList::PutToReady() {
    bool woken = false;
    _current_interrupt ++;
    for(std::list<sleepThread>::iterator it = _threadlist.begin();
        it != _threadlist.end(); ) {
        if(_current_interrupt >= it->when) {
            woken = true;
            cout << "sleepList::PutToReady Thread woken" << endl;
            kernel->scheduler->ReadyToRun(it->sleeper);
            it = _threadlist.erase(it);
        } else {
            it++;
        }
    }
    return woken;
}
```

## Test 
We use the two file ```test1``` and ```teat2```.

```c++
// test/test1.c

#include "syscall.h"
main() {
    int i;
    for(i = 0; i < 5; i++) {
        Sleep(1000000);
        PrintInt(2222);
    }
    return 0;
}
```
```c++
// test/test2.c

#include "syscall.h"
main() {
    int i;
    for(i = 0; i < 20; i++) {
        Sleep(100000);
        PrintInt(10);
    }
    return 0;
}
```

# Result 
```
./nachos -e ../test/test1 -e ../test/test2
```
![image](https://github.com/lillianone02/Operating-System/blob/b749ec63d80678cd67b53184b0483d09283bdecd/OS%20Project2/OSproject2.png)
![image](https://github.com/lillianone02/Operating-System/blob/dcdb2977b5a25b5c6478400a79c7de3792a3a7b3/OS%20Project2/OSproject2-1.png)
