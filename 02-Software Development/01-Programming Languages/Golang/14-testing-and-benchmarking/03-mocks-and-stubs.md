# Mocks and Stubs

## Summary
In Go, mocking is primarily achieved through **Interfaces**. Unlike dynamic languages that can monkey-patch classes at runtime, Go requires you to design your code for testability by accepting interfaces rather than concrete types. This allows you to inject "mock" or "stub" implementations during tests to control behavior (e.g., simulating database errors) and verify interactions.

## Detailed Explanation

### The Philosophy
Go does not have a built-in mocking framework in the standard library (though specialized libraries exist). The idiomatic approach is:
1.  Define an `interface` for the dependency (e.g., Database, API Client).
2.  Accept the interface in your function/struct.
3.  Inject a real implementation in production.
4.  Inject a mock struct in tests.

### Definitions
*   **Stub**: An object that provides predefined answers to calls (e.g., "return error" or "return user Bob"). Used to test state.
*   **Mock**: An object that verifies *how* it was called (e.g., "expect Save() to be called once with argument X"). Used to test behavior.

### Implementation Pattern

#### 1. Define the Interface
Do not depend on `*sql.DB` directly.
```go
type DataStore interface {
    GetUser(id int) (*User, error)
}
```

#### 2. The Code Under Test
```go
type UserService struct {
    store DataStore // Interface dependency
}

func (s *UserService) GetUserName(id int) (string, error) {
    user, err := s.store.GetUser(id)
    if err != nil {
        return "", err
    }
    return user.Name, nil
}
```

#### 3. The Mock (Hand-rolled)
```go
type MockStore struct {
    // Fields to control behavior
    mockUser *User
    mockErr  error
}

func (m *MockStore) GetUser(id int) (*User, error) {
    return m.mockUser, m.mockErr
}
```

#### 4. The Test
```go
func TestGetUserName_Error(t *testing.T) {
    // Inject stub with error behavior
    mock := &MockStore{mockErr: errors.New("db down")}
    service := UserService{store: mock}

    _, err := service.GetUserName(1)
    if err == nil {
        t.Fatal("expected error, got nil")
    }
}
```

### Mocking Libraries
For complex interactions, hand-rolling mocks can be tedious. Popular libraries include:
*   **gomock**: Official Go mocking framework. Uses code generation (`mockgen`) to create type-safe mocks.
*   **testify/mock**: Part of the popular `testify` toolkit. Uses a dynamic approach.

#### Example with `testify/mock`
```go
type MockStore struct {
    mock.Mock
}

func (m *MockStore) GetUser(id int) (*User, error) {
    args := m.Called(id)
    return args.Get(0).(*User), args.Error(1)
}

// In test:
m := new(MockStore)
m.On("GetUser", 123).Return(&User{Name: "Alice"}, nil)
```

## Interview Questions

**Q: How do you mock a function in Go that doesn't belong to a struct (a standalone function)?**
**A:** You technically cannot "mock" a standalone function directly in Go without monkey-patching (which is discouraged). The idiomatic solution is to refactor the code to assign that function to a variable (e.g., `var TimeNow = time.Now`) so you can swap it in tests, or wrap it in an interface.

**Q: Why are interfaces crucial for testing in Go?**
**A:** Interfaces decouple the component under test from its concrete dependencies. This allows tests to substitute heavy, external dependencies (like databases or HTTP APIs) with lightweight, controllable mocks, making unit tests fast, deterministic, and isolated.

**Q: What is the difference between a Mock and a Stub?**
**A:** A Stub simply returns hardcoded data to facilitate the test (state verification). A Mock expects specific calls to be made with specific arguments and will fail the test if those expectations aren't met (behavior verification).
