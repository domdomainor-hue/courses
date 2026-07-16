<img width="800" height="1200" alt="image" src="https://github.com/user-attachments/assets/e0ee711e-952f-4da9-a1f9-029de85958f7" />

🔥 Hibernate N+1 Problem

The **N+1 problem** occurs when an application loads a parent entity (1 query) and then executes a separate database query for each child association of those parents (N queries). If you have 100 users, you end up making 101 database queries, which severely impacts application performance.

Here is a detailed breakdown of the four solutions highlighted in your summary, with practical examples using standard JPA/Hibernate.

---

## 1. JOIN FETCH

### The Explanation

Instead of loading the parent entities first and then retrieving the related child records one by one, `JOIN FETCH` instructs the persistence provider to perform an SQL `JOIN` and retrieve both the parents and their children in a **single database query**.

This is the most common and straightforward way to solve the N+1 problem when you know you need the associated data immediately.

### The Example

If you are writing a custom JPQL (Java Persistence Query Language) query:

```java
// JPQL Query
@Query("SELECT u FROM User u JOIN FETCH u.posts")
List<User> findAllUsersWithPosts();

```

**Generated SQL:**

```sql
SELECT u.id, u.username, p.id, p.title, p.user_id 
FROM users u 
INNER JOIN posts p ON u.id = p.user_id;

```

> **Result:** Exactly **1 query** executed.

---

## 2. @EntityGraph

### The Explanation

`@EntityGraph` is a declarative solution introduced in JPA 2.1. It allows you to dynamically define which associations should be eagerly fetched for a specific query, without having to change the global fetch plan (keeping your entity mappings clean with default `FetchType.LAZY`).

It is highly reusable and avoids cluttering your repository methods with complex `JOIN FETCH` JPQL strings.

### The Example

In a Spring Data JPA Repository, you can simply annotate your query method:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @EntityGraph(attributePaths = {"posts"})
    List<User> findAll(); 
}

```

**Generated SQL:**
Hibernate automatically inspects the entity graph and generates a query containing the necessary SQL `LEFT JOIN` to fetch the `posts` collection alongside the `User` records in one go.

> **Result:** Exactly **1 query** executed.

---

## 3. Batch Fetching

### The Explanation

If you have multiple collections to load, using joins can lead to a "Cartesian Product" issue (duplicate rows returned by the database). **Batch Fetching** is an alternative that allows lazy loading but does it in chunks (batches) rather than one by one.

Instead of executing a query for *each* individual user's posts, Hibernate waits until you access a lazy collection, then fetches the posts for a batch of users at once using an `IN` clause.

### The Example

You configure this directly on the entity relationship using `@BatchSize`:

```java
@Entity
public class User {
    @Id
    private Long id;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    @BatchSize(size = 10) // Fetches posts for 10 users at a time
    private List<Post> posts;
}

```

If you load 30 users and try to access their posts, instead of 30 queries, Hibernate executes just **3 queries** for the posts:

```sql
-- Query 1: Get all users
SELECT * FROM users;

-- Query 2: Get posts for the first 10 users
SELECT * FROM posts WHERE user_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

-- Query 3: Get posts for the next 10 users
SELECT * FROM posts WHERE user_id IN (11, 12, 13, 14, 15, 16, 17, 18, 19, 20);

-- Query 4: Get posts for the remaining 10 users
SELECT * FROM posts WHERE user_id IN (21, 22, 23, 24, 25, 26, 27, 28, 29, 30);

```

> **Result:** Reduced from 31 queries down to **4 queries**.

---

## 4. Subselect Fetching

### The Explanation

**Subselect Fetching** is a powerful Hibernate-specific optimization. When you enable this, Hibernate loads the associated collections for *all* retrieved parent entities using a **single additional query** containing a sub-query.

Regardless of whether you have 10, 100, or 10,000 parent entities, subselect fetching will always resolve the relationship in exactly **2 queries** total.

### The Example

You configure this by adding the `@Fetch` annotation to your collection:

```java
@Entity
public class User {
    @Id
    private Long id;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    @Fetch(FetchMode.SUBSELECT)
    private List<Post> posts;
}

```

When you query all users and access their posts, Hibernate will run these two queries:

```sql
-- Query 1: Fetch the users (e.g., users from Paris)
SELECT * FROM users WHERE city = 'Paris';

-- Query 2: Fetch all posts using the original query as a subselect
SELECT * FROM posts WHERE user_id IN (SELECT id FROM users WHERE city = 'Paris');

```

> **Result:** Exactly **2 queries** executed, no matter how many users match the criteria.
