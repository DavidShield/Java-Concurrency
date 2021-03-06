# Java-Concurrency

## Notes of https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html
## This lesson introduces the platform's basic concurrency support and summarizes some of the high-level APIs in the java.util.concurrent packages.

## Processes and Threads
### processes

Processing time for a single core is shared among processes and threads through an OS feature called time slicing.

A process has a self-contained execution environment. A process generally has a complete, private set of basic run-time resources; in particular, each process has its own memory space.

Processes are often seen as synonymous with programs or applications. A single application may in fact be a set of cooperating processes. To facilitate communication between processes, most operating systems support Inter Process Communication (IPC) resources, such as pipes and sockets. IPC is used not just for communication between processes on the same system, but processes on different systems.

Most implementations of the Java virtual machine run as a single process. A Java application can create additional processes using a ProcessBuilder object.

### Threads

Threads exist within a process — every process has at least one. Threads share the process's resources, including memory and open files. This makes for efficient, but potentially problematic, communication.

Multithreaded execution is an essential feature of the Java platform. Every application has at least one thread — or several, if you count "system" threads that do things like memory management and signal handling. But from the application programmer's point of view, you start with just one thread, called the main thread. This thread has the ability to create additional threads.

## Thread Objects
Each thread is associated with an instance of the class Thread.
 There are two basic strategies for using Thread objects to create a concurrent application.

* To directly control thread creation and management, simply instantiate Thread each time the application needs to initiate an asynchronous task.
* To abstract thread management from the rest of your application, pass the application's tasks to an executor.

## Defining and Starting a Thread

There are two ways to do this:

Provide a Runnable object. The Runnable interface defines a single method, run, meant to contain the code executed in the thread. The Runnable object is passed to the Thread constructor, as in the HelloRunnable example:

    public class HelloRunnable implements Runnable {

        public void run() {
            System.out.println("Hello from a thread!");
        }

        public static void main(String args[]) {
            (new Thread(new HelloRunnable())).start();
        }

    }
Subclass Thread. The Thread class itself implements Runnable, though its run method does nothing. An application can subclass Thread, providing its own implementation of run, as in the HelloThread example:

    public class HelloThread extends Thread {

        public void run() {
            System.out.println("Hello from a thread!");
        }

        public static void main(String args[]) {
            (new HelloThread()).start();
        }

    }

The first idiom, which employs a Runnable object, is more general, because the Runnable object can subclass a class other than Thread. The second idiom is easier to use in simple applications, but is limited by the fact that your task class must be a descendant of Thread.

## Pausing Execution with Sleep
Thread.sleep causes the current thread to suspend execution for a specified period. sleep times are not guaranteed to be precise, because they are limited by the facilities provided by the underlying OS. 

    public class SleepMessages {
        public static void main(String args[])
            throws InterruptedException {
            String importantInfo[] = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
            };

            for (int i = 0;
                 i < importantInfo.length;
                 i++) {
                //Pause for 4 seconds
                Thread.sleep(4000);
                //Print a message
                System.out.println(importantInfo[i]);
            }
        }
    }
    
Notice that main declares that it throws InterruptedException. This is an exception that sleep throws when another thread interrupts the current thread while sleep is active. Since this application has not defined another thread to cause the interrupt, it doesn't bother to catch InterruptedException.

## Interrupts
An interrupt is an indication to a thread that it should stop what it is doing and do something else.

    public class Test implements Runnable {
        public void run() {
            System.out.println("the name of the thread " + Thread.currentThread().getName());
            if (Thread.interrupted()) {
                System.out.println("The thread is interrupted");
            }
        }

        public static void main(String[] args) {
            Thread t1 = new Thread(new Test(), "t1");
            t1.start();
            t1.interrupt();
        }
    }
    
## Joins
The join method allows one thread to wait for the completion of another. If t is a Thread object whose thread is currently executing,

    t.join();
causes the current thread to pause execution until t's thread terminates. 
  
    t.join(int32)
Blocks the calling thread until the thread represented by this instance terminates or the specified time elapses
# Synchronization 
## Thread Interference
Interference happens when two operations, running in different threads, but acting on the same data, interleave. This means that the two operations consist of multiple steps, and the sequences of steps overlap.

## Memory Consistency Errors
Memory consistency errors occur when different threads have inconsistent views of what should be the same data. The causes of memory consistency errors are complex and beyond the scope of this tutorial. 

## Synchronized Methods
To make a method synchronized, simply add the synchronized keyword to its declaration:

    public class SynchronizedCounter {
        private int c = 0;

        public synchronized void increment() {
            c++;
        }

        public synchronized void decrement() {
            c--;
        }

        public synchronized int value() {
            return c;
        }
    }
If count is an instance of SynchronizedCounter, then making these methods synchronized has two effects:

