# asterisk

> A minimal Swift operator for inline configuration of objects using an infix style.

Small syntactic idea that turned out surprisingly clean. 

### The `.*` Operator

Custom infix operator `.*` applies a mutation closure to a value-type instance and returns the result.

Useful for situations where you want to set properties at creation without extra boilerplate.


```swift
infix operator .*: AdditionPrecedence

func .*<T>(lhs: T, rhs: (inout T) -> Void) -> T {
  var copy = lhs
  rhs(&copy)
  return copy
}
```

### Usages

#### View configuration

```swift

class SomeViewController: UIViewController {
  lazy var label: UILabel = {
    let label = UILabel()
    label.text = "Title"
    label.textColor = .white
    label.font = .boldSystemFont(ofSize: 20)
    label.numberOfLines = 0
    return label
  }
}

✨
class SomeViewController: UIViewController {
  lazy var label = UILabel() .* {
    $0.text = "Title"
    $0.textColor = .white
    $0.font = .boldSystemFont(ofSize: 20)
    $0.numberOfLines = 0
}
```

#### Asserting non equatable tuples without intermediate variables

```swift
func test_method_someBehaviourOnSomeEvent() async throws {

  let result = try await sut.getResult()
  
  let anyTuple = someExpectedTuple()
  XCTAssertEqual(result.value1, anyTuple.value1)
  XCTAssertEqual(result.value2, anyTuple.value2)
  XCTAssertEqual(result.value2, anyTuple.value2)
}

✨
func test_method_someBehaviourOnSomeEvent() async throws {

  let result = try await sut.getResult()
  
  someExpectedTuple() .* {
    XCTAssertEqual(result.value1, $0.value1)
    XCTAssertEqual(result.value2, $0.value2)
    XCTAssertEqual(result.value2, $0.value2)
  }
}

```

#### Unify object updates into a single api

```swift
let todo = Todo()

class CodableTodoStore {
  func checkTodo(id: UUID) {
    let newTodo = ...
    persist(newTodo)
  }
  
  func changeTitle(id: UUID, newTitle: STring) {
    let newTodo = ...
    persist(newTodo)
  }
  
  func asignProject(id: UUID, projectsName: String) {
    let newTodo = ...
    persist(newTodo)
  }
}

store.checkTodo(id: todo.id)
store.changeTitle(id: todo.id, newTitle: "New title")
store.asignProject(id: todo.id, projectsName: "Some project")

✨
class CodableTodoStore {
  func upsert(todo) {
    persist(todo)
  }
}

store.upsert(todo .* {$0.isChecked = true})
store.upsert(todo .* {$0.newTitle = "New title"})
store.upsert(todo .* {$0.parentProject = "Some project"})
```

### Theme configuration overriding only different values:

```swift
struct Theme {
  // Variable values between dark & light theme
  var background = Color.white
  var textColor = Color.black
  
  // Common values between dark & light theme
  var accentColor = Color.blue
  var cornerRadius: CGFloat = 10
}

let lightTheme = Theme()

✨ 
// Override only different values
let darkTheme = Theme() .* { 
  $0.background = .black
  $0.textColor = .white
}

let lightTheme = Theme()
let darkTheme = Theme() .* { 
  $0.background = .black
}

struct Main: View {
  @State theme = .lightTheme
  var body: some View {}
}
```

#### Intermediate transformations

```swift
func selectOnlyActiveUsers(users: [User]) {
  state = users
    .filter { $0.isActive }
    .map { user in
      var copy = user
      copy.isSelected = true
      return copy
    }
}

✨
func selectOnlyActiveUsers(users: [User]) {
  state = users
    .filter { $0.isActive }
    .map { $0 .* { $0.isSelected = true } }
}
```

### Init

Something like this can be achieved through protocol extensions:

```swift
let object = SomeObject { $0.someProperty = "new value" }
```

Implementation:

```swift
protocol Initiable {init()}
extension Initiable {
  init(transform: (inout Self) -> Void) {
    var copy = Self.init()
    transform(&copy)
    self = copy
  }
}

extension SomeObject: Initiable {}

let obj = SomeObject { $0.someProperty = "new value" }
```

While cool, that seems too esoteric and brings no real value (but the burden of having to conform to protocol on each time you want to use it)