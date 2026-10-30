# Session 6: Strings

- **Course:** Java Programming (PGCP-BDA)
- **Base Package:** `co.vinod`
- **Duration / Allocated Hours:** 2T + 2L + 2SL (2 Hours Theory + 2 Hours Lab + 2 Hours Self-Learning)
- **Programming Standard:** Java 17 (LTS)

---

## 1. Learning Objectives
By the end of this session, students will be able to:
1. Deconstruct the internal memory architecture of `java.lang.String` and understand Java 9+ **Compact Strings** (`byte[]` with latin1/UTF-16 encoding).
2. Explain the architectural rationale for String immutability regarding security, caching, synchronization, and the String Constant Pool (SCP).
3. Differentiate between String literals, `new String()`, reference equality (`==`), and content equality (`.equals()`).
4. Master the `intern()` method and observe pool memory management.
5. Contrast `String`, `StringBuilder`, and `StringBuffer` in performance benchmarks and thread safety.
6. Design and apply Regular Expressions (`java.util.regex`) to build data validators and text statistics analyzers.

---

## 2. String Architecture and Immutability

### 2.1 What is a String in Java?
In Java, a `String` is not a primitive array of characters; it is an instance of the class `java.lang.String`.
Starting with **Java 9**, Java adopted **Compact Strings**:
- Previously: stored as `char[]` (where every character consumed 2 bytes, 16 bits).
- Now: stored as `byte[]` accompanied by a `coder` flag (Latin-1 = 1 byte per char; UTF-16 = 2 bytes per char). This reduces heap consumption by up to 50% for standard Western text.

### 2.2 Why is `String` Immutable in Java?
Once constructed, the contents of a `String` instance cannot be modified. Any method that appears to modify a String (e.g., `toLowerCase()`, `concat()`, `replace()`) creates and returns a brand-new `String` object on the heap.

```
                           Why String is Immutable
                                      |
         +-----------------+----------+----------+-----------------+
         |                 |                     |                 |
+--------v-------+  +------v-------+      +------v-------+  +------v-------+
| String Constant|  |   Security   |      | Thread Safety|  |   HashCode   |
|   Pool (SCP)   |  |   Integrity  |      |  Concurrency |  |   Caching    |
+----------------+  +--------------+      +--------------+  +--------------+
(Permits sharing    (Protects DB   |      (Immutable     |  (Calculated    |
 identical strings  passwords,     |      objects are    |  once and cached|
 across threads     network sockets|      inherently     |  drastically    |
 saving huge heap)  from mutation) |      thread-safe)   |  speeds HashMaps)
```

---

## 3. The String Constant Pool (SCP) & `intern()`

### 3.1 String Literals vs `new String()`

```java
String s1 = "Bangalore";               // Stored in String Constant Pool
String s2 = "Bangalore";               // Reuses same instance from SCP (s1 == s2 is TRUE)
String s3 = new String("Bangalore");   // Creates new object in general Heap (s1 == s3 is FALSE)
String s4 = s3.intern();               // Retrieves canonical reference from SCP (s1 == s4 is TRUE)
```

```
STACK MEMORY                             HEAP MEMORY
+--------------+                         +-----------------------------------------------+
| s1: 0x1000   | ------------+           |  String Constant Pool (SCP)                   |
| s2: 0x1000   | ------------+---------> |  +--------------------+                       |
|              |                         |  | 0x1000: "Bangalore"|                       |
| s4: 0x1000   | ------------+           |  +--------------------+                       |
|              |                         |                                               |
| s3: 0x2500   | ----------------------> |  General Heap Area                            |
+--------------+                         |  +-----------------------------------------+  |
                                         |  | 0x2500: String Object (points to 0x1000)|  |
                                         |  +-----------------------------------------+  |
                                         +-----------------------------------------------+
```

---

## 4. `String` vs `StringBuilder` vs `StringBuffer`

| Metric | `java.lang.String` | `java.lang.StringBuilder` | `java.lang.StringBuffer` |
|---|---|---|---|
| **Mutability** | Immutable | Mutable | Mutable |
| **Thread Safety** | Thread-Safe (Immutable) | **Not Thread-Safe** | **Thread-Safe** (`synchronized`) |
| **Speed / Performance** | Slow for concatenation (creates garbage) | **Fastest** (No lock overhead) | Slower due to synchronization |
| **Recommended Use** | Constants, Keys, Domain Fields | Single-threaded text building | Multi-threaded shared buffers |

