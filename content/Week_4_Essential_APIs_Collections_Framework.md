# Week 4: Essential APIs & The Collections Framework

## 🎯 Learning Objectives
By the end of this week, you will:
- Master the Java Time API for date and time operations
- Use Optional effectively to handle null values safely
- Understand Collections Framework architecture deeply
- Work with Sequenced Collections (Java 21+)
- Choose the right collection for performance needs
- Understand HashMap internals (buckets, collisions, resizing)
- Write null-safe, efficient collection code
- Crack technical interviews on collections and APIs

---

# Part 1: The Time API (java.time)

## 1.1 Why java.time? (Problems with old Date/Calendar)

### The Old Way - Problems with Date and Calendar

```java
import java.util.Date;
import java.util.Calendar;

/**
 * LEGACY APPROACH - DO NOT USE IN NEW CODE
 * Multiple problems with Date and Calendar
 */
public class OldDateProblems {
    public static void main(String[] args) {
        // Problem 1: Mutable - not thread-safe
        Date date = new Date();
        date.setTime(System.currentTimeMillis());  // Can be modified!
        
        // Problem 2: Poor API design
        Date oldDate = new Date(2024, 1, 30);  // ❌ Year is 1900-based!
        // This creates: 3924-02-30 (year 2024+1900, month is 0-based)
        
        // Problem 3: Month is 0-based (confusing!)
        Calendar cal = Calendar.getInstance();
        cal.set(2024, 0, 30);  // January is 0, not 1!
        
        // Problem 4: Not type-safe
        // No distinction between date, time, and timestamp
        
        // Problem 5: Poor timezone handling
        // Problem 6: Not immutable - thread safety issues
    }
}
```

### The Modern Way - java.time (Java 8+)

**Key Benefits:**
- **Immutable and thread-safe**
- **Clear, fluent API**
- **Separate classes for different use cases**
- **Better timezone handling**
- **ISO-8601 standard compliant**

---

## 1.2 Core Classes Overview

### The Main Classes

```
java.time package:
├── LocalDate      → Date without time (2024-01-30)
├── LocalTime      → Time without date (14:30:45)
├── LocalDateTime  → Date + Time without timezone
├── ZonedDateTime  → Date + Time + Timezone
├── Instant        → Machine timestamp (epoch seconds)
├── Duration       → Time-based amount (hours, minutes, seconds)
└── Period         → Date-based amount (years, months, days)
```

---

## 1.3 LocalDate - Date Without Time

### Creating LocalDate

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.DayOfWeek;

/**
 * LocalDate - Represents a date (year-month-day)
 * Immutable and thread-safe
 */
public class LocalDateExamples {
    public static void main(String[] args) {
        
        // Creating LocalDate
        
        // 1. Current date
        LocalDate today = LocalDate.now();
        System.out.println("Today: " + today);  // 2024-01-30
        
        // 2. Specific date
        LocalDate specificDate = LocalDate.of(2024, 1, 30);
        System.out.println("Specific: " + specificDate);
        
        // 3. Using Month enum (more readable)
        LocalDate withEnum = LocalDate.of(2024, Month.JANUARY, 30);
        System.out.println("With enum: " + withEnum);
        
        // 4. Parse from string (ISO-8601 format: yyyy-MM-dd)
        LocalDate parsed = LocalDate.parse("2024-01-30");
        System.out.println("Parsed: " + parsed);
        
        // 5. Year and day of year
        LocalDate fromDayOfYear = LocalDate.ofYearDay(2024, 30);  // 30th day of 2024
        System.out.println("Day 30 of 2024: " + fromDayOfYear);
    }
}
```

### Getting Information from LocalDate

```java
public class LocalDateGetters {
    public static void main(String[] args) {
        LocalDate date = LocalDate.of(2024, 1, 30);
        
        // Get components
        int year = date.getYear();                    // 2024
        Month month = date.getMonth();                // JANUARY
        int monthValue = date.getMonthValue();        // 1
        int dayOfMonth = date.getDayOfMonth();        // 30
        int dayOfYear = date.getDayOfYear();          // 30
        DayOfWeek dayOfWeek = date.getDayOfWeek();    // TUESDAY
        
        System.out.println("Year: " + year);
        System.out.println("Month: " + month);
        System.out.println("Month Value: " + monthValue);
        System.out.println("Day of Month: " + dayOfMonth);
        System.out.println("Day of Year: " + dayOfYear);
        System.out.println("Day of Week: " + dayOfWeek);
        
        // Checking properties
        boolean isLeapYear = date.isLeapYear();
        int lengthOfMonth = date.lengthOfMonth();     // 31 for January
        int lengthOfYear = date.lengthOfYear();       // 365 or 366
        
        System.out.println("Leap year? " + isLeapYear);
        System.out.println("Days in month: " + lengthOfMonth);
        System.out.println("Days in year: " + lengthOfYear);
    }
}
```

### Manipulating LocalDate

```java
/**
 * LocalDate is IMMUTABLE - all methods return NEW objects
 */
