# Sessions 1 & 2: Getting started with Java

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 4T + 2L + 2SL (4 Hours Theory + 2 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Explain the evolution, philosophy, and architectural design of the Java platform (WORA).
2. Deconstruct the Java Virtual Machine (JVM) internal runtime architecture, class loading mechanism, and memory areas.
3. Use JDK 17 CLI development tools (`javac`, `java`, `javap`, `jar`, `jshell`) with command-line confidence.
4. Structure Java applications properly using package hierarchies under `co.vinod`.
5. Differentiate strictly between primitive types (stored in stack memory) and reference variables (pointing to heap objects).
6. Implement robust decision and repetition control structures, including traditional statements and modern Java 17 Switch Expressions (`->` syntax and `yield`).
7. Construct a fully functioning, interactive console application with robust exception and boundary condition handling.

---

## 2. Overview and Evolution of Java

### 2.1 The Genesis of Java
Java was developed in 1991 by the "Green Team" at Sun Microsystems led by **James Gosling**, Mike Sheridan, and Patrick Naughton. Initially named **Oak** (after an oak tree outside Gosling's office), the project was designed for consumer electronics (interactive television set-top boxes). When the cable television industry proved unready, the team redirected the language toward the emerging World Wide Web, renaming it **Java** in 1995.

### 2.2 The Paradigm Shift: Write Once, Run Anywhere (WORA)
In traditional languages like C and C++, source code is compiled directly into host-specific native machine instructions (ELF binaries on Linux, Mach-O on macOS, PE executables on Windows). This architecture creates two severe bottlenecks:
1. **Platform Lock-in:** Code compiled on an x86 Intel architecture running Linux cannot run on an ARM processor or a Windows machine without recompilation.
2. **Undefined Data Sizes:** An `int` in C/C++ can be 16, 32, or 64 bits depending on the compiler and operating system architecture.

Java eliminated these bottlenecks through an architectural intermediate abstraction: **Bytecode**.

```
+-------------------------------------------------------------------------+
|                           C/C++ Model (Native)                          |
|                                                                         |
|  [Source Code .c] ---> [Compiler] ---> [Native Machine Code (x86_64)]   |
|                               (Direct OS/Hardware Execution)            |
+-------------------------------------------------------------------------+

+-------------------------------------------------------------------------+
|                            Java Model (WORA)                            |
|                                                                         |
|  [Source Code .java]                                                    |
|           |                                                             |
|           v (javac Compiler)                                            |
|  [Bytecode .class] (Architecturally Neutral Binary)                     |
|           |                                                             |
|           +-------------------+--------------------+                    |
|           |                   |                    |                    |
|           v                   v                    v                    |
|     +-----------+       +-----------+        +-----------+              |
|     | JVM Linux |       | JVM macOS |        | JVM Win   |              |
|     +-----+-----+       +-----+-----+        +-----+-----+              |
|           |                   |                    |                    |
|           v                   v                    v                    |
|     [Intel Linux]       [Apple Silicon]      [AMD Windows]              |
+-------------------------------------------------------------------------+
```

Java standardizes data types across all platforms: on every JVM in the world, an `int` is strictly a 32-bit signed two's complement integer.

---

## 3. The Java Buzzwords (Core Features)

The official Java White Paper defines ten core characteristics:
1. **Simple:** Stripped of complex, memory-unsafe constructs from C++ (no explicit pointers, no pointer arithmetic, no multiple inheritance of state, no operator overloading).
2. **Object-Oriented:** Clean focus on objects, state, behavior, encapsulation, and inheritance. In Java, with the exception of 8 primitive data types, everything is modeled as a class.
3. **Platform-Independent & Portable:** The bytecode runs on any system with a compliant JVM. No implementation-dependent aspects exist in the core language.
4. **Robust:** Strong compile-time and runtime type checking, explicit exception handling, and automatic garbage collection eliminate dangling pointers and memory leaks.
5. **Secure:** Sandboxed execution inside the JVM, bytecode verification before execution, absence of direct memory addressing, and classloader security boundaries.
6. **Architecture-Neutral:** The compiler generates bytecode instructions that can be executed on any processor running a JVM.
7. **High Performance:** While interpreted bytecode was once slow, modern Just-In-Time (JIT) compilers dynamically translate frequently executed bytecode ("hotspots") into optimized native machine code.
8. **Multithreaded:** Java integrates multithreading primitives (`Thread`, `Runnable`, `synchronized`) directly into the language and runtime library.
9. **Distributed:** Native support for TCP/IP networking, HTTP protocols, and remote method invocation allows building network-aware applications seamlessly.
10. **Dynamic:** Classes are linked and loaded lazily at runtime, allowing classes and libraries to evolve without breaking existing client code.

---

## 4. JVM Architecture & Runtime Data Areas

The Java Virtual Machine (JVM) is the runtime execution engine. Understanding its internal architecture is crucial for writing performant code and diagnosing memory issues.

```
+-----------------------------------------------------------------------+
|                        Class Loader Subsystem                         |
|   1. Loading: Bootstrap ClassLoader -> Platform -> Application        |
|   2. Linking: Verification -> Preparation -> Resolution               |
|   3. Initialization: Static initializers & static fields initialized   |
+-----------------------------------+-----------------------------------+
                                    |
+-----------------------------------v-----------------------------------+
|                          Runtime Data Areas                           |
|                                                                       |
|  +-------------------------------+  +------------------------------+  |
|  |          Method Area          |  |         Heap Memory          |  |
|  |  (Class Metadata, Bytecode,   |  |  (All Objects, Class         |  |
|  |   Static Fields, Metaspace)   |  |   Instances, and Arrays)     |  |
|  +-------------------------------+  +------------------------------+  |
|                                                                       |
|  +--------------------+  +--------------------+  +-----------------+  |
|  |   JVM Stack Area   |  |    PC Registers    |  |  Native Method  |  |
|  | (Stack Frames per  |  | (Program Counter   |  |   Stack (JNI)   |  |
|  |  thread: Locals)   |  |  per thread)       |  |  (C/C++ Calls)  |  |
|  +--------------------+  +--------------------+  +-----------------+  |
+-----------------------------------+-----------------------------------+
                                    |
+-----------------------------------v-----------------------------------+
|                           Execution Engine                            |
|   +-------------------+  +--------------------+  +----------------+   |
|   |    Interpreter    |  |    JIT Compiler    |  | Garbage        |   |
|   | (Bytecode line by |  | (HotSpot C1/C2 to  |  | Collector (GC) |   |
|   |  line execution)  |  |  Native Machine)   |  | (ZGC/G1)       |   |
|   +-------------------+  +--------------------+  +----------------+   |
+-----------------------------------+-----------------------------------+
                                    |
                     +--------------v---------------+
                     | Native Method Interface (JNI)|
                     +------------------------------+
```

### 4.1 Class Loader Subsystem
1. **Loading:** Locates `.class` files on disk or network and reads the binary data.
   - *Bootstrap ClassLoader:* Loads core JDK classes (`java.lang.*`, `java.util.*`) from runtime modules (`lib/modules`).
   - *Platform/Extension ClassLoader:* Loads platform-specific extensions.
   - *Application/System ClassLoader:* Loads classes found on the application classpath (`-cp` or environment variable).
2. **Linking:**
   - *Verification:* Ensures bytecode adheres to JVM specifications and does not violate security constraints.
   - *Preparation:* Allocates memory for static fields and initializes them to default values.
   - *Resolution:* Replaces symbolic memory references with direct references.
3. **Initialization:** Executes static initializers and assigns explicit static values.

### 4.2 Runtime Memory Areas
- **Method Area (Metaspace since Java 8):** Shared across all threads. Stores class metadata, bytecode instructions, static variables, and the runtime constant pool. It resides in native memory.
- **Heap Area:** Shared across all threads. All objects created with the `new` keyword and all arrays reside here. This is the primary domain of the Garbage Collector (GC).
- **JVM Stack:** Created per thread. Each method invocation pushes a new **Stack Frame** containing:
  - Local Variable Array (primitives and object reference pointers).
  - Operand Stack (workspace for intermediate arithmetic and logical calculations).
  - Frame Data (return values and exception dispatch tables).
- **Program Counter (PC) Register:** One per thread; stores the address of the currently executing JVM instruction.
- **Native Method Stack:** Stores frames for native methods written in C/C++ invoked via the Java Native Interface (JNI).

---

## 5. JDK CLI Tools and Usage

A software engineer must understand the core CLI tools provided by the JDK 17 installation.

| Tool | Purpose | Typical Syntax |
|---|---|---|
| `javac` | Compiles `.java` files into `.class` bytecode | `javac -d bin src/co/vinod/basics/Welcome.java` |
| `java` | Launches the JVM and runs the class `main` method | `java -cp bin co.vinod.basics.Welcome` |
| `javap` | Bytecode disassembler for inspecting compiled classes | `javap -c -p bin/co/vinod/basics/Welcome.class` |
| `jar` | Packages multiple classes and assets into a single archive | `jar -cvfe app.jar co.vinod.basics.Welcome -C bin .` |
| `jshell` | Interactive REPL (Read-Eval-Print-Loop) for quick tests | `jshell` |

### Step-by-Step CLI Walkthrough
```bash
# 1. Create directory structure
mkdir -p src/co/vinod/basics bin

# 2. Compile Java file into target bin directory
javac -d bin src/co/vinod/basics/Welcome.java

# 3. Inspect generated bytecode
javap -c bin/co/vinod/basics/Welcome.class

# 4. Run the application
java -cp bin co.vinod.basics.Welcome
```

---

## 6. Structure of a Java Class & Packages (`co.vinod`)

### 6.1 Package Naming & Directory Conventions
Java mandates that package names reflect reverse internet domain names in all lowercase letters:
- Company/Author: `co.vinod`
- Module: `co.vinod.basics`
- Directory path: `co/vinod/basics/`

### 6.2 Complete Java Class Anatomy
```java
package co.vinod.basics;

// 1. Explicit import of external classes
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

// 2. Static import for constants or utility methods
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

/**
 * Demonstrates the anatomy of a production-grade Java class.
 * Author: Vinod Kumar (Bangalore)
 */
public class EmployeeProfile {

    // 3. Class Variable (Static: one copy shared across all instances)
    public static final String COMPANY_NAME = "Vinod Tech Solutions Bangalore";
    private static int totalRegistrations = 0;

    // 4. Instance Variables (State: each instance on heap gets its own copy)
    private int employeeId;
    private String employeeName;
    private String city;
    private double basicSalary;

    // 5. Parameterized Constructor
    public EmployeeProfile(int employeeId, String employeeName, String city, double basicSalary) {
        this.employeeId = employeeId;
        this.employeeName = employeeName;
        this.city = city;
        this.basicSalary = basicSalary;
        totalRegistrations++;
    }

    // 6. Instance Methods (Behavior)
    public void displaySummary() {
        System.out.println("==========================================");
        System.out.println("Organization : " + COMPANY_NAME);
        System.out.println("Employee ID  : " + this.employeeId);
        System.out.println("Name         : " + this.employeeName);
        System.out.println("City         : " + this.city);
        System.out.printf("Salary       : ₹%,.2f%n", this.basicSalary);
        System.out.println("==========================================");
    }

    // 7. Static Method
    public static int getTotalRegistrations() {
        return totalRegistrations;
    }

    // 8. Application Entry Point
    public static void main(String[] args) {
        System.out.println("Running EmployeeProfile Driver...");
        
        // Creating instances on the heap
        EmployeeProfile emp1 = new EmployeeProfile(1001, "Vinod", "Bangalore", 125000.0);
        EmployeeProfile emp2 = new EmployeeProfile(1002, "Shyam", "Mysore", 95000.0);
        EmployeeProfile emp3 = new EmployeeProfile(1003, "Murali", "Bangalore", 110000.0);

        emp1.displaySummary();
        emp2.displaySummary();
        emp3.displaySummary();

        System.out.println("Total Employees Registered: " + EmployeeProfile.getTotalRegistrations());
    }
}
```

---

## 7. Working with Data Types: Primitives vs References

Java has two distinct categories of data types:

```
                            Data Types in Java
                                    |
            +-----------------------+-----------------------+
            |                                               |
     Primitive Types                                 Reference Types
   (Stored directly on stack)                     (Memory address on stack
            |                                      pointing to heap object)
    +-------+-------+                                       |
    |               |                                +------+------+------+
 Numeric         Logical                             |      |      |      |
    |               |                              Class  Array Interface Record
+---+---+       boolean
|       |
Integer Floating
|       |
byte    float
short   double
int
long
char
```

### 7.1 Primitive Data Types Summary
| Primitive Type | Bits | Bytes | Default | Range | Wrapper Class |
|---|---|---|---|---|---|
| `byte` | 8 | 1 | `0` | -128 to 127 | `java.lang.Byte` |
| `short` | 16 | 2 | `0` | -32,768 to 32,767 | `java.lang.Short` |
| `int` | 32 | 4 | `0` | -2³¹ to 2³¹ - 1 (~2.14 Billion) | `java.lang.Integer` |
| `long` | 64 | 8 | `0L` | -2⁶³ to 2⁶³ - 1 | `java.lang.Long` |
| `float` | 32 | 4 | `0.0f` | ±1.4E-45 to ±3.4E+38 (7 digits precision) | `java.lang.Float` |
| `double` | 64 | 8 | `0.0d` | ±4.9E-324 to ±1.7E+308 (15 digits precision) | `java.lang.Double` |
| `char` | 16 | 2 | `'\u0000'` | Unicode characters (0 to 65,535) | `java.lang.Character` |
| `boolean` | JVM | JVM | `false` | `true` or `false` | `java.lang.Boolean` |

### 7.2 Memory Breakdown: Primitive vs Reference Variable
Consider the following snippet:
```java
int empId = 101;
String trainer = "Vinod";
EmployeeProfile profile = new EmployeeProfile(101, "Vinod", "Bangalore", 150000.0);
```

```
STACK MEMORY (Thread Specific)               HEAP MEMORY (Shared)
+------------------------------------+       +------------------------------------+
| Variable   | Value                 |       | Address  | Object Content          |
+------------+-----------------------+       +----------+-------------------------+
| empId      | 101                   |       | 0x5000   | "Vinod" [String Pool]   |
| trainer    | 0x5000 (Pointer)      | ----> |          |                         |
| profile    | 0x7A00 (Pointer)      | ----> | 0x7A00   | EmployeeProfile Object  |
+------------------------------------+       |          | - employeeId: 101       |
                                             |          | - employeeName: ->0x5000|
                                             |          | - city: ->0x5020        |
                                             |          | - basicSalary: 150000.0 |
                                             +------------------------------------+
```

### 7.3 Local Variable Type Inference (`var`) - Java 10+
Since Java 10, the `var` keyword allows compiler-inferred types for local variables:
```java
var city = "Bangalore";               // Inferred as String
var salary = 125000.50;               // Inferred as double
var emp = new EmployeeProfile(...);   // Inferred as EmployeeProfile
// var unassigned;                   // COMPILE ERROR: Cannot infer without initializer!
```
*Key Rule:* Java remains strictly statically typed. `var` is not like JavaScript's `var`. Once inferred at compile time, the variable type is immutable.

---

## 8. Operators and Control Statements

### 8.1 Operators in Java
- **Arithmetic:** `+`, `-`, `*`, `/`, `%`
- **Unary:** `++`, `--`, `+`, `-`, `!`
- **Relational:** `==`, `!=`, `>`, `<`, `>=`, `<=`
- **Logical:** Short-circuit `&&` (AND), `||` (OR); Non-short-circuit `&`, `|`
- **Bitwise:** `&`, `|`, `^`, `~`, `<<`, `>>`, `>>>` (unsigned right shift)
- **Ternary Operator:** `variable = (condition) ? expressionIfTrue : expressionIfFalse;`

### 8.2 Control Flow Statements
Java provides three categories of control statements:
1. **Selection:** `if`, `if-else`, nested `if`, and `switch`.
2. **Iteration:** `for`, enhanced `for-each`, `while`, `do-while`.
3. **Jump:** `break`, labeled `break`, `continue`, `return`.

### 8.3 Evolution: Traditional Switch vs Java 17 Switch Expression
Traditional switch statements had fall-through bugs if `break` was omitted:
```java
// Traditional switch
switch (city) {
    case "Bangalore":
        allowance = 5000.0;
        break;
    case "Mysore":
        allowance = 3000.0;
        break;
    default:
        allowance = 1000.0;
}
```

Modern Java 17 Switch Expressions introduce arrow syntax (`->`), yield values, and require exhaustive matching:
```java
// Java 17 Enhanced Switch Expression
double allowance = switch (city) {
    case "Bangalore" -> 5000.0;
    case "Mysore", "Mangalore" -> 3500.0;
    case "Hubli" -> 2500.0;
    default -> {
        System.out.println("Standard tier applicable for city: " + city);
        yield 1500.0;
    }
};
```

---

## 9. Comprehensive Lab Assignment: Console Calculator Application

### 9.1 Problem Statement
Develop an enterprise-grade interactive command-line calculator in package `co.vinod.calculator`. The application must accept two numeric operands, prompt for an arithmetic operation (`+`, `-`, `*`, `/`, `%`, `^`), handle edge cases including division by zero and invalid inputs gracefully, and continue executing until the user explicitly requests to quit (`q`).

### 9.2 Complete Implementation Source Code

```java
package co.vinod.calculator;

import java.util.Scanner;

/**
 * Interactive Console Calculator Application
 * Package: co.vinod.calculator
 * Author: Vinod Kumar (Bangalore)
 */
public class CalculatorApp {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        boolean keepRunning = true;

        System.out.println("==================================================");
        System.out.println("  Welcome to the CDAC BDA Java 17 Console Calc   ");
        System.out.println("  Author: Vinod Kumar | Location: Bangalore       ");
        System.out.println("==================================================");

        while (keepRunning) {
            System.out.print("
Choose Operation [ +, -, *, /, %, ^ ] or 'q' to Quit: ");
            String inputChoice = scanner.next().trim();

            if (inputChoice.equalsIgnoreCase("q")) {
                System.out.println("Thank you for using Vinod's Calculator. Exiting system...");
                keepRunning = false;
                break;
            }

            if (inputChoice.length() != 1 || "+-*/%^".indexOf(inputChoice.charAt(0)) == -1) {
                System.out.println("Error: Unsupported operator '" + inputChoice + "'. Please choose valid operator.");
                continue;
            }

            char operator = inputChoice.charAt(0);

            // Read Operands with Defensive Validation
            double num1 = readValidDouble(scanner, "Enter First Number  (Operand 1): ");
            double num2 = readValidDouble(scanner, "Enter Second Number (Operand 2): ");

            // Calculate Result using Java 17 Switch Expression
            try {
                double result = evaluate(operator, num1, num2);
                System.out.printf("%n>>> Result: %,.4f %c %,.4f = %,.4f%n", num1, operator, num2, result);
            } catch (ArithmeticException ex) {
                System.err.println(">>> Math Error: " + ex.getMessage());
            }
        }

        scanner.close();
    }

    /**
     * Evaluates arithmetic operations using modern Java 17 Switch Expression.
     */
    public static double evaluate(char operator, double a, double b) {
        return switch (operator) {
            case '+' -> a + b;
            case '-' -> a - b;
            case '*' -> a * b;
            case '/' -> {
                if (b == 0.0) {
                    throw new ArithmeticException("Division by zero is undefined.");
                }
                yield a / b;
            }
            case '%' -> {
                if (b == 0.0) {
                    throw new ArithmeticException("Modulo by zero is undefined.");
                }
                yield a % b;
            }
            case '^' -> Math.pow(a, b);
            default -> throw new IllegalArgumentException("Unknown operator: " + operator);
        };
    }

    /**
     * Helper to read valid doubles and prevent InputMismatchException crashes.
     */
    private static double readValidDouble(Scanner scanner, String prompt) {
        while (true) {
            System.out.print(prompt);
            if (scanner.hasNextDouble()) {
                return scanner.nextDouble();
            } else {
                String invalidToken = scanner.next();
                System.out.println("Invalid input: '" + invalidToken + "' is not a valid number. Try again.");
            }
        }
    }
}
```

### 9.3 Expected Console Output
```text
==================================================
  Welcome to the CDAC BDA Java 17 Console Calc   
  Author: Vinod Kumar | Location: Bangalore       
==================================================

Choose Operation [ +, -, *, /, %, ^ ] or 'q' to Quit: +
Enter First Number  (Operand 1): 150000
Enter Second Number (Operand 2): 25000

>>> Result: 150,000.0000 + 25,000.0000 = 175,000.0000

Choose Operation [ +, -, *, /, %, ^ ] or 'q' to Quit: /
Enter First Number  (Operand 1): 98000
Enter Second Number (Operand 2): 0

>>> Math Error: Division by zero is undefined.

Choose Operation [ +, -, *, /, %, ^ ] or 'q' to Quit: q
Thank you for using Vinod's Calculator. Exiting system...
```

---

## 10. Self-Study & Review Questions
1. Why is the `main` method in Java declared as `public static void`? What would happen if `static` were omitted?
2. Explain the step-by-step role of the JIT compiler. How does the HotSpot profiler determine when to compile bytecode to native code?
3. Contrast stack allocation vs heap allocation in the JVM. Under what conditions does an application throw `java.lang.StackOverflowError` vs `java.lang.OutOfMemoryError`?
4. How do Java 17 Switch Expressions enhance code safety compared to legacy switch statements?
