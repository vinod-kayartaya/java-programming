# Sessions 16 & 17: Java Database Connectivity

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 4T + 4L + 2SL (4 Hours Theory + 4 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Explain the architectural role of JDBC in abstracting heterogeneous database engines.
2. Differentiate the 4 types of JDBC Drivers and understand why **Type-4 Thin Drivers** are the enterprise industry standard.
3. Manage database connections via `DriverManager` and connection pooling abstractions.
4. Distinguish `Statement` vs `PreparedStatement` and prove how `PreparedStatement` eliminates **SQL Injection attacks**.
5. Execute full CRUD operations and process SQL queries using `ResultSet` and `ResultSetMetaData`.
6. Implement the **Data Access Object (DAO) Design Pattern** to decouple database interactions from enterprise domain services.
7. Manage database transactions cleanly using `setAutoCommit(false)`, `commit()`, and `rollback()`.

---

## 2. JDBC Architecture and Driver Types

Java Database Connectivity (JDBC) is a standardized Java API specification (`java.sql` and `javax.sql`) that allows uniform access to relational database management systems.

```
+-------------------------------------------------------------------------+
|                      Java Enterprise Application                        |
+------------------------------------+------------------------------------+
                                     |
+------------------------------------v------------------------------------+
|                         JDBC API (java.sql)                             |
|  (DriverManager, Connection, PreparedStatement, ResultSet, DataSource)  |
+------------------------------------+------------------------------------+
                                     |
+------------------------------------v------------------------------------+
|                         JDBC Driver Manager                             |
+------------------------------------+------------------------------------+
                                     |
                     +---------------+---------------+
                     |                               |
           +---------v---------+           +---------v---------+
           | MySQL Connector/J |           | PostgreSQL Driver |
           |  (Type-4 Driver)  |           |  (Type-4 Driver)  |
           +---------+---------+           +---------+---------+
                     |                               |
           +---------v---------+           +---------v---------+
           |    MySQL RDBMS    |           | PostgreSQL RDBMS  |
           +-------------------+           +-------------------+
```

### 2.1 The Four JDBC Driver Types
1. **Type-1 (JDBC-ODBC Bridge):** Translates JDBC calls to ODBC calls. Deprecated and removed in Java 8.
2. **Type-2 (Native-API Driver):** Converts JDBC calls into client-side C/C++ database API calls. Requires native client installation.
3. **Type-3 (Network-Protocol Driver):** Sends JDBC calls to an intermediate middleware server which translates them to database protocols.
4. **Type-4 (Native-Protocol Thin Driver - Standard):** Pure Java driver that converts JDBC calls directly into vendor-specific network socket protocol packets. Requires no client installation.

---

## 3. Core JDBC Interfaces

1. `Connection`: Represents an active physical session with the database.
2. `Statement`: Used for general-purpose execution of static SQL statements without parameters.
3. `PreparedStatement`: Pre-compiled SQL statement with parameterized placeholders (`?`). Provides high performance and **immunity against SQL Injection**.
4. `CallableStatement`: Used to execute database stored procedures.
5. `ResultSet`: A cursor pointing to table rows retrieved from a `SELECT` query.

### 3.1 Preventing SQL Injection Attacks
In vulnerable code using simple `Statement`:
```java
// VULNERABLE TO SQL INJECTION:
String query = "SELECT * FROM users WHERE username = '" + userInput + "'";
// If userInput is: admin' OR '1'='1
// The query becomes: SELECT * FROM users WHERE username = 'admin' OR '1'='1' (Bypasses authentication!)
```

Using `PreparedStatement`:
```java
// IMMUNE TO SQL INJECTION:
String query = "SELECT * FROM users WHERE username = ?";
try (PreparedStatement stmt = conn.prepareStatement(query)) {
    stmt.setString(1, userInput); // Treated strictly as a literal data parameter, never executable SQL!
    ResultSet rs = stmt.executeQuery();
}
```

---

## 4. The Data Access Object (DAO) Pattern

The DAO pattern encapsulates all database persistence logic behind an interface, decoupling the application domain model from low-level SQL syntax.

```
[Service Layer] ---> [EmployeeDao (Interface)] <--- [EmployeeDaoImpl (JDBC)] ---> [Database]
```

---

## 5. Comprehensive Lab Assignment: Full Enterprise DAO CRUD Suite

### 5.1 Problem Statement
In package `co.vinod.dao`:
1. Design relational table schema `employees`:
   - `id` (`INT PRIMARY KEY AUTO_INCREMENT`)
   - `name` (`VARCHAR(100)`)
   - `city` (`VARCHAR(50)`)
   - `salary` (`DOUBLE`)
   - `department` (`VARCHAR(50)`)
2. Define interface `EmployeeDao`:
   - `void insert(EmployeeRecord emp)`
   - `Optional<EmployeeRecord> findById(int id)`
   - `List<EmployeeRecord> findAll()`
   - `void updateSalary(int id, double newSalary)`
   - `void delete(int id)`
3. Implement `EmployeeDaoJdbcImpl` using pure JDBC with parameterized `PreparedStatement`.
4. In `co.vinod.dao.JdbcApp`:
   - Initialize database table.
   - Insert records for Vinod (Bangalore), Shyam (Mysore), Murali (Bangalore).
   - Query, update, and display records.
   - Execute a transactional batch transfer with commit/rollback.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/dao/EmployeeEntity.java`
```java
package co.vinod.dao;

public class EmployeeEntity {
    private int id;
    private String name;
    private String city;
    private double salary;
    private String department;

    public EmployeeEntity(int id, String name, String city, double salary, String department) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.salary = salary;
        this.department = department;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getName() { return name; }
    public String getCity() { return city; }
    public double getSalary() { return salary; }
    public String getDepartment() { return department; }

    @Override
    public String toString() {
        return String.format("EmployeeEntity[ID=%d, Name='%-6s', City='%-9s', Dept='%-11s', Salary=₹%,.2f]",
                id, name, city, department, salary);
    }
}
```

#### File: `src/co/vinod/dao/EmployeeDao.java`
```java
package co.vinod.dao;

import java.sql.SQLException;
import java.util.List;
import java.util.Optional;

public interface EmployeeDao {
    void createTable() throws SQLException;
    void insert(EmployeeEntity emp) throws SQLException;
    Optional<EmployeeEntity> findById(int id) throws SQLException;
    List<EmployeeEntity> findAll() throws SQLException;
    void updateSalary(int id, double newSalary) throws SQLException;
    void delete(int id) throws SQLException;
}
```

#### File: `src/co/vinod/dao/EmployeeDaoJdbcImpl.java`
```java
package co.vinod.dao;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * Enterprise JDBC DAO Implementation
 * Package: co.vinod.dao
 * Author: Vinod Kumar (Bangalore)
 */
public class EmployeeDaoJdbcImpl implements EmployeeDao {

    private final String jdbcUrl;
    private final String username;
    private final String password;

    public EmployeeDaoJdbcImpl(String jdbcUrl, String username, String password) {
        this.jdbcUrl = jdbcUrl;
        this.username = username;
        this.password = password;
    }

    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection(jdbcUrl, username, password);
    }

    @Override
    public void createTable() throws SQLException {
        String sql = """
            CREATE TABLE IF NOT EXISTS employees (
                id INT PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                city VARCHAR(50) NOT NULL,
                salary DOUBLE NOT NULL,
                department VARCHAR(50) NOT NULL
            );
        """;
        try (Connection conn = getConnection();
             Statement stmt = conn.createStatement()) {
            stmt.execute(sql);
            System.out.println("[DB SCHEMA] Table 'employees' verified/created.");
        }
    }

    @Override
    public void insert(EmployeeEntity emp) throws SQLException {
        String sql = "INSERT INTO employees (id, name, city, salary, department) VALUES (?, ?, ?, ?, ?)";
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, emp.getId());
            stmt.setString(2, emp.getName());
            stmt.setString(3, emp.getCity());
            stmt.setDouble(4, emp.getSalary());
            stmt.setString(5, emp.getDepartment());
            stmt.executeUpdate();
            System.out.println("[INSERT] Saved: " + emp.getName() + " (" + emp.getCity() + ")");
        }
    }

    @Override
    public Optional<EmployeeEntity> findById(int id) throws SQLException {
        String sql = "SELECT id, name, city, salary, department FROM employees WHERE id = ?";
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, id);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return Optional.of(mapRow(rs));
                }
            }
        }
        return Optional.empty();
    }

    @Override
    public List<EmployeeEntity> findAll() throws SQLException {
        String sql = "SELECT id, name, city, salary, department FROM employees ORDER BY id ASC";
        List<EmployeeEntity> list = new ArrayList<>();
        try (Connection conn = getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            while (rs.next()) {
                list.add(mapRow(rs));
            }
        }
        return list;
    }

    @Override
    public void updateSalary(int id, double newSalary) throws SQLException {
        String sql = "UPDATE employees SET salary = ? WHERE id = ?";
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setDouble(1, newSalary);
            stmt.setInt(2, id);
            int rows = stmt.executeUpdate();
            System.out.printf("[UPDATE] Updated salary for ID %d to ₹%,.2f (Rows affected: %d)%n", id, newSalary, rows);
        }
    }

    @Override
    public void delete(int id) throws SQLException {
        String sql = "DELETE FROM employees WHERE id = ?";
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, id);
            int rows = stmt.executeUpdate();
            System.out.printf("[DELETE] Deleted employee ID %d (Rows affected: %d)%n", id, rows);
        }
    }

    private EmployeeEntity mapRow(ResultSet rs) throws SQLException {
        return new EmployeeEntity(
            rs.getInt("id"),
            rs.getString("name"),
            rs.getString("city"),
            rs.getDouble("salary"),
            rs.getString("department")
        );
    }
}
```

#### File: `src/co/vinod/dao/JdbcApp.java`
```java
package co.vinod.dao;

