
# Reader Writer Problem

The Reader Writer problem is a classic synchronization problem in computer science where multiple threads compete for access to a shared resource. The challenge arises when some threads only read the resource, while others need to write to it. To ensure data consistency and avoid issues like deadlocks or starvation, the problem requires synchronization techniques to manage access to the shared resource. In particular, it involves managing the conflicts between reading and writing operations, ensuring that multiple readers can access the resource simultaneously while blocking writers and ensuring that only one writer has exclusive access to the resource at a time. Different synchronization techniques like semaphores, locks, or monitors can be used to solve the Reader Writer problem.

# Implementation using semaphores
## Assumptions:

* There is a shared resource that can be read and written to.
* Multiple readers can access the resource simultaneously, but only one writer can access it at a time.
* Readers have priority over writers (i.e., if there are readers waiting to access the resource, they should be served before any waiting writers).
## Initialization:

* Two semaphores: "read_mutex" and "write_mutex", both initialized to 1.
* An integer variable: "read_count" initialized to 0.

## Reader thread:

* Wait on "read_mutex" semaphore to acquire read access.
* Increment "read_count" to signal that a reader is accessing the resource.
* If "read_count" is 1 (i.e., this is the first reader), wait on "write_mutex" to block writers.
* Signal "read_mutex" semaphore to release read access.
* Read the shared resource.
* Wait on "read_mutex" semaphore to acquire read access.
* Decrement "read_count" to signal that a reader has finished accessing the resource.
* If "read_count" is 0 (i.e., this is the last reader), signal "write_mutex" to unblock writers.
* Signal "read_mutex" semaphore to release read access.
## Writer thread:

* Wait on "write_mutex" semaphore to acquire write access.
* Write to the shared resource.
* Signal "write_mutex" semaphore to release write access.
## Explanation:

The "read_mutex" semaphore is used to ensure that multiple readers don't access the resource simultaneously and to allow only one reader to update the "read_count" variable at a time.
The "write_mutex" semaphore is used to block writers while readers are accessing the resource, so that no writer can modify the resource while it is being read.
The "read_count" variable is used to keep track of how many readers are accessing the resource at any given time.
When a reader acquires the "read_mutex" semaphore, it increments "read_count" to signal that it is accessing the resource, and if it is the first reader, it blocks writers by waiting on the "write_mutex" semaphore. When the reader finishes accessing the resource, it decrements "read_count" and if it is the last reader, it unblocks writers by signaling the "write_mutex" semaphore.
When a writer acquires the "write_mutex" semaphore, it blocks all other readers and writers, writes to the shared resource, and releases the semaphore when it is done.

# Problem In Above Solution (Starvation of Writers)
The above solution suffers from the "Starvation of Writers" problem. This occurs when there are one or more writers waiting to acquire the "write_mutex" semaphore but keep getting blocked by the "readers". Since multiple readers can access the resource simultaneously, a writer may keep waiting indefinitely if there are always new readers arriving and updating the "read_count" variable, which prevents the writer from acquiring the "write_mutex" semaphore.

In other words, the solution prioritizes readers over writers, and as a result, writers may starve for an indefinite period, waiting for the readers to finish accessing the resource.

To address this issue, one possible solution is to implement a mechanism that gives priority to the waiting writers over the readers. This can be achieved by modifying the solution to allow writers to acquire the "write_mutex" semaphore even if there are readers currently accessing the resource. However, this approach requires careful consideration to ensure that data consistency is maintained while avoiding potential issues like writer starvation or reader-writer conflicts.

# Solution of above Problem (Starve Free Solution)
## A possible solution by using an additional semaphore

## Initialization:

* A "read_mutex" semaphore initialized to 1.
* A "write_mutex" semaphore initialized to 1.
* A "no_readers" semaphore initialized to 1.
* An integer variable "read_count" initialized to 0.
## Reader thread:

