# Session 3: Introduction to OOP

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 2T + 2L + 2SL (2 Hours Theory + 2 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Master Java package creation and compile/execute applications using the `-classpath` (`-cp`) option.
2. Declare, allocate, initialize, and traverse single-dimensional, multi-dimensional, and jagged/ragged arrays.
3. Formulate software abstractions using Classes and Objects adhering to core Object-Oriented principles.
4. Trace the complete lifecycle of a Java object from heap allocation (`new`) to garbage collection eligibility.
5. Understand JVM garbage collection mechanisms, recognize memory reachability states (Live, Unreachable, Island of Isolation), and explain why `finalize()` is deprecated in modern Java.
6. Use primitive Wrapper classes (`Integer`, `Double`, `Boolean`) for real-world entity modeling.

---

## 2. Packages and the Classpath (`co.vinod`)

### 2.1 The Purpose of Packages
A Java package provides:
1. **Namespace Isolation:** Prevents naming conflicts between classes written by different developers (e.g., `co.vinod.model.Order` vs `org.apache.commons.Order`).
2. **Access Protection:** Package-private (default access) allows friendly classes inside the same package to collaborate without exposing internals to external packages.
3. **Directory Organization:** Maps logically to file-system directories on the host operating system.

### 2.2 Compilation and Execution with Classpath
Consider the following project structure:
```
project-root/
 ├── src/
 │    └── co/
 │         └── vinod/
 │              ├── model/
 │              │    └── Employee.java
 │              └── app/
 │                   └── Application.java
 └── bin/
```

To compile and execute from the terminal:
```bash
# 1. Compile source files from src into bin target
javac -d bin src/co/vinod/model/Employee.java src/co/vinod/app/Application.java

# 2. Execute by specifying classpath (-cp) pointing to the root of compiled packages
java -cp bin co.vinod.app.Application
```

---

## 3. Arrays in Java

### 3.1 Array Fundamentals
In Java, **arrays are dynamically allocated objects on the heap**. Even arrays of primitives (`int[]`) inherit from `java.lang.Object`.

```java
// 1. Declaration and Instantiation
int[] salaries = new int[3]; // Allocates contiguous heap memory; initialized to zeros
salaries[0] = 75000;
salaries[1] = 92000;
salaries[2] = 110000;

// 2. Array Literal (Combined declaration, allocation, initialization)
String[] cities = {"Bangalore", "Mysore", "Mangalore", "Hubli"};

// 3. Length attribute (Read-only field, NOT a method)
int count = cities.length; // 4
```

### 3.2 Multi-Dimensional and Ragged (Jagged) Arrays
Java does not support true matrix memory layouts; multi-dimensional arrays are **arrays of arrays**. This enables ragged arrays where rows have variable columns:

```java
package co.vinod.arrays;

public class RaggedArrayDemo {
    public static void main(String[] args) {
        // A corporate training center with 3 instructors having different batch sizes
        // Instructor 0: Vinod (Bangalore) -> 3 batches
        // Instructor 1: Shyam (Mysore)    -> 2 batches
        // Instructor 2: Murali (Bangalore)-> 4 batches

        int[][] centerBatches = new int[3][];
        centerBatches[0] = new int[]{30, 28, 35};
        centerBatches[1] = new int[]{25, 26};
        centerBatches[2] = new int[]{32, 34, 30, 31};

        for (int i = 0; i < centerBatches.length; i++) {
            System.out.print("Instructor " + i + " batch sizes: ");
            for (int j = 0; j < centerBatches[i].length; j++) {
                System.out.print(centerBatches[i][j] + " ");
            }
            System.out.println();
        }
    }
}
```

---

## 4. Object-Oriented Programming Concepts

1. **Abstraction:** Isolating what an object does from how it does it.
2. **Encapsulation:** Wrapping state and behavior into a single class unit, keeping fields `private` and exposing public accessor methods.
3. **Inheritance:** Creating new classes based on existing ones, inheriting state and behavior to maximize code reuse.
4. **Polymorphism:** Allowing a single interface or parent reference to represent multiple underlying forms.

---

## 5. The Object Lifecycle & Garbage Collection

### 5.1 The Four Stages of an Object's Life
1. **Creation:** 
   - Class loaded if not already in Metaspace.
   - Memory allocated on Heap.
   - Instance variables set to default values (`0`, `null`, `false`).
   - Constructor executed to initialize fields.
   - Reference address returned and stored in a stack variable.
2. **In-Use:** Actively referenced by at least one live thread stack frame.
3. **Eligible for Garbage Collection:** When an object is no longer reachable from any GC Root (Live Thread, Stack Variable, Static Field).
4. **Finalized & Reclaimed:** Memory freed back to the JVM heap manager.

```
+-------------------------------------------------------------------------+
|                  GC Reachability & Island of Isolation                  |
|                                                                         |
|  Stack (GC Roots)              Heap Memory                              |
|  +--------------+              +---------------------+                  |
|  | refA (Live)  | -----------> | Object A            |                  |
|  +--------------+              +----------+----------+                  |
|                                           | references                  |
|                                           v                             |
|                                +---------------------+                  |
|                                | Object B            |                  |
|                                +---------------------+                  |
|                                                                         |
|  (No stack root)               +---------------------+                  |
|                                | Object C (Isolated) | <----+           |
|                                +----------+----------+      |           |
|                                           | references      | references|
|                                           v                 |           |
|                                +---------------------+      |           |
|                                | Object D (Isolated) | -----+           |
|                                +---------------------+                  |
|                                (Both C & D are ELIGIBLE for GC!)         |
+-------------------------------------------------------------------------+
```

### 5.2 How an Object Becomes Eligible for GC
1. **Nullifying the reference:**
   ```java
   Employee emp = new Employee("Vinod", "Bangalore");
   emp = null; // Object previously pointed to is now eligible for GC
   ```
2. **Reassigning the reference variable:**
   ```java
   Employee emp1 = new Employee("Vinod", "Bangalore");
   Employee emp2 = new Employee("Shyam", "Mysore");
   emp1 = emp2; // Original "Vinod" object is now eligible for GC
   ```
3. **Out of Scope (Method Frame Pop):**
   ```java
   void createTemp() {
       Employee temp = new Employee("Murali", "Bangalore");
   } // Once createTemp() finishes and stack frame pops, "Murali" is eligible
   ```
4. **Island of Isolation:** Two or more objects reference each other, but no active stack variable or GC root references either of them.

### 5.3 Deprecation of `finalize()` in Modern Java
- In classic Java, `protected void finalize()` was called before an object was reclaimed.
- **Why `finalize()` is deprecated (Java 9) and terminally deprecated (Java 18+):**
  - Unpredictable execution timing (no guarantee when or if it will run).
  - Severe performance overhead on the GC.
  - Can resurrect objects by assigning `this` back to a static reference.
  - Exception suppression bugs.
- **Modern Solution:** Use `java.lang.AutoCloseable` with `try-with-resources` or `java.lang.ref.Cleaner`.

---

## 6. Comprehensive Lab Assignment: Real-World Entity Modeling & Lifecycle

### 6.1 Requirements
1. Design an entity class `Employee` in package `co.vinod.model` with 6 attributes using primitive wrappers:
   - `id` (`Long`)
   - `name` (`String`)
   - `city` (`String`)
   - `salary` (`Double`)
   - `performanceScore` (`Integer`)
   - `active` (`Boolean`)
2. In driver class `co.vinod.app.LifecycleApp`:
   - Create instances representing "Vinod" (Bangalore), "Shyam" (Mysore), and "Murali" (Bangalore).
   - Demonstrate reference reassignment.
   - Demonstrate reference nullification.
   - Trigger and observe garbage collection behavior via `System.gc()`.

### 6.2 Implementation Source Code

#### File: `src/co/vinod/model/Employee.java`
```java
package co.vinod.model;

/**
 * Enterprise Employee Domain Entity
 * Package: co.vinod.model
 */
public class Employee {

    private Long id;
    private String name;
    private String city;
    private Double salary;
    private Integer performanceScore;
    private Boolean active;

    public Employee(Long id, String name, String city, Double salary, Integer performanceScore, Boolean active) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.salary = salary;
        this.performanceScore = performanceScore;
        this.active = active;
        System.out.printf("[CONSTRUCTOR] Created Employee #%d: %s (%s)%n", id, name, city);
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }

    public Double getSalary() { return salary; }
    public void setSalary(Double salary) { this.salary = salary; }

    public Integer getPerformanceScore() { return performanceScore; }
    public void setPerformanceScore(Integer score) { this.performanceScore = score; }

    public Boolean isActive() { return active; }
    public void setActive(Boolean active) { this.active = active; }

    @Override
    public String toString() {
        return String.format("Employee[id=%d, name='%s', city='%s', salary=₹%,.2f, score=%d, active=%b]",
                id, name, city, salary, performanceScore, active);
    }
}
```

#### File: `src/co/vinod/app/LifecycleApp.java`
```java
package co.vinod.app;

import co.vinod.model.Employee;

/**
 * Demonstrates Object Lifecycle, Reference Reassignment, and GC Eligibility.
 * Package: co.vinod.app
 * Author: Vinod Kumar (Bangalore)
 */
public class LifecycleApp {

    public static void main(String[] args) {
        System.out.println("=== 1. Object Instantiation on Heap ===");
        Employee emp1 = new Employee(101L, "Vinod", "Bangalore", 145000.0, 95, true);
        Employee emp2 = new Employee(102L, "Shyam", "Mysore", 98000.0, 88, true);
        Employee emp3 = new Employee(103L, "Murali", "Bangalore", 112000.0, 91, true);

        System.out.println("
=== Current Object References ===");
        System.out.println("emp1 points to -> " + emp1);
        System.out.println("emp2 points to -> " + emp2);
        System.out.println("emp3 points to -> " + emp3);

        System.out.println("
=== 2. Reference Reassignment ===");
        System.out.println("Executing: emp1 = emp2;");
        // emp1 now points to the object created for Shyam (102).
        // The original object (Vinod - 101) has lost its primary reference!
        emp1 = emp2;
        System.out.println("emp1 now points to -> " + emp1);
        System.out.println("emp2 still points to -> " + emp2);

        System.out.println("
=== 3. Reference Nullification ===");
        System.out.println("Executing: emp3 = null;");
        // The object for Murali (103) is now completely unreachable!
        emp3 = null;
        System.out.println("emp3 is now: " + emp3);

        System.out.println("
=== 4. Requesting JVM Garbage Collection ===");
        System.out.println("Invoking System.gc()...");
        System.gc(); // Hint to JVM to run Garbage Collector

        System.out.println("Main method execution complete. Stack frames will be popped.");
    }
}
```

### 6.3 Expected Execution Output
```text
=== 1. Object Instantiation on Heap ===
[CONSTRUCTOR] Created Employee #101: Vinod (Bangalore)
[CONSTRUCTOR] Created Employee #102: Shyam (Mysore)
[CONSTRUCTOR] Created Employee #103: Murali (Bangalore)

=== Current Object References ===
emp1 points to -> Employee[id=101, name='Vinod', city='Bangalore', salary=₹1,45,000.00, score=95, active=true]
emp2 points to -> Employee[id=102, name='Shyam', city='Mysore', salary=₹98,000.00, score=88, active=true]
emp3 points to -> Employee[id=103, name='Murali', city='Bangalore', salary=₹1,12,000.00, score=91, active=true]

=== 2. Reference Reassignment ===
Executing: emp1 = emp2;
emp1 now points to -> Employee[id=102, name='Shyam', city='Mysore', salary=₹98,000.00, score=88, active=true]
emp2 still points to -> Employee[id=102, name='Shyam', city='Mysore', salary=₹98,000.00, score=88, active=true]

=== 3. Reference Nullification ===
Executing: emp3 = null;
emp3 is now: null

=== 4. Requesting JVM Garbage Collection ===
Invoking System.gc()...
Main method execution complete. Stack frames will be popped.
```

---

## 7. Self-Study & Review Questions
1. How does Java manage array boundaries at runtime, and why does accessing index `-1` or `length` result in `ArrayIndexOutOfBoundsException`?
2. What constitutes an "Island of Isolation" in Java heap memory? Can modern GC algorithms identify and collect it?
3. Contrast primitive types (`int`, `double`) with their wrapper equivalents (`Integer`, `Double`). What are the memory and performance trade-offs?
