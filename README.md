# Java-Multithreading-and-Concurrency

## Single Threaded App Example
### Serial Threaded vs Multi-threaded -
Multi-threaded applications normally do parallel processing of tasks where as single threaded applications do one task at a time i.e. If there are two tasks such as T1, T2 they are executed in serial order i.e T1 after T2 or T2 after T1, where as multi threading enables us to execute them simultaneously i.e. T1 along with T2. We can choose single threaded applications when parallel processing is not required example use cases such as simple text editor. 

Below is a simple example for single threaded application, in this the Task need to print 1500 T's and Main is another task which prints 1500 M's. If we execute this in serial order then it is referred as serial execution.



Serial Execution Example -

```java
/*
Some task; here it will print 1500 T's.
*/
class Task {
 
    public void doTask() {
	for(int i=1; i <= 1500; i++) {
	    System.out.print("T");
	}
    }
}
 
/* Main */
public class Main {
 
    public static void main(String[] args) {
		
	// Print M's
	for(int i=1; i <= 1500; i++) {
	    System.out.print("M");
	}
		
	// Call the task to print T's
	Task t1 = new Task();
	t1.doTask();
    }
 
}
```
When you run the above example you will see 1500 M's first and then followed by 1500 T's.

<hr>


## Parallelism
### True Parallelism vs Logical Parallelism
True parallelism is achieved by assigning tasks to individual CPUs or Processors. This is possible through multicore processors or executing the tasks using multiple CPUs.

If there is one CPU and you want to perform multiple tasks in parallel, then CPU will be shared across those tasks for some stipulated time interval, which stands for interleaved execution, and this way of executing the tasks is known as logical parallelism or psuedo parallelism.

In case of logical parallelism, lets assume there are two tasks T1 and T2, when executed in parallel and using one CPU, CPU is switched between T1 and T2 i.e. CPU executed T1 for some stipulated time interval and then switched to T2. Once T2's time slice is completed then it is switched back to T1 and starts from where it stops.



Example - (Explained in detail in future lectures.)
In the below example Task is a Thread(explained later), and run is the entry point of the thread execution where it starts printing 1500 T's.

`main()` runs in Thread i.e. the Main thread which is started by the JVM.

Note In the main method we are not calling doTask directly, instead we are using the start() method of the Thread class, which runs the Task using a separate Thread.


```java
class Task extends Thread {
	
    // Thread execution begins here.
    public void run() {
        // performs the task i.e. prints 1500 T's
	doTask(); 
    }
	
    public void doTask() {
	for(int i=1; i <= 1500; i++) {
	    System.out.print("T");
	}
    }
}
 
public class Main {
 
    // Runs with in the Main thread started by JVM.
    public static void main(String[] args) {
		
	Task t1 = new Task();
        // Starts a separate Thread using the 
        // the start method of the Thread class.
	t1.start(); 
 
	// runs in the Main thread and prints 1500 M's
	for(int i=1; i <= 1500; i++) {
	    System.out.print("M");
	}	
    }
}
```

Here `main()` and Task are run using two separate threads, which means they are executed in parallel (logical parallelism in case of single CPU) and hence you will see output like MMMTTTMMMTTT...

<hr>

## Designing Threads
### Thread -
A thread is a light weight process, it is given its own context and stack etc. for preserving the state. Thread state enables the CPU to switch back and continue from where it stopped.

### Creating Threads in Java -
When you launch Java application, JVM internally creates few threads, e.g. Main thread for getting started with main() and GarbageCollector for performing garbage collection and few other threads which are need for execution of a Java application.

You can create a thread and execute the tasks with in the application, this enables you to perform parallel activities with in the application.

There are two approaches,

1. Extending the Thread class and performing the task. This is not a preferred approach because you are not extending the Thread functionality, instead you are using the Thread to execute a task, hence you should prefer the second approach.
2. Implementing the Runnable interface and then submitting this task for execution. Similarly there is a Callable interface(explained later) as well.



