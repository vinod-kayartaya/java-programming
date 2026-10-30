# Session 9: Enums and Wrapper Class

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 2T + 4L (2 Hours Theory + 4 Hours Lab)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Design type-safe, feature-rich Java Enumerations containing state, custom constructors, and business behavior.
2. Integrate Enums with modern Java 17 Switch Expressions.
3. Master Autoboxing and Unboxing mechanics and avoid critical traps (Integer caching boundaries and NullPointerExceptions).
4. Utilize primitive Wrapper utility methods (`parseInt`, `valueOf`, `compareTo`, `toHexString`).
5. Apply standard annotations (`@Override`, `@Deprecated`, `@SuppressWarnings`, `@FunctionalInterface`).
6. Leverage utility classes across `java.lang` and `java.util` (`Math`, `Objects`, `Random`, `Arrays`, `Collections`).

---

## 2. Java Enumerations Architecture

In Java, enums are not simple integer flags (like in C/C++); **an enum is a specialized class** that implicitly extends `java.lang.Enum<E>`.

### 2.1 Key Enum Properties
1. Enums are strictly type-safe; passing invalid integer constants is impossible.
2. Enum constants are `public static final` singleton instances instantiated at class load time.
3. Enum constructors are implicitly `private`.
4. Enums can contain instance fields, concrete methods, abstract methods, and implement interfaces.

### 2.2 Rich Enum with Behavior Example
```java
package co.vinod.enums;

public enum EmployeeType {
    PERMANENT(1.20, "Full-Time Staff"),
    CONTRACT(1.00, "Fixed-Term Contractor"),
    INTERN(0.60, "Graduate Trainee");

    private final double salaryMultiplier;
    private final String description;

    EmployeeType(double salaryMultiplier, String description) {
        this.salaryMultiplier = salaryMultiplier;
        this.description = description;
    }

    public double getSalaryMultiplier() {
        return salaryMultiplier;
    }

    public String getDescription() {
        return description;
    }

    public double computeBonus(double baseSalary) {
        return baseSalary * salaryMultiplier * 0.15;
    }
}
```

---

## 3. Autoboxing, Unboxing and Memory Pitfalls

### 3.1 Conversions
- **Autoboxing:** Automatic conversion of a primitive to its wrapper: `Integer val = 100;` $	o$ `Integer.valueOf(100)`.
- **Unboxing:** Automatic conversion of a wrapper to its primitive: `int x = val;` $	o$ `val.intValue()`.

### 3.2 The Integer Cache Trap
Java caches `Integer` instances for values between **`-128` and `127`** (Byte cache pool):
```java
Integer a = 125, b = 125;
System.out.println(a == b); // TRUE (Same cached reference in heap)

Integer x = 200, y = 200;
System.out.println(x == y); // FALSE (Distinct heap instances allocated!)
System.out.println(x.equals(y)); // TRUE (Content comparison)
```
*Golden Rule:* **Always compare wrapper objects using `.equals()`**, never with `==`.

### 3.3 The Unboxing NullPointerException Trap
```java
Integer balance = null;
int amount = balance; // Throws NullPointerException! Compiles to balance.intValue()
```

---

## 4. Standard Annotations and Utilities

- `@Override`: Guarantees a method overrides a parent method.
- `@Deprecated(since="...", forRemoval=true)`: Warns against using outdated APIs.
- `@SuppressWarnings("unchecked")`: Disables specific compiler warnings.
- `java.util.Objects.requireNonNull(obj, "msg")`: Defensive null checking.
- `java.lang.Math`: Static mathematical calculations (`pow`, `sqrt`, `round`, `abs`).

---

## 5. Comprehensive Lab Assignment: Enums, Wrappers & Sorting

### 5.1 Problem Statement
In package `co.vinod.enums`:
1. Create `DayOfWeek` enum with methods `isWeekend()` and `isWeekday()`.
2. Create `EmployeeTier` enum (`EXECUTIVE`, `SENIOR_STAFF`, `ASSOCIATE`) with allowances.
3. In `co.vinod.enums.EnumWrapperApp`:
   - Parse numeric strings using `Double.parseDouble()` and `Integer.valueOf()`.
   - Populate a `List<EmployeeProfile>` (Vinod in Bangalore, Shyam in Mysore, Murali in Bangalore).
   - Sort employees using:
     - Natural ordering by ID via `Comparable`.
     - Custom sorting by City, then by Salary descending using `Comparator`.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/enums/DayOfWeek.java`
```java
package co.vinod.enums;

