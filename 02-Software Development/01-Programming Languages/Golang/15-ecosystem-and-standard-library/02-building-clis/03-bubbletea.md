# Bubbletea (TUI Framework)

## Summary
Bubbletea is a Go framework for building terminal user interfaces (TUIs) based on **The Elm Architecture**. It treats the terminal application as a pure state machine: State (Model) -> View (String) -> Update (Msg). This allows for creating rich, interactive, and beautiful CLI experiences (like lists, spinners, text inputs) that are robust and easy to reason about.

## Detailed Explanation

### The Elm Architecture in Go
Bubbletea applications consist of three main parts:

1.  **Model**: A struct that stores the application state.
2.  **View**: A method `View() string` that returns the UI string based on the current Model.
3.  **Update**: A method `Update(msg Msg) (Model, Cmd)` that handles events (keypresses, timer ticks) and returns a new Model and an optional Command (side effect).

### Code Example: A Simple Counter

```go
package main

import (
    "fmt"
    "os"
    tea "github.com/charmbracelet/bubbletea"
)

// 1. Model: The state
type model struct {
    count int
}

func initialModel() model {
    return model{count: 0}
}

// Init: Initial side effect (none here)
func (m model) Init() tea.Cmd {
    return nil
}

// 2. Update: Handle messages
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        case "q", "ctrl+c":
            return m, tea.Quit
        case "up":
            m.count++
        case "down":
            m.count--
        }
    }
    return m, nil
}

// 3. View: Render UI
func (m model) View() string {
    return fmt.Sprintf("Count: %d\n\nPress q to quit.\n", m.count)
}

func main() {
    p := tea.NewProgram(initialModel())
    if _, err := p.Run(); err != nil {
        fmt.Printf("Alas, there's been an error: %v", err)
        os.Exit(1)
    }
}
```

### Key Concepts
*   **Cmd (Command)**: A function that performs I/O (like an HTTP request) and returns a `Msg`. The update loop is pure; side effects are managed via Cmds.
*   **Msg (Message)**: An interface payload that carries data to the Update function.
*   **Bubbles**: A standard library of common components (spinners, list views, text inputs, viewports) maintained by the Charm team to speed up development.

## Interview Questions

**Q: What is the "Update" loop in Bubbletea?**
**A:** It is the core logic handler. It receives a `Msg` (event) and the current `Model` (state). Based on the message type (KeyMsg, WindowSizeMsg, custom struct), it calculates the *next* state of the Model and returns it, along with any asynchronous `Cmd` to run next. This ensures one-way data flow.

**Q: How do you handle asynchronous tasks (like fetching data) in Bubbletea?**
**A:** You return a `tea.Cmd` from the `Init` or `Update` function. A `Cmd` is essentially a function that performs the work in a goroutine and returns a `Msg` when done. Bubbletea's runtime manages the goroutine and feeds the result message back into the `Update` loop.

**Q: Why use Bubbletea over standard `fmt.Print` or `survey`?**
**A:** Standard print/scan is linear and blocking. Bubbletea allows for **full-screen**, **event-driven** applications where the UI can update dynamically in response to multiple inputs (keyboard, timers, network events) without blocking the user, enabling rich dashboards and interactive tools.