* Wait on "no_readers" semaphore to block writers while readers are accessing the resource.
* Wait on "read_mutex" semaphore to acquire read access.
* Increment "read_count" to signal that a reader is accessing the resource.
* If "read_count" is 1 (i.e., this is the first reader), wait on "write_mutex" semaphore to block writers.
* Signal "read_mutex" semaphore to release read access.
* Signal "no_readers" semaphore to allow other readers to access the resource.
* Read the shared resource.
* Wait on "read_mutex" semaphore to acquire read access.
* Decrement "read_count" to signal that a reader has finished accessing the resource.
* If "read_count" is 0 (i.e., this is the last reader), signal "write_mutex" semaphore to unblock writers.
* Signal "read_mutex" semaphore to release read access.

### Pseudocode
```cpp
do {

    wait(no_reader);    //  the next section executes if the process is not blocked by other semaphore
                        //  this mutex is present in both reader and writer and prevents starvation by giving equal opportunity to both

    wait(read_mutex);
    read_count++; //The reader count increases by 1.

    if (read_count == 1) {  // this implies that the first reader tries to access

        wait(write_mutex);  // this makes sure that even if there is one reader, writere is not able to enter
    }

    signal(read_mutex); // other readers that have acquired the no_reader can enter while this current reader is inside the critical section

    signal(no_reader);
    //  after the reader enters the critical section, it frees the no_reader semaphore
    //  This no_reader semaphore can be acquired by either a reader or writer, whichever arrives first

    // CRITICAL_SECTION

    wait(read_mutex);
    read_count--; //The reader count decreases by 1.

    if (read_count == 0) { // this denotes the last reader
        signal(write_mutex);
        // when the last reader leaves then a writer can enter
    }
    signal(read_mutex);


} while (true);

```
## Writer thread:

* Wait on "no_readers" semaphore to block writers while readers are accessing the resource.
* Wait on "write_mutex" semaphore to acquire write access.
* Write to the shared resource.
* Signal "write_mutex" semaphore to release write access.
* Signal "no_readers" semaphore to allow other readers and writers to access the resource.
### Pseudocode

```cpp
do {
 
    wait(no_reader);
    //  writer process also waits for the no_reader simultaneously 
    //  this can be aquired even if a reader is present inside the critical section

    wait(write_mutex);
    //  the writer process would be blocked while waiting
    //  once free it will acquire the write_mutex and enter the critical section

    signal(no_reader);
    //  after the writer enters the critical section it frees the no_reader
    //	now any of reader or writer which comes first can aquire the no_reader mutex 

        // CRITICAL_SECTION

    signal(write_mutex);
} while(true);
```

## Explanation:

* The "read_mutex" semaphore is used to ensure that multiple readers don't access the resource simultaneously.
* The "write_mutex" semaphore is used to provide exclusive access to the shared resource and to block writers.
* The "no_readers" semaphore is used to block writers while readers are accessing the resource. It is initialized to 1 to ensure that writers are initially blocked until a reader has accessed the resource at least once.
* When a reader acquires the "no_readers" semaphore, it blocks writers and prevents them from acquiring the "write_mutex" semaphore. It then acquires the "read_mutex" semaphore, increments "read_count", and if it is the first reader, waits on the "write_mutex" semaphore to block writers. It then signals the "read_mutex" semaphore to release read access and the "no_readers" semaphore to allow other readers to access the resource. When the reader finishes accessing the resource, it acquires the "read_mutex" semaphore, decrements "read_count", and if it is the last reader, signals the "write_mutex" semaphore to unblock writers, and releases the "read_mutex" semaphore.
* When a writer acquires the "no_readers" semaphore, it blocks other writers and readers and waits on the "write_mutex" semaphore to acquire exclusive access to the shared resource. * It then writes to the resource and releases the "write_mutex" semaphore. Finally, it signals the "no_readers" semaphore to allow other readers and writers to access the resource.
* By using the "no_readers" semaphore to block writers and ensuring that at least one reader has accessed the resource before allowing writers to acquire the "write_mutex" semaphore, this solution guarantees that writers will not starve. At the same time, it ensures data consistency by using the "read_mutex" and "write_mutex".
