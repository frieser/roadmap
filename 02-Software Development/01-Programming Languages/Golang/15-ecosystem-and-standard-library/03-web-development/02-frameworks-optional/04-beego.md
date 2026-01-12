# Beego Web Framework

## Summary
Beego is a full-featured, "batteries-included" MVC framework for Go. Unlike micro-frameworks (Gin, Echo) that focus on routing and HTTP, Beego provides an entire ecosystem: ORM, caching, logging, configuration parsing, and an admin dashboard. It is designed for rapid enterprise application development where you want an all-in-one solution.

## Detailed Explanation

### 1. MVC Architecture
Beego enforces a classic Model-View-Controller structure.
*   **Router**: Dispatches URLs to Controllers.
*   **Controller**: Handles business logic.
*   **Model**: Interacts with the database (Beego ORM).

### 2. The `bee` Tool
Beego comes with a powerful CLI tool called `bee`.
*   `bee new myapp`: Scaffolds a new project structure.
*   `bee run`: Runs the app with "hot reload" (recompiles on file save).
*   `bee generate`: Generates code for controllers/routers.

### 3. Code Example (Controller)
```go
type MainController struct {
    beego.Controller
}

func (c *MainController) Get() {
    c.Data["Website"] = "beego.me"
    c.Data["Email"] = "astaxie@gmail.com"
    c.TplName = "index.tpl" // Renders a template
}
```

## Interview Questions

**Q: When would you choose Beego over Gin?**
**A:** Choose Beego if you want a framework like Django or Rails—where everything (ORM, caching, logging) is integrated and follows a strict convention. Choose Gin if you want a lightweight, flexible router (like Flask or Express) and prefer to pick your own libraries for database and logging.

**Q: What is the downside of using Beego?**
**A:** It is "heavy" and opinionated. It uses a lot of reflection and magic (init functions) which can make the application startup slower and harder to debug compared to the explicit, idiomatic Go style favored by micro-frameworks. It also has a steeper learning curve due to its large surface area.

**Q: Does Beego support hot reloading?**
**A:** Yes, via the `bee` command-line tool. Running `bee run` watches your filesystem for changes and automatically recompiles and restarts the binary, which speeds up the development loop significantly.
