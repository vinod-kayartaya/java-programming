# Session 14: Multithreading

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 2T + 2L (2 Hours Theory + 2 Hours Lab)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Explain Java concurrency primitives, differentiating operating system processes from JVM threads.
2. Trace the 6 states of the Java Thread Lifecycle (`NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, `TERMINATED`).
3. Create concurrent tasks by extending `Thread` vs implementing `Runnable` (and understand why `Runnable` is superior).
4. Identify critical sections and diagnose Race Conditions in shared mutable data structures.
5. Apply synchronization mechanisms using intrinsic monitor locks (`synchronized` methods and `synchronized` blocks).
6. Implement Inter-Thread Communication using `wait()`, `notify()`, and `notifyAll()` in the classic Producer-Consumer pattern.

---

## 2. Java Concurrency & Thread Architecture

### 2.1 Process vs Thread
- **Process:** An isolated execution program with its own dedicated virtual address space and resources assigned by the Operating System.
- **Thread:** A lightweight dispatchable path of execution within a process. All threads of a JVM process **share the same Heap Memory**, but maintain their own private **JVM Stacks** and **PC Registers**.

```
+-------------------------------------------------------------------------+
|                              JVM Process                                |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |             Shared Heap Memory & Metaspace (Objects)              |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +--------------------+  +--------------------+  +-------------------+  |
|  | Thread 1 (Vinod)   |  | Thread 2 (Shyam)   |  | Thread 3 (Murali) |  |
|  | - Private Stack    |  | - Private Stack    |  | - Private Stack   |  |
|  | - Private PC Reg   |  | - Private PC Reg   |  | - Private PC Reg  |  |
|  +--------------------+  +--------------------+  +-------------------+  |
+-------------------------------------------------------------------------+
```

### 2.2 The 6 States of a Thread Lifecycle (`Thread.State`)

```
       +-------+
       |  NEW  |
       +---+---+
           | t.start()
       +---v----+
+----->|RUNNABLE|<-----------------------------+
|      +---+----+                              |
|          |                                   |
| scheduler| yield()                           | lock acquired / notified
|          |                                   |
|      +---v-------------------------------+   |
|      | BLOCKED / WAITING / TIMED_WAITING |---+
|      | (Lock contention / wait() / sleep)|
|      +---+-------------------------------+
           | run() completes
       +---v--------+
       | TERMINATED |
       +------------+
```

---

## 3. Creating Threads: `Thread` vs `Runnable`

```java
// Approach 1: Extending java.lang.Thread (Limits single inheritance)
public class WorkerThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in " + getName());
    }
}

// Approach 2: Implementing Runnable (Preferred: preserves class inheritance & supports Lambdas)
Runnable task = () -> System.out.println("Running in " + Thread.currentThread().getName());
Thread t = new Thread(task, "Worker-1");
t.start();
```

---

## 4. Race Conditions, Critical Sections & Synchronization

### 4.1 What is a Race Condition?
When multiple threads read and write shared mutable memory simultaneously, the final outcome depends on thread scheduling order, causing silent data corruption.

```java
// VULNERABLE TO RACE CONDITIONS:
public void deposit(double amount) {
    // 1. Read balance from heap
    // 2. Add amount to local register
    // 3. Write back to balance
    // If Thread A and Thread B interleave, deposits are lost!
    balance += amount; 
}
```

### 4.2 Intrinsic Locks (Monitor Locks)
Every Java object has an **Intrinsic Lock (Monitor)**:
- A thread must acquire the monitor lock before entering a `synchronized` block/method.
- Only one thread can hold the monitor at any given instant. Other competing threads enter the `BLOCKED` state.
- Upon method completion, the lock is automatically released.

```java
// Synchronized Method (Locks on 'this')
public synchronized void deposit(double amount) {
    this.balance += amount;
}