public enum DayOfWeek {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    public boolean isWeekend() {
        return this == SATURDAY || this == SUNDAY;
    }

    public boolean isWeekday() {
        return !isWeekend();
    }
}
```

#### File: `src/co/vinod/enums/EmployeeTier.java`
```java
package co.vinod.enums;

public enum EmployeeTier {
    EXECUTIVE(50000.0, 0.25),
    SENIOR_STAFF(30000.0, 0.18),
    ASSOCIATE(15000.0, 0.10);

    private final double travelAllowance;
    private final double bonusPercentage;

    EmployeeTier(double travelAllowance, double bonusPercentage) {
        this.travelAllowance = travelAllowance;
        this.bonusPercentage = bonusPercentage;
    }

    public double getTravelAllowance() { return travelAllowance; }
    public double getBonusPercentage() { return bonusPercentage; }

    public double calculateTotalBonus(double baseSalary) {
        return travelAllowance + (baseSalary * bonusPercentage);
    }
}
```

#### File: `src/co/vinod/enums/EmployeeRecord.java`
```java
package co.vinod.enums;

import java.util.Objects;

public class EmployeeRecord implements Comparable<EmployeeRecord> {
    private final int id;
    private final String name;
    private final String city;
    private final double salary;
    private final EmployeeTier tier;

    public EmployeeRecord(int id, String name, String city, double salary, EmployeeTier tier) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.salary = salary;
        this.tier = tier;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCity() { return city; }
    public double getSalary() { return salary; }
    public EmployeeTier getTier() { return tier; }

    @Override
    public int compareTo(EmployeeRecord o) {
        return Integer.compare(this.id, o.id);
    }

    @Override
    public String toString() {
        return String.format("EmployeeRecord[ID=%d, Name='%-6s', City='%-9s', Tier=%-12s, Salary=₹%,.2f]",
                id, name, city, tier, salary);
    }
}
```

#### File: `src/co/vinod/enums/EnumWrapperApp.java`
```java
package co.vinod.enums;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

/**
 * Driver demonstrating Enums, Wrapper Pitfalls, and Multi-Level Sorting
 * Package: co.vinod.enums
 * Author: Vinod Kumar (Bangalore)
 */
