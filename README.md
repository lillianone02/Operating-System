# OS Nachos Project 
* Project 1: Thread Management
* Project 2: System Call 
* Project 3: CPU Scheduling
* Project 4: Memory management  

# VirtualBox 
* Oracle VM VirtualBox   
  + <https://www.virtualbox.org/wiki/Downloads>
* 32-bit Ubuntu 22.10 
  + <https://ubuntu.com/download/desktop> 
 
# Setup 
Install g++ csh 
```
sudo apt-get install g++
sudo apt-get install csh
sudo apt-get install make
```
Download NachOS and Cross Compiler 
```
wget -d http://cc.ee.ntu.edu.tw/~farn/courses/OS/OS2015/projects/project.1/nachos-4.0.tar.gz
wget -d http://cc.ee.ntu.edu.tw/~farn/courses/OS/OS2015/projects/project.1/mips-x86.linux-xgcc.tar.gz
```

# Install NachOS 
```
tar -xvf nachos-4.0.tar.gz
```
```
sudo mv mips-x86.linux-xgcc.tar.gz /
cd /
sudo tar -zxvf mips-x86.linux-xgcc.tar.gz
```
```
GCCDIR = /mips-x86.linux-xgcc/
CPP = /mips-x86.linux-xgcc/cpp0
CFLAGS = -G 0 -c $(INCDIR) -B/mips-x86.linux-xgcc/
```
```
cd ~/nachos-4.0/code
make
```

# Test 
```
cd ./userprog
./nachos -e ../tset/test1
```
```
cd ./userprog
./nachos -e ../tset/test2
```