First, it is not possible for two invocations of synchronized methods on the same object to interleave. When one thread is executing a synchronized method for an object, all other threads that invoke synchronized methods for the same object block (suspend execution) until the first thread is done with the object.
Second, when a synchronized method exits, it automatically establishes a happens-before relationship with any subsequent invocation of a synchronized method for the same object. This guarantees that changes to the state of the object are visible to all threads.

Note that constructors cannot be synchronized — using the synchronized keyword with a constructor is a syntax error.

Synchronized methods enable a simple strategy for preventing thread interference and memory consistency errors. All read and write should be done synchronized while an exception is final variable which cannot be changed after constructed.

## Intrinsic Locks and Synchronization
Synchronization is built around an internal entity known as the intrinsic lock or monitor lock.

When a thread invokes a synchronized method, it automatically acquires the intrinsic lock for that method's object and releases it when the method returns. The lock release occurs even if the return was caused by an uncaught exception.

### Synchronized Statements
Another way to create synchronized code is with synchronized statements. Unlike synchronized methods, synchronized statements must specify the object that provides the intrinsic lock:
    
    Object object = new Object();
    public void addName(String name) {
        synchronized(object) {
            lastName = name;
            nameCount++;
        }
        nameList.add(name);
    }

A synchronized method is almost identical (see bottom) to synchronizing on this:

    synchroinzed void foo() {
      
    }

    void foo() {
        synchronized(this) {

        }
    }
    
    
    public class MsLunch {
        private long c1 = 0;
        private long c2 = 0;
        private Object lock1 = new Object();
        private Object lock2 = new Object();

        public void inc1() {
            synchronized(lock1) {
                c1++;
            }
        }

        public void inc2() {
            synchronized(lock2) {
                c2++;
            }
        }
    }

### Reentrant Synchronization
Recall that a thread cannot acquire a lock owned by another thread. But a thread can acquire a lock that it already owns. Allowing a thread to acquire the same lock more than once enables reentrant synchronization. This describes a situation where synchronized code, directly or indirectly, invokes a method that also contains synchronized code, and both sets of code use the same lock. Without reentrant synchronization, synchronized code would have to take many additional precautions to avoid having a thread cause itself to block.

### Atomic Access
An atomic action cannot stop in the middle: it either happens completely, or it doesn't happen at all.

Reads and writes are atomic for reference variables and for most primitive variables (all types except long and double).
Reads and writes are atomic for all variables declared volatile (including long and double variables).

   // volatile keyword here makes sure that
   // the changes made in one thread are 
   // immediately reflect in other thread
   static volatile int sharedVar = 6;
   
   
## Liveness
A concurrent application's ability to execute in a timely manner is known as its liveness.

### Deadlock
In order for deadlock to occur, four conditions must be true.

*Mutual exclusion - Each resource is either currently allocated to exactly one process or it is available. (Two processes cannot simultaneously control the same resource or be in their critical section).

*Hold and Wait - processes currently holding resources can request new resources.

*No preemption - Once a process holds a resource, it cannot be taken away by another process or the kernel.

*Circular wait - Each process is waiting to obtain a resource which is held by another process.
and all these condition are satisfied in above diagram.

## Starvation and Livelock

### Starvation
Starvation describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress. This happens when shared resources are made unavailable for long periods by "greedy" threads. For example, suppose an object provides a synchronized method that often takes a long time to return. If one thread invokes this method frequently, other threads that also need frequent synchronized access to the same object will often be blocked.

### Livelock
A thread often acts in response to the action of another thread. If the other thread's action is also a response to the action of another thread, then livelock may result. As with deadlock, livelocked threads are unable to make further progress. However, the threads are not blocked — they are simply too busy responding to each other to resume work. This is comparable to two people attempting to pass each other in a corridor: Alphonse moves to his left to let Gaston pass, while Gaston moves to his right to let Alphonse pass. Seeing that they are still blocking each other, Alphone moves to his right, while Gaston moves to his left. They're still blocking each other, so...

## Guarded Blocks
Threads often have to coordinate their actions. The most common coordination idiom is the guarded block. Such a block begins by polling a condition that must be true before the block can proceed.

simply loop until the condition is satisfied, but that loop is wasteful, since it executes continuously while waiting.

     public void guardedJoy() {
         // Simple loop guard. Wastes
         // processor time. Don't do this!
         while(!joy) {}
         System.out.println("Joy has been achieved!");
     }
     
     public synchronized void guardedJoy() {
         // This guard only loops once for each special event, which may not
         // be the event we're waiting for.
         while(!joy) {
             try {
                 wait();
             } catch (InterruptedException e) {}
         }
         System.out.println("Joy and efficiency have been achieved!");
    }