// Synchronized Block (Fine-grained lock on specific target object)
public void withdraw(double amount) {
    synchronized (this) {
        if (balance >= amount) {
            balance -= amount;
        }
    }
}
```

---

## 5. Comprehensive Lab Assignment: Concurrent Banking & Producer-Consumer

### 5.1 Problem Statement
In package `co.vinod.threads`:
1. Implement `BankAccount` representing an account in Bangalore holding ₹1,00,000.
2. Launch 4 concurrent threads simulating transactions from Vinod, Shyam, Murali, and Naveen:
   - Run 1: Unsynchronized operations $	o$ Observe corrupted balance.
   - Run 2: Synchronized operations $	o$ Verify perfect mathematical balance.
3. Implement a thread-safe `ProducerConsumerQueue` using `wait()` and `notifyAll()` coordinating worker threads.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/threads/BankAccount.java`
```java
package co.vinod.threads;

public class BankAccount {
    private final String accountNumber;
    private final String branch;
    private double balance;

    public BankAccount(String accountNumber, String branch, double initialBalance) {
        this.accountNumber = accountNumber;
        this.branch = branch;
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance;
    }

    // Thread-safe Deposit
    public synchronized void deposit(String customer, double amount) {
        System.out.printf("[%s] Depositing ₹%,.2f... (Current: ₹%,.2f)%n", customer, amount, balance);
        balance += amount;
        System.out.printf("  -> [%s] Deposit Complete. New Balance: ₹%,.2f%n", customer, balance);
    }

    // Thread-safe Withdrawal
    public synchronized void withdraw(String customer, double amount) {
        System.out.printf("[%s] Attempting withdrawal of ₹%,.2f... (Current: ₹%,.2f)%n", customer, amount, balance);
        if (balance >= amount) {
            try {
                // Simulate processing latency
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            balance -= amount;
            System.out.printf("  -> [%s] Withdrawal Successful. Remaining: ₹%,.2f%n", customer, balance);
        } else {
            System.err.printf("  -> [%s] WITHDRAWAL DECLINED! Insufficient funds (Available: ₹%,.2f)%n", customer, balance);
        }
    }
}
```

#### File: `src/co/vinod/threads/ProducerConsumerQueue.java`
```java
package co.vinod.threads;

import java.util.LinkedList;
import java.util.Queue;

public class ProducerConsumerQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    public ProducerConsumerQueue(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void produce(T item, String producer) throws InterruptedException {
        while (queue.size() == capacity) {
            System.out.println("  [Queue Full] " + producer + " is WAITING...");
            wait(); // Releases monitor lock and suspends thread
        }
        queue.add(item);
        System.out.printf("[%s] Produced item: %s (Queue Size: %d)%n", producer, item, queue.size());
        notifyAll(); // Wakes up waiting consumer threads
    }

    public synchronized T consume(String consumer) throws InterruptedException {
        while (queue.isEmpty()) {
            System.out.println("  [Queue Empty] " + consumer + " is WAITING...");
            wait(); // Releases monitor lock and suspends thread
        }
        T item = queue.poll();
        System.out.printf("[%s] Consumed item: %s (Queue Size: %d)%n", consumer, item, queue.size());
        notifyAll(); // Wakes up waiting producer threads
        return item;
    }
}
```

