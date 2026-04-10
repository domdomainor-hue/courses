# Kotlin for Senior Java Engineers

## 1. Basics

- `val vs var`: `val` = immutable reference, `var` = mutable → `val x = 1`
- Type inference: compiler infers types → `val name = "John"`
- Functions: concise syntax → `fun sum(a:Int,b:Int)=a+b`
- Expression style: no need for return → `fun max(a:Int,b:Int)=if(a>b)a else b`

## 2. Null Safety

- Non-null types: cannot be null → `val a:String="hi"`
- Nullable types: explicit null → `val b:String?=null`
- Safe call: avoid NPE → `b?.length`
- Elvis operator: default value → `b ?: "default"`
- Not-null assertion: unsafe → `b!!`

## 3. Classes

- Data class: auto methods → `data class User(val id:Int)`
- Regular class: like Java → `class Service`
- Final by default: must use `open` for inheritance → `open class A`
- Constructors: primary constructor → `class A(val x:Int)`

## 4. Collections

- Immutable by default → `val list=listOf(1,2)`
- Functional ops: map/filter → `list.map{it*2}`
- No streams needed → cleaner API

## 5. Control Flow

- `when`: powerful switch → `when(x){1->"A" else->"B"}`
- Smart cast: no manual cast → `if(x is String) x.length`

## 6. Functions Advanced

- Default params → `fun f(a:Int=0)`
- Named params → `f(a=2)`
- Extension functions → `fun String.upper()=this.uppercase()`
- Top-level functions → no utility class needed

## 7. JVM Under the Hood

- Compiles to bytecode → same as Java
- Top-level → static methods → `FileKt.class`
- Properties → getters/setters
- Null safety → compile-time only
- Default args → synthetic methods
- Coroutines → state machines (not threads)

## 8. Java Interop

- Full compatibility → same project
- Cannot mix in same file → `.kt` vs `.java`
- Java → Kotlin → getters as properties
- Kotlin → Java → `FileKt.method()`
- Platform types → `String!` (unsafe nullability)

## 9. Spring Boot Usage

- Constructor injection → `class S(val repo:Repo)`
- DTOs → `data class Request(...)`
- Controllers → concise functions
- Config → immutable properties
- Mapping → extension functions

## 10. JPA Pitfalls

- Avoid data class for entities
- Need `var` for fields
- No-arg constructor issues
- Lazy loading + equals problems

## 11. Coroutines

- Lightweight async → `suspend fun`
- Not threads → continuation-based
- Replace CompletableFuture
- Used with WebFlux

## 12. Best Practices

- Prefer `val` over `var`
- Keep functions small
- Avoid `!!`
- Use data classes for DTOs only
- Do not overuse scope functions

## 13. Kotlin vs Java Summary

- Less boilerplate → data classes
- Safer → null safety
- More expressive → functions/collections
- Same runtime → JVM
- Better async → coroutines

## 14. Interview Questions

- Why Kotlin over Java? → safer + concise + JVM compatible
- Is Kotlin faster? → same JVM performance
- Null safety? → compile-time enforced
- Platform types? → unknown nullability from Java
- Why final classes? → safer design
- Data class vs entity? → not suitable for JPA
- Coroutines vs threads? → lightweight, non-blocking
- val vs final? → similar but idiomatic
- Extension functions? → static helpers
- Default params? → synthetic methods
- Inline functions? → avoid lambda overhead
- Checked exceptions? → not enforced in Kotlin


# Kotlin Collections (Detailed Guide)

## 1. Mental Model

- Kotlin collections are concise and functional → `users.filter { it.active }.map { it.email }`
- No need for Streams API → built-in operators on collections
- Prefer readability over chaining complexity

---

## 2. Read-only vs Mutable

- Read-only (default) → `val list = listOf(1,2,3)`
- Mutable → `val list = mutableListOf(1,2,3); list.add(4)`
- Best practice → prefer read-only unless mutation is required

---