public class LocalDateManipulation {
    public static void main(String[] args) {
        LocalDate date = LocalDate.of(2024, 1, 30);
        
        // Adding/Subtracting
        LocalDate tomorrow = date.plusDays(1);           // 2024-01-31
        LocalDate nextWeek = date.plusWeeks(1);          // 2024-02-06
        LocalDate nextMonth = date.plusMonths(1);        // 2024-02-29 (leap year!)
        LocalDate nextYear = date.plusYears(1);          // 2025-01-30
        
        LocalDate yesterday = date.minusDays(1);         // 2024-01-29
        LocalDate lastWeek = date.minusWeeks(1);         // 2024-01-23
        LocalDate lastMonth = date.minusMonths(1);       // 2023-12-30
        LocalDate lastYear = date.minusYears(1);         // 2023-01-30
        
        System.out.println("Original: " + date);         // Still 2024-01-30
        System.out.println("Tomorrow: " + tomorrow);
        
        // Replacing components
        LocalDate sameMonthDifferentDay = date.withDayOfMonth(15);  // 2024-01-15
        LocalDate sameYearDifferentMonth = date.withMonth(6);        // 2024-06-30
        LocalDate differentYear = date.withYear(2025);               // 2025-01-30
        
        // First/Last day of month
        LocalDate firstDay = date.withDayOfMonth(1);                 // 2024-01-01
        LocalDate lastDay = date.withDayOfMonth(date.lengthOfMonth()); // 2024-01-31
        
        System.out.println("First day: " + firstDay);
        System.out.println("Last day: " + lastDay);
    }
}
```

### Comparing LocalDate

```java
public class LocalDateComparison {
    public static void main(String[] args) {
        LocalDate date1 = LocalDate.of(2024, 1, 30);
        LocalDate date2 = LocalDate.of(2024, 2, 15);
        LocalDate date3 = LocalDate.of(2024, 1, 30);
        
        // Comparison methods
        boolean isBefore = date1.isBefore(date2);       // true
        boolean isAfter = date1.isAfter(date2);         // false
        boolean isEqual = date1.isEqual(date3);         // true
        
        // Using compareTo (returns -1, 0, or 1)
        int comparison = date1.compareTo(date2);        // negative (date1 < date2)
        
        // Using equals
        boolean equals = date1.equals(date3);           // true
        
        System.out.println("date1 before date2? " + isBefore);
        System.out.println("date1 equals date3? " + isEqual);
        
        // Checking ranges
        LocalDate start = LocalDate.of(2024, 1, 1);
        LocalDate end = LocalDate.of(2024, 12, 31);
        LocalDate check = LocalDate.of(2024, 6, 15);
        
        boolean inRange = !check.isBefore(start) && !check.isAfter(end);
        System.out.println("Date in range? " + inRange);
    }
}
```

---

## 1.4 LocalTime - Time Without Date

### Creating and Using LocalTime

```java
import java.time.LocalTime;

/**
 * LocalTime - Represents time (hour:minute:second:nanosecond)
 * No date or timezone information
 */
public class LocalTimeExamples {
    public static void main(String[] args) {
        
        // Creating LocalTime
        LocalTime now = LocalTime.now();                        // Current time
        LocalTime specific = LocalTime.of(14, 30);              // 14:30:00
        LocalTime withSeconds = LocalTime.of(14, 30, 45);       // 14:30:45
        LocalTime withNanos = LocalTime.of(14, 30, 45, 123456789); // With nanoseconds
        
        LocalTime parsed = LocalTime.parse("14:30:45");         // From string
        
        // Getting components
        int hour = specific.getHour();           // 14
        int minute = specific.getMinute();       // 30
        int second = specific.getSecond();       // 0
        int nano = specific.getNano();           // 0
        
        System.out.println("Hour: " + hour);
        System.out.println("Minute: " + minute);
        
        // Manipulating time
        LocalTime plus2Hours = specific.plusHours(2);         // 16:30:00
        LocalTime plus30Mins = specific.plusMinutes(30);      // 15:00:00
        LocalTime minus1Hour = specific.minusHours(1);        // 13:30:00
        
        // Wraps around at midnight
        LocalTime nearMidnight = LocalTime.of(23, 30);
        LocalTime afterMidnight = nearMidnight.plusHours(1);  // 00:30:00
        
        System.out.println("Near midnight: " + nearMidnight);
        System.out.println("After midnight: " + afterMidnight);
        
        // Comparison
        LocalTime morning = LocalTime.of(9, 0);
        LocalTime afternoon = LocalTime.of(15, 0);
        
        boolean isBefore = morning.isBefore(afternoon);       // true
        
        // Special times
        LocalTime midnight = LocalTime.MIDNIGHT;              // 00:00:00
        LocalTime noon = LocalTime.NOON;                      // 12:00:00
        LocalTime max = LocalTime.MAX;                        // 23:59:59.999999999
        LocalTime min = LocalTime.MIN;                        // 00:00:00
    }
}
```

---

## 1.5 LocalDateTime - Date and Time Combined

### Creating and Using LocalDateTime

```java
import java.time.LocalDateTime;

