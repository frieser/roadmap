---
---

## Summary
The **Type Object Pattern** involves creating a class (the "Type Object") to represent the type of another object (the "Entity"). This allows new "types" of entities to be defined dynamically at runtime by creating new instances of the Type Object, rather than by creating new subclasses in code. Ideally used to avoid subclass explosions.

## Detailed Explanation

### Problem
Imagine a game with many monsters: Dragon, Troll, Orc. You could create a subclass for each (`type Dragon struct`, `type Troll struct`).
*   **Issue 1**: If you have 100 monsters, you need 100 structs.
*   **Issue 2**: To add a new monster, you must recompile the code.
*   **Issue 3**: Monsters differ only by data (Health, Attack Power, Name), not behavior.

### Solution
Split the object into two:
1.  **Monster (Entity)**: The concrete instance in the game (has position, current health).
2.  **MonsterType (Type Object)**: The configuration/blueprint (has Max Health, Base Attack, Name).

The `Monster` holds a reference to its `MonsterType`.

### Relationship to Other Patterns
*   **Strategy**: Type Object focuses on shared *data/properties* defining a category. Strategy focuses on interchangeable *behavior/algorithms*.
*   **Flyweight**: Type Objects are often implemented as Flyweights (shared instances).

## Go Example

```go
package main

import "fmt"

// 1. The Type Object (Blueprint/Configuration)
// Can be loaded from JSON/Database at runtime!
type MonsterType struct {
	Name          string
	StartingHealth int
	AttackPower    int
}

func (t *MonsterType) CreateMonster() *Monster {
	return &Monster{
		Type:          t,
		CurrentHealth: t.StartingHealth,
	}
}

// 2. The Entity (Instance)
type Monster struct {
	Type          *MonsterType // Reference to the Type Object
	CurrentHealth int
}

func (m *Monster) Attack() {
	fmt.Printf("The %s attacks for %d damage!\n", m.Type.Name, m.Type.AttackPower)
}

func main() {
	// Define types (could be loaded from file)
	dragonType := &MonsterType{Name: "Dragon", StartingHealth: 1000, AttackPower: 50}
	trollType := &MonsterType{Name: "Troll", StartingHealth: 200, AttackPower: 10}

	// Create instances
	dragon1 := dragonType.CreateMonster()
	dragon2 := dragonType.CreateMonster()
	troll1 := trollType.CreateMonster()

	dragon1.Attack()
	dragon2.Attack()
	troll1.Attack()
	
	// Dynamic runtime addition
	goblinType := &MonsterType{Name: "Goblin", StartingHealth: 50, AttackPower: 5}
	goblin1 := goblinType.CreateMonster()
	goblin1.Attack()
}
```

## Interview Questions

### Q: When should you use Type Object instead of Subclassing/Embedding?
**A:** Use Type Object when:
1.  The difference between types is primarily **data**, not behavior.
2.  You need to define new types at **runtime** (e.g., loaded from configuration files) without recompiling.
3.  You have a massive number of types that would clutter the codebase if defined as distinct structs.

### Q: How does Type Object relate to the Flyweight pattern?
**A:** The `MonsterType` instances are typically **Flyweights**. Since `DragonType` is the same for all dragons, we only need one instance of `DragonType` in memory, shared by thousands of `Monster` instances. This saves memory.