---

## 5. Regular Expressions in Java (`java.util.regex`)

Java provides `Pattern` and `Matcher` classes for regex pattern matching:
- `Pattern.compile(regex)`: Compiles the regex into an immutable pattern.
- `pattern.matcher(input)`: Generates an engine to match against candidate strings.

### Key Syntax:
- `^` and `$`: Start and end of line
- `\d`: Any digit `[0-9]`
- `\w`: Word character `[a-zA-Z_0-9]`
- `+`: One or more occurrences
- `{n,m}`: Between `n` and `m` repetitions

---

## 6. Comprehensive Lab Assignment: Text Analyzer & Regex Validator

### 6.1 Problem Statement
In package `co.vinod.strings`, implement:
1. `StringPoolDemo`: Verify memory references (`==` vs `.equals()`) and demonstrate `intern()`.
2. `TextAnalyzer`: Accepts a passage, parses words, sentences, vowels, consonants, digits, and special characters.
3. `RegexValidator`: Validates:
   - Corporate Email: `user@vinod.co` or `user@domain.com`
   - Karnataka Mobile Number: `+91` or `0` followed by 10 digits starting with `6`, `7`, `8`, or `9`.
   - Employee Code: Format `EMP-BLR-1001` or `EMP-MYS-1002`.

### 6.2 Implementation Source Code

#### File: `src/co/vinod/strings/TextAnalyzerApp.java`
```java
package co.vinod.strings;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * Text Analysis and String Constant Pool Verification
 * Package: co.vinod.strings
 * Author: Vinod Kumar (Bangalore)
 */
public class TextAnalyzerApp {

    public static void main(String[] args) {
        System.out.println("==================================================================");
        System.out.println("    CDAC BDA Java 17 String Architecture & Regex Analyzer        ");
        System.out.println("==================================================================
");

        // 1. String Constant Pool Verification
        demonstrateStringPool();

        // 2. Performance Benchmark: String vs StringBuilder
        benchmarkStringPerformance();

        // 3. Text Statistical Analyzer
        String sampleText = "Vinod Kumar conducts Big Data Analytics and Java 17 training in Bangalore! " +
                            "Shyam leads the Mysore center. Over 1,500 students enrolled in 2026; " +
                            "contact support@vinod.co for admissions.";
        analyzeText(sampleText);

        // 4. Enterprise Regex Validation
        validateEnterpriseInputs();
    }

    private static void demonstrateStringPool() {
        System.out.println("--- 1. String Constant Pool & intern() Mechanics ---");
        String city1 = "Bangalore";
        String city2 = "Bangalore";
        String city3 = new String("Bangalore");
        String city4 = city3.intern();

        System.out.println("city1 == city2 (Both literals from SCP)    : " + (city1 == city2));      // true
        System.out.println("city1 == city3 (Literal vs new Heap obj)   : " + (city1 == city3));      // false
        System.out.println("city1.equals(city3) (Content comparison)   : " + city1.equals(city3));   // true
        System.out.println("city1 == city4 (Literal vs interned obj)   : " + (city1 == city4));      // true
        System.out.println();
    }

    private static void benchmarkStringPerformance() {
        System.out.println("--- 2. Performance: String vs StringBuilder (50,000 Appends) ---");
        int iterations = 50_000;

        // String Concatenation
        long startStr = System.currentTimeMillis();
        String str = "";
        for (int i = 0; i < iterations; i++) {
            str += "A";
        }
        long timeStr = System.currentTimeMillis() - startStr;
        System.out.printf("String (+) Concatenation Time       : %d ms%n", timeStr);

        // StringBuilder
        long startSb = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < iterations; i++) {
            sb.append("A");
        }
        long timeSb = System.currentTimeMillis() - startSb;
        System.out.printf("StringBuilder (append) Time         : %d ms%n%n", timeSb);
    }

    private static void analyzeText(String text) {
        System.out.println("--- 3. Text Statistical Analyzer ---");
        System.out.println("Source Text:
"" + text + ""
");

        int vowels = 0, consonants = 0, digits = 0, spaces = 0, specials = 0;

        for (char ch : text.toCharArray()) {
            if (Character.isLetter(ch)) {
                char lower = Character.toLowerCase(ch);
                if (lower == 'a' || lower == 'e' || lower == 'i' || lower == 'o' || lower == 'u') {
                    vowels++;
                } else {
                    consonants++;
                }
            } else if (Character.isDigit(ch)) {
                digits++;
            } else if (Character.isWhitespace(ch)) {
                spaces++;
            } else {
                specials++;
            }
        }

        String[] words = text.split("\s+");
        String[] sentences = text.split("[.!?]+");

        System.out.println("Total Characters : " + text.length());
        System.out.println("Total Words      : " + words.length);
        System.out.println("Total Sentences  : " + sentences.length);
        System.out.println("Vowels Count     : " + vowels);
        System.out.println("Consonants Count : " + consonants);
        System.out.println("Digits Count     : " + digits);
        System.out.println("Spaces Count     : " + spaces);
        System.out.println("Special Chars    : " + specials);
        System.out.println();
    }

    private static void validateEnterpriseInputs() {
        System.out.println("--- 4. Enterprise Regex Pattern Validation ---");

        Pattern emailPattern = Pattern.compile("^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}$");
        Pattern phonePattern = Pattern.compile("^(?:\+91|0)?[6-9]\d{9}$");
        Pattern empCodePattern = Pattern.compile("^EMP-(BLR|MYS)-\d{4}$");

        String[] testEmails = {"vinod@vinod.co", "shyam.mysore@cdac.in", "invalid-email@", "murali@bangalore"};
        for (String email : testEmails) {
            System.out.printf("Email '%-24s' -> Valid: %b%n", email, emailPattern.matcher(email).matches());
        }

        System.out.println();
        String[] testPhones = {"+919845012345", "0802345678", "9845098765", "1234567890"};
        for (String phone : testPhones) {
            System.out.printf("Phone '%-15s' -> Valid: %b%n", phone, phonePattern.matcher(phone).matches());
        }

        System.out.println();
        String[] testEmpCodes = {"EMP-BLR-1001", "EMP-MYS-1002", "EMP-DEL-5000", "BLR-1001"};
        for (String code : testEmpCodes) {
            System.out.printf("Emp Code '%-14s' -> Valid: %b%n", code, empCodePattern.matcher(code).matches());
        }
    }
}
```

