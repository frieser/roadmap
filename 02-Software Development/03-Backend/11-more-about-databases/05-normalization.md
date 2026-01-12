---
---

## Summary
Database Normalization is the process of structuring a relational database in a way that reduces data redundancy and improves data integrity. It involves organizing columns and tables to ensure that dependencies are properly enforced and data is stored logically.

## Detailed Explanation
The goal of normalization is to isolate data so that additions, deletions, and modifications can be made with minimal impact on the rest of the database.

### The Normal Forms (NF)
1.  **First Normal Form (1NF)**: Data must be atomic (no lists in cells), and each record must be unique (Primary Key).
2.  **Second Normal Form (2NF)**: Must be in 1NF, and all non-key columns must depend on the *entire* primary key.
3.  **Third Normal Form (3NF)**: Must be in 2NF, and non-key columns must not depend on other non-key columns (no transitive dependencies). "The data depends on the key, the whole key, and nothing but the key."

### Denormalization
Sometimes, developers intentionally violate normalization rules for performance. This is called Denormalization. It reduces the number of `JOIN` operations needed for read-heavy workloads but increases the risk of data inconsistency.

## Go-specific Context
In Go, normalization impacts how you design your structs. Highly normalized databases lead to many small structs and frequent use of `Preload` or `Joins` in ORMs.

### Example: Normalizing User Data
**Unnormalized**:
```go
type User struct {
    ID      int
    Name    string
    Address string // "123 Main St, New York, NY" -> Violates 1NF (not atomic)
}
```

**Normalized (3NF)**:
```go
type User struct {
    ID        int
    Name      string
    AddressID int
}

type Address struct {
    ID     int
    Street string
    CityID int
}

type City struct {
    ID   int
    Name string
    Zip  string
}
```

## Interview Questions
**Q: What is an 'Update Anomaly'?**
**A:** An update anomaly occurs in unnormalized databases when data is duplicated. If you update the value in one place but not another, the data becomes inconsistent.

**Q: When would you choose to denormalize your data?**
**A:** Denormalization is used when read performance is critical and the overhead of complex `JOINs` is too high. Common in data warehousing, analytics, or high-scale NoSQL applications.

**Q: Explain the difference between 2NF and 3NF.**
**A:** 2NF deals with partial functional dependencies (an attribute depends on only part of a composite key). 3NF deals with transitive dependencies (an attribute depends on another attribute that is not a key).
