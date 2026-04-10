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
