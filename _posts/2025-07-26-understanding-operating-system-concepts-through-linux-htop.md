---
title: "Understanding Operating System Concepts through linux htop"
date: 2025-07-26 00:00:00 +0000
categories: [Linux, htop]
tags: [linux, htop, operating systems]
---

## Install htop
~~~
sudo apt update
sudo apt install htop
~~~

![htop screenshot](images/htop-ss.png)


## 🖥️ CPU Cores

At the top of the htop window, you'll see several bars like []. These represent CPU cores, and the number of bars depends on your processor. Mine shows 12 cores.

In each core we can see a %. The linux kernel is constantly swicthing tasks between cores, and the % is a snapshot of how busy each core at that moment. 

### 🔄 Task switching

This is key function of the operating system. There are two main concepts. Quntum(time slice) and context swtiching. 

#### Quntum: 

The CPU gives fair time for each thread(small task) to run. When the time ends, it paused and the CPU moves to next thread even if the current task isn't finished. So the CPU quickly switching between tasks and feels like multitasking, but in reality it just rapidly switching tasks.

##### Context Switching:

If a task ends early before the quantum runs out, the CPU switches to the next one.

## 📊 Tasks: 151, 845 thr; 175 kthr; 1 running 

Processes(full task): 151

Threads(small sub-task): 845

Kernel Threads(task handle directly by kernel): 175

Runing threads: 1

Other thread are not active but they are ready to run(waiting in the queue for and event)

## 📈 Load Averaege: 0.58 0.77 0.66

On average, 0.69 tasks(threads) either running or waiting in the last 1 minute

0.81 over the last 5 minutes

0.66 over the last 15 minutes

Note: Thread is a small task within a process. Eventhough a process consist of many threads, OS counts only active ones. In my case, I have 12 cores. So, my system is healthy and not overloaded.

The system is healthy if --> Load < cores

## ⏳ Uptime 

This shows how long linux system has been actuvely running, excluding sleep/suspend time.

## 💾 Memory(RAM)

This is the short time memory. When we open a app, RAM holds data so the CPU can access it faster than reading from the permenent storage like hard disk.

This shows the available vs currently used memory.

## 💽 Swap

This is a extra memory CPU can use when the main memory is full. It uses the hard disk or SSD as backup. But this is slow. When we experience some lag, it might be because the system is using swap.

## 🧠 Main Column

Shows how much RAM (main memory) is being used by each process.

## 📀 I/O column

Shows how much disk read/write activity is happening for each process.

## ⌨️ Handy Keys in htop

F5(Tree view)

Shows processes in a hierarchical tree structure, so you can see parent-child relationships between processes.

Parent-child: Parent is like the main program, and child is the sub-task. 

Example:

MS word: Parent
New document: Child

F6(sort):

You can sort processes by CPU, memory, I/O, etc., by pressing F6.

/ (Search):

Opens a search bar where you can type a process name or PID(Process ID) to quickly locate it in the list.

F9 (Kill Process):

This is just like using "End task" in windows task manager.
