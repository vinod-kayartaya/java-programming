# Sessions 12 & 13: Java Streams

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 4T + 6L + 2SL (4 Hours Theory + 6 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Master the Stream API architecture: understand Lazy Evaluation, Pipelines, and Internal Iteration.
2. Differentiate Collections (in-memory data storage) from Streams (declarative compute pipelines).
3. Apply Intermediate operations: `filter()`, `map()`, `flatMap()`, `distinct()`, `sorted()`, `peek()`, and `limit()`.
4. Apply Terminal operations: `collect()`, `forEach()`, `reduce()`, `count()`, and matching predicates (`anyMatch()`, `allMatch()`).
5. Process high-throughput numeric data with Primitive Streams (`IntStream`, `DoubleStream`).
6. Perform multi-level aggregations and partitioning using `java.util.stream.Collectors` (`groupingBy()`, `partitioningBy()`).
7. Leverage the modern Java 8/17 Date-Time API (`java.time`) to calculate employee tenures, age, and period intervals.

---

## 2. Stream Architecture: Anatomy of a Processing Pipeline

A Java Stream is **not** a data storage structure; it is a pipeline of computational steps executed lazily over a sequence of elements.

```
+------------+       +-------------------+       +--------------------+       +--------------------+
|   Source   |  -->  |  filter() (Interm)|  -->  |   map() (Interm)   |  -->  | collect() (Terminal|
| Collection |       | (Lazy Evaluation) |       | (Lazy Evaluation)  |       | Triggers Execution)|
+------------+       +-------------------+       +--------------------+       +--------------------+
```

### 2.1 Streams vs Collections: Key Distinctions
| Feature | Collections (`List`, `Set`, `Map`) | Streams (`Stream<T>`) |
|---|---|---|
| **Primary Focus** | Data storage and memory layout | Computation, transformation, and query pipelines |
| **Iteration** | **External** (Explicit `for`, `while`, `Iterator`) | **Internal** (JVM controls iteration and parallelism) |
| **Reusability** | Fully reusable (can read indefinitely) | **Single-use only** (Consuming terminal op closes stream) |
| **Evaluation** | Eager (Constructed upfront in heap) | **Lazy** (Intermediate ops execute only on terminal op) |

---

## 3. Core Stream Operations

### 3.1 Intermediate Operations (Return `Stream<T>`)
- `filter(Predicate<T>)`: Selects elements satisfying a boolean predicate.
- `map(Function<T, R>)`: Transforms elements of type `T` into type `R`.
- `flatMap(Function<T, Stream<R>>)`: Flattens multiple streams/nested collections into a single stream.
- `sorted(Comparator<T>)`: Sorts elements according to natural or custom comparator.
- `distinct()`: Removes duplicate elements based on `equals()`.

### 3.2 Terminal Operations (Trigger Execution and Produce a Non-Stream Result)
- `collect(Collector)`: Gathers elements into collections, maps, or strings.
- `reduce(BinaryOperator)`: Performs reduction fold operations (sum, product, min, max).
- `count()`: Returns the number of elements.
- `anyMatch(Predicate)` / `allMatch(Predicate)`: Short-circuiting boolean checks.

---

## 4. Modern Java Date-Time API (`java.time`)

Java 8 introduced the `java.time` package (JSR-310), completely superseding the legacy mutable `java.util.Date` and `Calendar`:
- `LocalDate`: Date without time (`2026-10-30`).
- `LocalTime`: Time without date (`15:30:00`).
- `LocalDateTime`: Combined date and time without timezone.
- `Period`: Date-based amount of time (Years, Months, Days).
- `Duration`: Time-based amount of time (Seconds, Nanoseconds).
- `DateTimeFormatter`: Thread-safe parsing and formatting.

```java
LocalDate joined = LocalDate.of(2021, Month.AUGUST, 15);
LocalDate today = LocalDate.now();
Period tenure = Period.between(joined, today);
System.out.printf("Tenure: %d years, %d months, %d days%n", 
    tenure.getYears(), tenure.getMonths(), tenure.getDays());
```

---

## 5. Comprehensive Lab Assignment: Big Data Enterprise Stream Analytics

### 5.1 Problem Statement
In package `co.vinod.streams`:
1. Model enterprise workforce:
   - Vinod (Bangalore, Architecture, ₹1,65,000, Joined: 2018-05-10, Skills: ["Java", "Spark", "Kafka"])
   - Shyam (Mysore, Development, ₹98,000, Joined: 2021-03-15, Skills: ["Java", "Docker"])
   - Murali (Bangalore, Analytics, ₹1,25,000, Joined: 2019-11-01, Skills: ["Python", "Spark"])
   - Naveen (Mysore, Development, ₹88,000, Joined: 2022-07-20, Skills: ["Java", "SQL"])
   - Harish (Bangalore, Cloud, ₹1,40,000, Joined: 2020-01-12, Skills: ["Kubernetes", "AWS"])
2. Execute stream pipeline queries:
   - Filter Bangalore staff earning > ₹1,30,000.
   - Calculate total company salary expense and average salary.
   - Group employees by City (`Map<String, List<Staff>>`).
   - Group by City and compute average salary per city (`Map<String, Double>`).
   - Extract a **distinct set of technical skills** across all employees using `flatMap()`.
   - Calculate employee service tenure using `Period.between()`.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/streams/CorporateEmployee.java`
```java
package co.vinod.streams;

import java.time.LocalDate;
import java.time.Period;
import java.util.List;

public class CorporateEmployee {
    private final int id;
    private final String name;
    private final String city;
    private final String department;
    private final double salary;
    private final LocalDate joiningDate;
    private final List<String> technicalSkills;

    public CorporateEmployee(int id, String name, String city, String department, 
                             double salary, LocalDate joiningDate, List<String> skills) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.department = department;
        this.salary = salary;
        this.joiningDate = joiningDate;
        this.technicalSkills = skills;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCity() { return city; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public LocalDate getJoiningDate() { return joiningDate; }
    public List<String> getTechnicalSkills() { return technicalSkills; }

    public Period getTenure() {
        return Period.between(joiningDate, LocalDate.now());
    }

    @Override
    public String toString() {
        return String.format("CorporateEmployee[ID=%d, Name='%-6s', City='%-9s', Dept='%-11s', Salary=₹%,.2f]",
                id, name, city, department, salary);
    }
}
```

#### File: `src/co/vinod/streams/StreamAnalyticsApp.java`
```java
package co.vinod.streams;

import java.time.LocalDate;
import java.time.Month;
import java.time.Period;
import java.util.*;
import java.util.stream.Collectors;

/**
 * Enterprise Stream Analytics & Date-Time API Masterclass
 * Package: co.vinod.streams
 * Author: Vinod Kumar (Bangalore)
 */
public class StreamAnalyticsApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("       CDAC BDA Enterprise Java 17 Stream Analytics Suite         ");
        System.out.println("==================================================================
");

        List<CorporateEmployee> roster = initializeRoster();

        // Query 1: Filter & Sort Bangalore high earners
        queryHighEarners(roster);

        // Query 2: Aggregations (Sum, Average, Max, Min) via Primitive Streams
        computeSalaryStatistics(roster);

        // Query 3: Grouping & Multi-Level Collectors
        groupEmployeesByCity(roster);

        // Query 4: FlatMap - Extracting distinct skills
        extractDistinctSkills(roster);

        // Query 5: Date-Time API - Tenure calculations
        computeTenure(roster);
    }

    private static List<CorporateEmployee> initializeRoster() {
        return List.of(
            new CorporateEmployee(101, "Vinod", "Bangalore", "Architecture", 165000.0, 
                LocalDate.of(2018, Month.MAY, 10), List.of("Java", "Spark", "Kafka")),
            new CorporateEmployee(102, "Shyam", "Mysore", "Development", 98000.0, 
                LocalDate.of(2021, Month.MARCH, 15), List.of("Java", "Docker")),
            new CorporateEmployee(103, "Murali", "Bangalore", "Analytics", 125000.0, 
                LocalDate.of(2019, Month.NOVEMBER, 1), List.of("Python", "Spark", "SQL")),
            new CorporateEmployee(104, "Naveen", "Mysore", "Development", 88000.0, 
                LocalDate.of(2022, Month.JULY, 20), List.of("Java", "SQL")),
            new CorporateEmployee(105, "Harish", "Bangalore", "Cloud", 140000.0, 
                LocalDate.of(2020, Month.JANUARY, 12), List.of("Kubernetes", "AWS", "Docker"))
        );
    }

    private static void queryHighEarners(List<CorporateEmployee> roster) {
        System.out.println("--- 1. Query: Bangalore Staff Earning >= ₹1,30,000 ---");
        List<CorporateEmployee> highEarners = roster.stream()
            .filter(e -> "Bangalore".equalsIgnoreCase(e.getCity()))
            .filter(e -> e.getSalary() >= 130000.0)
            .sorted(Comparator.comparing(CorporateEmployee::getSalary).reversed())
            .toList();

        highEarners.forEach(System.out::println);
        System.out.println();
    }

    private static void computeSalaryStatistics(List<CorporateEmployee> roster) {
        System.out.println("--- 2. Primitive Stream Statistics: SummaryStatistics ---");
        DoubleSummaryStatistics stats = roster.stream()
            .mapToDouble(CorporateEmployee::getSalary)
            .summaryStatistics();

        System.out.printf("Total Count         : %d%n", stats.getCount());
        System.out.printf("Total Monthly Payroll: ₹%,.2f%n", stats.getSum());
        System.out.printf("Average Salary       : ₹%,.2f%n", stats.getAverage());
        System.out.printf("Max Salary           : ₹%,.2f%n", stats.getMax());
        System.out.printf("Min Salary           : ₹%,.2f%n%n", stats.getMin());
    }

    private static void groupEmployeesByCity(List<CorporateEmployee> roster) {
        System.out.println("--- 3. Collectors: Grouping by City & Average Salary ---");

        // Grouping by City
        Map<String, List<CorporateEmployee>> byCity = roster.stream()
            .collect(Collectors.groupingBy(CorporateEmployee::getCity));

        byCity.forEach((city, employees) -> {
            System.out.println("City: " + city);
            employees.forEach(e -> System.out.println("   - " + e.getName() + " (" + e.getDepartment() + ")"));
        });

        // Grouping by City and calculating Average Salary
        Map<String, Double> avgSalaryByCity = roster.stream()
            .collect(Collectors.groupingBy(CorporateEmployee::getCity, 
                     Collectors.averagingDouble(CorporateEmployee::getSalary)));

        System.out.println("
Average Salary by Location:");
        avgSalaryByCity.forEach((city, avg) -> System.out.printf("   - %-10s : ₹%,.2f%n", city, avg));
        System.out.println();
    }

    private static void extractDistinctSkills(List<CorporateEmployee> roster) {
        System.out.println("--- 4. FlatMap: Extracting All Distinct Technical Skills ---");
        List<String> distinctSkills = roster.stream()
            .flatMap(e -> e.getTechnicalSkills().stream())
            .distinct()
            .sorted()
            .toList();

        System.out.println("Unique Enterprise Tech Stack: " + String.join(", ", distinctSkills));
        System.out.println();
    }

    private static void computeTenure(List<CorporateEmployee> roster) {
        System.out.println("--- 5. Modern Date-Time API: Service Tenure Calculation ---");
        for (CorporateEmployee emp : roster) {
            Period p = emp.getTenure();
            System.out.printf("%-7s | Joined: %s | Tenure: %d years, %d months, %d days%n",
                    emp.getName(), emp.getJoiningDate(), p.getYears(), p.getMonths(), p.getDays());
        }
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
       CDAC BDA Enterprise Java 17 Stream Analytics Suite         
==================================================================

--- 1. Query: Bangalore Staff Earning >= ₹1,30,000 ---
CorporateEmployee[ID=101, Name='Vinod ', City='Bangalore', Dept='Architecture', Salary=₹1,65,000.00]
CorporateEmployee[ID=105, Name='Harish', City='Bangalore', Dept='Cloud      ', Salary=₹1,40,000.00]

--- 2. Primitive Stream Statistics: SummaryStatistics ---
Total Count         : 5
Total Monthly Payroll: ₹6,16,000.00
Average Salary       : ₹1,23,200.00
Max Salary           : ₹1,65,000.00
Min Salary           : ₹88,000.00

--- 3. Collectors: Grouping by City & Average Salary ---
City: Bangalore
   - Vinod (Architecture)
   - Murali (Analytics)
   - Harish (Cloud)
City: Mysore
   - Shyam (Development)
   - Naveen (Development)

Average Salary by Location:
   - Bangalore  : ₹1,43,333.33
   - Mysore     : ₹93,000.00

--- 4. FlatMap: Extracting All Distinct Technical Skills ---
Unique Enterprise Tech Stack: AWS, Docker, Java, Kafka, Kubernetes, Python, SQL, Spark

--- 5. Modern Date-Time API: Service Tenure Calculation ---
Vinod   | Joined: 2018-05-10 | Tenure: 8 years, 5 months, 20 days
Shyam   | Joined: 2021-03-15 | Tenure: 5 years, 7 months, 15 days
Murali  | Joined: 2019-11-01 | Tenure: 6 years, 11 months, 29 days
Naveen  | Joined: 2022-07-20 | Tenure: 4 years, 3 months, 10 days
Harish  | Joined: 2020-01-12 | Tenure: 6 years, 9 months, 18 days
```

---

## 6. Self-Study & Review Questions
1. What does "lazy evaluation" mean in the context of Java Streams? How does `Stream.peek()` prove that intermediate operations do not execute until a terminal operation is called?
2. Explain the difference between `map()` and `flatMap()`. When is `flatMap()` essential?
3. Contrast `java.time.Period` with `java.time.Duration`. Which one would you use to measure thread execution time vs employee age?