### 6.3 Expected Execution Output
```text
==================================================================
    CDAC BDA Java 17 String Architecture & Regex Analyzer        
==================================================================

--- 1. String Constant Pool & intern() Mechanics ---
city1 == city2 (Both literals from SCP)    : true
city1 == city3 (Literal vs new Heap obj)   : false
city1.equals(city3) (Content comparison)   : true
city1 == city4 (Literal vs interned obj)   : true

--- 2. Performance: String vs StringBuilder (50,000 Appends) ---
String (+) Concatenation Time       : 1240 ms
StringBuilder (append) Time         : 2 ms

--- 3. Text Statistical Analyzer ---
Source Text:
"Vinod Kumar conducts Big Data Analytics and Java 17 training in Bangalore! Shyam leads the Mysore center. Over 1,500 students enrolled in 2026; contact support@vinod.co for admissions."

Total Characters : 180
Total Words      : 27
Total Sentences  : 3
Vowels Count     : 53
Consonants Count : 87
Digits Count     : 5
Spaces Count     : 26
Special Chars    : 9

--- 4. Enterprise Regex Pattern Validation ---
Email 'vinod@vinod.co          ' -> Valid: true
Email 'shyam.mysore@cdac.in    ' -> Valid: true
Email 'invalid-email@          ' -> Valid: false
Email 'murali@bangalore        ' -> Valid: false

Phone '+919845012345   ' -> Valid: true
Phone '0802345678      ' -> Valid: false
Phone '9845098765      ' -> Valid: true
Phone '1234567890      ' -> Valid: false

Emp Code 'EMP-BLR-1001  ' -> Valid: true
Emp Code 'EMP-MYS-1002  ' -> Valid: true
Emp Code 'EMP-DEL-5000  ' -> Valid: false
Emp Code 'BLR-1001      ' -> Valid: false
```

---

## 7. Self-Study & Review Questions
1. How does the JVM handle string concatenation with `+` under the hood in Java 9+? Explain `invokedynamic` with `StringConcatFactory`.
2. Why is storing sensitive credentials like passwords in a `char[]` strongly recommended over `java.lang.String`?
3. What is the fundamental difference between `Matcher.find()` and `Matcher.matches()`?