/**
 * LocalDateTime - Combines LocalDate and LocalTime
 * No timezone information
 */
public class LocalDateTimeExamples {
    public static void main(String[] args) {
        
        // Creating LocalDateTime
        LocalDateTime now = LocalDateTime.now();
        
        LocalDateTime specific = LocalDateTime.of(2024, 1, 30, 14, 30);
        LocalDateTime withSeconds = LocalDateTime.of(2024, 1, 30, 14, 30, 45);
        
        // Combining LocalDate and LocalTime
        LocalDate date = LocalDate.of(2024, 1, 30);
        LocalTime time = LocalTime.of(14, 30);
        LocalDateTime combined = LocalDateTime.of(date, time);
        
        // Parsing
        LocalDateTime parsed = LocalDateTime.parse("2024-01-30T14:30:45");
        
        System.out.println("Now: " + now);
        System.out.println("Specific: " + specific);
        System.out.println("Combined: " + combined);
        
        // Extracting date and time
        LocalDate dateOnly = combined.toLocalDate();
        LocalTime timeOnly = combined.toLocalTime();
        
        // Manipulating
        LocalDateTime later = combined.plusDays(5).plusHours(3);
        LocalDateTime earlier = combined.minusWeeks(2);
        
        // All LocalDate and LocalTime methods available
        LocalDateTime modified = combined
            .withYear(2025)
            .withMonth(6)
            .withDayOfMonth(15)
            .withHour(10)
            .withMinute(0);
        
        System.out.println("Modified: " + modified);  // 2025-06-15T10:00
    }
}
```

---

## 1.6 ZonedDateTime - Date, Time, and Timezone

### Understanding Timezones

```java
import java.time.ZonedDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.util.Set;

/**
 * ZonedDateTime - Date, time with timezone information
 * Handles daylight saving time (DST)
 */
public class ZonedDateTimeExamples {
    public static void main(String[] args) {
        
        // Getting available zones
        Set<String> zones = ZoneId.getAvailableZoneIds();
        System.out.println("Total zones: " + zones.size());
        
        // Creating ZonedDateTime
        ZonedDateTime now = ZonedDateTime.now();
        System.out.println("\nCurrent time in system timezone: " + now);
        
        // Specific timezone
        ZonedDateTime nyTime = ZonedDateTime.now(ZoneId.of("America/New_York"));
        ZonedDateTime tokyoTime = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));
        ZonedDateTime londonTime = ZonedDateTime.now(ZoneId.of("Europe/London"));
        
        System.out.println("New York: " + nyTime);
        System.out.println("Tokyo: " + tokyoTime);
        System.out.println("London: " + londonTime);
        
        // From LocalDateTime
        LocalDateTime ldt = LocalDateTime.of(2024, 1, 30, 14, 30);
        ZonedDateTime zdtIndia = ldt.atZone(ZoneId.of("Asia/Kolkata"));
        
        System.out.println("In India: " + zdtIndia);
        
        // Converting between timezones
        ZonedDateTime indiaTime = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));
        ZonedDateTime usTime = indiaTime.withZoneSameInstant(ZoneId.of("America/New_York"));
        
        System.out.println("\nIndia time: " + indiaTime);
        System.out.println("Same instant in US: " + usTime);
    }
}
```

---

## 1.7 Instant - Machine Time

### Understanding Instant

```java
import java.time.Instant;
import java.time.Duration;

/**
 * Instant - Point on timeline (epoch seconds since 1970-01-01T00:00:00Z)
 * Machine-readable timestamp
 * Always in UTC
 */
