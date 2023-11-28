# 2-rtos concurrent programming-RTOS: Thread synchronization methods
## 3.
The code uses the Round Robin scheduling algorithm. Due to the nature and way of writing in the code, different values of var 1 and var 2 are expected.
In the written code there are 4 threads with priority 10 and the Round-Robin scheduling method, and they have to execute the update_thread function until they are stopped after the end of the sleep function (60). The work of threads consists in checking the values, including the equality of the variables var 1 and var 2. In case of inequality, they will strive to equalize it. (the phenomenon of incrementing threads one after the other)
For a properly functioning program, the variables should be equal. The inequality may be influenced by the way the scheduling algorithm is written in the code (RR).

## 4.
#### Program work
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/7ecdaab8-2ddf-4fc7-9bbe-a6c9f2b6609b)

When the program is started, it starts the threads that perform the work. The system console shows that the values of var and var 2 are not equal when performing all actions. You could say that the program is not working properly. Threads are observed competing with each other for CPU time at the moment between the increment of var 1 and var 2. At this point, the CPU time is observed at the moment between the increment of var 1 and var 2. Then the CPU time is divided into subsequent threads, comparing variables and determining their equality or equality status. not equality. The purpose of the solution may be to use a different scheduling method, e.g. FIFO. Reducing the number of threads, condensing them into 1. Synchronization of program operation, including threads.
In the original version of the program, the values of var 1 and var 2 are not equal:

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/4d257201-4ff8-46f8-b372-63b10cb24ede)

Threads become mutually exclusive.

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/aa232f05-df25-41b9-b38c-17ca87835c15)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/d227c759-3730-4f7b-9a0b-d63e53f5a63b)

Initially, the unmodified code does not include commands to declare a shared global variable Mutex. Additionally, the previously defined mutex is not initialized. This process is also influenced by the beginning and end of the critical variable. Placing or retrieving an item from the buffer is a critical section. (queuing, waiting, lack of consistency in relation to e.g. different values of var 1 and var2). The solution to this situation is to declare the mutex global variable declaration command.
pthread_mutex_t var_mutex; and its initiation and definition. After completing the original version of the code, the process adopts the same scheduling values, actions var1=var2. The process is running correctly.

## 5.
Mutex after modification – program execution result:

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/37244b67-c63d-4e0e-a111-1fa5fa61fd7f)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/f459c1d9-fb6f-4e3f-8d28-a8610bc79444)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/25c05360-019f-43a3-aa80-339b40e25f4d)

After modification and completion of the code, equal values of var 1 and var 2 are observed. The thread scheduling has been changed.

## 6.
Changing the state in the Condvar program from 0 to 1 and from 1 to 0. As per the comment, changing 2 states.
(assume: thread 0 is the producer - start of critical section)

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/5fecf42d-4a2a-4422-b7f2-51d70aa5f7ab)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/dd1d7b84-fb1d-4531-ac50-d1970cfb9dc5)

The state variable is supposed to do the work only 1 time, unless otherwise defined.
The state flag specifies the state value of which thread should do its work. They control all threads but only with the correct state setting.

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/9f4423bd-2de7-4e67-bcba-f9a2f1373ea3)

## 7.
The program changed from state 0 to 1 and then went to 3. It does not work properly, it hangs. The pthread_cond_signal function informs about the variable the thread waiting the longest, not the thread waiting for this change, e.g. here the state thread. The pthread_cond_broadcast function signals the modification of the condition variable so that all threads can react, which is the solution to the problem.

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/7ea56472-12ff-4d9f-92f6-49adff376c95)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/21b80bbd-fc9a-4629-9410-810037805786)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/33c5e8bb-1ad7-451e-a9ad-b8721bdc38dd)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/de9c66d9-cdff-4be7-8297-1dac2ef14ba3)

Observed state 1 to 3 change as per comment, only after modification. In the initial state, the process does not match the expectation written in the comment.

