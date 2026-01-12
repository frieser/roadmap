---
---

## Summary
**Federation** (Functional Partitioning) is a database architecture strategy where a monolithic database is split into multiple distinct databases based on business domain or function (e.g., Users, Products, Forums). This approach decouples systems, reduces the "blast radius" of failures, and allows independent scaling.

## Detailed Explanation

### How It Works
Instead of a single `monolith.db` containing 500 tables, you split it into:
1.  **Identity DB**: Stores `users`, `roles`, `sessions`.
2.  **Catalog DB**: Stores `products`, `categories`, `inventory`.
3.  **Order DB**: Stores `orders`, `invoices`, `payments`.

### Benefits
*   **Isolation**: High load on the "Catalog" (Black Friday browsing) doesn't impact the "Identity" DB (logins).
*   **Scaling**: You can put the `Order DB` on a high-CPU server and the `Catalog DB` on a high-RAM server.
*   **Maintainability**: Smaller schemas are easier to migrate and manage.

### Challenges
*   **No Cross-DB Joins**: You cannot write `SELECT * FROM orders JOIN users ON ...` because tables live in different physical databases.
*   **Data Integrity**: You lose Foreign Key constraints between domains.
*   **Distributed Transactions**: Updating `orders` and `inventory` atomically requires complex patterns like **Two-Phase Commit (2PC)** or **Sagas**.

## Go Context: Multiple Connections
In Go, you handle federation by maintaining separate connection pools for each database.

```go
type App struct {
	UserDB    *sql.DB
	ProductDB *sql.DB
}

func NewApp(userDSN, productDSN string) *App {
	u, _ := sql.Open("postgres", userDSN)
	p, _ := sql.Open("postgres", productDSN)
	return &App{UserDB: u, ProductDB: p}
}

// Application-side Join
func (a *App) GetOrderWithUser(orderID int) (*OrderDTO, error) {
	// 1. Fetch Order
	var order Order
	a.ProductDB.QueryRow("SELECT user_id, ... FROM orders WHERE id=?", orderID).Scan(&order.UserID, ...)

	// 2. Fetch User
	var user User
	a.UserDB.QueryRow("SELECT name FROM users WHERE id=?", order.UserID).Scan(&user.Name)

	// 3. Combine
	return &OrderDTO{Order: order, UserName: user.Name}, nil
}
```

## Interview Questions

### Q: How do you handle joins in a Federated database architecture?
**A:** You perform **Application-Side Joins**. You query the first database to get the foreign keys (e.g., `user_id`s), collect them, and then run a second query against the other database (e.g., `SELECT * FROM users WHERE id IN (...)`).

### Q: What is the main downside of Federation compared to Sharding?
**A:** Federation splits by *function* (schema), whereas Sharding splits by *data* (rows). Federation has a limit: if your "Users" table grows to 10 billion rows, Federation alone doesn't solve the size problem; you would still need to shard the User DB.