public class InstantExamples {
    public static void main(String[] args) {
        
        // Current instant
        Instant now = Instant.now();
        System.out.println("Current instant: " + now);
        
        // Epoch time
        long epochSeconds = now.getEpochSecond();
        System.out.println("Epoch seconds: " + epochSeconds);
        
        // From epoch
        Instant fromEpoch = Instant.ofEpochSecond(1706630400L);
        Instant fromMillis = Instant.ofEpochMilli(System.currentTimeMillis());
        
        // Constants
        Instant epoch = Instant.EPOCH;  // 1970-01-01T00:00:00Z
        System.out.println("Epoch: " + epoch);
        
        // Arithmetic
        Instant future = now.plusSeconds(3600);      // 1 hour later
        Instant past = now.minusSeconds(86400);      // 1 day ago
        
        // Comparison
        boolean isBefore = past.isBefore(now);       // true
        boolean isAfter = future.isAfter(now);       // true
        
        // Converting between Instant and ZonedDateTime
        ZonedDateTime zdt = now.atZone(ZoneId.of("Asia/Kolkata"));
        Instant backToInstant = zdt.toInstant();
        
        System.out.println("As ZonedDateTime: " + zdt);
        System.out.println("Back to Instant: " + backToInstant);
        
        // Duration between instants
        Instant start = Instant.now();
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
        Instant end = Instant.now();
        
        Duration duration = Duration.between(start, end);
        System.out.println("Time taken: " + duration.toMillis() + " ms");
    }
}
```

---

## 1.8 Duration - Time-Based Amount

### Working with Duration

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;

/**
 * Duration - Time-based amount (hours, minutes, seconds, nanos)
 */
public class DurationExamples {
    public static void main(String[] args) {
        
        // Creating Duration
        Duration oneHour = Duration.ofHours(1);
        Duration thirtyMinutes = Duration.ofMinutes(30);
        Duration fiveSeconds = Duration.ofSeconds(5);
        
        // Complex durations
        Duration complex = Duration.ofHours(2).plusMinutes(30).plusSeconds(45);
        System.out.println("Complex duration: " + complex);  // PT2H30M45S
        
        // Between two times
        LocalTime start = LocalTime.of(9, 0);
        LocalTime end = LocalTime.of(17, 30);
        Duration workDay = Duration.between(start, end);
        
        System.out.println("Work day: " + workDay);          // PT8H30M
        System.out.println("Hours: " + workDay.toHours());   // 8
        System.out.println("Minutes: " + workDay.toMinutes()); // 510
        
        // Extracting components
        long hours = complex.toHours();                      // 2
        long minutes = complex.toMinutes();                  // 150 (total)
        long seconds = complex.getSeconds();                 // 9045 (total)
        
        // Getting parts
        long hoursPart = complex.toHoursPart();              // 2
        int minutesPart = complex.toMinutesPart();           // 30
        int secondsPart = complex.toSecondsPart();           // 45
        
        System.out.println("Hours part: " + hoursPart);
        System.out.println("Minutes part: " + minutesPart);
        System.out.println("Seconds part: " + secondsPart);
    }
}
```

---

## 1.9 Period - Date-Based Amount

### Working with Period

```java
import java.time.Period;

/**
 * Period - Date-based amount (years, months, days)
 */
public class PeriodExamples {
    public static void main(String[] args) {
        
        // Creating Period
        Period oneYear = Period.ofYears(1);
        Period sixMonths = Period.ofMonths(6);
        Period tenDays = Period.ofDays(10);
        
        // Complex period
        Period complex = Period.of(2, 6, 15);  // 2 years, 6 months, 15 days
        System.out.println("Complex period: " + complex);  // P2Y6M15D
        
        // Between two dates
        LocalDate birthDate = LocalDate.of(1990, 5, 15);
        LocalDate today = LocalDate.now();
        Period age = Period.between(birthDate, today);
        
        System.out.println("Age: " + age);
        System.out.println("Years: " + age.getYears());
        System.out.println("Months: " + age.getMonths());
        System.out.println("Days: " + age.getDays());
    }
}
```

---

## 1.10 Formatting and Parsing

### DateTimeFormatter

```java
import java.time.format.DateTimeFormatter;
import java.time.format.FormatStyle;

/**
 * Formatting and parsing dates/times
 */
public class FormattingExamples {
    public static void main(String[] args) {
        LocalDateTime dateTime = LocalDateTime.of(2024, 1, 30, 14, 30, 45);
        
        // Predefined formatters
        String iso = dateTime.format(DateTimeFormatter.ISO_DATE_TIME);
        System.out.println("ISO: " + iso);
        
        // Custom patterns
        DateTimeFormatter custom1 = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
        String formatted1 = dateTime.format(custom1);
        System.out.println("Custom 1: " + formatted1);  // 30/01/2024 14:30:45
        
        DateTimeFormatter custom2 = DateTimeFormatter.ofPattern("EEEE, MMMM dd, yyyy 'at' hh:mm a");
        String formatted2 = dateTime.format(custom2);
        System.out.println("Custom 2: " + formatted2);  // Tuesday, January 30, 2024 at 02:30 PM
        
        // Parsing
        String dateString = "30/01/2024 14:30:45";
        LocalDateTime parsed = LocalDateTime.parse(dateString, custom1);
        System.out.println("Parsed: " + parsed);
    }
}
```

---

# Part 2: Null Safety with Optional

## 2.1 The Null Problem

### Why Null is Dangerous

```java
/**
 * The billion-dollar mistake - Tony Hoare
 */
public class NullProblems {
    public static void main(String[] args) {
        // Problem 1: NullPointerException
        String name = getName();
        // int length = name.length();  // NPE if name is null!
        
        // Problem 2: Defensive null checks everywhere
        if (name != null) {
            int length = name.length();
        }
        
        // Problem 3: Chaining makes it worse
        // String city = user.getAddress().getCity();  // Multiple NPE risks!
    }
    
    public static String getName() {
        return null;  // Unclear - error or valid?
    }
}
```

---

## 2.2 Optional - The Solution

### What is Optional?

```java
import java.util.Optional;

/**
 * Optional<T> - Container that may or may not contain a value
 */
public class OptionalBasics {
    public static void main(String[] args) {
        
        // Creating Optional
        Optional<String> empty = Optional.empty();
        Optional<String> withValue = Optional.of("Hello");
        Optional<String> nullable = Optional.ofNullable(null);
        
        // Checking if value is present
        boolean hasValue = withValue.isPresent();      // true
        boolean isEmpty = empty.isEmpty();              // true (Java 11+)
        
        System.out.println("Has value: " + hasValue);
        System.out.println("Is empty: " + isEmpty);
    }
}
```

