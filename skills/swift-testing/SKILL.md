---
name: swift_testing
description: Modern Swift Testing framework patterns and best practices
---

# Swift Testing

You are a Swift testing expert using the modern Swift Testing framework (not XCTest). Apply these patterns when writing tests.

## Framework Basics

### Import and Structure

```swift
import Testing
@testable import MyApp

struct UserTests {
    @Test func createsUserWithValidEmail() {
        let user = User(email: "test@example.com")
        #expect(user.isValid)
    }
}
```

### Key Differences from XCTest

| XCTest | Swift Testing |
|--------|---------------|
| `class` with `XCTestCase` | `struct` (preferred) or `class` |
| `func testSomething()` | `@Test func something()` |
| `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| `XCTAssertTrue(x)` | `#expect(x)` |
| `XCTAssertNil(x)` | `#expect(x == nil)` |
| `XCTAssertThrowsError` | `#expect(throws:)` |
| `setUpWithError()` | `init()` |
| `tearDown()` | `deinit` |

## Expectations

### Basic Expectations

```swift
@Test func basicExpectations() {
    let value = 42

    // Boolean
    #expect(value > 0)
    #expect(value == 42)

    // Optionals
    let optional: String? = "hello"
    #expect(optional != nil)

    // Collections
    let array = [1, 2, 3]
    #expect(array.contains(2))
    #expect(array.count == 3)
}
```

### Require (Unwrapping)

```swift
@Test func requireUnwrapping() throws {
    let optional: String? = "hello"

    // Unwrap or fail test
    let value = try #require(optional)
    #expect(value == "hello")

    // With custom message
    let user = try #require(fetchUser(), "User should exist")
    #expect(user.isActive)
}
```

### Error Expectations

```swift
@Test func errorHandling() {
    // Expect any error
    #expect(throws: (any Error).self) {
        try riskyOperation()
    }

    // Expect specific error type
    #expect(throws: ValidationError.self) {
        try validate(email: "invalid")
    }

    // Expect specific error value
    #expect(throws: ValidationError.invalidEmail) {
        try validate(email: "")
    }
}
```

## Test Organization

### Suites (Grouping)

```swift
@Suite("User Authentication")
struct AuthenticationTests {

    @Suite("Login")
    struct LoginTests {
        @Test func validCredentials() { }
        @Test func invalidPassword() { }
    }

    @Suite("Registration")
    struct RegistrationTests {
        @Test func newUser() { }
        @Test func duplicateEmail() { }
    }
}
```

### Setup and Teardown

```swift
struct DatabaseTests {
    let database: Database

    // Setup in init
    init() async throws {
        database = try await Database.createTest()
    }

    @Test func insertsRecord() async throws {
        try await database.insert(record)
        #expect(await database.count == 1)
    }
}
```

## Parameterized Tests

### Basic Parameterization

```swift
@Test(arguments: ["hello", "world", "swift"])
func stringIsNotEmpty(value: String) {
    #expect(!value.isEmpty)
}

@Test(arguments: 1...10)
func positiveNumbers(n: Int) {
    #expect(n > 0)
}
```

### Multiple Parameters

```swift
@Test(arguments: [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0)
])
func addition(a: Int, b: Int, expected: Int) {
    #expect(a + b == expected)
}
```

### Zip vs Product

```swift
// Zip: pairs elements (3 tests)
@Test(arguments: zip([1, 2, 3], ["a", "b", "c"]))
func zipped(number: Int, letter: String) { }

// Product: all combinations (9 tests)
@Test(arguments: [1, 2, 3], ["a", "b", "c"])
func product(number: Int, letter: String) { }
```

## Tags and Traits

### Tagging Tests

```swift
extension Tag {
    @Tag static var slow: Self
    @Tag static var network: Self
    @Tag static var critical: Self
}

@Test(.tags(.slow, .network))
func networkRequest() async { }

@Test(.tags(.critical))
func paymentProcessing() { }
```

### Display Names

```swift
@Test("User can update profile picture")
func updateProfilePicture() { }

@Suite("Payment Processing")
struct PaymentTests { }
```

### Conditional Tests

```swift
// Disable with reason
@Test(.disabled("Waiting for API update"))
func upcomingFeature() { }

// Platform-specific
@Test(.enabled(if: ProcessInfo.processInfo.environment["CI"] != nil))
func ciOnly() { }

// Runtime conditions
@Test
func featureFlagTest() throws {
    try #require(FeatureFlags.newCheckout, "Feature not enabled")
    // Test new checkout...
}
```

### Time Limits

```swift
@Test(.timeLimit(.minutes(1)))
func longRunningOperation() async { }

@Test(.timeLimit(.seconds(5)))
func quickOperation() { }
```

## Async Testing

### Async Tests

```swift
@Test func asyncFetch() async throws {
    let service = DataService()
    let result = try await service.fetchData()
    #expect(!result.isEmpty)
}
```

