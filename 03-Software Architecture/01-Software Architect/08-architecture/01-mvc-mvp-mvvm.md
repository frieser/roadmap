---
---

# MVC, MVP, and MVVM Architecture Patterns

Architectural patterns that structure the presentation layer of an application, focusing on the **Separation of Concerns (SoC)** between data, UI, and logic.

## 1. Core Components

### **Model**
- Represents the business logic and data.
- Not just data classes, but also the logic to retrieve, store, and manipulate it (e.g., database interactions, API calls).
- Independent of the UI.

### **View**
- The visual representation of the data (UI).
- Displays information and forwards user actions (clicks, inputs) to the logic layer.

---

## 2. Comparison of Patterns

| Feature | MVC (Controller) | MVP (Presenter) | MVVM (ViewModel) |
| :--- | :--- | :--- | :--- |
| **Logic Layer** | Controller | Presenter | ViewModel |
| **View's Role** | Smart/Active | Passive | Observes/Binds |
| **Coupling** | High (View knows Model) | Low (Via Interface) | Very Low (Data Binding) |
| **Testing** | Difficult (UI coupled) | Easy (Mocks interfaces) | Easiest (Unit test VM) |
| **Data Flow** | Bidirectional/Complex | Bidirectional (Presenter updates View) | Bidirectional (Data Binding) |

---

## 3. Evolution and Variations

### **MVC: The Foundation**
- **Structure**: Controller receives input -> updates Model -> View updates (sometimes via Model directly).
- **Issue**: "Massive View Controller" (especially in iOS/Android). View and Model are often tightly coupled, making unit testing hard.
- **Use Cases**: Classic Web Frameworks (Ruby on Rails, Django, ASP.NET MVC).

### **MVP: Decoupling via Interfaces**
- **Structure**: View is a "Passive View". Presenter handles all logic and updates the View through an interface.
- **Evolution**: Introduced to make UI logic unit-testable without relying on framework UI classes.
- **Use Cases**: Early Android development, WinForms, GWT.

### **MVVM: Reactive and Data-Driven**
- **Structure**: ViewModel exposes observable data. View "binds" to these properties. When data changes in VM, View updates automatically.
- **Key Tech**: **Data Binding**. ViewModel has no reference to the View.
- **Use Cases**: Modern Web (Vue, Angular), WPF/MAUI, Modern Mobile (Jetpack Compose, SwiftUI).

---

## 4. Modern Variation: Unidirectional Data Flow (UDF)

As applications grew complex (especially in Web), bidirectional patterns like MVVM could lead to "state explosion" and unpredictable side effects.

### **Flux / Redux**
- **Concept**: State is immutable and lives in a single source of truth (**Store**).
- **Flow**: Action -> Dispatcher -> Store -> View.
- **Benefits**: Predicable state changes, easy "time-travel" debugging.
- **Evolution**: Often seen as the modern successor to MVVM for complex state management.

### **MVI (Model-View-Intent)**
- A reactive version of UDF often used in Mobile.
- User **Intent** -> Model (State) -> View (Renders State).

---

## 5. Architectural Trade-offs

- **Small Apps**: MVC is often sufficient and faster to implement.
- **Heavy UI Logic**: MVP/MVVM reduce complexity and improve testability.
- **Complex State**: UDF (Flux/Redux) provides the most predictability at the cost of boilerplate.

## Summary for Architects
Choose the pattern based on **testability requirements** and **platform capabilities**. If the framework supports powerful data binding, MVVM is usually the best choice. For high-reliability systems with complex state, look toward Unidirectional Data Flow.