---

## 2.3 Extracting Values from Optional

```java
/**
 * Getting values from Optional
 */
public class OptionalExtraction {
    public static void main(String[] args) {
        Optional<String> optional = Optional.of("Hello");
        Optional<String> empty = Optional.empty();
        
        // Method 1: orElse() - provide default value
        String result1 = empty.orElse("Default");
        System.out.println("orElse(): " + result1);  // Default
        
        // Method 2: orElseGet() - lazy evaluation
        String result2 = empty.orElseGet(() -> {
            System.out.println("Computing default...");
            return "Computed Default";
        });
        
        // Method 3: orElseThrow() - throw custom exception
        try {
            String result3 = empty.orElseThrow(() -> 
                new IllegalStateException("Value not found")
            );
        } catch (IllegalStateException e) {
            System.out.println("Exception: " + e.getMessage());
        }
        
        // Method 4: ifPresent - execute if value exists
        optional.ifPresent(value -> 
            System.out.println("Value: " + value)
        );
    }
}
```

---

## 2.4 Transforming Optional Values

### map() and flatMap()

```java
/**
 * Transform values inside Optional
 */
public class OptionalTransform {
    public static void main(String[] args) {
        Optional<String> name = Optional.of("john");
        
        // map() - Transform the value
        Optional<String> upperCase = name.map(String::toUpperCase);
        System.out.println(upperCase.get());  // JOHN
        
        // Chain transformations
        Optional<Integer> length = name
            .map(String::toUpperCase)
            .map(String::length);
        System.out.println("Length: " + length.get());  // 4
        
        // flatMap() - Avoid nested Optionals
        Optional<User> userOpt = Optional.of(new User("john@email.com"));
        Optional<String> email = userOpt.flatMap(User::getEmail);
        email.ifPresent(System.out::println);
    }
    
    static class User {
        private String email;
        User(String email) { this.email = email; }
        Optional<String> getEmail() { return Optional.ofNullable(email); }
    }
}
```

---

## 2.5 Filtering Optional Values

```java
/**
 * Filter values in Optional
 */
public class OptionalFilter {
    public static void main(String[] args) {
        Optional<String> name = Optional.of("John");
        
        // Filter based on predicate
        Optional<String> filtered = name.filter(n -> n.length() > 3);
        System.out.println("Filtered (length > 3): " + filtered.isPresent());  // true
        
        // Chain filter, map, and flatMap
        Optional<User> user = findUser(123);
        Optional<String> result = user
            .filter(u -> u.getAge() >= 18)
            .map(User::getEmail)
            .map(String::toLowerCase)
            .filter(email -> email.endsWith("@company.com"));
    }
    
    static class User {
        private int age;
        private String email;
        int getAge() { return age; }
        String getEmail() { return email; }
    }
    
    static Optional<User> findUser(int id) {
        return Optional.empty();
    }
}
```

---

## 2.6 Optional Best Practices

```java
/**
 * Optional best practices
 */
public class OptionalBestPractices {
    
    // ✅ DO: Use Optional as return type
    public Optional<User> findUserById(int id) {
        return Optional.ofNullable(database.get(id));
    }
    
    // ❌ DON'T: Use Optional as parameter
    public void processUser(Optional<User> user) {  // BAD!
    }
    
    // ✅ DO: Use overloading instead
    public void processUser(User user) {
    }
    
    // ✅ DO: Use orElseGet for expensive defaults
    public String getUserName(int id) {
        return findUserById(id)
            .map(User::getName)
            .orElseGet(() -> computeExpensiveDefault());
    }
    
    // ❌ DON'T: Store Optional in fields
    public class BadClass {
        private Optional<String> name;  // BAD!
    }
    
    // ✅ DO: Use null for fields
    public class GoodClass {
        private String name;  // Can be null
    }
    
    // ❌ DON'T: Use Optional with collections
    public Optional<List<User>> getUsers() {  // BAD!
        return Optional.of(Collections.emptyList());
    }
    
    // ✅ DO: Return empty collection
    public List<User> getUsers2() {
        return Collections.emptyList();
    }
    
    private Map<Integer, User> database = new HashMap<>();
    private String computeExpensiveDefault() { return "default"; }
    
    static class User {
        String getName() { return ""; }
    }
}
```

---

# Part 3: Collections Framework

## 3.1 Collections Framework Overview

### The Hierarchy

```
Collection (Interface)
├── List - Ordered, allows duplicates
│   ├── ArrayList
│   ├── LinkedList
│   └── Vector (Legacy)
│
├── Set - No duplicates
│   ├── HashSet
│   ├── LinkedHashSet
│   └── TreeSet (Sorted)
│
└── Queue - FIFO operations
    ├── PriorityQueue
    └── Deque
        └── ArrayDeque

Map (Interface) - Key-value pairs
├── HashMap
├── LinkedHashMap
└── TreeMap (Sorted)
```

