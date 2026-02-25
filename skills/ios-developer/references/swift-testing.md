# Swift Testing

## Overview

Modern Swift Testing framework patterns for writing, reviewing, migrating, and debugging tests with emphasis on readability and parallel execution safety.

## Core Principles

1. **Framework choice** - Swift Testing for unit/integration; XCTest for UI automation
2. **Assertion defaults** - `#expect` as default, `#require` for prerequisites
3. **Parallelization-first** - Address shared state before using `.serialized`
4. **Use traits** - Metadata via traits, not naming conventions
5. **Parameterization** - Consolidate repetitive tests
6. **Import isolation** - `Testing` only in test targets

## Basic Test Structure

```swift
import Testing

@Test("User can log in with valid credentials")
func loginWithValidCredentials() async throws {
    let auth = AuthService()
    let user = try await auth.login(email: "test@example.com", password: "valid")
    #expect(user.isAuthenticated)
}

@Test("User cannot log in with invalid password")
func loginWithInvalidPassword() async throws {
    let auth = AuthService()
    await #expect(throws: AuthError.invalidCredentials) {
        try await auth.login(email: "test@example.com", password: "wrong")
    }
}
```

## Assertions

```swift
// Basic expectation
#expect(value == expected)
#expect(array.contains(element))
#expect(string.hasPrefix("Hello"))

// Required values (unwrapping)
let user = try #require(await fetchUser(id: "123"))
#expect(user.name == "John")

// Error expectations
await #expect(throws: NetworkError.self) {
    try await fetch(invalidURL)
}

// Known issues (instead of skipping)
withKnownIssue("Server returns wrong format") {
    #expect(response.format == .json)
}
```

## Parameterized Tests

```swift
@Test(arguments: ["admin", "user", "guest"])
func roleHasReadPermission(role: String) {
    let permissions = Permissions(role: role)
    #expect(permissions.canRead)
}

@Test(arguments: [
    (input: "", expected: false),
    (input: "valid@email.com", expected: true),
    (input: "invalid", expected: false)
])
func emailValidation(input: String, expected: Bool) {
    #expect(Email.isValid(input) == expected)
}
```

## Traits

```swift
// Tagging tests
@Test(.tags(.critical))
func criticalPathTest() { }

@Test(.tags(.slow))
func performanceTest() { }

// Conditional execution
@Test(.enabled(if: ProcessInfo.processInfo.environment["CI"] != nil))
func ciOnlyTest() { }

@Test(.disabled("Flaky on CI"))
func flakyTest() { }

// Time limits
@Test(.timeLimit(.minutes(1)))
func longRunningTest() async { }

// Bug tracking
@Test(.bug("https://github.com/org/repo/issues/123"))
func bugReproduction() { }
```

## Test Suites

```swift
@Suite("Authentication Tests")
struct AuthTests {
    let auth = AuthService()

    @Test func loginSuccess() async throws { }
    @Test func loginFailure() async throws { }
    @Test func logout() { }
}

// Serialized suite for shared state
@Suite(.serialized)
struct DatabaseTests {
    @Test func createRecord() { }
    @Test func readRecord() { }
    @Test func deleteRecord() { }
}
```

## Async Testing

```swift
@Test func asyncDataFetch() async throws {
    let service = DataService()
    let data = try await service.fetch()
    #expect(!data.isEmpty)
}

@Test func confirmationPattern() async {
    await confirmation("Callback received") { confirm in
        NotificationCenter.default.addObserver(
            forName: .dataUpdated,
            object: nil,
            queue: .main
        ) { _ in
            confirm()
        }
        triggerUpdate()
    }
}
```

## XCTest Migration

| XCTest | Swift Testing |
|--------|---------------|
| `XCTestCase` | `@Suite` struct |
| `func testXxx()` | `@Test func xxx()` |
| `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| `XCTAssertThrowsError` | `#expect(throws:)` |
| `XCTUnwrap(optional)` | `try #require(optional)` |
| `XCTSkip` | `.disabled()` trait |
| `setUp/tearDown` | init/deinit |

## External References

For comprehensive Swift Testing guidance, see:

- **[Swift-Testing-Agent-Skill](https://github.com/AvdLee/Swift-Testing-Agent-Skill)** - Complete Swift Testing patterns with migration guides
