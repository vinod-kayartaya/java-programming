# Session 4: Pillars of OOP

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 2T + 4L + 2SL (2 Hours Theory + 4 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Implement strict encapsulation using appropriate access modifiers, constructor chaining (`this()`, `super()`), and create immutable classes.
2. Build polymorphic inheritance hierarchies using abstract classes and interfaces with Java 8/9 default, static, and private methods.
3. Master Inner Classes: Member Inner Classes, Static Nested Classes, Local Inner Classes, and Anonymous Inner Classes.
4. Distinguish between Reference Type (compile-time) and Object Type (runtime) during dynamic method dispatch.
5. Leverage Java 16/17 Pattern Matching for `instanceof` to eliminate boilerplate object casting.

---

## 2. Encapsulation & Method Mechanics

### 2.1 Java Access Modifiers
| Access Modifier | Within Same Class | Within Same Package | Subclass (Other Package) | Universal (Anywhere) |
|---|---|---|---|---|
| `private` | Yes | No | No | No |
| *(default / package-private)* | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes (via inheritance) | No |
| `public` | Yes | Yes | Yes | Yes |

### 2.2 Strict Pass-by-Value Semantics
In Java, **everything is strictly passed by value**.
- **Primitive arguments:** The method receives a copy of the primitive value. Changes to the parameter variable inside the method do not affect the caller's variable.
- **Reference arguments:** The method receives a **copy of the reference address**. While mutating the object via the copied pointer alters the shared heap object, reassigning the reference parameter itself (`param = new Object()`) has zero effect on the caller's reference.

```
Caller Frame:                        Callee Frame:
+--------------------------+         +--------------------------+
| empRef: 0x8800 (Pointer) | ------> | param: 0x8800 (Copy)     |
+--------------------------+         +--------------------------+
             |                                    |
             +-----------------> HEAP <-----------+
                            [Employee Object]
```

### 2.3 Immutable Classes
An immutable class cannot be modified once instantiated (like `String` or `Integer`):
1. Declare the class `final` so it cannot be subclassed.
2. Make all fields `private` and `final`.
3. Provide no mutator (setter) methods.
4. Perform defensive copying for mutable fields in constructors and getters.

*(Note: Java 14+ introduces `record` as a native construct for immutable data carriers: `public record EmployeeRecord(int id, String name) {}`).*

---

## 3. Inheritance, Abstract Classes & Interfaces

### 3.1 Abstract Classes vs Interfaces (Java 17 Standards)

| Characteristic | Abstract Class | Interface |
|---|---|---|
| **Multiple Inheritance** | Single class inheritance (`extends`) | Multiple interface implementation (`implements`) |
| **State / Fields** | Can have instance variables with state | Only `public static final` constants |
| **Constructors** | Yes (invoked via `super()`) | No constructors allowed |
| **Method Types** | Abstract, concrete, final, static | Abstract, `default`, `static`, `private` (Java 9+) |
| **Conceptual Goal** | "Is-A" core identity relationship | "Can-Do" behavioral capability contract |

### 3.2 Dynamic Method Dispatch (Runtime Polymorphism)
```java
Employee emp = new Manager("Vinod", 150000.0, 30000.0);
emp.calculatePay(); 
// The compiler verifies calculatePay() exists on Employee.
// At runtime, the JVM inspects the heap object (Manager) and invokes Manager's calculatePay().
```

### 3.3 Modern Pattern Matching for `instanceof` (Java 16/17 LTS)
```java
// Traditional verbose casting:
if (emp instanceof Manager) {
    Manager m = (Manager) emp;
    m.approveExpense(5000.0);
}

// Modern Java 16/17 Pattern Matching:
if (emp instanceof Manager m) {
    m.approveExpense(5000.0); // Variable m is immediately in scope and type-safe
}
```

---

## 4. Inner Classes Architecture

1. **Member Inner Class:** Non-static class defined within an enclosing class. Holds an implicit reference to `Outer.this`.
2. **Static Nested Class:** Defined with `static`. Can be instantiated without an enclosing class instance: `new Outer.StaticNested()`.
3. **Local Inner Class:** Defined inside a method block; can access effectively final local variables.
4. **Anonymous Inner Class:** Nameless class defined and instantiated on the fly to override a class or implement an interface.

---

## 5. Comprehensive Lab Assignment: Enterprise Hierarchy & Inner Classes

### 5.1 Problem Statement
Build an enterprise compensation hierarchy in package `co.vinod.payroll`:
1. Abstract base class `Employee` with `id`, `name`, `city`, `baseSalary`, and an abstract method `double calculateTotalEarnings()`.
2. Concrete subclasses:
   - `Manager`: adds `bonus` and `departmentSize`.
   - `Developer`: adds `overtimePay` and `primarySkill`.
3. Add a **Member Inner Class** `BenefitsEnrollment` inside `Employee` to manage medical insurance and retirement fund deductions.
4. In `co.vinod.payroll.PayrollApp`:
   - Instantiate managers and developers (Vinod, Shyam, Murali).
   - Demonstrate runtime polymorphism using parent references.
   - Use Java 17 pattern matching with `instanceof`.
   - Create an **Anonymous Inner Class** implementing a `PerformanceBonusCalculator` interface.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/payroll/Employee.java`
```java
package co.vinod.payroll;

/**
 * Abstract Base Employee Class
 * Package: co.vinod.payroll
 */
public abstract class Employee {

    private int id;
    private String name;
    private String city;
    protected double baseSalary;

    public Employee(int id, String name, String city, double baseSalary) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.baseSalary = baseSalary;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCity() { return city; }
    public double getBaseSalary() { return baseSalary; }

    // Abstract method: mandatory contract for subclasses
    public abstract double calculateTotalEarnings();

    public void printEmployeeSlip() {
        System.out.printf("Employee #%d | Name: %-8s | City: %-10s | Total: ₹%,.2f%n",
                id, name, city, calculateTotalEarnings());
    }

    // -------------------------------------------------------------
    // MEMBER INNER CLASS: BenefitsEnrollment
    // -------------------------------------------------------------
    public class BenefitsEnrollment {
        private String planName;
        private double coverageAmount;

        public BenefitsEnrollment(String planName, double coverageAmount) {
            this.planName = planName;
            this.coverageAmount = coverageAmount;
        }

        public void displayBenefits() {
            // Member inner classes access outer class private fields directly (e.g. name, baseSalary)
            System.out.printf("  [Benefits] %s enrolled in plan '%s' with coverage of ₹%,.2f (Monthly Premium: ₹%,.2f)%n",
                    name, planName, coverageAmount, (baseSalary * 0.02));
        }
    }
}
```

#### File: `src/co/vinod/payroll/Manager.java`
```java
package co.vinod.payroll;

public class Manager extends Employee {

    private double executiveBonus;
    private int teamSize;

    public Manager(int id, String name, String city, double baseSalary, double executiveBonus, int teamSize) {
        super(id, name, city, baseSalary);
        this.executiveBonus = executiveBonus;
        this.teamSize = teamSize;
    }

    public double getExecutiveBonus() { return executiveBonus; }
    public int getTeamSize() { return teamSize; }

    @Override
    public double calculateTotalEarnings() {
        return baseSalary + executiveBonus;
    }

    public void conductStrategyMeeting() {
        System.out.println("  -> Manager " + getName() + " is conducting quarterly strategy review for team of " + teamSize);
    }
}
```

#### File: `src/co/vinod/payroll/Developer.java`
```java
package co.vinod.payroll;

public class Developer extends Employee {

    private double overtimePay;
    private String primarySkill;

    public Developer(int id, String name, String city, double baseSalary, double overtimePay, String primarySkill) {
        super(id, name, city, baseSalary);
        this.overtimePay = overtimePay;
        this.primarySkill = primarySkill;
    }

    public double getOvertimePay() { return overtimePay; }
    public String getPrimarySkill() { return primarySkill; }

    @Override
    public double calculateTotalEarnings() {
        return baseSalary + overtimePay;
    }

    public void deployModule() {
        System.out.println("  -> Developer " + getName() + " is deploying " + primarySkill + " service to production.");
    }
}
```

#### File: `src/co/vinod/payroll/PerformanceBonusCalculator.java`
```java
package co.vinod.payroll;

@FunctionalInterface
public interface PerformanceBonusCalculator {
    double computeSpecialBonus(Employee emp);
}
```

#### File: `src/co/vinod/payroll/PayrollApp.java`
```java
package co.vinod.payroll;

/**
 * Driver Application demonstrating Polymorphism, Inner Classes, and Pattern Matching.
 * Package: co.vinod.payroll
 * Author: Vinod Kumar (Bangalore)
 */
public class PayrollApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("     CDAC BDA Enterprise Payroll System - Vinod Tech Bangalore     ");
        System.out.println("==================================================================
");

        // 1. Polymorphic Collection of Employees
        Employee[] staff = new Employee[]{
            new Manager(1001, "Vinod", "Bangalore", 160000.0, 45000.0, 12),
            new Developer(1002, "Shyam", "Mysore", 95000.0, 15000.0, "Java 17 & Microservices"),
            new Developer(1003, "Murali", "Bangalore", 110000.0, 20000.0, "Kafka & Cloud")
        };

        // 2. Dynamic Method Dispatch & Pattern Matching
        for (Employee emp : staff) {
            emp.printEmployeeSlip();

            // Modern Java 16/17 Pattern Matching for instanceof
            if (emp instanceof Manager mgr) {
                mgr.conductStrategyMeeting();
            } else if (emp instanceof Developer dev) {
                dev.deployModule();
            }

            // Instantiating Member Inner Class via outer object instance
            Employee.BenefitsEnrollment benefits = emp.new BenefitsEnrollment("Platinum Health Cover", 1000000.0);
            benefits.displayBenefits();
            System.out.println();
        }

        // 3. Anonymous Inner Class Implementation
        System.out.println("=== Applying Ad-Hoc Performance Bonus (Anonymous Inner Class) ===");
        PerformanceBonusCalculator festiveBonus = new PerformanceBonusCalculator() {
            @Override
            public double computeSpecialBonus(Employee emp) {
                // Bangalore branch employees receive festive festival multiplier
                if ("Bangalore".equalsIgnoreCase(emp.getCity())) {
                    return emp.getBaseSalary() * 0.10;
                }
                return emp.getBaseSalary() * 0.05;
            }
        };

        for (Employee emp : staff) {
            double bonus = festiveBonus.computeSpecialBonus(emp);
            System.out.printf("Festival Bonus for %s (%s): ₹%,.2f%n", emp.getName(), emp.getCity(), bonus);
        }
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
     CDAC BDA Enterprise Payroll System - Vinod Tech Bangalore     
==================================================================

Employee #1001 | Name: Vinod    | City: Bangalore  | Total: ₹2,05,000.00
  -> Manager Vinod is conducting quarterly strategy review for team of 12
  [Benefits] Vinod enrolled in plan 'Platinum Health Cover' with coverage of ₹10,00,000.00 (Monthly Premium: ₹3,200.00)

Employee #1002 | Name: Shyam    | City: Mysore     | Total: ₹1,10,000.00
  -> Developer Shyam is deploying Java 17 & Microservices service to production.
  [Benefits] Shyam enrolled in plan 'Platinum Health Cover' with coverage of ₹10,00,000.00 (Monthly Premium: ₹1,900.00)

Employee #1003 | Name: Murali   | City: Bangalore  | Total: ₹1,30,000.00
  -> Developer Murali is deploying Kafka & Cloud service to production.
  [Benefits] Murali enrolled in plan 'Platinum Health Cover' with coverage of ₹10,00,000.00 (Monthly Premium: ₹2,200.00)

=== Applying Ad-Hoc Performance Bonus (Anonymous Inner Class) ===
Festival Bonus for Vinod (Bangalore): ₹16,000.00
Festival Bonus for Shyam (Mysore): ₹4,750.00
Festival Bonus for Murali (Bangalore): ₹11,000.00
```

---

## 6. Self-Study & Review Questions
1. How does the JVM resolve method calls during dynamic method dispatch? Explain the role of the Virtual Method Table (vtable).
2. Why can't an abstract class be instantiated with `new` even if all its methods are fully concrete?
3. Contrast a static nested class with a member inner class regarding their memory footprint and access to enclosing class instances.
