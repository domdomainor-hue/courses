In Java, every class inherits from `java.lang.Object`, which provides default implementations for `equals(Object)` and `hashCode()`. However, these methods must almost always be overridden to work correctly with hash-based collections (like `HashMap` or `HashSet`) and to enable logical equality checks.

Here is a detailed explanation of their purpose, the "Contract," practical examples, and common pitfalls.

---

### 1. The Core Purpose

#### **`equals(Object o)`**
*   **Purpose:** To determine if two objects are logically identical based on their content (data), not just memory location.
*   **Default Behavior:** Compares object references using the `==` operator. Returns `true` only if both variables point to the exact same instance in memory.
*   **Requirement:** When comparing data (e.g., two `String`s with "Hello"), you override this method so that `"Hello".equals("Hello")` returns true.

#### **`hashCode()`**
*   **Purpose:** To generate a unique integer value for the object based on its content. This acts as an index to store objects efficiently in collections.
*   **Default Behavior:** Returns `System.identityHashCode(this)`, which is usually unique per instance but based on memory address.
*   **Requirement:** When using `HashMap` or `HashSet`, you override this so that two logically equal objects return the same integer hash code.

---

### 2. The Contract (The Golden Rule)

Java enforces a strict contract between `equals` and `hashCode`. You cannot violate this without causing bugs.

1.  **If** two objects are equal according to `equals(Object)`, **then** they MUST have the same integer result from `hashCode()`.
2.  **Conversely**, if they do not return the same hash code, they cannot be equal.
3.  **Exception:** Two objects having the same hash code does not mean they are equal (collisions are normal).

> **Example:** If object A and B have different content, their `equals()` must return `false`, and their `hashCode()` values *can* be different (though it is okay if they happen to be the same by coincidence). However, if A and B have the same content, their `equals()` returns true, and their `hashCode()` **must** be identical.

---

### 3. Concrete Example: The `Student` Class

Let's see what happens when you implement them manually versus relying on defaults.

#### **A. Default Implementation (The Problem)**
If you do nothing:
```java
class Student {
    private String name;
    public Student(String n) { this.name = n; }
}
```

```java
Student s1 = new Student("Alice");
Student s2 = new Student("Alice");

// Reference Equality (Default equals)
System.out.println(s1 == s2);      // false (Different memory addresses)
System.out.println(s1.equals(s2)); // false (Same behavior as above)

// Hash Set Example
Set<Student> set = new HashSet<>();
set.add(s1);
set.add(s2);
System.out.println(set.size());     // 2 (Both objects added because default equals says they are different)
```
**Why?** The JVM doesn't know "Alice" and "Alice" are logically the same student. It sees them as two distinct memory references.

#### **B. Correct Implementation**
You must override both methods to ensure consistency.

```java
import java.util.Objects;

class Student {
    private String name;
    
    public Student(String name) { this.name = name; }

    @Override
    public boolean equals(Object o) {
        // Check for null and type safety first
        if (this == o) return true;         // Optimized check for same ref
        if (!(o instanceof Student)) return false; 
        
        Student student = (Student) o;      // Cast
        
        // Compare logic. Using Objects.equals handles null safely.
        return Objects.equals(this.name, student.name); 
    }

    @Override
    public int hashCode() {
        // Must be consistent with equals!
        return name != null ? Objects.hash(name) : 0;
    }
}
```

**Testing the Fixed Version:**
```java
Student s1 = new Student("Alice");
Student s2 = new Student("Alice");
Student s3 = new Student("Bob");

System.out.println(s1.equals(s2));   // true (Content match)
System.out.println(s1.hashCode());   // e.g., 5028
System.out.println(s2.hashCode());   // e.g., 5028 (Must be same as s1)

// Set Example
Set<Student> set = new HashSet<>();
set.add(s1);
set.add(s2); // s2 is ignored because it has same hash AND equals s1
System.out.println(set.size());      // 1 (Correct!)
```

---

### 4. How `HashMap` and `HashSet` Use Them Internally

When you use a hash-based collection, here is the exact process happening inside:

#### **Scenario:** Putting an object into a `HashMap<String, Integer>`

