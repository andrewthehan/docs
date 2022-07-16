# Concurrency

*Summarized version of https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html*

## What?

Being able to do multiple things at once.

Two basic units:

- Processes
  - self-contained execution environment
  - has complete, private set of basic runtime resources

- Threads
  - exists as part of a process
  - shares process's resources

## Threads

```java
class Thread implements Runnable
```

```java
@FunctionalInterface
interface Runnable {
  void run()
}
```

Performing some work on a thread:

```java
new Thread() {
  @Override
  void run() {
    expensiveWork();
  }
};

new Thread(() -> expensiveWork()).start();
```

`Thread.sleep` can be used to make the processor available to other threads (though should not be used for precise timings).

Use `Thread#join` to wait for the completion of another thread.

## Synchronization

Used to avoid memory consistency issues.

```java
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
```

Has memory consistency issues when accessed from multiple threads. Key to memory consistency is establishing a *happens-before* relationship.

Two ways to synchronize:

### Synchronized methods

```java
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
```

Only one thread can access any synchronized method of the same object. Other access block until the first thread is done with the object.

### Synchronized statements

```java
public class SynchronizedCounter {
  private Object lock = new Object();
  private int c = 0;

  public void increment() {
    synchronized (lock) {
      c++;
    }
  }

  public void decrement() {
    synchronized (lock) {
      c--;
    }
  }

  public int value() {
    return c;
  }
}
```

One one thread can access any synchronized block that acquires the same lock.

Synchronized statements are more complex than synchronized methods but provides more granularity and control.

---

**Atomicity** is the property of being all or nothing. All reads and writes for primitives (except `long` and `double`) and `volatile` variables are atomic.

**Reentrant synchronization** is the capability for a thread that already acquired a lock to safely acquire the same lock.

## Liveness

**Deadlock** is when 2+ threads are blocked forever waiting for each other.

**Livelock** is when 2+ threads are busy responding to each other forever.

**Starvation** is when a thread is unable to gain access to resources and cannot make progress.

## Guarded blocks

Use `wait` and `notify` to coordinate progress between multiple threads.

```java
// Thread A
while (!condition) {
  try {
    wait();
  } catch (InterruptedException e) {}
}

work();
```

```java
// Thread B
condition = true;
notify(); // or notifyAll() depending on how many waiting threads you want to wake up
```

## Immutability

The state of not changing. Makes objects simpler to reason about and use concurrently.

## High level concurrency objects

### Lock

Similar to the locking mechanism provided by `synchronized` but supports the ability to back out of an attempt to acquire a lock.

```java
Lock l = ...;
l.lock();
try {
  // access the resource protected by this lock
} finally {
  l.unlock();
}
```

`tryLock` backs out if the lock is not available or before a timeout expires.

### Executor

Allows decoupling of thread management and application logic.

Three interfaces:

- **`Executor`**, a simple interface that supports launching new tasks.
- **`ExecutorService`**, a subinterface of Executor, which adds features that help manage the life cycle, both of the individual tasks and of the executor itself.
- **`ScheduledExecutorService`**, a subinterface of ExecutorService, supports future and/or periodic execution of tasks.

Instead of
```java
new Thread(runnable).start();
```

use
```java
executor.execute(runnable);
```

Most `Executor`s use thread pools. Thread pools can be created using factory methods from `java.util.concurrent.Executors` (`newFixedThreadPool`, `newCachedThreadPool`, `newSingleThreadExecutor`, etc.)

### Concurrent collections

### Atomic variables