## 3. Core Types

- List (ordered, duplicates) → `listOf("A","B","A")`
- Set (unique) → `setOf("A","B","A")`
- Map (key/value) → `mapOf("A" to 1)`

---

## 4. Transformations

- `map` → transform elements → `users.map { it.email }`
- `mapNotNull` → transform + filter nulls → `values.mapNotNull { it.toIntOrNull() }`
- `flatMap` → flatten nested lists → `orders.flatMap { it.items }`

---

## 5. Filtering

- `filter` → keep matching → `users.filter { it.active }`
- `filterNotNull` → remove nulls → `list.filterNotNull()`
- `partition` → split in two → `val (valid, invalid) = list.partition { it.isValid() }`

---

## 6. Finding Elements

- `find` / `firstOrNull` → safe lookup → `users.firstOrNull { it.id == 1 }`
- `first` → throws if not found → `users.first { it.active }`

---

## 7. Checks & Counts

- `any` → at least one → `users.any { it.active }`
- `all` → all match → `users.all { it.active }`
- `none` → none match → `users.none { it.active }`
- `count` → count matches → `users.count { it.active }`

---

## 8. Sorting

- `sorted` → natural order → `list.sorted()`
- `sortedBy` → custom field → `users.sortedBy { it.email }`
- `sortedByDescending` → reverse → `users.sortedByDescending { it.id }`

---

## 9. Distinct

- `distinct` → remove duplicates → `list.distinct()`
- `distinctBy` → by key → `users.distinctBy { it.email }`

---

## 10. Mapping to Maps

- `associateBy` → build map → `users.associateBy { it.id }`
- Use case → avoid nested loops

Example:
```kotlin
val usersById = users.associateBy { it.id }
val result = orders.map { usersById[it.userId] }
```

---

## 11. Grouping

- `groupBy` → group elements → `users.groupBy { it.active }`
- `groupingBy` + `eachCount` → count per group → `list.groupingBy { it }.eachCount()`

---

## 12. Map Operations

- `mapValues` → transform values → `map.mapValues { it.value * 2 }`
- `getOrDefault` → default value → `map.getOrDefault("A", 0)`
- `getOrElse` → lazy default → `map.getOrElse("A") { 0 }`

---

## 13. Flattening

- `flatten` → flatten nested lists → `listOfLists.flatten()`

---

## 14. Zipping

- Combine collections → `names.zip(ages)`
- Transform → `names.zip(ages) { n, a -> "$n:$a" }`

---

## 15. Slicing

- `take` → first elements → `list.take(2)`
- `drop` → skip elements → `list.drop(2)`
- `chunked` → batch → `list.chunked(2)`

---

## 16. Reduction

- `reduce` → combine → `list.reduce { acc, v -> acc + v }`
- `fold` → with initial → `list.fold(0) { acc, v -> acc + v }`

---

## 17. Sequences (Lazy)

- Convert to lazy → `list.asSequence()`
- Avoid intermediate collections → `.filter { }.map { }.toList()`
- Use for large datasets or long pipelines

---

## 18. Performance Patterns

- Avoid nested `find` → use `associateBy`
- Use `Set` for membership → `ids.toSet()`
- Use `chunked` for batching APIs

---

## 19. Real-world Patterns

- DTO mapping → `entities.map { it.toDto() }`
- Filter + map → `users.filter { it.active }.map { it.email }`
- Distinct IDs → `orders.map { it.userId }.distinct()`
- Lookup map → `users.associateBy { it.id }`
- Group counts → `list.groupingBy { it.status }.eachCount()`

---

## 20. Java vs Kotlin

- Java Streams → `stream().filter().map()`
- Kotlin → `filter { }.map { }`
- Less boilerplate → more readable pipelines

---

## 21. Best Practices

- Prefer read-only collections
- Keep pipelines short and readable
- Use `associateBy` instead of nested loops
- Avoid overusing `asSequence()`
- Favor clarity over clever chaining

