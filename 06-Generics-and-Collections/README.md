# Sessions 7 & 8: Generics and Collections

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 4T + 4L + 4SL (4 Hours Theory + 4 Hours Lab + 4 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Master Java Generics to achieve compile-time type safety and eliminate runtime `ClassCastException`.
2. Understand Type Erasure, Generic bounds (`<T extends Number>`), and Wildcard mechanics (`? extends T`, `? super T`, PECS rule).
3. Navigate the Java Collections Framework (JCF) hierarchy (`Iterable`, `Collection`, `List`, `Set`, `Queue`, `Map`).
4. Benchmark data structures: `ArrayList` vs `LinkedList`, `HashSet` vs `TreeSet`, `HashMap` vs `TreeMap`.
5. Master the critical `equals()` and `hashCode()` contract and prevent memory leaks in Hash-based structures.
6. Design and implement a reusable generic repository `EntityManager<T>` managing domain entities.

---

## 2. Generics Fundamentals and Type Safety

### 2.1 The Problem with Legacy Raw Types (Pre-Java 5)
```java
List staff = new ArrayList();
staff.add("Vinod");
staff.add(101); // Autoboxed to Integer

for (Object obj : staff) {
    String name = (String) obj; // Crashes with ClassCastException when processing 101!
}
```

### 2.2 Compile-Time Safety with Generics
Generics allow parameters for types:
```java
List<String> staff = new ArrayList<>();
staff.add("Vinod");
// staff.add(101); // COMPILE ERROR: The compiler rejects incorrect types before execution!
```

### 2.3 Type Erasure
Java implements Generics via **Type Erasure** to maintain backward compatibility with legacy JVM bytecode:
- At compile time, the compiler inspects generic types and inserts necessary type casts.
- At runtime, the type parameter `<T>` is erased and replaced by its bound (or `Object` if unbounded).
- Consequently, `List<String>` and `List<Integer>` share the identical runtime class `java.util.ArrayList.class`.

### 2.4 Wildcards and the PECS Principle
- `<?>`: Unbounded wildcard (any type).
- `<? extends Number>`: Upper bounded wildcard (read-only producer).
- `<? super Integer>`: Lower bounded wildcard (writeable consumer).
- **PECS:** **P**roducer **E**xtends, **C**onsumer **S**uper.
  - If a collection *produces* data for you to read, use `<? extends T>`.
  - If a collection *consumes* data you write to it, use `<? super T>`.

---

## 3. Java Collections Framework (JCF) Architecture

```
                             +---------------+
                             |   Iterable    |
                             +-------+-------+
                                     |
                             +-------v-------+
                             |  Collection   |
                             +---+---+---+---+
                                 |   |   |
          +----------------------+   |   +-----------------------+
          |                          |                           |
  +-------v-------+          +-------v-------+           +-------v-------+
  |     List      |          |     Queue     |           |      Set      |
  +-------+-------+          +-------+-------+           +-------+-------+
          |                          |                           |
    +-----+-----+              +-----+-----+               +-----+-----+
    |           |              |           |               |           |
+---v---+   +---v----+     +---v---+   +---v----+      +---v---+   +---v---+
| Array |   | Linked |     | Prior |   | Array  |      | Hash  |   | Tree  |
| List  |   | List   |     | Queue |   | Deque  |      | Set   |   | Set   |
+-------+   +--------+     +-------+   +--------+      +-------+   +-------+
```

### 3.1 List: Ordered, Index-based, Allows Duplicates
- `ArrayList`: Backed by a resizable dynamic array. Fast random access $O(1)$. Slow middle insertion/deletion $O(n)$ due to element shifting.
- `LinkedList`: Doubly linked list. Fast head/tail insertion/deletion $O(1)$. Slow random access $O(n)$ requiring linear traversal.

### 3.2 Set: Unordered, Unique Elements
- `HashSet`: Backed by a `HashMap`. Constant-time $O(1)$ performance.
- `LinkedHashSet`: Preserves element insertion order via a doubly-linked bucket list.
- `TreeSet`: Backed by a Red-Black Tree. Elements sorted naturally or via a `Comparator`. $O(\log n)$ search/insertion.

### 3.3 Map: Key-Value Associations (Keys are Unique)
- `HashMap`: Array of buckets (Node<K,V>[]). Java 8+ converts buckets with $\ge 8$ colliding nodes from a LinkedList into a balanced Red-Black Tree, preventing $O(n)$ collision denial-of-service degradation.
- `TreeMap`: NavigableMap sorted by keys.

---

## 4. The `equals()` and `hashCode()` Contract

When using objects as keys in `HashMap` or elements in `HashSet`:
1. If `o1.equals(o2)` is `true`, then `o1.hashCode() == o2.hashCode()` **must strictly be true**.
2. If two objects have the same `hashCode()`, they are not necessarily equal (a **hash collision** occurs, handled via chaining/treeification).
3. If `equals()` is overridden without overriding `hashCode()`, equal objects will hash into different buckets, causing `map.get()` to return `null`!

---

## 5. Comprehensive Lab Assignment: Generic EntityManager & Benchmarking

### 5.1 Problem Statement
In package `co.vinod.collections`:
1. Implement a generic interface `Repository<T, ID>` with operations: `save(T entity)`, `findById(ID id)`, `findAll()`, `deleteById(ID id)`.
2. Implement `EntityManager<T, ID>` backed internally by a `HashMap<ID, T>`.
3. Create domain entity `Employee` in Bangalore/Mysore with `id`, `name`, `city`, `salary`, properly overriding `equals()` and `hashCode()`.
4. Implement a comprehensive benchmark driver measuring:
   - `ArrayList` vs `LinkedList` on 100,000 element random lookups.
   - `HashSet` vs `TreeSet` on lookup throughput.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/collections/Repository.java`
```java
package co.vinod.collections;

import java.util.List;
import java.util.Optional;

public interface Repository<T, ID> {
    void save(ID id, T entity);
    Optional<T> findById(ID id);
    List<T> findAll();
    boolean deleteById(ID id);
    int count();
}
```

#### File: `src/co/vinod/collections/EntityManager.java`
```java
package co.vinod.collections;

import java.util.*;

/**
 * Generic In-Memory Entity Manager
 * Package: co.vinod.collections
 * Author: Vinod Kumar (Bangalore)
 */
public class EntityManager<T, ID> implements Repository<T, ID> {

    private final Map<ID, T> storage = new HashMap<>();

    @Override
    public void save(ID id, T entity) {
        storage.put(id, entity);
    }

    @Override
    public Optional<T> findById(ID id) {
        return Optional.ofNullable(storage.get(id));
    }

    @Override
    public List<T> findAll() {
        return new ArrayList<>(storage.values());
    }

    @Override
    public boolean deleteById(ID id) {
        return storage.remove(id) != null;
    }

    @Override
    public int count() {
        return storage.size();
    }
}
```

#### File: `src/co/vinod/collections/Employee.java`
```java
package co.vinod.collections;

import java.util.Objects;

public class Employee implements Comparable<Employee> {
    private final int id;
    private final String name;
    private final String city;
    private final double salary;

    public Employee(int id, String name, String city, double salary) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.salary = salary;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCity() { return city; }
    public double getSalary() { return salary; }

    // Natural ordering by ID for TreeSet / Collections.sort
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }

    // Strict equals and hashCode contract for HashSet / HashMap
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Employee employee)) return false;
        return id == employee.id && Objects.equals(name, employee.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

    @Override
    public String toString() {
        return String.format("Employee[ID=%d, Name='%s', City='%s', Salary=₹%,.2f]", id, name, city, salary);
    }
}
```

#### File: `src/co/vinod/collections/CollectionBenchmarkApp.java`
```java
package co.vinod.collections;