public class EnumWrapperApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("     CDAC BDA Enums, Wrapper Mechanics & Sorting Lab             ");
        System.out.println("==================================================================
");

        // 1. Enum Methods Demonstration
        demonstrateEnums();

        // 2. Wrapper Pool Caching Inspection
        demonstrateWrapperCache();

        // 3. Multi-Level Sorting with Comparable and Comparator
        demonstrateSorting();
    }

    private static void demonstrateEnums() {
        System.out.println("--- 1. Enum Operations ---");
        DayOfWeek today = DayOfWeek.SATURDAY;
        System.out.printf("Today is %s -> Is Weekend: %b | Is Weekday: %b%n",
                today, today.isWeekend(), today.isWeekday());

        EmployeeTier tier = EmployeeTier.EXECUTIVE;
        double base = 150000.0;
        System.out.printf("Tier: %s | Bonus for ₹%,.2f is ₹%,.2f%n%n",
                tier, base, tier.calculateTotalBonus(base));
    }

    private static void demonstrateWrapperCache() {
        System.out.println("--- 2. Wrapper Integer Cache Boundaries (-128 to 127) ---");
        Integer num1 = 120, num2 = 120;
        System.out.printf("num1 = 120, num2 = 120 -> num1 == num2: %b (Cached pool)%n", (num1 == num2));

        Integer num3 = 350, num4 = 350;
        System.out.printf("num3 = 350, num4 = 350 -> num3 == num4: %b (Outside cache!)%n", (num3 == num4));
        System.out.printf("num3.equals(num4)       -> %b (Correct content check)%n%n", num3.equals(num4));
    }

    private static void demonstrateSorting() {
        System.out.println("--- 3. Employee Sorting Demonstration ---");
        List<EmployeeRecord> employees = new ArrayList<>();
        employees.add(new EmployeeRecord(103, "Murali", "Bangalore", 115000.0, EmployeeTier.SENIOR_STAFF));
        employees.add(new EmployeeRecord(101, "Vinod", "Bangalore", 165000.0, EmployeeTier.EXECUTIVE));
        employees.add(new EmployeeRecord(102, "Shyam", "Mysore", 98000.0, EmployeeTier.SENIOR_STAFF));
        employees.add(new EmployeeRecord(104, "Naveen", "Mysore", 125000.0, EmployeeTier.EXECUTIVE));

        System.out.println("Initial Order:");
        employees.forEach(System.out::println);

        // Natural Ordering by ID (Comparable)
        Collections.sort(employees);
        System.out.println("
Sorted by Natural Order (ID via Comparable):");
        employees.forEach(System.out::println);

        // Custom Multi-Level Sorting: City ASC, then Salary DESC (Comparator)
        Comparator<EmployeeRecord> citySalaryComparator = Comparator
                .comparing(EmployeeRecord::getCity)
                .thenComparing(Comparator.comparing(EmployeeRecord::getSalary).reversed());

        employees.sort(citySalaryComparator);
        System.out.println("
Sorted by City ASC, then Salary DESC (Comparator):");
        employees.forEach(System.out::println);
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
     CDAC BDA Enums, Wrapper Mechanics & Sorting Lab             
==================================================================

--- 1. Enum Operations ---
Today is SATURDAY -> Is Weekend: true | Is Weekday: false
Tier: EXECUTIVE | Bonus for ₹1,50,000.00 is ₹87,500.00

--- 2. Wrapper Integer Cache Boundaries (-128 to 127) ---
num1 = 120, num2 = 120 -> num1 == num2: true (Cached pool)
num3 = 350, num4 = 350 -> num3 == num4: false (Outside cache!)
num3.equals(num4)       -> true (Correct content check)

--- 3. Employee Sorting Demonstration ---
Initial Order:
EmployeeRecord[ID=103, Name='Murali', City='Bangalore', Tier=SENIOR_STAFF , Salary=₹1,15,000.00]
EmployeeRecord[ID=101, Name='Vinod ', City='Bangalore', Tier=EXECUTIVE    , Salary=₹1,65,000.00]
EmployeeRecord[ID=102, Name='Shyam ', City='Mysore   ', Tier=SENIOR_STAFF , Salary=₹98,000.00]
EmployeeRecord[ID=104, Name='Naveen', City='Mysore   ', Tier=EXECUTIVE    , Salary=₹1,25,000.00]

Sorted by Natural Order (ID via Comparable):
EmployeeRecord[ID=101, Name='Vinod ', City='Bangalore', Tier=EXECUTIVE    , Salary=₹1,65,000.00]
EmployeeRecord[ID=102, Name='Shyam ', City='Mysore   ', Tier=SENIOR_STAFF , Salary=₹98,000.00]
EmployeeRecord[ID=103, Name='Murali', City='Bangalore', Tier=SENIOR_STAFF , Salary=₹1,15,000.00]
EmployeeRecord[ID=104, Name='Naveen', City='Mysore   ', Tier=EXECUTIVE    , Salary=₹1,25,000.00]

Sorted by City ASC, then Salary DESC (Comparator):
EmployeeRecord[ID=101, Name='Vinod ', City='Bangalore', Tier=EXECUTIVE    , Salary=₹1,65,000.00]
EmployeeRecord[ID=103, Name='Murali', City='Bangalore', Tier=SENIOR_STAFF , Salary=₹1,15,000.00]
EmployeeRecord[ID=104, Name='Naveen', City='Mysore   ', Tier=EXECUTIVE    , Salary=₹1,25,000.00]
EmployeeRecord[ID=102, Name='Shyam ', City='Mysore   ', Tier=SENIOR_STAFF , Salary=₹98,000.00]
```

---

## 6. Self-Study & Review Questions
1. Can an Enum extend another class? Can an Enum implement multiple interfaces? Explain the design reasons.
2. How does the JVM serialize Enum constants? Why is Enum considered the safest singleton pattern according to Joshua Bloch?
3. What is the internal performance difference between `Integer.parseInt("100")` and `Integer.valueOf("100")`?
