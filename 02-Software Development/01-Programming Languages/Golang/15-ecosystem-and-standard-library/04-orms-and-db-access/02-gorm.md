# GORM

## Summary
GORM is the most popular Object-Relational Mapping (ORM) library for Go. It provides a developer-friendly API for interacting with databases (Postgres, MySQL, SQLite) using Go structs. Key features include Auto-Migration (creating tables from structs), Associations (Has One, Has Many), Hooks (BeforeSave, AfterCreate), and Soft Deletes.

## Detailed Explanation

### 1. Declaring Models
You define database tables using standard Go structs with tags.
```go
import "gorm.io/gorm"

type User struct {
    gorm.Model        // Adds ID, CreatedAt, UpdatedAt, DeletedAt
    Name      string
    Email     string  `gorm:"uniqueIndex"` // Struct tag for constraints
    Active    bool    `gorm:"default:true"`
}
```

### 2. CRUD Operations
GORM relies on method chaining.
```go
// Create
db.Create(&User{Name: "Jinzhu", Email: "jinzhu@gorm.io"})

// Read
var user User
db.First(&user, 1) // find user with integer primary key 1
db.First(&user, "name = ?", "Jinzhu") // find user with name Jinzhu

// Update
db.Model(&user).Update("Name", "NewName")

// Delete (Soft Delete if gorm.Model is used)
db.Delete(&user, 1)
```

### 3. Associations & Preloading
GORM handles relationships automatically if defined in the struct.
```go
type User struct {
    gorm.Model
    Orders []Order
}

// Fetch user AND their orders in one go
db.Preload("Orders").Find(&users)
```

## Interview Questions

**Q: What is the "N+1 problem" in GORM and how do you solve it?**
**A:** The N+1 problem occurs when you fetch a list of N objects (1 query) and then iterate over them to fetch a related child object for each (N queries), resulting in N+1 total queries. In GORM, this is solved using **Preloading** (`db.Preload("Orders").Find(&users)`), which fetches the main records and the related records in just two queries efficiently.

**Q: What are GORM Hooks?**
**A:** Hooks are functions that are automatically called before or after database operations (Create, Save, Update, Delete, Find). For example, `BeforeSave` can be used to hash a password before it is written to the database, or `AfterCreate` can be used to send a welcome email.

**Q: Why might you choose *not* to use GORM in a high-performance Go service?**
**A:** GORM relies heavily on **reflection** to map structs to SQL, which incurs a CPU penalty compared to raw SQL or generated code (like `sqlc`). Additionally, for very complex queries, the ORM abstraction can become leaky or generate suboptimal SQL, making raw `pgx` or `database/sql` a better choice for performance-critical paths.