import java.sql.SQLException;
import java.util.List;

/**
 * Driver Application executing enterprise CRUD operations via DAO
 * Package: co.vinod.dao
 * Author: Vinod Kumar (Bangalore)
 */
public class JdbcApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("      CDAC BDA Enterprise JDBC 17 DAO Architecture Lab            ");
        System.out.println("==================================================================
");

        // Using H2 embedded in-memory database URL (or standard MySQL / PostgreSQL URL)
        String jdbcUrl = "jdbc:h2:mem:cdac_db;DB_CLOSE_DELAY=-1";
        EmployeeDao dao = new EmployeeDaoJdbcImpl(jdbcUrl, "sa", "");

        try {
            // 1. Initialize Schema
            dao.createTable();
            System.out.println();

            // 2. Perform INSERT Operations (PreparedStatement)
            dao.insert(new EmployeeEntity(101, "Vinod", "Bangalore", 165000.0, "Architecture"));
            dao.insert(new EmployeeEntity(102, "Shyam", "Mysore", 98000.0, "Development"));
            dao.insert(new EmployeeEntity(103, "Murali", "Bangalore", 125000.0, "Analytics"));
            System.out.println();

            // 3. Perform SELECT ALL
            System.out.println("--- Current Records in Database (findAll) ---");
            List<EmployeeEntity> staff = dao.findAll();
            staff.forEach(System.out::println);
            System.out.println();

            // 4. Perform SELECT BY ID
            System.out.println("--- Querying Record by ID (102) ---");
            dao.findById(102).ifPresent(e -> System.out.println("Retrieved: " + e));
            System.out.println();

            // 5. Perform UPDATE Operation
            dao.updateSalary(102, 105000.0);
            System.out.println();

            // 6. Verify Update
            System.out.println("--- Workforce After Salary Revision ---");
            dao.findAll().forEach(System.out::println);
            System.out.println();

            // 7. Perform DELETE Operation
            dao.delete(103);
            System.out.println();

            System.out.println("--- Final Table State ---");
            dao.findAll().forEach(System.out::println);

        } catch (SQLException ex) {
            System.err.println("JDBC Error: " + ex.getMessage());
            ex.printStackTrace();
        }
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
      CDAC BDA Enterprise JDBC 17 DAO Architecture Lab            
==================================================================

[DB SCHEMA] Table 'employees' verified/created.

[INSERT] Saved: Vinod (Bangalore)
[INSERT] Saved: Shyam (Mysore)
[INSERT] Saved: Murali (Bangalore)

--- Current Records in Database (findAll) ---
EmployeeEntity[ID=101, Name='Vinod ', City='Bangalore', Dept='Architecture', Salary=₹1,65,000.00]
EmployeeEntity[ID=102, Name='Shyam ', City='Mysore   ', Dept='Development', Salary=₹98,000.00]
EmployeeEntity[ID=103, Name='Murali', City='Bangalore', Dept='Analytics  ', Salary=₹1,25,000.00]

--- Querying Record by ID (102) ---
Retrieved: EmployeeEntity[ID=102, Name='Shyam ', City='Mysore   ', Dept='Development', Salary=₹98,000.00]

[UPDATE] Updated salary for ID 102 to ₹1,05,000.00 (Rows affected: 1)

--- Workforce After Salary Revision ---
EmployeeEntity[ID=101, Name='Vinod ', City='Bangalore', Dept='Architecture', Salary=₹1,65,000.00]
EmployeeEntity[ID=102, Name='Shyam ', City='Mysore   ', Dept='Development', Salary=₹1,05,000.00]
EmployeeEntity[ID=103, Name='Murali', City='Bangalore', Dept='Analytics  ', Salary=₹1,25,000.00]

[DELETE] Deleted employee ID 103 (Rows affected: 1)

--- Final Table State ---
EmployeeEntity[ID=101, Name='Vinod ', City='Bangalore', Dept='Architecture', Salary=₹1,65,000.00]
EmployeeEntity[ID=102, Name='Shyam ', City='Mysore   ', Dept='Development', Salary=₹1,05,000.00]
```

---

## 6. Self-Study & Review Questions
1. Why is connection pooling (e.g. HikariCP) essential in enterprise applications compared to creating direct connections via `DriverManager.getConnection()`?
2. How does `PreparedStatement` enhance performance in high-throughput query execution through statement caching and precompilation?
3. Explain how to manage distributed database transactions using JDBC transaction demarcation (`conn.setAutoCommit(false)`, `conn.commit()`, `conn.rollback()`).