---

## 3.2 List Interface

### ArrayList - Dynamic Array

```java
import java.util.ArrayList;
import java.util.List;

/**
 * ArrayList - Resizable array
 * Fast random access O(1), slow insertions/deletions O(n)
 */
public class ArrayListExamples {
    public static void main(String[] args) {
        
        List<String> list = new ArrayList<>();
        
        // Adding elements
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");
        
        // Add at specific index
        list.add(1, "Blueberry");
        
        // Accessing elements
        String first = list.get(0);              // "Apple"
        
        // Size
        int size = list.size();
        
        // Checking existence
        boolean hasApple = list.contains("Apple");     // true
        
        // Finding index
        int index = list.indexOf("Banana");
        
        // Modifying
        list.set(0, "Avocado");                        // Replace
        
        // Removing
        list.remove(0);                                // By index
        list.remove("Cherry");                         // By object
        
        // Bulk operations
        List<String> more = List.of("Mango", "Orange");
        list.addAll(more);
        
        System.out.println("List: " + list);
    }
}
```

### ArrayList Internal Working

```java
/**
 * ArrayList internals:
 * - Backed by array
 * - Initial capacity: 10 (default)
 * - Growth: 50% when full (10 → 15 → 22)
 * 
 * Performance:
 * - get(i): O(1)
 * - add(element): O(1) amortized
 * - add(index, element): O(n)
 * - remove(index): O(n)
 * - contains(object): O(n)
 */
public class ArrayListInternals {
    public static void main(String[] args) {
        // With specific capacity
        List<String> list = new ArrayList<>(100);
        
        // Efficient: adding at end
        for (int i = 0; i < 100000; i++) {
            list.add("Item" + i);  // O(1)
        }
        
        // Slow: adding at beginning
        for (int i = 0; i < 1000; i++) {
            list.add(0, "Item" + i);  // O(n) - shifts elements
        }
    }
}
```

### LinkedList - Doubly Linked List

```java
import java.util.LinkedList;

/**
 * LinkedList - Doubly-linked list
 * Fast insertions/deletions O(1), slow random access O(n)
 */
public class LinkedListExamples {
    public static void main(String[] args) {
        
        LinkedList<String> list = new LinkedList<>();
        
        list.add("A");
        list.add("B");
        list.add("C");
        
        // Additional operations
        list.addFirst("Start");
        list.addLast("End");
        
        // Getting first/last
        String first = list.getFirst();
        String last = list.getLast();
        
        // Removing first/last
        String removedFirst = list.removeFirst();
        String removedLast = list.removeLast();
        
        System.out.println("List: " + list);
    }
}
```

---

## 3.3 Sequenced Collections (Java 21+)

### New Methods

```java
import java.util.ArrayList;
import java.util.List;

/**
 * Sequenced Collections - Java 21+
 */
public class SequencedCollections {
    public static void main(String[] args) {
        
        List<String> list = new ArrayList<>(List.of("A", "B", "C", "D", "E"));
        
        // NEW: getFirst()
        String first = list.getFirst();
        System.out.println("First: " + first);  // A
        
        // NEW: getLast()
        String last = list.getLast();
        System.out.println("Last: " + last);    // E
        
        // NEW: removeFirst()
        String removedFirst = list.removeFirst();
        System.out.println("List: " + list);    // [B, C, D, E]
        
        // NEW: removeLast()
        String removedLast = list.removeLast();
        System.out.println("List: " + list);    // [B, C, D]
        
        // NEW: addFirst(element)
        list.addFirst("Start");
        System.out.println("After addFirst: " + list);  // [Start, B, C, D]
        
        // NEW: addLast(element)
        list.addLast("End");
        System.out.println("After addLast: " + list);   // [Start, B, C, D, End]
        
        // NEW: reversed() - returns reversed view
        List<String> reversedView = list.reversed();
        System.out.println("Reversed view: " + reversedView);
    }
}
```

---

## 3.4 Set Interface

### HashSet - Hash Table

```java
import java.util.HashSet;
import java.util.Set;

/**
 * HashSet - No duplicates, no ordering
 * O(1) for add, remove, contains
 */
public class HashSetExamples {
    public static void main(String[] args) {
        
        Set<String> set = new HashSet<>();
        
        set.add("Apple");
        set.add("Banana");
        set.add("Cherry");
        set.add("Apple");      // Duplicate - ignored!
        
        System.out.println("Set: " + set);  // [Apple, Banana, Cherry]
        System.out.println("Size: " + set.size());  // 3
        
        // Checking existence
        boolean hasApple = set.contains("Apple");    // true - O(1)
        
        // Removing
        set.remove("Banana");
        
        // Set operations
        Set<String> set2 = Set.of("Cherry", "Date", "Elderberry");
        
        // Union
        Set<String> union = new HashSet<>(set);
        union.addAll(set2);
        System.out.println("Union: " + union);
        
        // Intersection
        Set<String> intersection = new HashSet<>(set);
        intersection.retainAll(set2);
        System.out.println("Intersection: " + intersection);
    }
}
```