### `run()` method -
```
Once you choose your approach, you can consider the run() method as the entry point for thread execution.
To simplify just think like main() for a program, run() for a thread.
```
### `start()` method -
```
Execution of the thread should be initiated using the start() method of the Thread class.
This submits the thread for execution. This takes the associated thread to ready state, 
this doesn't mean it is started immediately .i.e. in simple terms,when you call the start() method,
it marks the thread as ready for execution and waits for the CPU turn.
```

```java
// 1. Extending the Thread class.
class MyThread extends Thread {
	
	// Thread execution begins here.
	public void run() {
		// DO THAT TASK HERE.
		for(int i=0; i <= 1000; i++) {
			System.out.print("T");
		}
	}
	
}
 
// 2. Implementing the Runnable interface,
// This marks this class as Runnable and 
// assures that this class contains the run()
// method. Because any class implementing the
// interface should define the abstract method
// of the interface, otherwise it becomes abstract.
class MyTask implements Runnable {
 
    // Thread execution begins here.
    @Override
    public void run() {
	// DO THAT TASK HERE.
	for(int i=0; i <= 1000; i++) {
	    System.out.print("-");
	}
    }
}
 
public class Main {
 
    // Will run with in the main thread.
    public static void main(String[] args) {
		
	// Because MyThread extends the Thread
	// class, you can call the start() method
	// directly, as it is also a member of this
	// class, courtesy inheritance relation.
	MyThread thr = new MyThread();
	thr.start();
		
	// MyTask is a Runnable task and not a 
	// Thread and hence we need to create a 
	// Thread object and assign it the task 
	// Note here we are calling the start 
	// method over Thread object and not on 
	// task object.
	MyTask task = new MyTask();
	Thread thr2 = new Thread(task);
	thr2.start();
		
	for(int i=0; i <= 1000; i++) {
	    System.out.print("M");
	}	
    }
}
```

There are three threads in the above program (system threads ignored). One the Main thread that prints "M" and thr that prints "T" and thr2 that prints "-". Because they are executed in parallel, you will see the output like MMMTTT---MMMTTT--- ...

<hr>

## Transform code to achieve parallelism
### Transforming serial code to parallel code using Threads.


```java
package com.example.io.utils;
 
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
 
/* A simple utility to copy the content 
 * of the given source file to the given 
 * destination file. */
public class IOUtils {
	
    /*
     * Copies the given source stream 
     * i.e. src to the given
     * destination stream i.e. dest.
     */
    public static void copy(InputStream src, OutputStream dest) throws IOException {
	int value;
	while ((value = src.read()) != -1) {
	    dest.write(value);
	}
    }
	
    /*
     * Copies the given srcFile to the destFile.
     */
    public static void copyFile(String srcFile, String destFile) throws IOException {
	FileInputStream fin = new FileInputStream(srcFile);
	FileOutputStream fout = new FileOutputStream(destFile);
		
	copy(fin, fout);
		
	fin.close();
	fout.close();
    }
}
```

### Serial Mode -
In the below example we are making a direct call to copyFile and it is executed in serial order i.e. first Copy a.txt to c.txt is executed and then once it is done, the next copy i.e. b.txt to d.txt will be initiated.

<strong>Note</strong> - <em>For this program to work you need to create two files a.txt and b.txt in the src/ folder and place some content in it.</em>


```java
import java.io.IOException;
import com.example.io.utils.IOUtils;
 
public class Main {
 
    public static void main(String[] args) throws IOException {
		
	String sourceFile1 = "a.txt";
	String sourceFile2 = "b.txt";
		
	String destFile1 = "c.txt";
	String destFile2 = "d.txt";
		
	// 1. Copy a.txt to c.txt
	IOUtils.copyFile(sourceFile1, destFile1);
 
	// 2. Copy b.txt to d.txt
	IOUtils.copyFile(sourceFile2, destFile2);
    }
}
```

### Parallel Mode -
The two copy operations above are initiated through two different threads, which enables us to perform the operation in parallel. For this we defined a CopyTask which is a Runnable task, you should pass the source and the destination to the constructor which are then used to perform the copy operation once the task execution begins.