Let's use guarded blocks to create a Producer-Consumer application. This kind of application shares data between two threads: the producer, that creates the data, and the consumer, that does something with it. The two threads communicate using a shared object. Coordination is essential: the consumer thread must not attempt to retrieve the data before the producer thread has delivered it, and the producer thread must not attempt to deliver new data if the consumer hasn't retrieved the old data.

refer to ProducerConsumerExample.java

## Strategy for defining Immutable Objects
1. Don't provide "setter" methods — methods that modify fields or objects referred to by fields.

2. Make all fields final and private.

3. Don't allow subclasses to override methods. The simplest way to do this is to declare the class as final. A more sophisticated approach is to make the constructor private and construct instances in factory methods.

4. If the instance fields include references to mutable objects, don't allow those objects to be changed:

*Don't provide methods that modify the mutable objects.

*Don't share references to the mutable objects. Never store references to external, mutable objects passed to the constructor; if necessary, create copies, and store references to the copies. Similarly, create copies of your internal mutable objects when necessary to avoid returning the originals in your methods.

## Lock Objects
As with implicit locks, only one thread can own a Lock object at a time. Lock objects also support a wait/notify mechanism, through their associated Condition objects.

The biggest advantage of Lock objects over implicit locks is their ability to back out of an attempt to acquire a lock. The tryLock method backs out if the lock is not available immediately or before a timeout expires (if specified). The lockInterruptibly method backs out if another thread sends an interrupt before the lock is acquired.

## Executor
The Executor interface provides a single method, execute, designed to be a drop-in replacement for a common thread-creation idiom. If r is a Runnable object, and e is an Executor object you can replace

(new Thread(r)).start();
with

e.execute(r);

However, the definition of execute is less specific. The low-level idiom creates a new thread and launches it immediately. Depending on the Executor implementation, execute may do the same thing, but is more likely to use an existing worker thread to run r, or to place r in a queue to wait for a worker thread to become available.

The executor implementations in java.util.concurrent are designed to make full use of the more advanced ExecutorService and ScheduledExecutorService interfaces, although they also work with the base Executor interface.

## Thread Pools
thread pools, which consist of worker threads.

Using worker threads minimizes the overhead due to thread creation. Thread objects use a significant amount of memory, and in a large-scale application, allocating and deallocating many thread objects creates a significant memory management overhead.

One common type of thread pool is the fixed thread pool. This type of pool always has a specified number of threads running; if a thread is somehow terminated while it is still in use, it is automatically replaced with a new thread. Tasks are submitted to the pool via an internal queue, which holds extra tasks whenever there are more active tasks than threads.

An important advantage of the fixed thread pool is that applications using it degrade gracefully. To understand this, consider a web server application where each HTTP request is handled by a separate thread. If the application simply creates a new thread for every new HTTP request, and the system receives more requests than it can handle immediately, the application will suddenly stop responding to all requests when the overhead of all those threads exceed the capacity of the system. With a limit on the number of the threads that can be created, the application will not be servicing HTTP requests as quickly as they come in, but it will be servicing them as quickly as the system can sustain.

A simple way to create an executor that uses a fixed thread pool is to invoke the newFixedThreadPool factory method in java.util.concurrent.Executors

## Fork/Join
It is designed for work that can be broken into smaller pieces recursively. The goal is to use all the available processing power to enhance the performance of your application.

1. Create a task that represents all of the work to be done.

// source image pixels are in src
// destination image pixels are in dst
ForkBlur fb = new ForkBlur(src, 0, src.length, dst);

2. Create the ForkJoinPool that will run the task.

ForkJoinPool pool = new ForkJoinPool();

3. Run the task.

pool.invoke(fb);

### Atomic Variables
The java.util.concurrent.atomic package defines classes that support atomic operations on single variables. All classes have get and set methods that work like reads and writes on volatile variables.

Example: the Counter class we originally used to demonstrate thread interference:

    class Counter {
        private int c = 0;

        public void increment() {
            c++;
        }

        public void decrement() {
            c--;
        }

        public int value() {
            return c;
        }

    }

One way to make Counter safe from thread interference is to make its methods synchronized, as in SynchronizedCounter:

    class SynchronizedCounter {
        private int c = 0;

        public synchronized void increment() {
            c++;
        }

        public synchronized void decrement() {
            c--;
        }

        public synchronized int value() {
            return c;
        }
    }
     
For this simple class, synchronization is an acceptable solution. But for a more complicated class, we might want to avoid the liveness impact of unnecessary synchronization. Replacing the int field with an AtomicInteger allows us to prevent thread interference without resorting to synchronization, as in AtomicCounter:

import java.util.concurrent.atomic.AtomicInteger;

    class AtomicCounter {
        private AtomicInteger c = new AtomicInteger(0);

        public void increment() {
            c.incrementAndGet();
        }

        public void decrement() {
            c.decrementAndGet();
        }

        public int value() {
            return c.get();
        }
    }

## Fair Lock and Unfair Lock