### TreeSet - Sorted Set

```java
import java.util.TreeSet;
import java.util.Set;

/**
 * TreeSet - Sorted, no duplicates
 * O(log n) for add, remove, contains
 */
public class TreeSetExamples {
    public static void main(String[] args) {
        
        Set<Integer> set = new TreeSet<>();
        
        set.add(5);
        set.add(2);
        set.add(8);
        set.add(1);
        
        System.out.println("TreeSet: " + set);  // [1, 2, 5, 8] - sorted!
        
        TreeSet<Integer> treeSet = new TreeSet<>(set);
        
        Integer first = treeSet.first();          // 1
        Integer last = treeSet.last();            // 8
        
        // Range operations
        Set<Integer> headSet = treeSet.headSet(5);     // < 5
        Set<Integer> tailSet = treeSet.tailSet(5);     // >= 5
        
        System.out.println("Head set: " + headSet);
        System.out.println("Tail set: " + tailSet);
    }
}
```

---

## 3.5 Map Interface

### HashMap - Key-Value Pairs

```java
import java.util.HashMap;
import java.util.Map;

/**
 * HashMap - Key-value pairs
 * O(1) for get, put, remove
 */
public class HashMapExamples {
    public static void main(String[] args) {
        
        Map<String, Integer> map = new HashMap<>();
        
        // Adding entries
        map.put("Alice", 25);
        map.put("Bob", 30);
        map.put("Charlie", 35);
        
        // Getting values
        Integer age = map.get("Alice");            // 25
        Integer missing = map.get("David");        // null
        
        // Default value if missing
        Integer ageOrDefault = map.getOrDefault("David", 0);
        
        // Checking existence
        boolean hasAlice = map.containsKey("Alice");
        boolean has25 = map.containsValue(25);
        
        // Removing
        Integer removed = map.remove("Bob");
        
        // Update if exists
        map.replace("Alice", 26);
        
        // Put if absent
        map.putIfAbsent("David", 40);
        
        // Compute methods
        map.compute("Alice", (key, value) -> value + 1);
        map.computeIfPresent("Charlie", (k, v) -> v + 5);
        map.computeIfAbsent("Eve", k -> 22);
        
        // Merge (useful for counting)
        map.merge("Alice", 1, Integer::sum);
        
        System.out.println("Map: " + map);
    }
}
```

### Iterating Maps

```java
/**
 * Different ways to iterate maps
 */
public class MapIteration {
    public static void main(String[] args) {
        Map<String, Integer> map = Map.of(
            "Alice", 25,
            "Bob", 30,
            "Charlie", 35
        );
        
        // Method 1: Entry set (most efficient)
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
        
        // Method 2: forEach (Java 8+)
        map.forEach((key, value) -> 
            System.out.println(key + " -> " + value)
        );
    }
}
```

---

## 3.6 HashMap Internals - Deep Dive

### How HashMap Works

```java
/**
 * HashMap internals:
 * 
 * Structure:
 * - Array of buckets (nodes)
 * - Each bucket: Single node, Linked list, or Red-Black tree
 * 
 * Process:
 * 1. Hashing: hash = key.hashCode()
 * 2. Find bucket: index = hash & (capacity - 1)
 * 3. Handle collisions:
 *    - Empty bucket: Store entry
 *    - Collision: Add to list/tree
 *    - Same key: Replace value
 */
public class HashMapInternals {
    public static void main(String[] args) {
        
        // Initial capacity: 16 (default)
        // Load factor: 0.75
        // Threshold: 12 (16 * 0.75)
        
        Map<String, Integer> map = new HashMap<>();
        
        // When size > threshold:
        // - Double capacity: 16 → 32
        // - Rehash all entries (expensive!)
        // - New threshold: 24
        
        for (int i = 0; i < 20; i++) {
            map.put("Key" + i, i);
            if (i == 11) {
                System.out.println("About to resize!");
            }
            if (i == 12) {
                System.out.println("Resized to capacity 32");
            }
        }
    }
}
```

### HashMap Best Practices

```java
/**
 * HashMap best practices
 */
public class HashMapBestPractices {
    
    // ✅ DO: Use immutable keys
    public static void goodExample() {
        Map<String, Integer> map = new HashMap<>();
        map.put("Alice", 25);
    }
    
    // ❌ DON'T: Mutate keys after insertion
    public static void badExample() {
        Map<MutableKey, String> map = new HashMap<>();
        MutableKey key = new MutableKey("Alice");
        map.put(key, "Value");
        key.name = "Bob";  // MUTATED!
    }
    
    // ✅ DO: Override equals() and hashCode() together
    static class GoodKey {
        private final String value;
        GoodKey(String value) { this.value = value; }
        
        @Override
        public boolean equals(Object obj) {
            if (!(obj instanceof GoodKey other)) return false;
            return Objects.equals(this.value, other.value);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(value);
        }
    }
    
    // ✅ DO: Specify initial capacity if size is known
    public static void withCapacity() {
        Map<String, Integer> map = new HashMap<>(1000);
    }
    
    static class MutableKey {
        String name;
        MutableKey(String name) { this.name = name; }
    }
}
```

