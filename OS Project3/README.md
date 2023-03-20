# Operating System CPU Scheduling 
We implement and compare three CPU Scheduling Algorithms: First-Come-First-Service(FCFS), Shortest-Fob-First(SJF),and Priority.  
We use teo cases to compare the order and CPU burst time between the three Algorithms.  

# Implementation 
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
```
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
We add SchedularType
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
