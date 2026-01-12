#API
---
---

## Summary
API pagination is a technique used to divide large datasets into smaller, manageable chunks (pages) when returning them through an endpoint. It is essential for improving API performance, reducing network latency, and enhancing User Experience (UX) by preventing the server from being overwhelmed by massive data transfers. Common strategies include offset-based, cursor-based, and page-based pagination.

## Detailed Explanation

### 1. Why Paginate?
- **Performance**: Reduces database load and memory usage on the server.
- **UX**: Faster response times for users.
- **Reliability**: Prevents timeouts and crashes when dealing with millions of records.

### 2. Pagination Strategies

#### Offset-based Pagination
The most common strategy, using `LIMIT` and `OFFSET` in SQL queries.
- **Parameters**: `limit` (items per page), `offset` (number of items to skip).
- **Pros**: Easy to implement, allows jumping to specific pages.
- **Cons**: 
  - **Performance**: As offset increases, the database must still read and skip all previous rows.
  - **Data Drift**: If items are added or deleted while paginating, items might be duplicated or skipped.

#### Cursor-based (Keyset) Pagination
Uses a unique, sequential identifier (like an ID or timestamp) as a pointer to where the next page should start.
- **Parameters**: `cursor` (the ID of the last item from the previous page), `limit`.
- **Pros**: 
  - **Performance**: Constant time complexity if the cursor column is indexed.
  - **Stability**: Not affected by insertions/deletions (no drift).
- **Cons**: Cannot easily jump to a specific page (e.g., "Go to page 50").

#### Page-based Pagination
A high-level abstraction over offset pagination.
- **Parameters**: `page`, `per_page`.
- **Logic**: `offset = (page - 1) * per_page`.

### 3. Standards and Headers
- **Link Header (RFC 5988)**: The standard way to provide pagination links in the response.
  ```http
  Link: <https://api.example.com/items?page=3>; rel="next",
        <https://api.example.com/items?page=1>; rel="prev"
  ```
- **X-Total-Count**: A custom header often used to tell the client the total number of records available.

### 4. Implementation in Go

#### Offset Pagination Example
```go
func GetItems(db *gorm.DB, page, pageSize int) ([]Item, error) {
    var items []Item
    offset := (page - 1) * pageSize
    
    err := db.Limit(pageSize).Offset(offset).Find(&items).Error
    return items, err
}
```

#### Cursor Pagination Example
```go
func GetItemsCursor(db *gorm.DB, lastID int, limit int) ([]Item, error) {
    var items []Item
    
    // We fetch items with ID greater than the last seen ID
    err := db.Where("id > ?", lastID).
        Order("id asc").
        Limit(limit).
        Find(&items).Error
        
    return items, err
}
```

#### Setting Response Headers
```go
func handleItems(w http.ResponseWriter, r *http.Request) {
    // ... logic to fetch items ...
    
    w.Header().Set("X-Total-Count", "1050")
    w.Header().Set("Link", `<https://api.example.com/v1/items?offset=20&limit=10>; rel="next"`)
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(items)
}
```

## Interview Questions

- **Q: What is the main disadvantage of Offset Pagination for large datasets?**
- **A:** The database must scan and skip all rows before the offset. For an offset of 1,000,000, the DB reads 1,000,000 rows just to discard them, leading to severe performance degradation.

- **Q: How does Cursor Pagination solve the "Data Drift" problem?**
- **A:** Since it relies on a specific value (e.g., `id > 100`) rather than a relative position (`skip 10`), inserting a new item with `id=101` doesn't shift the position of existing items relative to the pointer, ensuring no item is skipped or duplicated.

- **Q: When would you choose Offset over Cursor pagination?**
- **A:** When the dataset is small, or when the user requirement includes jumping to specific pages (e.g., a search results page with "1 2 3 ... 10" navigation).

- **Q: What is the purpose of the `Link` header in an API?**
- **A:** It provides a standard, hypermedia-driven way for clients to discover the next, previous, first, and last pages of a collection without having to manually construct URLs.
