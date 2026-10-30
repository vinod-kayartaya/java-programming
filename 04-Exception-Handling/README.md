# Session 5: Exception Handling

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 2T + 2L + 2SL (2 Hours Theory + 2 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Explain the architectural role of exceptions in separating operational business logic from defensive error-handling logic.
2. Deconstruct the Java `Throwable` hierarchy (`Error` vs `Exception`, Checked vs Unchecked).
3. Apply structured error-handling mechanisms using `try`, `catch`, `finally`, `throw`, and `throws`.
4. Implement modern multi-catch blocks (`|`), nested try structures, and `AutoCloseable` resource management.
5. Engineer production-grade custom domain checked and unchecked exceptions.
6. Build a resilient data repository for employee records with validation and exception recovery.

---

## 2. The Architecture of Exceptions in Java

### 2.1 Why Exceptions?
In legacy languages like C, errors were indicated by sentinel return codes (e.g., returning `-1` or `NULL`). This led to brittle code where every function call was interspersed with error-checking logic. Java separates normal control flow from abnormal operational paths using **Exceptions**.

### 2.2 The `Throwable` Class Hierarchy

```
                                +-------------------+
                                |  java.lang.Object |
                                +---------+---------+
                                          |
                                +---------v---------+
                                |     Throwable     |
                                +---------+---------+
                                          |
                     +--------------------+--------------------+
                     |                                         |
           +---------v---------+                     +---------v---------+
           |       Error       |                     |     Exception     |
           +---------+---------+                     +---------+---------+
                     |                                         |
          (JVM / Hardware Failures:             +--------------+--------------+
           OutOfMemoryError,                    |                             |
           StackOverflowError)        +---------v---------+         +---------v---------+
                                      | Checked Exception |         | RuntimeException  |
                                      +-------------------+         +-------------------+
                                      (Compile-Time Enforced:       (Unchecked Errors:
                                       IOException,                  NullPointerException,
                                       SQLException,                 ArithmeticException,
                                       ClassNotFoundException)       IllegalArgumentException)
```

### 2.3 Checked vs Unchecked Exceptions: The Golden Comparison

| Dimension | Checked Exceptions (`Exception`) | Unchecked Exceptions (`RuntimeException`) |
|---|---|---|
| **Compiler Enforcement** | Mandatory: Compiler enforces catch-or-declare (`try-catch` or `throws`). | Optional: Compiler does not require explicit handling. |
| **Philosophical Purpose** | Anticipated, recoverable environmental faults (e.g., file missing, server unreachable). | Programmer defects, contract violations, or logical bugs. |
| **Recovery Expectation** | The application is expected to gracefully handle and recover. | The code should ideally be corrected to prevent the condition. |
| **Examples** | `IOException`, `SQLException`, `RecordNotFoundException` | `NullPointerException`, `IndexOutOfBoundsException` |

---

## 3. Exception Handling Keywords & Semantics

### 3.1 The Five Keywords
1. `try`: Encloses code that may trigger an exceptional condition.
2. `catch`: Catches and handles specific exceptions thrown by the `try` block.
3. `finally`: Guaranteed execution block for cleanup operations.
4. `throw`: Explicitly triggers an exception instance: `throw new InvalidDataException("Invalid salary");`.
5. `throws`: Method signature clause declaring the checked exceptions a method may propagate to its caller.

### 3.2 The Guaranteed Nature of `finally`
The `finally` block executes under all normal and exceptional conditions, even if:
- An exception was caught.
- An exception was uncaught and propagates up the call stack.
- A `return` statement was executed inside the `try` or `catch` block.

*(Sole exception: `System.exit(0)` terminates the JVM process immediately, bypassing `finally`).*

### 3.3 Multi-Catch Block (Java 7+)
```java
try {
    processData();
} catch (RecordNotFoundException | InvalidDataException ex) {
    // Variable 'ex' is implicitly final
    System.err.println("Validation or retrieval failed: " + ex.getMessage());
}
```

---

## 4. Designing Custom Domain Exceptions

In enterprise applications, standard JDK exceptions (`NullPointerException`, `RuntimeException`) are too generic. We create custom exceptions to communicate domain-specific failures:

```java
package co.vinod.exceptions;

// Custom Checked Exception (Caller MUST handle or declare)
public class RecordNotFoundException extends Exception {
    private final String entityId;

    public RecordNotFoundException(String entityId, String message) {
        super(message);
        this.entityId = entityId;
    }

    public String getEntityId() {
        return entityId;
    }
}
```

```java
package co.vinod.exceptions;

// Custom Unchecked Exception (Represents illegal arguments or state violations)
public class InvalidDataException extends RuntimeException {
    public InvalidDataException(String message) {
        super(message);
    }
}
```

---

## 5. Comprehensive Lab Assignment: Resilient Employee Repository

### 5.1 Problem Statement
In package `co.vinod.repository`, implement an `EmployeeRepository` managing employee records (Vinod in Bangalore, Shyam in Mysore, Murali in Bangalore).
1. `Employee findById(int id)` throws custom checked `RecordNotFoundException` if the ID is missing.
2. `void addEmployee(Employee emp)` throws custom unchecked `InvalidDataException` if:
   - The employee object is `null`.
   - Name or City is blank.
   - Base salary is negative or below ₹30,000.
3. In `co.vinod.app.ExceptionApp`:
   - Test successful retrievals.
   - Test non-existent records and recover gracefully in `catch` blocks.
   - Test validation errors and log clear diagnostics.
   - Demonstrate `finally` execution in all scenarios.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/exceptions/RecordNotFoundException.java`
```java
package co.vinod.exceptions;

public class RecordNotFoundException extends Exception {
    private final int searchId;

    public RecordNotFoundException(int searchId, String message) {
        super(message);
        this.searchId = searchId;
    }

    public int getSearchId() {
        return searchId;
    }
}
```

#### File: `src/co/vinod/exceptions/InvalidDataException.java`
```java
package co.vinod.exceptions;

public class InvalidDataException extends RuntimeException {
    public InvalidDataException(String message) {
        super(message);
    }
}
```

#### File: `src/co/vinod/repository/EmployeeRepository.java`
```java
package co.vinod.repository;

import co.vinod.exceptions.InvalidDataException;
import co.vinod.exceptions.RecordNotFoundException;
import co.vinod.payroll.Employee;
import co.vinod.payroll.Developer;
import co.vinod.payroll.Manager;

/**
 * In-Memory Employee Repository with Resilient Error Handling
 * Package: co.vinod.repository
 * Author: Vinod Kumar (Bangalore)
 */
public class EmployeeRepository {

    private Employee[] store = new Employee[10];
    private int size = 0;

    public EmployeeRepository() {
        // Pre-populate seed data
        store[size++] = new Manager(101, "Vinod", "Bangalore", 165000.0, 45000.0, 10);
        store[size++] = new Developer(102, "Shyam", "Mysore", 95000.0, 15000.0, "Java 17");
        store[size++] = new Developer(103, "Murali", "Bangalore", 115000.0, 20000.0, "Spring Boot");
    }

    public Employee findById(int id) throws RecordNotFoundException {
        System.out.println("[AUDIT] Initiating search for Employee ID: " + id);
        for (int i = 0; i < size; i++) {
            if (store[i].getId() == id) {
                return store[i];
            }
        }
        throw new RecordNotFoundException(id, "Employee with ID #" + id + " does not exist in Bangalore/Mysore registers.");
    }

    public void addEmployee(Employee emp) {
        if (emp == null) {
            throw new InvalidDataException("Employee payload cannot be null.");
        }
        if (emp.getName() == null || emp.getName().trim().isEmpty()) {
            throw new InvalidDataException("Employee name is a mandatory field.");
        }
        if (emp.getBaseSalary() < 30000.0) {
            throw new InvalidDataException(String.format("Salary ₹%,.2f is below minimum wage threshold (₹30,000).", emp.getBaseSalary()));
        }
        if (size >= store.length) {
            throw new IllegalStateException("Repository capacity exceeded. Expansion required.");
        }

        store[size++] = emp;
        System.out.printf("[SUCCESS] Added %s (%s) to repository.%n", emp.getName(), emp.getCity());
    }
}
```

#### File: `src/co/vinod/app/ExceptionApp.java`
```java
package co.vinod.app;

import co.vinod.exceptions.InvalidDataException;
import co.vinod.exceptions.RecordNotFoundException;
import co.vinod.payroll.Developer;
import co.vinod.payroll.Employee;
import co.vinod.repository.EmployeeRepository;

/**
 * Driver testing Checked and Unchecked Exceptions with try-catch-finally
 * Package: co.vinod.app
 * Author: Vinod Kumar (Bangalore)
 */
public class ExceptionApp {

    public static void main(String[] args) {
        EmployeeRepository repo = new EmployeeRepository();

        System.out.println("==================================================================");
        System.out.println("      CDAC BDA Resilient Repository Exception Handling Demo       ");
        System.out.println("==================================================================
");

        // Scenario 1: Search for existing employee (Vinod - 101)
        testSearch(repo, 101);

        // Scenario 2: Search for non-existent employee (Naveen - 999) -> Checked Exception
        testSearch(repo, 999);

        // Scenario 3: Add employee with invalid data -> Unchecked Exception
        testAddition(repo, new Developer(104, "Harish", "Mangalore", 18000.0, 5000.0, "React"));

        // Scenario 4: Add employee with null name -> Unchecked Exception
        testAddition(repo, new Developer(105, "", "Hubli", 45000.0, 5000.0, "DevOps"));
    }

    private static void testSearch(EmployeeRepository repo, int id) {
        System.out.println("--- Test: Fetching Record #" + id + " ---");
        try {
            Employee emp = repo.findById(id);
            System.out.println(">>> Record Found: " + emp.getName() + " from " + emp.getCity() + ", Salary: ₹" + emp.calculateTotalEarnings());
        } catch (RecordNotFoundException ex) {
            System.err.println(">>> [HANDLED CHECKED EXCEPTION] Error: " + ex.getMessage() + " (Searched ID: " + ex.getSearchId() + ")");
        } finally {
            System.out.println("[FINALLY BLOCK] Search audit logging complete for ID: " + id + "
");
        }
    }

    private static void testAddition(EmployeeRepository repo, Employee emp) {
        System.out.println("--- Test: Adding Employee ---");
        try {
            repo.addEmployee(emp);
        } catch (InvalidDataException ex) {
            System.err.println(">>> [HANDLED UNCHECKED EXCEPTION] Validation Failed: " + ex.getMessage());
        } finally {
            System.out.println("[FINALLY BLOCK] Registration transaction closed.
");
        }
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
      CDAC BDA Resilient Repository Exception Handling Demo       
==================================================================

--- Test: Fetching Record #101 ---
[AUDIT] Initiating search for Employee ID: 101
>>> Record Found: Vinod from Bangalore, Salary: ₹210000.0
[FINALLY BLOCK] Search audit logging complete for ID: 101

--- Test: Fetching Record #999 ---
[AUDIT] Initiating search for Employee ID: 999
>>> [HANDLED CHECKED EXCEPTION] Error: Employee with ID #999 does not exist in Bangalore/Mysore registers. (Searched ID: 999)
[FINALLY BLOCK] Search audit logging complete for ID: 999

--- Test: Adding Employee ---
>>> [HANDLED UNCHECKED EXCEPTION] Validation Failed: Salary ₹18,000.00 is below minimum wage threshold (₹30,000).
[FINALLY BLOCK] Registration transaction closed.

--- Test: Adding Employee ---
>>> [HANDLED UNCHECKED EXCEPTION] Validation Failed: Employee name is a mandatory field.
[FINALLY BLOCK] Registration transaction closed.
```

---

## 6. Self-Study & Review Questions
1. Why is catching `Throwable` or `Error` in application code considered a catastrophic anti-pattern?
2. What is "Exception Chaining", and how does `initCause()` or `super(message, cause)` preserve the root cause of an error across architectural boundaries?
3. What is the impact of placing a `return` statement inside a `finally` block? Why does the compiler warn against it?
