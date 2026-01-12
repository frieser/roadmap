---
---

# Database Caching

**Database Caching** refers to the internal mechanisms used by DBMS to speed up data retrieval.

## **Types**
- **Buffer Pool**: An area in main memory where the database caches data and indexes as they are read from disk.
- **Query Cache**: (Now deprecated in most modern DBs) Caches the exact results of a query string.
- **Materialized Views**: Pre-computed results of a complex query stored as a table.

## **Pros**
- **Automatic**: Mostly managed by the database engine itself.
- **Disk I/O Reduction**: Minimizes slow disk reads.