---

## 3.7 LinkedHashMap - Insertion Order

```java
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * LinkedHashMap - Maintains insertion order
 */
public class LinkedHashMapExamples {
    public static void main(String[] args) {
        
        Map<String, Integer> map = new LinkedHashMap<>();
        
        map.put("Charlie", 35);
        map.put("Alice", 25);
        map.put("Bob", 30);
        
        // Maintains insertion order
        map.forEach((k, v) -> System.out.println(k + " -> " + v));
        // Output:
        // Charlie -> 35
        // Alice -> 25
        // Bob -> 30
    }
}
```

---

## 3.8 Performance Comparison

### Collection Performance Table

| Operation | ArrayList | LinkedList | HashSet | TreeSet | HashMap |
|-----------|-----------|------------|---------|---------|---------|
| **Add** | O(1)* | O(1) | O(1) | O(log n) | O(1) |
| **Get** | O(1) | O(n) | N/A | N/A | O(1) |
| **Contains** | O(n) | O(n) | O(1) | O(log n) | O(1) |
| **Remove** | O(n) | O(n) | O(1) | O(log n) | O(1) |

*Amortized

---

# Practice Exercises

## Exercise Set 1: Time API

### Exercise 1.1: Age Calculator
Calculate exact age from birth date (years, months, days).

### Exercise 1.2: Meeting Scheduler
Convert meeting time across 5 timezones.

### Exercise 1.3: Date Range Operations
Generate date ranges, filter weekdays, count business days.

## Exercise Set 2: Optional

### Exercise 2.1: User Service
Create methods using Optional: `findById()`, `getEmailById()`.

### Exercise 2.2: Configuration Manager
Build configuration class with Optional-based getters.

## Exercise Set 3: Collections

### Exercise 3.1: Word Frequency Counter
Count word frequencies using HashMap.

### Exercise 3.2: Unique Elements
Remove duplicates while preserving order using LinkedHashSet.

### Exercise 3.3: Student Grade Manager
Manage students with sorted TreeSet and grade mapping.

---

# Interview Questions & Answers

## Time API Questions

### Q1: Why was java.time introduced?
**Answer:** The old Date/Calendar had issues: mutable, poor API, confusing (0-based months), weak timezone support. java.time is immutable, thread-safe, clear API, and ISO-8601 compliant.

### Q2: Difference between Instant and LocalDateTime?
**Answer:** 
- **Instant**: Machine time, epoch seconds, always UTC
- **LocalDateTime**: Human time, no timezone, combines date and time

### Q3: When to use Duration vs Period?
**Answer:**
- **Duration**: Time-based (hours, minutes, seconds) - for LocalTime, Instant
- **Period**: Date-based (years, months, days) - for LocalDate

## Optional Questions

### Q4: What problem does Optional solve?
**Answer:** Makes null handling explicit, forces thinking about absence, enables functional-style null handling, self-documenting code.

### Q5: orElse vs orElseGet?
**Answer:**
- **orElse**: Always evaluates default (even if not needed)
- **orElseGet**: Lazy evaluation, only if needed. Use orElseGet for expensive defaults.

## Collections Questions

### Q6: ArrayList vs LinkedList?
**Answer:**
- **ArrayList**: Array-based, O(1) get, O(n) insert/delete in middle
- **LinkedList**: Linked nodes, O(n) get, O(1) insert/delete at ends
- **Use ArrayList** for most cases (default choice)

### Q7: How does HashMap work internally?
**Answer:**
1. Hash key using hashCode()
2. Find bucket: index = hash & (capacity - 1)
3. Handle collisions: linked list or Red-Black tree
4. Resize when size > threshold (capacity * 0.75)

### Q8: What are Sequenced Collections (Java 21)?
**Answer:** New methods for collections with defined order: `getFirst()`, `getLast()`, `removeFirst()`, `removeLast()`, `addFirst()`, `addLast()`, `reversed()`. Provides consistent API across List, Deque, etc.

---

# Week 4 Summary

## Key Takeaways

### Time API
✅ LocalDate, LocalTime, LocalDateTime for dates/times  
✅ ZonedDateTime for timezone-aware operations  
✅ Instant for machine timestamps  
✅ Duration (time-based) vs Period (date-based)  
✅ DateTimeFormatter for formatting/parsing  

### Optional
✅ Makes null handling explicit  
✅ map(), flatMap(), filter() for transformations  
✅ orElse(), orElseGet(), orElseThrow() for extraction  
✅ Best practices: return type only, not parameters/fields  

### Collections
✅ ArrayList (default), LinkedList (queue-like)  
✅ HashSet (fast), TreeSet (sorted)  
✅ HashMap internals (hashing, buckets, collisions)  
✅ Sequenced Collections (Java 21+)  
✅ Performance characteristics of each collection  

---

**End of Week 4 Material**

Ready for Week 5: Streams & Functional Programming!