With Signal, the program will run when we have 2 threads, the second thread waits for a change in the state of the conditional variable. Program locking problem. The solution to the problem is to change the way of signaling a change in the condition variable from pethret_cont_signal to pethret_cont_broadcast. It tells us which thread needs to do the work. All of them react using pethret_cont_broadcast, but the work will be done by the one that has the state variable properly set. When it is equal to 0, the zero state will do the work. If the variable =1, then the first one and so on. All of them are launched, but they end their operation quickly, they will check whether the variable is different, but if it is different from the state, they will not go inside, if it is equal, they will go inside and you will be able to do your work. A change to the scheduling method should be applied.

The program works incorrectly because it blocks, because information about the state of variables does not reach all threads where it should be sent. The solution is to change the way of signaling the change of the condition variable from the function from pethret_cont_signal to pethret_cont_broadcast..

comment the states are to change from 0 to 1, from 1 to 2 when 1 is even, or from 1 to 3 when odd, and from 2 to 0 and 3 to 0.
With the Signal function, when the first thread reacts to a change in the state of the conditional variable, the information is removed from the system so the remaining threads have no information about the change. With the pethret_cont_broadcast function, the information is held until all threads respond to the request.
changing the state of the conditional variable. Only the variable with a properly set state value will do the work.

## 8.
After changing to "broadcast" you may notice slightly more buffering

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/db5c04a6-ccbf-428f-87e7-6325b6e4b09b)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/177ab714-1ef2-4844-b92a-dd90bb3be1ec)

After changing the signaling of the conditional variable from Signal to broadcast, the program works correctly, the states change as follows: 3 to 0, 0 to 1, then m to 3, 0 again, and then 1, then 2, etc.

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/1c04488f-8bbe-4138-b62d-681c57688ccb)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/5eae703a-8c33-4e66-911e-7b514d0ccb37)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/9bc9e02f-0b82-4cb8-874d-48c539cebd57)

Change action from 0 to 1 from 1 to 2. As described in the comment, there are 3 state changes.

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/953c50ad-8e37-4d0f-b1ee-f2957bfbbaf3)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/c5364e9a-5cd2-4feb-8875-0ecde0afccb9)

The states change much faster, threads do not block, it was enough to change from pthread_cond_ Signac(&var_cond) to pthread_cond_ broadcast(&var_cond)

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/16664bea-8194-43c6-b182-50ba24a2df67)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/5c455c78-ca7e-430f-acfe-555fed1a244e)

## 9.
a) Path: cd /dev/shm

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/55e08b51-d4c9-436c-b78f-be1e5e0983f7)

b) semaphore name: sem.Semex.
(sem- "shortcut" as semaphore identifier, Semex-name).

c) During the first run, there is no noticeable problem, but it occurs again. Its solution is to unblock and enable (add) the sem_unlink function. This function removes the name of the semaphore depending on its handling function. Without unlocking this function, the written code can only work once, and after running it, we get any number of possible runs. With the sem_unlink function we do not remove the semaphore from the namespace, it exists as long as it is necessary for the process to restart.
Starting for the first time:

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/8c070f77-369f-4bd4-92a2-7715eba6aaab)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/ffdf419e-5716-4288-a491-31de92d7dc95)

Starting for the second time and states during operation:

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/9e898910-2ead-4cac-bf0a-321e38c164bd)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/58900837-df74-451d-903a-e0f2d3912ae2)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/21364d3b-a53f-4903-888d-b4ce8555f535)

Run 3 times:

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/2ea78493-3055-4070-b398-fc40d740a6b2)

Run 4th time:

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/888457a3-dabd-4f8b-a4c9-a9e647fec8e0)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/4121a60f-0aa2-4e35-a552-c25cacabaca1)

After unlocking sem_unlink:

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/5ca4facd-d42f-453e-96b0-948ffbf6cee8)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/070d9656-fed2-4ce3-a77a-5bb1f246bd9f)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/f06127ab-f816-435e-8571-211042d7774d)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/e0cdd8b1-f721-40c1-b225-8844b52facfb)