1.  **Key Hash Calculation:** The map calculates `key.hashCode()`.
2.  **Bucket Indexing:** It maps that integer to a "bucket" (index) in its internal array. If the hash is $H$ and capacity is $C$, the bucket index is often calculated using a mix function (like `(n - 1) & h`).
3.  **Collision Handling:** Often, multiple keys end up in the same bucket (e.g., key "A" and "B" might have hash codes ending in 0).
4.  **The Bucket Traversal:** Inside that specific bucket, there is a list of entries. The map iterates through this list.
5.  **The Comparison:** For every entry found in the bucket, it calls `key.equals(key_candidate)`.
6.  **Result:** If `equals()` returns `true`, the old value is replaced. If it returns `false` for all entries, a new Entry is inserted at that bucket.

#### **Why you need `hashCode`:**
Without `hashCode()`, every single element in the map would have to be checked with `equals()`. If the map has 1 million items, adding one item becomes an $O(N)$ operation. By using `hashCode()`, finding an item becomes an $O(1)$ operation (Constant time) on average.

---

### 5. Common Pitfalls and Mistakes

#### **1. Inconsistent Mutable State**
The biggest bug occurs when a field is changed after the object is put into a collection, but that field is used for `equals` or `hashCode`.

```java
class Product {
    private String name;
    // ... constructor ...

    @Override
    public int hashCode() { return name.hashCode(); } 
    @Override
    public boolean equals(Object o) {
       if (!(o instanceof Product)) return false;
       Product p = (Product) o;
       return this.name.equals(p.name);
    }
}

// Usage Error:
Product p1 = new Product("Laptop");
Map<Product, Integer> map = new HashMap<>();
map.put(p1, 1000); // Map stores hash based on "Laptop"

p1.setName("Phone"); // MODIFICATION! 

// Now we cannot find it using the old name key:
Product p2 = new Product("Laptop"); 
System.out.println(map.containsKey(p2)); // Returns FALSE!
```
**Why?** The `hashCode` was calculated for "Laptop". You changed `name` to "Phone". `p2` ("Laptop") has a different hash code and ends up in a different bucket, so it never reaches the logic where it checks if it equals `p1`.

#### **2. Implementing Only One**
You must override both. If you override `equals` but not `hashCode`:
*   `HashMap` might find an entry based on reference (if it doesn't know `equals` overrides), or worse, `HashSet` will treat identical objects as unique duplicates because their default `hashCode` is different.

#### **3. Ignoring Transient Fields**
If you have a field marked `transient` (like a session ID stored in memory), do not use it in `hashCode()`. The hash code changes when the object is deserialized from disk, which breaks the contract if the class was serialized and then loaded into a Map.

---

### 6. Best Practices for Implementation

1.  **Use `Objects.equals()` and `Objects.hash()`:** These helper methods handle nulls gracefully without throwing `NullPointerException` inside your logic.
    ```java
    public int hashCode() {
        return Objects.hash(id, name);
    }

    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof MyClass)) return false;
        MyClass other = (MyClass) o;
        return id == other.id && // int check
            Objects.equals(name, other.name);
    }
    ```
2.  **Ensure Immutability:** If you override these methods, make your class `final` or its fields `private final`. This prevents the hash code from changing after the object is created (or put into a Map).
3.  **Consistency:** Ensure that the fields contributing to the Hash Code are exactly the same as the ones used in the `equals` check. If you use `id` and `name` for both, but change `name` at some point, the contract breaks.

### Summary Table

| Aspect | Default (`Object`) | Custom (Recommended) |
| :--- | :--- | :--- |
| **Equality Check** | Checks memory address (`==`). | Checks content/value (e.g., strings). |
| **Hash Code** | Memory-specific integer. | Content-specific integer. |
| **HashSet Behavior** | Two objects with same data are treated as distinct duplicates. | Two objects with same data act as a single entity. |
| **HashMap Lookups** | Cannot be used effectively for keys (always finds wrong ref). | Efficient lookup using value. |

By following the contract and ensuring immutability, `equals` and `hashCode` allow Java's collections to perform fast lookups while respecting logical object equality.
