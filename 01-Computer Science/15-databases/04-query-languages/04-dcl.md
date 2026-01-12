---
---

## Summary
**DCL (Data Control Language)** consists of SQL commands used to control access to data stored in a database. It handles authorization, permissions, and security. The two main commands are **GRANT** and **REVOKE**.

## Detailed Explanation
DCL ensures that only authorized users can perform DDL, DML, or DQL operations.

### Key Commands
1.  **GRANT**: Gives specific privileges to a user or role.
    *   `GRANT SELECT, INSERT ON users TO 'app_user';`
2.  **REVOKE**: Removes specific privileges from a user or role.
    *   `REVOKE INSERT ON users FROM 'app_user';`

### Privileges
*   **Object Privileges**: SELECT, INSERT, UPDATE, DELETE, EXECUTE (on procedures).
*   **System Privileges**: CREATE TABLE, CREATE USER, DROP ANY TABLE (admin tasks).

### Roles
Instead of granting permissions to every user individually, DBAs create **Roles** (e.g., `read_only`, `admin`), grant permissions to the role, and then assign users to that role.

### Go Context
Go applications typically connect to the database using a specific user (defined in the connection string). DCL is rarely executed by the application code itself; it is usually part of the database setup scripts or migration (infrastructure as code).

```go
// Application doesn't usually run DCL.
// Instead, the connection string defines the user capabilities:
// "postgres://app_user:password@localhost/dbname"

// If 'app_user' was only GRANTed SELECT, 
// db.Exec("DELETE FROM users") will return a "permission denied" error.
```

## Interview Questions
**Q: What is the Principle of Least Privilege in Database Security?**
A: Users/Applications should only be granted the minimum permissions necessary to perform their function. An application that only reads data should not have DELETE or DROP permissions.

**Q: What happens if you REVOKE a permission that was never granted?**
A: Usually nothing. The command completes successfully but has no effect.

**Q: Can you GRANT permissions to a Role?**
A: Yes, this is the best practice. You GRANT permissions to a Role, and then GRANT the Role to a User.

## Diagram
```mermaid
graph TD
    Admin[DB Admin]
    User[App User]
    Role[Role: ReadOnly]
    Table[Table: Data]
    
    Admin -- GRANT SELECT --> Role
    Role -- Assigned To --> User
    User -- Has Access --> Table
    
    Admin -- REVOKE DELETE --> User
```
