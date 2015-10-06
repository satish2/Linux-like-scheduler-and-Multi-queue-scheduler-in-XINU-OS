# Linux-like-scheduler-and-Multi-queue-scheduler-in-XINU-OS
Process Scheduling - Linux like scheduler and Multi-Queue scheduling


<h2>1) Linux-like Scheduler (based loosely	on the 2.2 Linux kernel)</h2>

This scheduling algorithm tries to loosely emulate the Linux scheduler in 2.2 kernel. In this assignment, we consider all the processes "conventional processes" and uses the policies of the SCHED_OTHER scheduling class within 2.2 kernel. With this algorithm, the CPU time is divided into epochs. In each epoch, every process has a specified time quantum, whose duration is computed at the beginning of the epoch. An epoch will end when all the runnable processes have used up their quantum. If a process has used up its quantum, it will not be scheduled until the next epoch starts, but a process can be selected many times during the epoch if it has not used up its quantum.

When a new epoch starts, the scheduler will recalculate the time quantum of all processes (including blocked ones). This way, a blocked process will start in the epoch when it becomes runnable again. New processes created in the middle of an epoch will wait till the next epoch. For a process that has never executed or has exhausted its time quantum in the previous epoch, its new quantum value is set to its process priority (i.e., quantum = priority). A quantum of 10 allows a process to execute for 10 ticks (10 timer interrupts) within an epoch. For a process that did not get to use up its previously assigned quantum, we allow part of the unused quantum to be carried over to the new epoch. Suppose for each process, a variable counter describes how many ticks are left from its quantum, then at the beginning of the next epoch, quantum = floor(counter/2) + priority. For example, a counter of 5 and a priority of 10 will produce a new quantum value of 12.

During each epoch, runnable processes are scheduled according to their goodness. For processes that have used up their quantum, their goodness value is 0. For other runnable processes, their goodness value is set considering both their priority and the amount of quantum allocation	left: goodness = counter + priority. Again, round-robin is used among processes with equal goodness.

The priority can be changed by explicitly specifying the priority of the	process during the create() system call or through the chprio() function. Priority changes made in the middle of an epoch, however, will only take effect in the next epoch.

An example of how processes should be scheduled under this scheduler is as follows:

If there are processes P1,P2,P3 with priority 10,20,15 then the epoch would be equal to 10+20+15=45 and the possible schedule (with quantum duration specified in the braces) can be: P2(20), P3(15),	P1(10), P2(20), P3(15), P1(10), but not: P2(20), P3(15), P2(20), P1(10).
 
<h2>2) Multiqueue</h2>

This scheduler is similar to the scheduler in part 1, except that it supports two queues: a Real-Time queue and a Normal queue.

You need to add to XINU a modified version of the create() function; name the new function createReal(), which does the same work as create() does, except that the processes it creates are considered as Real-Time processes. Processes created by the original create() function are considered as Normal processes. Real-Time processes go into the Real-Time queue and Normal processes go into the normal queue.

At the start of an epoch, the scheduler generates a random number to decide which queue to schedule in this epoch. It should ensure that in 70% time, the real-time queue is selected and in 30% time, the normal queue is selected.

For processes in the Real-Time queue, the scheduling is round robin. Each process gets a 100-tick quantum. When all runnable processes run up their quantum, the epoch ends.

For processes in the Normal queue, the scheduling algorithm is the same as in part 1.

If a queue is selected but it contains no runnable processes, the scheduler automatically selects the other queue. Again, the NULL process is selected to run when and only when there are no other ready processes in both queues.

Processes created by default (e.g., the master process) are Normal processes.
