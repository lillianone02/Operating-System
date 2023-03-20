# Operating System CPU Scheduling 
## Motivation
We implement and compare three CPU Scheduling Algorithms: First-Come-First-Service(FCFS), Shortest-Fob-First(SJF),and Priority.  
We use teo cases to compare the order and CPU burst time between the three Algorithms.  

## Implementation 
We first creat ```ThreadInfo```to print the running thread and the remaining CPU burst time.  
```c++
// threads/thread.cc

void
ThreadInfo() {
    Thread *thread = kernel->currentThread;
    while (thread->getCpuBurstTime() > 0) {
        thread->setCpuBurstTime(thread->getCpuBurstTime() - 1);
        kernel->interrupt->OneTick();
        cout << "Running thread " << kernel->currentThread->getName() 
        << ": cpu burst time remaining " << kernel->currentThread->getCpuBurstTime() << endl;
    }
}
```
Then we write test case in ```Thread::SchedulingTest()```.
```c++
// threads/thread.cc

void
Thread::SchedulingTest()
{
    //Test case 1
    const int THREAD_NUM = 4;
    char *name[THREAD_NUM] = {"A", "B", "C", "D"};
    int thread_priority[THREAD_NUM] = {3, 1, 4, 5};
    int cpu_burst[THREAD_NUM] = {8, 4, 9, 5};
    /* //Test case 2
    const int THREAD_NUM = 4;
    char *name[THREAD_NUM] = {"A", "B", "C", "D"};
    int thread_priority[THREAD_NUM] = {5, 1, 3, 2};
    int cpu_burst[THREAD_NUM] = {3, 9, 7, 3};
    */
    Thread *t;
    for (int i = 0; i < THREAD_NUM; i++) {
        t = new Thread(name[i]);
        t->setThreadPriority(thread_priority[i]);
        t->setCpuBurstTime(cpu_burst[i]);
        t->Fork((VoidFunctionPtr) ThreadInfo, (void *)NULL);
    }
    kernel->currentThread->Yield();
}
```
We add new function in class Thread.
```c++
// threads/thread.h

class Thread {
  private:
    //add
    void setCpuBurstTime(int t)    {cpuburstTime = t;}
    int getCpuBurstTime()      {return cpuburstTime;}
    void setThreadStartTime(int t)    {threadstartTime = t;}
    int getThreadStartTime()      {return threadstartTime;}
    void setThreadPriority(int t) {threadPriority = t;}
    int getThreadPriority()       {return threadPriority;}
    static void SchedulingTest();
  private:
    //add
    int cpuburstTime;  // cpu burst time
    int threadstartTime;  // the start time of a thread
    int threadPriority;   // the thread priority 
```
Then we add constructor.
```c++
void
ThreadedKernel::Initialize() {
    Initialize(RR);
}

void
ThreadedKernel::Initialize(SchedulerType type)
{
    scheduler = new Scheduler(type); 
}
void
ThreadedKernel::SelfTest() {
   currentThread->SelfTest();	// test thread switching
   Thread::SchedulingTest();
}

class ThreadedKernel {
  public:
    ThreadedKernel(int argc, char **argv);
    void Initialize(SchedulerType type);
    void Initialize()
}
```
We add SchedulerType
```c++
// threads/main.cc

int
main(int argc, char **argv)
{
    //add
    SchedulerType type;
    if(strcmp(argv[1], "FCFS") == 0) {
        type = FIFO;
    } else if (strcmp(argv[1], "SJF") == 0) {
        type = SJF;
    } else if (strcmp(argv[1], "PRIORITY") == 0) {
        type = Priority;
    } else {
        type = RR;
    }

    kernel = new KernelType(argc, argv);
    kernel->Initialize(type); // add
    
    CallOnUserAbort(Cleanup);		// if user hits ctl-C

    kernel->SelfTest();
    kernel->Run();
    
    return 0;
}
```
We define SchedulerType.
```c++
// threads/scheduler.h

enum SchedulerType {
        RR,     // Round Robin
        SJF,
        Priority,
        FIFO //add
};

class Scheduler {
  public:
	Scheduler();		// Initialize list of ready threads 
	Scheduler(SchedulerType type);		//  add Initialize list of ready threads 
    SchedulerType getSchedulerType() {return schedulerType;}
    void setSchedulerType(SchedulerType t) {schedulerType = t;}
};
```
We define different scheduler algorithms.
```c++
// threads/scheduler.cc

int SJFCompare(Thread *a, Thread *b) {
    if(a->getCpuBurstTime() == b->getCpuBurstTime())
        return 0;
    else if (a->getCpuBurstTime() > b->getCpuBurstTime())
        return 1;
    else
        return -1;
}
int PRIORITYCompare(Thread *a, Thread *b) {
    if(a->getThreadPriority() == b->getThreadPriority())
        return 0;
    else if (a->getThreadPriority() > b->getThreadPriority())
        return 1;
    else
        return -1;
}
int FIFOCompare(Thread *a, Thread *b) {
    return 1;
}
```
According to different arguments, OS put threads into SortedList.
```c++
// threads/scheduler.cc

Scheduler::Scheduler() {
    Scheduler(RR);
}
Scheduler::Scheduler(SchedulerType type)
{
    schedulerType = type;
    switch(schedulerType) {
    case RR:
        readyList = new List<Thread *>;
        break;
    case SJF:
        readyList = new SortedList<Thread *>(SJFCompare);
        break;
    case Priority:
        readyList = new SortedList<Thread *>(PRIORITYCompare);
        break;
    case FIFO:
        readyList = new SortedList<Thread *>(FIFOCompare);
        break;
    }
    toBeDestroyed = NULL;
}
```
## Test 
```./nachos FCFS/SJF/PRIORITY```
## Result 
### Test case 1 
#### FCFS 
Order:ABCD
```
*** thread 0 looped 0 times
*** thread 1 looped 0 times
*** thread 0 looped 1 times
*** thread 1 looped 1 times
*** thread 0 looped 2 times
*** thread 1 looped 2 times
*** thread 0 looped 3 times
*** thread 1 looped 3 times
*** thread 0 looped 4 times
*** thread 1 looped 4 times
Running thread A: cpu burst time remaining 7
Running thread A: cpu burst time remaining 6
Running thread A: cpu burst time remaining 5
Running thread A: cpu burst time remaining 4
Running thread A: cpu burst time remaining 3
Running thread A: cpu burst time remaining 2
Running thread A: cpu burst time remaining 1
Running thread A: cpu burst time remaining 0
Running thread B: cpu burst time remaining 3
Running thread B: cpu burst time remaining 2
Running thread B: cpu burst time remaining 1
Running thread B: cpu burst time remaining 0
Running thread C: cpu burst time remaining 8
Running thread C: cpu burst time remaining 7
Running thread C: cpu burst time remaining 6
Running thread C: cpu burst time remaining 5
Running thread C: cpu burst time remaining 4
Running thread C: cpu burst time remaining 3
Running thread C: cpu burst time remaining 2
Running thread C: cpu burst time remaining 1
Running thread C: cpu burst time remaining 0
Running thread D: cpu burst time remaining 4
Running thread D: cpu burst time remaining 3
Running thread D: cpu burst time remaining 2
Running thread D: cpu burst time remaining 1
Running thread D: cpu burst time remaining 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting! 
```
#### SJF 
Order:BDAC
```
*** thread 0 looped 0 times
*** thread 1 looped 0 times
*** thread 0 looped 1 times
*** thread 1 looped 1 times
*** thread 0 looped 2 times
*** thread 1 looped 2 times
*** thread 0 looped 3 times
*** thread 1 looped 3 times
*** thread 0 looped 4 times
*** thread 1 looped 4 times
Running thread B: cpu burst time remaining 3
Running thread B: cpu burst time remaining 2
Running thread B: cpu burst time remaining 1
Running thread B: cpu burst time remaining 0
Running thread D: cpu burst time remaining 4
Running thread D: cpu burst time remaining 3
Running thread D: cpu burst time remaining 2
Running thread D: cpu burst time remaining 1
Running thread D: cpu burst time remaining 0
Running thread A: cpu burst time remaining 7
Running thread A: cpu burst time remaining 6
Running thread A: cpu burst time remaining 5
Running thread A: cpu burst time remaining 4
Running thread A: cpu burst time remaining 3
Running thread A: cpu burst time remaining 2
Running thread A: cpu burst time remaining 1
Running thread A: cpu burst time remaining 0
Running thread C: cpu burst time remaining 8
Running thread C: cpu burst time remaining 7
Running thread C: cpu burst time remaining 6
Running thread C: cpu burst time remaining 5
Running thread C: cpu burst time remaining 4
Running thread C: cpu burst time remaining 3
Running thread C: cpu burst time remaining 2
Running thread C: cpu burst time remaining 1
Running thread C: cpu burst time remaining 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!
```
#### PRIORITY 
Order:BACD
```
*** thread 0 looped 0 times
*** thread 1 looped 0 times
*** thread 0 looped 1 times
*** thread 1 looped 1 times
*** thread 0 looped 2 times
*** thread 1 looped 2 times
*** thread 0 looped 3 times
*** thread 1 looped 3 times
Preemptive scheduling: interrupt->YieldOnReturn
*** thread 1 looped 4 times
*** thread 0 looped 4 times
Preemptive scheduling: interrupt->YieldOnReturn
Running thread B: cpu burst time remaining 3
Running thread B: cpu burst time remaining 2
Running thread B: cpu burst time remaining 1
Running thread B: cpu burst time remaining 0
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Running thread A: cpu burst time remaining 7
Running thread A: cpu burst time remaining 6
Running thread A: cpu burst time remaining 5
Running thread A: cpu burst time remaining 4
Running thread A: cpu burst time remaining 3
Running thread A: cpu burst time remaining 2
Running thread A: cpu burst time remaining 1
Running thread A: cpu burst time remaining 0
Preemptive scheduling: interrupt->YieldOnReturn
Running thread C: cpu burst time remaining 8
Running thread C: cpu burst time remaining 7
Running thread C: cpu burst time remaining 6
Running thread C: cpu burst time remaining 5
Running thread C: cpu burst time remaining 4
Running thread C: cpu burst time remaining 3
Running thread C: cpu burst time remaining 2
Running thread C: cpu burst time remaining 1
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Running thread C: cpu burst time remaining 0
Preemptive scheduling: interrupt->YieldOnReturn
Running thread D: cpu burst time remaining 4
Running thread D: cpu burst time remaining 3
Running thread D: cpu burst time remaining 2
Running thread D: cpu burst time remaining 1
Running thread D: cpu burst time remaining 0
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!
```
### Test case 2 
#### FCFS 
Order:ABCD
```
*** thread 0 looped 0 times
*** thread 1 looped 0 times
*** thread 0 looped 1 times
*** thread 1 looped 1 times
*** thread 0 looped 2 times
*** thread 1 looped 2 times
*** thread 0 looped 3 times
*** thread 1 looped 3 times
*** thread 0 looped 4 times
*** thread 1 looped 4 times
Running thread A: cpu burst time remaining 2
Running thread A: cpu burst time remaining 1
Running thread A: cpu burst time remaining 0
Running thread B: cpu burst time remaining 8
Running thread B: cpu burst time remaining 7
Running thread B: cpu burst time remaining 6
Running thread B: cpu burst time remaining 5
Running thread B: cpu burst time remaining 4
Running thread B: cpu burst time remaining 3
Running thread B: cpu burst time remaining 2
Running thread B: cpu burst time remaining 1
Running thread B: cpu burst time remaining 0
Running thread C: cpu burst time remaining 6
Running thread C: cpu burst time remaining 5
Running thread C: cpu burst time remaining 4
Running thread C: cpu burst time remaining 3
Running thread C: cpu burst time remaining 2
Running thread C: cpu burst time remaining 1
Running thread C: cpu burst time remaining 0
Running thread D: cpu burst time remaining 2
Running thread D: cpu burst time remaining 1
Running thread D: cpu burst time remaining 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting! 
```
#### SJF 
Order:ADCB
```
*** thread 0 looped 0 times
*** thread 1 looped 0 times
*** thread 0 looped 1 times
*** thread 1 looped 1 times
*** thread 0 looped 2 times
*** thread 1 looped 2 times
*** thread 0 looped 3 times
*** thread 1 looped 3 times
*** thread 0 looped 4 times
*** thread 1 looped 4 times
Running thread A: cpu burst time remaining 2
Running thread A: cpu burst time remaining 1
Running thread A: cpu burst time remaining 0
Running thread D: cpu burst time remaining 2
Running thread D: cpu burst time remaining 1
Running thread D: cpu burst time remaining 0
Running thread C: cpu burst time remaining 6
Running thread C: cpu burst time remaining 5
Running thread C: cpu burst time remaining 4
Running thread C: cpu burst time remaining 3
Running thread C: cpu burst time remaining 2
Running thread C: cpu burst time remaining 1
Running thread C: cpu burst time remaining 0
Running thread B: cpu burst time remaining 8
Running thread B: cpu burst time remaining 7
Running thread B: cpu burst time remaining 6
Running thread B: cpu burst time remaining 5
Running thread B: cpu burst time remaining 4
Running thread B: cpu burst time remaining 3
Running thread B: cpu burst time remaining 2
Running thread B: cpu burst time remaining 1
Running thread B: cpu burst time remaining 0
No threads ready or runnable, and no pending interrupts.
Assuming the program completed.
Machine halting!
```
#### PRIORITY
Order:BDCA
```
*** thread 0 looped 0 times
*** thread 1 looped 0 times
*** thread 0 looped 1 times
*** thread 1 looped 1 times
*** thread 0 looped 2 times
*** thread 1 looped 2 times
*** thread 0 looped 3 times
*** thread 1 looped 3 times
Preemptive scheduling: interrupt->YieldOnReturn
*** thread 1 looped 4 times
*** thread 0 looped 4 times
Preemptive scheduling: interrupt->YieldOnReturn
Running thread B: cpu burst time remaining 8
Running thread B: cpu burst time remaining 7
Running thread B: cpu burst time remaining 6
Running thread B: cpu burst time remaining 5
Running thread B: cpu burst time remaining 4
Running thread B: cpu burst time remaining 3
Preemptive scheduling: interrupt->YieldOnReturn
Running thread B: cpu burst time remaining 2
Running thread B: cpu burst time remaining 1
Running thread B: cpu burst time remaining 0
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Running thread D: cpu burst time remaining 2
Running thread D: cpu burst time remaining 1
Running thread D: cpu burst time remaining 0
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Running thread C: cpu burst time remaining 6
Running thread C: cpu burst time remaining 5
Running thread C: cpu burst time remaining 4
Running thread C: cpu burst time remaining 3
Running thread C: cpu burst time remaining 2
Running thread C: cpu burst time remaining 1
Running thread C: cpu burst time remaining 0
Preemptive scheduling: interrupt->YieldOnReturn
Preemptive scheduling: interrupt->YieldOnReturn
Running thread A: cpu burst time remaining 2
Running thread A: cpu burst time remaining 1
Running thread A: cpu burst time remaining 0
```