#### File: `src/co/vinod/threads/ConcurrencyApp.java`
```java
package co.vinod.threads;

/**
 * Driver demonstrating Thread Synchronization and Producer-Consumer
 * Package: co.vinod.threads
 * Author: Vinod Kumar (Bangalore)
 */
public class ConcurrencyApp {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("==================================================================");
        System.out.println("      CDAC BDA Java 17 Concurrency & Synchronization Lab          ");
        System.out.println("==================================================================
");

        // Part 1: Synchronized Bank Account Simulation
        testBankTransactions();

        // Part 2: Producer-Consumer Simulation with wait() & notifyAll()
        testProducerConsumer();
    }

    private static void testBankTransactions() throws InterruptedException {
        System.out.println("--- Part 1: Synchronized Banking Operations ---");
        BankAccount account = new BankAccount("SB-BLR-9001", "Bangalore Main", 100000.0);

        Thread t1 = new Thread(() -> account.deposit("Vinod", 25000.0), "Thread-Vinod");
        Thread t2 = new Thread(() -> account.withdraw("Shyam", 40000.0), "Thread-Shyam");
        Thread t3 = new Thread(() -> account.deposit("Murali", 15000.0), "Thread-Murali");
        Thread t4 = new Thread(() -> account.withdraw("Naveen", 30000.0), "Thread-Naveen");

        t1.start();
        t2.start();
        t3.start();
        t4.start();

        // Wait for all 4 threads to complete execution
        t1.join();
        t2.join();
        t3.join();
        t4.join();

        System.out.printf("%n>>> FINAL AUDITED BALANCE: ₹%,.2f (Expected: ₹70,000.00)%n%n", account.getBalance());
    }

    private static void testProducerConsumer() throws InterruptedException {
        System.out.println("--- Part 2: Producer-Consumer Queue (Capacity: 2) ---");
        ProducerConsumerQueue<String> queue = new ProducerConsumerQueue<>(2);

        Thread producer = new Thread(() -> {
            String[] batches = {"Batch-Bangalore-1", "Batch-Mysore-2", "Batch-Bangalore-3", "Batch-Mangalore-4"};
            try {
                for (String batch : batches) {
                    queue.produce(batch, "Producer-Vinod");
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 4; i++) {
                    queue.consume("Consumer-Shyam");
                    Thread.sleep(250);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        consumer.start();

        producer.join();
        consumer.join();

        System.out.println("
Producer-Consumer execution successfully completed.");
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
      CDAC BDA Java 17 Concurrency & Synchronization Lab          
==================================================================

--- Part 1: Synchronized Banking Operations ---
[Vinod] Depositing ₹25,000.00... (Current: ₹1,00,000.00)
  -> [Vinod] Deposit Complete. New Balance: ₹1,25,000.00
[Shyam] Attempting withdrawal of ₹40,000.00... (Current: ₹1,25,000.00)
  -> [Shyam] Withdrawal Successful. Remaining: ₹85,000.00
[Murali] Depositing ₹15,000.00... (Current: ₹85,000.00)
  -> [Murali] Deposit Complete. New Balance: ₹1,00,000.00
[Naveen] Attempting withdrawal of ₹30,000.00... (Current: ₹1,00,000.00)
  -> [Naveen] Withdrawal Successful. Remaining: ₹70,000.00

>>> FINAL AUDITED BALANCE: ₹70,000.00 (Expected: ₹70,000.00)

--- Part 2: Producer-Consumer Queue (Capacity: 2) ---
[Producer-Vinod] Produced item: Batch-Bangalore-1 (Queue Size: 1)
[Consumer-Shyam] Consumed item: Batch-Bangalore-1 (Queue Size: 0)
[Producer-Vinod] Produced item: Batch-Mysore-2 (Queue Size: 1)
[Producer-Vinod] Produced item: Batch-Bangalore-3 (Queue Size: 2)
  [Queue Full] Producer-Vinod is WAITING...
[Consumer-Shyam] Consumed item: Batch-Mysore-2 (Queue Size: 1)
[Producer-Vinod] Produced item: Batch-Mangalore-4 (Queue Size: 2)
[Consumer-Shyam] Consumed item: Batch-Bangalore-3 (Queue Size: 1)
[Consumer-Shyam] Consumed item: Batch-Mangalore-4 (Queue Size: 0)

Producer-Consumer execution successfully completed.
```

---

## 6. Self-Study & Review Questions
1. What is Deadlock in multithreading? What are Coffman's four conditions necessary for deadlock to occur?
2. Why must `wait()` and `notify()` only be called from inside a `synchronized` context? What exception is thrown if violated?
3. Contrast `volatile` variables with `synchronized` blocks. Does `volatile` guarantee atomicity for compound operations like `count++`?
