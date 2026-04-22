---
title: "Write Skew: The Bug You Can't Reproduce"
tags: [Databases, Spring Boot]
style: border
color:
description: How a simple "create if not exists" flow silently broke under concurrency requests.
---
I remember a time when I was developing a clearly trivial flow: "create entity if none exists". It was so simple, I had to check if an entity already exists for a customer, if not, create one.

After weeks in QA, the release day came, and after the deploy, three of my team members went to the same customer to check this flow.

I didn't know it, but I was just about to discover in real life the **write skew**.

<br>

### The pattern

Write skew is a race condition situation that follows a really simple pattern:

1. Transaction A reads some data and makes a decision based on it.
2. Transaction B reads the **same** data and makes a decision based on it.
3. Each transaction writes to **different** rows.
4. Neither write is wrong in isolation — but together they break an invariant that was true when both transactions started. (In my case, just one entity of the class I was creating was allowed to exist).

The classic textbook example is the doctors-on-call problem: a hospital requires at least one doctor on call at all times. Alice sees two doctors on call and decides it's safe to go off call. Bob sees the same two doctors on call and makes the same decision. Both transactions commit. Now there are zero doctors on call — the invariant is broken.

In our duplicate-entity bug, the invariant is the mirror image: **at most one entity per customer**. Same pattern, opposite direction.


```
Window 1: SELECT → no entity exists  ┐
Window 2: SELECT → no entity exists  ├─ all three see "empty" before any write
Window 3: SELECT → no entity exists  ┘

Window 1: INSERT entity  ┐
Window 2: INSERT entity  ├─ 3 duplicates created
Window 3: INSERT entity  ┘
```

<br>

---
### Why Snapshot Isolation doesn't save you

Most databases default to **Snapshot Isolation** (called `REPEATABLE READ` in MySQL and InnoDB). Under snapshot isolation, each transaction gets a consistent snapshot of the database at the moment it started. This prevents different kinds of race conditions: dirty reads, read skew, and some lost updates.

But write skew is still a problem, because snapshot isolation prevents a transaction from reading uncommitted changes made by concurrent transactions. **It does not prevent two transactions from reading the same data**, each deciding it's safe to write, and writing to *different* rows.

Each of the three transactions reads a clean snapshot: "no entity exists." That's true — at the start of each transaction, no entity existed. The problem is that the premise became false the moment any of them committed.

**Snapshot isolation can't catch this because the transactions aren't modifying the same row. There's no write-write conflict to detect.**


<br>

---

### The fixes

##### 1. Unique constraint

For the "at most one" invariant, the cleanest fix is to add a unique constraint in the table:

```sql
ALTER TABLE entities ADD CONSTRAINT unique_entity_per_customer UNIQUE (customer_id);
```

That was the solution in our case, we completely forgot to add it at first, causing that unexpected bug.

Now two of the three inserts fail with a constraint violation. This is the preferred approach for this specific case: it makes the invariant explicit, durable, and enforced at the database level — not something you have to remember to check in application code.

In Spring Boot, catch the exception and handle it gracefully:

```java
@Transactional
public CustomerEntity getOrCreateEntity(Long customerId) {
    return repository.findByCustomerId(customerId)
        .orElseGet(() -> {
            try {
                return repository.save(new CustomerEntity(customerId));
            } catch (DataIntegrityViolationException e) {
                // Another transaction beat us to it
                return repository.findByCustomerId(customerId).orElseThrow();
            }
        });
}
```

On the other hand, you can use an upsert, so you never need to catch the error:

```java
@Modifying
@Query(value = """
    INSERT INTO entities (customer_id)
    VALUES (:customerId)
    ON DUPLICATE KEY UPDATE customer_id = customer_id
    """, nativeQuery = true)
void upsert(@Param("customerId") Long customerId);
```

##### 2. SELECT ... FOR UPDATE

This statement acquires an exclusive row lock on every row returned by the query. Any concurrent transaction that tries to read or write those rows has to wait until the lock is released.

```java
// Repository
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<CustomerEntity> findByCustomerId(Long customerId);

// Service
@Transactional
public CustomerEntity getOrCreateEntity(Long customerId) {
    Optional<CustomerEntity> existing = repository.findByCustomerId(customerId);
    return existing.orElseGet(() -> repository.save(new CustomerEntity(customerId)));
}
```

The problem with this approach is **there's no row to lock** in the situation I've described. When your query returns zero rows, there's nothing to put a lock on. All three transactions execute their `SELECT FOR UPDATE`, all get zero rows, all acquire no locks, and all proceed to insert.

`FOR UPDATE` is the right tool when **you need to lock *existing* rows before modifying them**. For the "check for absence" scenario, it doesn't help.

##### 3. SERIALIZABLE isolation

The only isolation level that fully prevents write skew is `SERIALIZABLE`. It guarantees that concurrent transactions produce results equivalent to running them one at a time, serially.

In Spring Boot, apply it specifically on the methods that need it:

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public CustomerEntity getOrCreateEntity(Long customerId) {
    return repository.findByCustomerId(customerId)
        .orElseGet(() -> repository.save(new CustomerEntity(customerId)));
}
```

Under `SERIALIZABLE`, MySQL InnoDB promotes every plain `SELECT` to a locking read. If two transactions run this concurrently, one blocks until the other commits — and then finds the entity that was just created.

Two important warnings:

**Performance cost:** `SERIALIZABLE` has a real overhead. Use it surgically on the specific method that needs it — not as a global default in `application.properties`.

**Self-invocation bypasses the proxy:** `@Transactional` works through a Spring AOP proxy. Calling a `@Transactional` method from within the same bean skips the proxy entirely, silently ignoring the isolation level:

```java
// This does NOT apply SERIALIZABLE — proxy is bypassed
public void outer() {
    this.getOrCreateEntity(id);  // ← self-invocation, isolation level ignored
}
```

**Always call `@Transactional` methods from another bean.**

<br>

---

### How to recognize this pattern

Write skew appears in any **check-then-act** flow on shared data:

- Read a condition (the presence or absence of rows).
- Make a decision based on what you read.
- Write something based on that decision.

The tell: *if two transactions ran this code simultaneously, could they each see a "safe" state that, after both commit, is no longer safe?*

If yes, choose your fix based on what's actually being protected:

| Invariant | Best fix |
|---|---|
| At most one / at least one row | Unique constraint |
| Existing rows need to be locked | `SELECT ... FOR UPDATE` |
| Complex multi-row invariant | `SERIALIZABLE` isolation |
