# Starve-Free-Readers-Writers-Problem
<b>CSN - 232 Assignment</b> 

Readers-Writers Problem is a classical situation where modification and reading of a shared resource simultaneously by concurrent threads occurs. 

There are two classes of users
- Readers - only read database
- Writers - read and modify database

There are three variations to this problem. We will be looking into the Third one.
## First Readers-Writers Problem
It gives  priority to the readers. When at least one reader is currently accessing the resource, it allow new readers to access it. It causes starvation if any writers waiting to modify the database and new readers arrive all the time, and writers will never get access. 

## Second Readers-Writers Problem
Here, writers are given priority. So, now the readers will starve until the writer is done modifying the resource.

## Third Readers-Writers Problem
As compared to the previous two problems, this problem overcomes the drawbacks of starvation. In this solution, all the processes are given priority according to their arrival (FIFO). When a writer wants to modify the resources, it will wait until the current readers are done and then enter a critical area. Then no new reader can enter the critical area until the writer is done. When leaving, the reader checks if a writer is waiting and if it was the last reading process. If true, the reader signals the writer to enter the critical area.


### Solution

First we define the global variables and use semaphore which is a more generic synchronisation concept.

```cpp
Semaphore in, out, write;
initialize(in, 1);            //
initialize(out, 1);
initialize(write, 0);

int counter_in = 0;           // Reader started reading
int counter_out = 0;          // Reader completed reading

bool writer_wait = false;     // If a writer is waiting then it set to true
```

#### Reader's Process

Multiple readers can read the resource simultaneously . 
Once a writer has entered the waiting list no new reading process can start until the writer gets access.

```cpp
    // All processes are given an unique id
    wait(in);                  // Wait for the permission
    counter_in++;              // increases as reader starts reading
    signal(in);                //  signal other readers 

    //Crtical Section
    
    wait(out);                 // Wait 
    counter_out++;             // increases after reading complete

    if (writer_wait && counter_in == counter_out){       //if writer is waiting and this was the last reafer then signal write semaphore
        signal(write);
    }

    signal(out);                // signal out semaphore
```

#### Writer's Process

The program knows if a writer is waiting if `writer_wait = true` and stops any new reading process using `in semaphore`. 
If another writer arrives it will get queued in the `in semaphore` according to FIFO. 

```cpp
    // All processes are given an unique id
    wait(in);
    wait(out);

    if (counter_in == counter_out){    // if no reading process active
        signal(out);
    }
    else{                              // wait for readers to finish and indicate that waiting writer is present
        writer_wait = true;
        signal(out);                   
        wait(write);
        writer_wait = false;           // writer is no longer waiting 
    }

    //Critical Section

    signal(in);                        // signal reader to wake                     
```


Therefore no reader or writer starve in this solution. 

### References 
- [arCiv:1309.4507](https://arxiv.org/abs/1309.4507)