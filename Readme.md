# Starve Free Reader-Writer's Problem

## The Problem Statement
To write pseudocode for Readers-Writers problem, such that neither the reading nor the writing processes are starved.

**Condition 1 -** Any number of reading processes can execute parallely.

**Condition 2 -** At most one writing process can execute at a time.

**Condition 3 -** Reading and writing processes can not be executed parallely.

## Thought Process

**Initial thought -** We can use two queues namely readqueue and writequeue. At any moment of time, if there is a waiting reading process, it immediately starts executing in parallel to already executing reading processes and all the writing processes in the writequeue wait. Only when the readqueue is completely empty, we give chance to one writing process from the writequeue to execute.

**Problem with the initial thought -** Assume a case, in which reading processes never stop arriving. In this case, readqueue is never empty, and thus the writing processes in the writequeue keep waiting indefinitely, i.e. they starve. This is happening because we are prioritizing reading processes over writing processes.

**What can be done better -** Rather than making two queues in the first place, we can make just one queue for both reading and writing processes.

## Solution

Make a common queue for both reading and writing processes. Now we iterate in the queue according to the FIFO principle.

**If a reading process is encountered -** If currently, a writing process is being executed, then wait; Otherwise start executing the reading process.

**If a writing process is encountered -** If currently, any process is being executed, then wait; Otherwise start executing the writing process.

**Why starvation will not occur -** As we can see, the algorithm must execute a writing process after exectuting all the processes arrived in the queue before that process. Thus, a writing process will not starve. Similarly, a reading process either executes after all the processes arrived in the queue before it, or it runs in parallel with the previous process. In both the cases, reading processes don't starve.

**Technical Details -** We will use three semaphores and a counter variable. Counter `readcount` stores current number of reading processes being executed. Semaphore `readcount_mutex` is responsible for mutual exclusion of operations related to `readcount`.  Semaphore `read_mutex` represents whether any reading process is being executed right now or not. Similarly, semaphore `write_mutex` represents whether any writing process is being executed right now or not.

## Global Variables
```cpp
Semaphore readcount_mutex = 1; // To mutually exclude access of readcount variable
Semaphore read_mutex = 1; // No reader is reading right now
Semaphore write_mutex = 1; // No writer is writing right now 
int readcount = 0; // Represents number of readers currently
```

## Implementation: Reader
```cpp
while (true){
    
    // Entry Section

    wait(write_mutex);
        
        wait(readcount_mutex);
            readcount++;
        signal(readcount_mutex);

    if (readcount > 0)
        wait(read_mutex);
    
    signal(write_mutex);

    // Critical Section

    // Exit Section

        wait(readcount_mutex);
            readcount--;
        signal(readcount_mutex);

    if (readcount == 0)
        signal(read_mutex);

}
```

## Implementation: Writer
```cpp
while (true){
    
    // Entry Section

    wait(read_mutex);
    wait(write_mutex);

    // Critical Section

    // Exit Section

    signal(write_mutex);
    signal(read_mutex);

}
```