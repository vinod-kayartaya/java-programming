# Session 15: File Handling

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 2T + 2L (2 Hours Theory + 2 Hours Lab)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Master the architecture of the `java.io` package: Byte Streams vs Character Streams.
2. Utilize buffered streams (`BufferedReader`, `BufferedWriter`) for performant text file processing.
3. Apply modern Java NIO.2 (`java.nio.file.Path`, `Files`) for concise file manipulation.
4. Implement Object Persistence using the `java.io.Serializable` marker interface.
5. Prevent version mismatch bugs using `serialVersionUID`.
6. Protect confidential domain attributes using the `transient` keyword.
7. Engineer a backup and restore subsystem for domain entity collections.

---

## 2. Java I/O Architecture: Byte Streams vs Character Streams

The `java.io` package is divided into two distinct hierarchies based on data representation:

```
                            java.io Stream Hierarchies
                                        |
            +---------------------------+---------------------------+
            |                                                       |
     Byte Streams (8-bit)                            Character Streams (16-bit)
  (Raw Binary: Images, Audio, PDF)                     (Unicode Text: UTF-8, CSV)
            |                                                       |
    +-------+-------+                                       +-------+-------+
    |               |                                       |               |
InputStream    OutputStream                               Reader          Writer
    |               |                                       |               |
FileInputStream FileOutputStream                        FileReader      FileWriter
BufferedInStream BufferedOutStream                      BufferedReader  BufferedWriter
ObjectInStream  ObjectOutStream                         StringReader    PrintWriter
```

### 2.1 The Try-With-Resources Pattern (`AutoCloseable`)
Introduced in Java 7, `try-with-resources` guarantees that any resource implementing `java.lang.AutoCloseable` is automatically closed at the end of the statement, eliminating file handle leaks:

```java
try (BufferedReader reader = Files.newBufferedReader(Path.of("employees.csv"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        processRow(line);
    }
} // 'reader' is automatically closed here!
```

---

## 3. Modern Java NIO.2 (`java.nio.file`)

Java NIO.2 (New I/O) provides concise, non-blocking, and high-performance file management:
```java
Path path = Path.of("data/report.txt");

// Writing text directly
Files.writeString(path, "Big Data Analytics Class - Vinod Tech Bangalore");

// Reading all content into a String
String content = Files.readString(path);

// Reading line-by-line as a Stream
try (Stream<String> lines = Files.lines(path)) {
    lines.filter(l -> l.contains("Bangalore")).forEach(System.out::println);
}
```

---

## 4. Object Serialization & The `transient` Keyword

### 4.1 What is Serialization?
**Serialization** is the process of translating an in-memory object graph into a portable sequence of bytes for file storage, database caching, or network transmission across distributed nodes. **Deserialization** reconstructs the byte stream back into a live heap object.

### 4.2 The Role of `serialVersionUID`
A `static final long serialVersionUID` acts as a version fingerprint. If a class definition changes (e.g. adding fields) without updating this ID, deserializing old data throws `InvalidClassException`.

### 4.3 The `transient` Keyword
Fields declared `transient` **are omitted from the serialized byte stream**. When deserialized, transient fields receive their default values (`null`, `0`, or `false`). This is essential for:
- Sensitive credentials (passwords, PINs, bank accounts).
- Ephemeral fields (cached values, thread handles, socket descriptors).

---

## 5. Comprehensive Lab Assignment: Entity Backup & Restore Subsystem

### 5.1 Problem Statement
In package `co.vinod.io`:
1. Design `EmployeeBackupEntity` implementing `Serializable`:
   - Attributes: `id`, `name`, `city`, `salary`, `transient String ssnPin`.
2. Implement `BackupService`:
   - `void backupToFile(List<EmployeeBackupEntity> list, String filePath)` using `ObjectOutputStream`.
   - `List<EmployeeBackupEntity> restoreFromFile(String filePath)` using `ObjectInputStream`.
3. In `co.vinod.io.BackupApp`:
   - Serialize employees (Vinod in Bangalore, Shyam in Mysore, Murali in Bangalore).
   - Deserialize the binary file.
   - Verify that data fields restore perfectly while `ssnPin` is safely reset to `null`.

### 5.2 Implementation Source Code

#### File: `src/co/vinod/io/EmployeeBackupEntity.java`
```java
package co.vinod.io;

import java.io.Serial;
import java.io.Serializable;

public class EmployeeBackupEntity implements Serializable {

    @Serial
    private static final long serialVersionUID = 1L;

    private final int id;
    private final String name;
    private final String city;
    private final double salary;

    // SENSITIVE FIELD: Omitted from binary serialization!
    private final transient String ssnPin;

    public EmployeeBackupEntity(int id, String name, String city, double salary, String ssnPin) {
        this.id = id;
        this.name = name;
        this.city = city;
        this.salary = salary;
        this.ssnPin = ssnPin;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getCity() { return city; }
    public double getSalary() { return salary; }
    public String getSsnPin() { return ssnPin; }

    @Override
    public String toString() {
        return String.format("EmployeeBackupEntity[ID=%d, Name='%-6s', City='%-9s', Salary=₹%,.2f, SSN_PIN='%s']",
                id, name, city, salary, ssnPin);
    }
}
```

