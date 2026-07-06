<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/4e3da90c-568e-46fe-bb2a-06237ba01d09" />


💳 Spring Transaction Management ("@Transactional")

One of the most common Spring Boot interview questions is:

«"I used "@Transactional", but my data didn't roll back. Why?"»

Understanding transaction management is essential for building reliable and consistent backend applications.

🔹 What is a Transaction?

A transaction is a group of database operations executed as a single unit of work following the ACID principles:

✅ Atomicity

✅ Consistency

✅ Isolation

✅ Durability

🔹 What does "@Transactional" do?

Spring automatically:

✔ Starts a transaction

✔ Commits if everything succeeds

✔ Rolls back when an eligible exception occurs

This eliminates manual transaction management.

🔹 Key Concepts Every Developer Should Know

✔ Rollback happens by default for RuntimeException and Error

✔ Checked exceptions require "rollbackFor"

✔ Understand Propagation ("REQUIRED", "REQUIRES_NEW")

✔ Choose the right Isolation Level

✔ Keep transactions short to improve performance

🌍 Real-World Example

In an e-commerce application:

🛒 Create Order

💳 Process Payment

📦 Update Inventory

If payment fails, all previous database changes should roll back to keep the system consistent.

⚠️ Common Mistakes

❌ Catching exceptions without rethrowing

❌ Self-invocation ("@Transactional" method calling another method in the same class)

❌ Assuming checked exceptions roll back automatically

❌ Wrapping long-running external API calls inside a transaction

🎯 Interview Questions

1️⃣ Why doesn't Spring roll back checked exceptions by default?

2️⃣ REQUIRED vs REQUIRES_NEW?

3️⃣ What are Isolation Levels?

4️⃣ Why doesn't "@Transactional" work on private methods?

5️⃣ What is the self-invocation problem?

💡 Senior Developer Tip

Transactions should protect database consistency, not long-running business logic. Keep them short, avoid unnecessary locks, and understand how Spring AOP proxies manage transactions behind the scenes.