### Actor Testing

```swift
@Test func actorState() async {
    let counter = Counter()
    await counter.increment()
    await counter.increment()

    let value = await counter.value
    #expect(value == 2)
}
```

### Confirmation (Async Expectations)

```swift
@Test func notificationReceived() async {
    await confirmation { confirm in
        NotificationCenter.default.addObserver(
            forName: .userLoggedIn,
            object: nil,
            queue: nil
        ) { _ in
            confirm()
        }

        await loginUser()
    }
}

// Multiple confirmations
@Test func multipleEvents() async {
    await confirmation(expectedCount: 3) { confirm in
        eventHandler.onEvent = { confirm() }
        triggerThreeEvents()
    }
}
```

## Best Practices

### Naming

```swift
// Good: describes behavior
@Test func calculatesDiscountForPremiumUsers() { }
@Test func throwsErrorWhenEmailInvalid() { }
@Test func returnsEmptyArrayWhenNoResults() { }

// Avoid: vague or implementation-focused
@Test func testDiscount() { }  // What about discount?
@Test func testMethod() { }    // Which method?
```

### Arrange-Act-Assert

```swift
@Test func appliesCouponCode() throws {
    // Arrange
    let cart = ShoppingCart()
    cart.addItem(Product(price: 100))
    let coupon = Coupon(code: "SAVE20", discount: 0.20)

    // Act
    try cart.apply(coupon)

    // Assert
    #expect(cart.total == 80)
    #expect(cart.appliedCoupons.contains(coupon))
}
```

### One Concept Per Test

```swift
// Good: focused tests
@Test func addsItemToCart() {
    cart.add(item)
    #expect(cart.items.contains(item))
}

@Test func updatesCartTotal() {
    cart.add(item)
    #expect(cart.total == item.price)
}

// Avoid: testing multiple concepts
@Test func cartOperations() {
    cart.add(item)
    #expect(cart.items.contains(item))
    cart.remove(item)
    #expect(cart.items.isEmpty)
    cart.add(item)
    cart.add(item2)
    #expect(cart.total == item.price + item2.price)
}
```

### Test Fixtures

```swift
struct OrderTests {
    // Shared fixtures
    let validProduct = Product(id: "1", name: "Widget", price: 9.99)
    let customer = Customer(id: "c1", tier: .standard)

    @Test func createsOrder() {
        let order = Order(customer: customer, items: [validProduct])
        #expect(order.isValid)
    }

    @Test func calculatesShipping() {
        let order = Order(customer: customer, items: [validProduct])
        #expect(order.shippingCost == 5.99)
    }
}
```

## Migration from XCTest

Swift Testing was introduced at WWDC 2024 and ships with Xcode 16. XCTest is not deprecated - migrate incrementally at your own pace.

### What Can Be Migrated

- Unit tests with assertions
- Async/await tests
- Tests using setUp/tearDown

### What Cannot Be Migrated

- **UI tests** using XCUIApplication
- **Performance tests** using XCTMetric
- **Objective-C tests**

Continue using XCTest for these cases.

### Critical Rule

**Never mix frameworks within a single test.** Don't call XCTAssert from Swift Testing or #expect from XCTest.

### Complete Migration Table

| XCTest | Swift Testing |
|--------|---------------|
| `class MyTests: XCTestCase` | `struct MyTests` |
| `func testSomething()` | `@Test func something()` |
| `override func setUp()` | `init()` |
| `override func tearDown()` | `deinit` |
| `XCTAssert(x)` | `#expect(x)` |
| `XCTAssertTrue(x)` | `#expect(x)` |
| `XCTAssertFalse(x)` | `#expect(!x)` |
| `XCTAssertNil(x)` | `#expect(x == nil)` |
| `XCTAssertNotNil(x)` | `#expect(x != nil)` |
| `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| `XCTAssertNotEqual(a, b)` | `#expect(a != b)` |
| `XCTAssertGreaterThan(a, b)` | `#expect(a > b)` |
| `XCTAssertLessThan(a, b)` | `#expect(a < b)` |
| `XCTUnwrap(x)` | `try #require(x)` |
| `XCTFail("message")` | `Issue.record("message")` |
| `XCTAssertThrowsError` | `#expect(throws:)` |
| `XCTAssertNoThrow` | `#expect(throws: Never.self)` |
| `expectation` + `fulfill` + `wait` | `await confirmation` |

## Running Tests

### Command Line

```bash
# Run all tests
swift test

# Run specific suite
swift test --filter UserTests

# Run with tags
swift test --filter "tag:critical"

# Parallel execution (default)
swift test --parallel
```

### Xcode

- `Cmd+U` - Run all tests
- `Cmd+Ctrl+U` - Run test at cursor
- Test Navigator shows results organized by suite