## 10.
The unnamed semaphore is run directly by the kernel and is a bit faster. The named semaphore is added to the queue, which makes the process slower and its name cannot be repeated. (named semaphore cannot occur more than 1 with the same name)
A defined named semaphore

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/aabcdc2e-d2c4-49a5-8258-b9b94a37c79b)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/c72aa657-8cd6-4c37-abb0-0b757143d1eb)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/e166cf99-b95e-4f68-a3f3-e2c51ffa92cd)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/8da6ec79-3748-4974-9a8a-9c915a6a2d45)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/be689b99-0cc2-43c5-8505-eaf9bffa19bb)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/f6471491-4e50-493c-a5c4-1e091ece8d5f)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/bce729b4-42c2-46a1-aa78-cfe6807e4537)

Undefined, unnamed semaphore

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/eb9378b3-0644-4357-a92e-cde4ec87a5e9)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/f00629ba-c360-4c83-94ba-c8e931e2e124)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/9793aed6-c532-4809-85bf-cbc4af515b5f)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/deec8555-a3f5-4e17-ba08-ad9458c01720)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/b9f718c6-20f8-46ce-8c7c-345d384cdc0e)

Unnamed semaphores are suitable for synchronizing threads within a single process. An unnamed semaphore is accessed via its address. It can also be used to synchronize processes as long as it is placed in shared memory.

An unnamed semaphore is accessed via the semaphore address. Hence the name unnamed semaphore.
Named semaphores are accessed by name. This type of semaphore is more suitable for synchronizing processes than threads. Unnamed semaphores run faster than named ones.

Before a semaphore can be used, it must be declared as an object of type sem_t and the memory used by the semaphore must be explicitly allocated to it. If the semaphore is to be used in different processes, it should be placed in previously allocated shared memory.

The problem is in sem_unlink deleting the semaphore when we unlock
There is a sharing of common process resources in the code. The global variable state indicates which thread is working.

## 11.
Barriers work in such a way that after creating two threads, the thread associated with the process stops at pthread_barrier_wait(&barrier) and the remaining threads work at this time. After finishing work, the thread associated with the process continues (threads are deleted).
Barriers are used to synchronize threads. They meet at a precisely defined place in the code, all of them reach the barrier and only when they are all at it do they continue working after passing through it. The thread that has waited the longest and reached the barrier last has the priority to pass with the same priority. In the case of different priorities, priority is determined by the value of the priority, and the one with the highest value goes first.

Memory barrier - a point in a program at which the computer system is forced to synchronize the contents of memory, i.e. to line up all writes and reads. This is needed because some microprocessors do not immediately update the contents of main memory or cache in order to obtain better performance, e.g. by replacing repeated writes to the same address with a single, delayed memory reference.
The memory barrier is implemented by a special processor instruction.

Without using a barrier, a situation may arise when threads or processes referring to the same memory address will read completely different data, which will be an error.

The barrier is a mechanism for synchronizing a group of threads. Threads after reaching the barrier are suspended until the last one reaches it - then they all continue. The barrier is a number greater than zero that determines how many threads are included in the group.
When the barrier is reached by all threads, its state (counter) is automatically initialized to the value set by the last call to pthread_barrier_init.

The barrier is initialized by the pthread_barrier_init function and removed by the pthread_barrier_destroy function. The barrier may have additional attributes.

## 12.
The modification consisted of inserting a while loop, which allows the program to run cyclically. While working, the expected result from the command is observed. New threads are created and the signal frequency changes. A noticeable variable is the higher percentage of CPU time used %Cpu(s).
Without while

![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/3cdbce5d-ffa9-45c5-ad4f-3cdf00412ec9)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/5e07db5d-08d0-431c-a891-fd989a08b412)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/3424b472-b8be-4166-bc61-c2adaa5f6947)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/7c9b8141-a7e4-4881-ae11-83797d053c47)

With while
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/f13de3b3-c847-4572-8739-e7db78437369)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/f9d4ce06-980f-4a97-8746-8a75d9769b96)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/a49fbacf-33d0-473d-8c19-c070974d952e)

#### Result:
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/034c7581-f233-4932-8d7b-2d301c56078b)
![image](https://github.com/AsiaEwa/2-rtos/assets/101841759/f7176559-ce81-4075-b582-fc21cbbc077a)