#### File: `src/co/vinod/io/BackupService.java`
```java
package co.vinod.io;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

public class BackupService {

    public void backupToFile(List<EmployeeBackupEntity> list, String filePath) throws IOException {
        Path path = Path.of(filePath);
        if (path.getParent() != null && !Files.exists(path.getParent())) {
            Files.createDirectories(path.getParent());
        }

        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(filePath))) {
            oos.writeObject(list);
            System.out.println("[SUCCESS] Backed up " + list.size() + " records to: " + filePath);
        }
    }

    @SuppressWarnings("unchecked")
    public List<EmployeeBackupEntity> restoreFromFile(String filePath) throws IOException, ClassNotFoundException {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(filePath))) {
            List<EmployeeBackupEntity> list = (List<EmployeeBackupEntity>) ois.readObject();
            System.out.println("[SUCCESS] Restored " + list.size() + " records from: " + filePath);
            return list;
        }
    }
}
```

#### File: `src/co/vinod/io/BackupApp.java`
```java
package co.vinod.io;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.ArrayList;
import java.util.List;

/**
 * Driver demonstrating Object Persistence and Transient Fields
 * Package: co.vinod.io
 * Author: Vinod Kumar (Bangalore)
 */
public class BackupApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("     CDAC BDA Enterprise File Handling & Serialization Lab        ");
        System.out.println("==================================================================
");

        String backupFile = "target/backups/employees_archive.dat";
        BackupService backupService = new BackupService();

        // 1. Create Source Data
        List<EmployeeBackupEntity> originalStaff = new ArrayList<>();
        originalStaff.add(new EmployeeBackupEntity(101, "Vinod", "Bangalore", 165000.0, "PIN-BLR-9876"));
        originalStaff.add(new EmployeeBackupEntity(102, "Shyam", "Mysore", 98000.0, "PIN-MYS-4321"));
        originalStaff.add(new EmployeeBackupEntity(103, "Murali", "Bangalore", 125000.0, "PIN-BLR-5544"));

        System.out.println("--- Original Workforce Records Before Serialization ---");
        originalStaff.forEach(System.out::println);
        System.out.println();

        // 2. Perform Binary Serialization to Disk
        try {
            backupService.backupToFile(originalStaff, backupFile);
            System.out.printf("File size on disk: %d bytes%n%n", Files.size(Path.of(backupFile)));
        } catch (IOException e) {
            System.err.println("Backup failed: " + e.getMessage());
            return;
        }

        // 3. Perform Binary Deserialization from Disk
        try {
            System.out.println("--- Restored Workforce Records After Deserialization ---");
            List<EmployeeBackupEntity> restoredStaff = backupService.restoreFromFile(backupFile);
            restoredStaff.forEach(System.out::println);

            System.out.println("
Security Verification:");
            System.out.println("Notice that 'SSN_PIN' is null for all restored records because it was marked transient!");
        } catch (IOException | ClassNotFoundException e) {
            System.err.println("Restore failed: " + e.getMessage());
        }
    }
}
```

### 5.3 Expected Execution Output
```text
==================================================================
     CDAC BDA Enterprise File Handling & Serialization Lab        
==================================================================

--- Original Workforce Records Before Serialization ---
EmployeeBackupEntity[ID=101, Name='Vinod ', City='Bangalore', Salary=₹1,65,000.00, SSN_PIN='PIN-BLR-9876']
EmployeeBackupEntity[ID=102, Name='Shyam ', City='Mysore   ', Salary=₹98,000.00, SSN_PIN='PIN-MYS-4321']
EmployeeBackupEntity[ID=103, Name='Murali', City='Bangalore', Salary=₹1,25,000.00, SSN_PIN='PIN-BLR-5544']

[SUCCESS] Backed up 3 records to: target/backups/employees_archive.dat
File size on disk: 348 bytes

--- Restored Workforce Records After Deserialization ---
[SUCCESS] Restored 3 records from: target/backups/employees_archive.dat
EmployeeBackupEntity[ID=101, Name='Vinod ', City='Bangalore', Salary=₹1,65,000.00, SSN_PIN='null']
EmployeeBackupEntity[ID=102, Name='Shyam ', City='Mysore   ', Salary=₹98,000.00, SSN_PIN='null']
EmployeeBackupEntity[ID=103, Name='Murali', City='Bangalore', Salary=₹1,25,000.00, SSN_PIN='null']

Security Verification:
Notice that 'SSN_PIN' is null for all restored records because it was marked transient!
```

---

## 6. Self-Study & Review Questions
1. What occurs if an object graph contains references to non-serializable classes during `ObjectOutputStream.writeObject()`? How does `NotSerializableException` manifest?
2. Compare standard I/O streams with Java NIO Channels and ByteBuffers. Where does NIO provide major throughput benefits?
3. How can customized serialization be achieved using `writeObject()` and `readObject()` private methods or the `Externalizable` interface?
