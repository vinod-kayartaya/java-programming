# Sessions 10 & 11: Functional Programming

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 4T + 2L + 2SL (4 Hours Theory + 2 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Understand the paradigm shift from imperative object-oriented programming to declarative functional programming in Java.
2. Define and validate custom Single Abstract Method (SAM) interfaces using `@FunctionalInterface`.
3. Master the core functional interfaces in `java.util.function` (`Predicate<T>`, `Function<T, R>`, `Consumer<T>`, `Supplier<T>`, and their bi-variants/primitive specializations).
4. Author concise, expressive Lambda Expressions, recognizing variable capture rules (effectively final).
5. Apply the four categories of Method References (`::`) to eliminate lambda boilerplate.
6. Refactor legacy collection algorithms into functional pipelines using `forEach`, `removeIf`, and `sort`.

---

## 2. Functional Programming Fundamentals in Java

### 2.1 The Paradigm Shift
- **Imperative Programming (How to do it):** Focuses on explicit step-by-step instructions, mutability, and looping constructs.
- **Declarative / Functional Programming (What to do):** Focuses on expressions, mathematical functions, immutability, and zero side-effects.

```java
// Imperative Approach: Explicit loop and state mutation
List<Employee> bangaloreStaff = new ArrayList<>();
for (Employee emp : employees) {
    if ("Bangalore".equalsIgnoreCase(emp.getCity())) {
        bangaloreStaff.add(emp);
    }
}

// Declarative Functional Approach: Describe the predicate contract
List<Employee> bangaloreStaff = employees.stream()
    .filter(emp -> "Bangalore".equalsIgnoreCase(emp.getCity()))
    .toList();
```

---

## 3. Functional Interfaces (`java.util.function`)

A **Functional Interface** contains exactly one abstract method (Single Abstract Method - SAM). It may contain any number of `default` or `static` methods.

### 3.1 The Big Four Functional Interfaces
| Interface | Signature | Abstract Method | Semantic Purpose | Real-World CDAC Example |
|---|---|---|---|---|
| `Predicate<T>` | `T -> boolean` | `boolean test(T t)` | Conditional filtering / verification | Check if employee earns > ₹1,00,000 |
| `Function<T, R>` | `T -> R` | `R apply(T t)` | Transformation / mapping | Extract name or convert Employee to DTO |
| `Consumer<T>` | `T -> void` | `void accept(T t)` | Terminal action / side effect | Print employee slip or log audit |
| `Supplier<T>` | `() -> T` | `T get()` | Factory / deferred generation | Produce default fallback employee |

### 3.2 Bi-Variants and Primitive Specializations
To eliminate the performance penalty of autoboxing, Java provides specialized versions:
- `BiPredicate<T, U>`, `BiFunction<T, U, R>`, `BiConsumer<T, U>`
- `IntPredicate`, `DoubleFunction<R>`, `ToLongFunction<T>`

---

## 4. Lambda Expressions & Variable Capture

### 4.1 Syntax Structure
```java
// Syntax: (parameter_list) -> { lambda_body }
(Employee e) -> { return e.getSalary() > 100000; }

// Concise syntax with type inference & single-expression return:
e -> e.getSalary() > 100000
```

### 4.2 Variable Capture Rules: Effectively Final
Lambdas can capture local variables defined outside their body **only if they are final or effectively final** (their value is never modified after initialization). This guarantees thread safety and avoids race conditions.

```java
double minThreshold = 80000.0; // Effectively final
Predicate<Employee> filter = emp -> emp.getSalary() > minThreshold; // VALID

// minThreshold = 90000.0; // ERROR! Reassignment makes variable not effectively final!
```

---

## 5. Method References (`::`)

Method references provide an ultra-clean syntax when a lambda simply invokes an existing method:

| Category | Syntax Pattern | Lambda Equivalent | Example |
|---|---|---|---|
| **Static Method Reference** | `ClassName::staticMethod` | `x -> Math.abs(x)` | `Math::abs` |
| **Instance Method of Existing Object** | `objectRef::instanceMethod` | `x -> System.out.println(x)` | `System.out::println` |
| **Instance Method of Arbitrary Object** | `ClassName::instanceMethod` | `emp -> emp.getName()` | `Employee::getName` |
| **Constructor Reference** | `ClassName::new` | `() -> new ArrayList<>()` | `ArrayList::new` |

---

## 6. Comprehensive Lab Assignment: Refactoring Imperative to Functional

### 6.1 Problem Statement
In package `co.vinod.fp`:
1. Model employee dataset (Vinod in Bangalore, Shyam in Mysore, Murali in Bangalore, Naveen in Mysore, Harish in Bangalore).
2. Refactor procedural iteration loops into declarative functional pipelines using `Predicate`, `Function`, `Consumer`, and `Supplier`.
3. Chain predicates using `.and()`, `.or()`, and `.negate()`.
4. Compose functions using `.andThen()`.
5. Apply method references for sorting and output formatting.

### 6.2 Implementation Source Code

#### File: `src/co/vinod/fp/StaffMember.java`
```java
package co.vinod.fp;

public class StaffMember {
    private final int id;
    private final String name;
    private final String city;
    private final double salary;
    private final String department;

    public StaffMember(int id, String name, String city, double salary, String department) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.salary = salary;
        this.department = department;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCity() { return city; }
    public double getSalary() { return salary; }
    public String getDepartment() { return department; }

    @Override
    public String toString() {
        return String.format("StaffMember[ID=%d, Name='%-6s', City='%-9s', Dept='%-11s', Salary=₹%,.2f]",
                id, name, city, department, salary);
    }
}
```

#### File: `src/co/vinod/fp/FunctionalProgrammingApp.java`
```java
package co.vinod.fp;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

/**
 * Enterprise Functional Programming & Lambda Refactoring Demo
 * Package: co.vinod.fp
 * Author: Vinod Kumar (Bangalore)
 */
public class FunctionalProgrammingApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("      CDAC BDA Java 17 Functional Programming Masterclass         ");
        System.out.println("==================================================================
");

        List<StaffMember> staffList = initializeStaff();

        // 1. Predicate Demonstration: Filtering and Chaining
        demonstratePredicates(staffList);

        // 2. Function Demonstration: Transformation and Composition
        demonstrateFunctions(staffList);

        // 3. Consumer & Supplier Demonstration
        demonstrateConsumerAndSupplier(staffList);

        // 4. Refactoring Collections via Lambdas & Method References
        demonstrateCollectionRefactoring(staffList);
    }

    private static List<StaffMember> initializeStaff() {
        List<StaffMember> list = new ArrayList<>();
        list.add(new StaffMember(101, "Vinod", "Bangalore", 165000.0, "Architecture"));
        list.add(new StaffMember(102, "Shyam", "Mysore", 98000.0, "Development"));
        list.add(new StaffMember(103, "Murali", "Bangalore", 125000.0, "Analytics"));
        list.add(new StaffMember(104, "Naveen", "Mysore", 88000.0, "Development"));
        list.add(new StaffMember(105, "Harish", "Bangalore", 140000.0, "Cloud"));
        return list;
    }

    private static void demonstratePredicates(List<StaffMember> staffList) {
        System.out.println("--- 1. Predicates: Filtering & Boolean Composition ---");
        
        Predicate<StaffMember> isBangalore = emp -> "Bangalore".equalsIgnoreCase(emp.getCity());
        Predicate<StaffMember> isHighSalary = emp -> emp.getSalary() >= 130000.0;

        // Composed Predicate: Bangalore AND Salary >= 130,000
        Predicate<StaffMember> seniorBangaloreStaff = isBangalore.and(isHighSalary);

        System.out.println("Senior Bangalore Staff (City=Bangalore AND Salary >= ₹1,30,000):");
        filterAndPrint(staffList, seniorBangaloreStaff);

        // Negated Predicate: NOT Bangalore (e.g. Mysore)
        System.out.println("
Non-Bangalore Staff (Negation):");
        filterAndPrint(staffList, isBangalore.negate());
        System.out.println();
    }

    private static void filterAndPrint(List<StaffMember> list, Predicate<StaffMember> condition) {
        for (StaffMember emp : list) {
            if (condition.test(emp)) {
                System.out.println("  " + emp);
            }
        }
    }

    private static void demonstrateFunctions(List<StaffMember> staffList) {
        System.out.println("--- 2. Functions: Mapping & Function Chaining (.andThen) ---");

        // Function 1: StaffMember -> String (Formatted Name & City)
        Function<StaffMember, String> toLabel = emp -> emp.getName() + " [" + emp.getCity() + "]";

        // Function 2: String -> String (Uppercase)
        Function<String, String> toUpper = String::toUpperCase;

        // Composed Pipeline: toLabel AND THEN toUpper
        Function<StaffMember, String> badgeFormatter = toLabel.andThen(toUpper);

        for (StaffMember emp : staffList) {
            System.out.println("Badge: " + badgeFormatter.apply(emp));
        }
        System.out.println();
    }

    private static void demonstrateConsumerAndSupplier(List<StaffMember> staffList) {
        System.out.println("--- 3. Consumer & Supplier Mechanics ---");

        // Consumer: prints salary notification
        Consumer<StaffMember> notifier = emp -> 
            System.out.printf("Notification sent to %s for monthly credit of ₹%,.2f%n", emp.getName(), emp.getSalary());

        staffList.forEach(notifier);

        // Supplier: provides a fallback employee record
        Supplier<StaffMember> fallbackSupplier = () -> 
            new StaffMember(999, "Guest", "Bangalore", 50000.0, "Consulting");

        System.out.println("
Default Fallback from Supplier: " + fallbackSupplier.get());
        System.out.println();
    }

    private static void demonstrateCollectionRefactoring(List<StaffMember> staffList) {
        System.out.println("--- 4. Collection Refactoring: Lambdas & Method References ---");

        // In-place sorting with Comparator.comparing & reversed
        staffList.sort(Comparator.comparing(StaffMember::getSalary).reversed());

        System.out.println("Staff sorted by Salary descending (using Method References):");
        staffList.forEach(System.out::println);

        // removeIf with Predicate
        staffList.removeIf(emp -> emp.getSalary() < 90000.0);
        System.out.println("
After removeIf(salary < 90,000), total staff: " + staffList.size());
    }
}
```

### 6.3 Expected Execution Output
```text
==================================================================
      CDAC BDA Java 17 Functional Programming Masterclass         
==================================================================

--- 1. Predicates: Filtering & Boolean Composition ---
Senior Bangalore Staff (City=Bangalore AND Salary >= ₹1,30,000):
  StaffMember[ID=101, Name='Vinod ', City='Bangalore', Dept='Architecture', Salary=₹1,65,000.00]
  StaffMember[ID=105, Name='Harish', City='Bangalore', Dept='Cloud      ', Salary=₹1,40,000.00]

Non-Bangalore Staff (Negation):
  StaffMember[ID=102, Name='Shyam ', City='Mysore   ', Dept='Development', Salary=₹98,000.00]
  StaffMember[ID=104, Name='Naveen', City='Mysore   ', Dept='Development', Salary=₹88,000.00]

--- 2. Functions: Mapping & Function Chaining (.andThen) ---
Badge: VINOD [BANGALORE]
Badge: SHYAM [MYSORE]
Badge: MURALI [BANGALORE]
Badge: NAVEEN [MYSORE]
Badge: HARISH [BANGALORE]

--- 3. Consumer & Supplier Mechanics ---
Notification sent to Vinod for monthly credit of ₹1,65,000.00
Notification sent to Shyam for monthly credit of ₹98,000.00
Notification sent to Murali for monthly credit of ₹1,25,000.00
Notification sent to Naveen for monthly credit of ₹88,000.00
Notification sent to Harish for monthly credit of ₹1,40,000.00

Default Fallback from Supplier: StaffMember[ID=999, Name='Guest ', City='Bangalore', Dept='Consulting ', Salary=₹50,000.00]

--- 4. Collection Refactoring: Lambdas & Method References ---
Staff sorted by Salary descending (using Method References):
StaffMember[ID=101, Name='Vinod ', City='Bangalore', Dept='Architecture', Salary=₹1,65,000.00]
StaffMember[ID=105, Name='Harish', City='Bangalore', Dept='Cloud      ', Salary=₹1,40,000.00]
StaffMember[ID=103, Name='Murali', City='Bangalore', Dept='Analytics  ', Salary=₹1,25,000.00]
StaffMember[ID=102, Name='Shyam ', City='Mysore   ', Dept='Development', Salary=₹98,000.00]
StaffMember[ID=104, Name='Naveen', City='Mysore   ', Dept='Development', Salary=₹88,000.00]

After removeIf(salary < 90,000), total staff: 4
```

---

## 7. Self-Study & Review Questions
1. Why does Java restrict local variable access in lambdas to variables that are final or effectively final?
2. What happens under the hood during the compilation of a Lambda Expression? Explain `invokedynamic` vs creating anonymous inner classes.
3. Compare `Function.compose()` with `Function.andThen()`. In what sequence are the composed functions evaluated?
