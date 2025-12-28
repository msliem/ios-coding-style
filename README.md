# SwiftUI Coding Style & Architecture Rules

> **Scope**

- SwiftUI Views
- View Models
- Contracts (Actions & State)
- Routers
- Reusable Components
- Use Cases (UC)
- File & Naming Conventions

---

## General Rules

- **Max file length**: `210 lines`
  - Applies to **Views**, **ViewModels**, and **Routers**
- Prefer **composition over inheritance**
- Keep **business logic out of Views**
- Use **extensions** to group related functionality
- Naming should optimize **call-site readability**

---

## SwiftUI View Structure

### File Ordering (Top → Bottom)

```swift
struct ExampleView: View {

    // 1. Environment
    @Environment(\.dismiss) private var dismiss

    // 2. Public / Private let
    private let title: String

    // 3. @State / Stored properties
    @State private var selectedIndex: Int?

    // 4. Computed vars (non-view)
    private var hasSelection: Bool { selectedIndex != nil }

    // 5. Init
    init(title: String) {
        self.title = title
    }

    // 6. Body
    var body: some View {
        mainContent
    }

    // 7. Computed View Builders / Helpers
    private var mainContent: some View {
        ...
    }
}
```

---

### Large Views

- When a view grows:
  - Move **view builders** to `extension ViewName`
  - Keep the main struct readable

```swift
extension ExampleView {
    var header: some View { ... }
    var footer: some View { ... }
}
```

---

## View Models Structure

### Property Ordering

```swift
final class ExampleVM: ObservableObject {

    // 1. Injected Dependencies
    @Injected(\.exampleUseCase) private var exampleUseCase

    // 2. Public properties
    @Published var viewState: ViewState

    // 3. Private properties
    private var cancellables = Set<AnyCancellable>()

    // 4. Computed properties
    private var canLoadMore: Bool { ... }

    // 5. Init
    init(viewState: ViewState = .init()) {
        self.viewState = viewState
    }

    // 6. onAction
    func onAction(_ action: Action) {
        ...
    }

    // 7. General lifecycle methods
    func onAppear() {
        ...
    }
}
```

---

### ViewModel Extensions

- Split by **responsibility**
- Naming pattern: `// MARK: - FeatureName`

```swift
// MARK: - GetSections
extension ExampleVM {

    func getSections() {
        ...
    }

    private func handleSections(_ result: Result<[Section], Error>) {
        ...
    }
}
```

**Rules**

- Public methods → first
- Private helpers → last
- Each extension should represent **one responsibility**

---

## Contract

### Actions

**Rules**

- Follow **SwiftUI naming conventions**
- No `Action` suffix
- Use `on` prefix for UI-driven events

```swift
enum Action {
    case onAppear
    case onTapMore(section)
}
```

**Associated Values**

- Optimize call-site readability

```swift
onTapMore(section)   // ✅ readable
onTapMore(index: section) // ❌ noisy
```

---

### State

#### Lists

- Use **descriptive plural names**

```swift
var sections: [Section]
var customers: [Customer]
```

❌ Avoid:

```swift
sectionsItems
updateSections
```

---

#### Bool Values

**UI State (loading / refreshing / pagination)**\
➡️ No `is` prefix

```swift
var loading: Bool
var refreshing: Bool
```

**Show / Hide UI**\
➡️ Use `show` prefix

```swift
var showNotificationIcon: Bool
var showAddressBanner: Bool
```

---

#### Single UI Models

- Use `Content` suffix

```swift
var headerContent: HeaderContent
var descriptionContent: DescriptionContent
```

---

#### State Helpers

- Helper methods belong to **ViewState**, not the ViewModel

```swift
struct ViewState {
    var items: [Item]

    func item(for id: Item.ID) -> Item? {
        items.first { $0.id == id }
    }
}
```

---

## Router

- **Protocol composition based**
- Keep navigation logic out of ViewModels
- Router should express **intent**, not implementation

```swift
protocol ExampleFlowRouting: BaseRouterProtocol {
    func navigateToDetails(item: Item)
    func presentDeleteConfirmation(confirm: @escaping () -> Void)
}
```

---

## Reusable Components — Naming

### Reusable Cells

```swift
ExpandableTitleItem
ExpandableTitleItemModel
```

---

### Fields (TextField)

```swift
PhoneField
EmailField
PasswordField
```

---

### General Views

```swift
ServiceDetailsView
AddressPickerView
```

---

## Naming Inside Views

### Empty State Handling

```swift
var mainContent: some View {
    if items.isEmpty {
        emptyState
    } else {
        content
    }
}
```

---

### Lists

```swift
var customersList: some View
var destinationsList: some View
```

---

### Buttons

- Separate **button** and **label** if label is complex
- Do NOT include `View` in naming

```swift
var serviceDetailsButton: some View {
    Button {
        ...
    } label: {
        serviceDetailsButtonLabel
    }
}

var serviceDetailsButtonLabel: some View {
    ...
}
```

---

### Custom Components

- Same rules as buttons
- Clear intent, no redundant suffixes

---

## Use Cases (UC)

- Group **2 or more related use cases** into a container

```swift
struct AddressUC {
    let fetchAddresses: FetchAddressesUseCase
    let deleteAddress: DeleteAddressUseCase
}
```

**Benefits**

- Cleaner dependency injection
- Easier mocking
- Scales better with features