```java
import java.io.IOException;
import com.example.io.utils.IOUtils;
 
/*
 * A Runnable task to copy the given source file 
 * to the given destination file.
 */
class CopyTask implements Runnable {
	
    String sourceFile;
    String destFile;
	
    public CopyTask(String sourceFile, String destFile) {
	this.sourceFile = sourceFile;
	this.destFile = destFile;
    }
	
    /*
     * Initiate the copy once thread execution begins.
     */
    public void run() {
	try {
	    IOUtils.copyFile(sourceFile, destFile);
	    System.out.println("Copied from - " + sourceFile + " to " + destFile);
	}catch(IOException e) {
	    e.printStackTrace();
	}	
    }
}
 
public class Main {
 
    public static void main(String[] args) throws IOException {
		
	String sourceFile1 = "a.txt";
	String sourceFile2 = "b.txt";
		
	String destFile1 = "c.txt";
	String destFile2 = "d.txt";
		
	// A new thread is created to initiate copy 
	// from a.txt to c.txt
	// Thread-1
 
        new Thread(new CopyTask(sourceFile1, destFile1)).start();		
	
        // A new thread to initiate copy from 
	// b.txt to d.txt
	// Thread-2
 
        new Thread(new CopyTask(sourceFile2, destFile2)).start();
    }
}
```
<strong>Important Note</strong> - <em>Although the main thread is completed after starting the two other threads, application won't be terminated until both the threads are done with the copy.</em>

<hr>

## Executor Service
### `ExecutorService` -
In the previous example we executed the copyTask using separate threads, but there is a critical point to consider here, Thread creation is a costly activity as it includes creating a separate execution context, stack etc.. Hence we should refrain from creating too many threads. And also creating a thread for each task is not a good idea, instead we can create a pool of threads and effectively utilise them in executing all the task. This could be achieved using ExecutorService in Java. Use the execute method of the ExecutorService to submit a Runnable task, if a thread is available in the pool then it assigns this task to the thread otherwise the task is added to the blocking queue and is kept till a thread is available. 

### Creating a ThreadPool -

Below statement creates a thread pool of size 5.
```java
ExecutorService executor = Executors.newFixedThreadPool(5);
```


### Submitting a Runnable task -

We can submit a task for execution using the execute method.

`executor.execute( runnableTaskInstance );`

e.g.
```java
executor.execute(new CopyTask(sourceFile1, destFile1));
```


Modified version of previous example to use ExecutorService -
```java
public class Main {
 
    public static void main(String[] args) throws IOException {
		
	String sourceFile1 = "a.txt";
	String sourceFile2 = "b.txt";
		
	String destFile1 = "c.txt";
	String destFile2 = "d.txt";
		
	// Creates a fixed thread pool of size 5.
		
	ExecutorService executor = Executors.newFixedThreadPool(5);
		
	// Assume you are submitting 100 copy tasks,
	// then executor service uses a fixed thread 
	// pool of size 5 to execute them.
 
        executor.execute(new CopyTask(sourceFile1, destFile1));
	executor.execute(new CopyTask(sourceFile2, destFile2));	
    }
}
```

<hr>

## Stopping Thread in the middle
### Stopping threads in the middle -
`stop()` method of the Thread class could be used to stop the thread in the middle. But this is the dangerous thing to do as it leaves the system in inconsistent state, because we are not giving the opportunity to the thread to rollback or reverse the actions that it has taken. And hence the stop method is deprecated and should not be used.

Correct approach would be to call the `interrupt()` method on the thread and then it is up to the thread to consider whether to stop or not.

A thread can then check if it was interrupted or not using `interrupted()` method of the Thread class. We can design the thread in a way that it reverses the actions/operations performed and then stop when interrupted.

<strong>Note</strong> - <em>If you are not extending Thread class and instead implementing `Runnable` interface, then you can use `Thread.isInterrupted()` to check if the current thread is interrupted.</em>

`interrupted()` and `Thread.isInterrupted()` both methods return true if the thread is interrupted when it is alive.



Example -