import java.util.*;

/**
 * Benchmark Driver: ArrayList vs LinkedList, HashSet vs TreeSet
 * Package: co.vinod.collections
 * Author: Vinod Kumar (Bangalore)
 */
public class CollectionBenchmarkApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("     CDAC BDA Generic Collections Architecture & Benchmarks       ");
        System.out.println("==================================================================
");

        // 1. Generic Entity Manager Verification
        testEntityManager();

        // 2. Benchmark ArrayList vs LinkedList
        benchmarkLists(100_000);

        // 3. Benchmark HashSet vs TreeSet
        benchmarkSets(100_000);
    }

    private static void testEntityManager() {
        System.out.println("--- 1. Testing Generic EntityManager<Employee, Integer> ---");
        EntityManager<Employee, Integer> manager = new EntityManager<>();

        manager.save(101, new Employee(101, "Vinod", "Bangalore", 160000.0));
        manager.save(102, new Employee(102, "Shyam", "Mysore", 95000.0));
        manager.save(103, new Employee(103, "Murali", "Bangalore", 115000.0));

        System.out.println("Total entities stored: " + manager.count());
        manager.findById(101).ifPresent(e -> System.out.println("Retrieved: " + e));

        manager.deleteById(102);
        System.out.println("After deleting ID 102, count is: " + manager.count());
        System.out.println();
    }

    private static void benchmarkLists(int elements) {
        System.out.println("--- 2. Benchmarking ArrayList vs LinkedList (" + elements + " elements) ---");
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();

        for (int i = 0; i < elements; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }

        // Random Access Benchmark at mid-point (Index 50,000)
        int targetIndex = elements / 2;

        long startAl = System.nanoTime();
        int valAl = arrayList.get(targetIndex);
        long timeAl = System.nanoTime() - startAl;
        System.out.printf("ArrayList  get(%d)  : %6d ns [Constant O(1)]%n", targetIndex, timeAl);

        long startLl = System.nanoTime();
        int valLl = linkedList.get(targetIndex);
        long timeLl = System.nanoTime() - startLl;
        System.out.printf("LinkedList get(%d)  : %6d ns [Linear O(n)]%n%n", targetIndex, timeLl);
    }

    private static void benchmarkSets(int elements) {
        System.out.println("--- 3. Benchmarking HashSet vs TreeSet Lookup (" + elements + " elements) ---");
        Set<Integer> hashSet = new HashSet<>();
        Set<Integer> treeSet = new TreeSet<>();

        for (int i = 0; i < elements; i++) {
            hashSet.add(i);
            treeSet.add(i);
        }

        int target = elements - 50;

        long startHs = System.nanoTime();
        boolean hasHs = hashSet.contains(target);
        long timeHs = System.nanoTime() - startHs;
        System.out.printf("HashSet contains(%d) : %6d ns [O(1) Hash Table]%n", target, timeHs);

        long startTs = System.nanoTime();
        boolean hasTs = treeSet.contains(target);
        long timeTs = System.nanoTime() - startTs;
        System.out.printf("TreeSet contains(%d) : %6d ns [O(log n) Red-Black Tree]%n", target, timeTs);
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
     CDAC BDA Generic Collections Architecture & Benchmarks       
==================================================================

--- 1. Testing Generic EntityManager<Employee, Integer> ---
Total entities stored: 3
Retrieved: Employee[ID=101, Name='Vinod', City='Bangalore', Salary=₹1,60,000.00]
After deleting ID 102, count is: 2

--- 2. Benchmarking ArrayList vs LinkedList (100000 elements) ---
ArrayList  get(50000)  :   1800 ns [Constant O(1)]
LinkedList get(50000)  : 654200 ns [Linear O(n)]

--- 3. Benchmarking HashSet vs TreeSet Lookup (100000 elements) ---
HashSet contains(99950) :   1200 ns [O(1) Hash Table]
TreeSet contains(99950) :  18400 ns [O(log n) Red-Black Tree]
```

---

## 6. Self-Study & Review Questions
1. Why does Java forbid creating generic arrays directly, such as `T[] array = new T[10];`? How is this resolved using reflection or `Object[]`?
2. Explain the internal working of `HashMap.put(K, V)`. How are bucket collisions resolved in Java 8+?
3. What is the difference between `Comparable<T>` (natural ordering) and `Comparator<T>` (external custom ordering)?