```java
class MyThread extends Thread {
	
    public void run() {
		
	// Intentionally kept in infinite loop
	for( ;; ) {
			
	    // Returns true if the thread is interrupted.
	    if (interrupted()) {
				
	        // You are supposed to rollback or reverse the operation
		// in progress and stop.
 
		System.out.println("Thread is interrupted hence stopping..");
				
		// Terminates the loop.
		break;
	    }
			
	    System.out.print("T");
	}	
    }
}
 
public class Main {
 
    public static void main(String[] args) {
 
	MyThread thr = new MyThread();
	thr.start();
		
	// Just for demo, wait for 2 seconds
	// before interrupting thr.
	try {
	    Thread.sleep(2000);
	} catch (InterruptedException e) {
	    e.printStackTrace();
	}
		
	// Interrupt the thread.
	thr.interrupt();
    }
}
```

### `sleep()` method -

`sleep()` method of the thread class is used to block the thread for the given time interval in milliseconds. This method throws `InterruptedException` if the thread is interrupted while it is in sleep.

<hr>

## Thread States
### Thread States -
A thread can be in one of the following states:

### NEW

A thread that is created but not yet started is in this state.

### RUNNABLE

A thread executing in the Java virtual machine is in this state, internally we can think of it as a combination of two sub states Ready and Running, i.e. when you start the thread it comes to Ready state and wait for the CPU, and if CPU is allocated then it goes into Running state. Once allocated CPU time is completed, in other words when the Thread schedular suspends the thread then it goes back to the Ready state and waits for its turn again.

The `yield()` method instructs the thread schedular to pass the CPU to other waiting thread if any.

### BLOCKED

A thread is blocked if it is waiting for a monitor lock is in this state. Refer synchronized methods and blocks.

### WAITING

A thread that is waiting indefinitely for another thread to perform a particular action is in this state. Refer `wait()`, `join()`

### TIMED_WAITING

A thread that is waiting for another thread to perform an action for up to a specified waiting time is in this state. Refer `wait(millis)`, `join(millis)`

### TERMINATED

A thread that has exited i.e. it has either completed its task or terminated in the middle of the execution.

![](https://github.com/dipjul/Java-Multithreading-and-Concurrency/blob/545ff854350c325820b1ce724709548ffbd461af/2018-10-23_05-09-17-d049f04475d6816447ea19b7777b57c5.png)

### `yield()` method -
yield() method is important in few scenarios, suppose a thread is given 5 min of CPU time, now after a minute thread knows that it doesn't need the CPU anymore with in that time period, in such scenarios do you think that blocking the CPU for the next four minutes is a good idea ? No, it is better to pass on the control to the threads if any waiting for CPU and that is when we can use the yield() method. Usage Thread.yield(), it is a static method of the Thread class and it affects the current thread from which the method is invoked.

<hr>

## Thread Priorities
Let us look at how we can change the thread priorities.

> Thread priorities range between 1 and 10.

### MIN_PRIORITY - 1 being the minimum priority

### NORM_PRIORITY - 5 is the normally priority, this is the default priority value.

### MAX_PRIORITY - 10 being the max priority.

### `setPriority(int newPriority)`  -
A method in the Thread class, this is used to set the new priority for the thread. If the newPriority value is more than the maximum priority allowed for the group then maximum priority is considered, i.e. if you try to set 15 then it takes only 10. And for a given ThreadGroup if the maximum allowed priority is 7 then any thread with in that group can have a maximum of 7.

Example -
Just think about a software installer app, the thread that copies the files should be given more priority than the thread which display the progress etc, that speeds up the installation process. Below example demonstrates setting higher priority for copyThread.
```java
class CopyTask implements Runnable {
    @Override
    public void run() {
	while(true) {
	    System.out.print("C");
	}
    }
}
 
class ProgressTask implements Runnable {
    @Override
    public void run() {
	while(true) {
	    System.out.print("-");
	}
    }
}
 
public class Main {
 
    public static void main(String[] args) {
	CopyTask copyTask = new CopyTask();
	Thread copyThread = new Thread(copyTask);
	copyThread.setPriority(Thread.NORM_PRIORITY + 3);
	copyThread.start();
		
	ProgressTask progressTask = new ProgressTask();
	Thread progressThread = new Thread(progressTask);
	progressThread.start();
    }
}
```

<hr>
