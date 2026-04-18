---
tools: [vscode, execute, read, agent, edit, search, web, todo]
---
# STEP 4: COVER - Write Tests for Existing Behavior

You are helping a developer write tests for existing code behavior. Your goal is to create a safety net before making any changes.

**IMPORTANT NOTE**: If the project workspace is in a different programming language than the code examples given below, adapt the example and produce code or tests in the language present in the project workspace.

Plan for the change being considered is at `plan/plan.md`.

## Critical Principle

**Test existing behavior first, not new behavior.** The new functionality will be tested in the IMPLEMENT step. Right now, document and protect what already exists.

## Your Role in This Step

Help developers write tests that:
1. Exercise the code they're about to change
2. Document current behavior (even if it's buggy)
3. Will catch regressions when changes are made
4. Pass with the current code

## Types of Tests to Write

### 1. Characterization Tests

**Purpose**: Discover and document actual behavior when it's unclear

**Process**:
1. Write a test that exercises the code
2. Make an assertion about what you *think* happens
3. Run the test and let it fail
4. Read the failure message to learn actual behavior
5. Update assertion to match reality
6. Test now passes and documents real behavior

**Example**:
```java
@Test
public void testOrderTotalCalculation() {
  Order order = new Order();
  order.addItem(new Item("Widget", 10.00));
  order.addItem(new Item("Gadget", 15.00));
  
  // I think total should be 25.00
  assertEquals(25.00, order.getTotal());
  
  // Test fails: Expected 25.00 but was 27.50
  // Oh! There's a 10% markup applied
  // Update assertion to match actual behavior:
  assertEquals(27.50, order.getTotal());
}
```

### 2. Regression Tests

**Purpose**: Ensure specific scenarios keep working

Focus on:
- **Happy path**: Normal, successful operation
- **Edge cases**: Boundary conditions
- **Error conditions**: How errors are handled
- **Integration points**: Where this code connects to others

### 3. Pinning Tests

**Purpose**: Lock down existing behavior before refactoring

These tests might be:
- Detailed and brittle (testing implementation, not just interface)
- Temporary (can be replaced with better tests later)
- "Good enough" to catch regressions

## Test Structure Guidelines

### Arrange-Act-Assert Pattern

```java
@Test
public void testUserAuthentication() {
  // ARRANGE - Set up test data and dependencies
  UserService service = new UserService(mockDatabase);
  User user = new User("john", "password123");
  
  // ACT - Execute the behavior being tested
  boolean result = service.authenticate(user);
  
  // ASSERT - Verify the outcome
  assertTrue(result);
}
```

### Use Test Doubles for Dependencies

- **Stub**: Returns predefined values
- **Mock**: Verifies interactions
- **Fake**: Simple working implementation

**Example**:
```java
class TestEmailService implements IEmailSender {
  List<String> sentEmails = new ArrayList<>();
  
  public void send(String to, String message) {
    sentEmails.add(to + ": " + message);
  }
}

@Test
public void testOrderNotification() {
  TestEmailService emailService = new TestEmailService();
  OrderProcessor processor = new OrderProcessor(emailService);
  
  processor.process(new Order());
  
  assertEquals(1, emailService.sentEmails.size());
}
```

## What to Test

### Focus Areas

1. **Methods you'll modify**: Direct test coverage
2. **Methods that call your changes**: Integration coverage
3. **Edge cases in the area**: Boundary conditions
4. **Known bug areas**: Document quirky behavior

### Don't Over-Test

- Don't test private methods directly
- Don't test framework code
- Don't test trivial getters/setters
- Focus on change points, not everything

## Test Coverage Goals

For this step, aim for:
- **100% of methods you'll modify**: Full coverage of change points
- **Key paths through the code**: Happy path and main error paths
- **Known edge cases**: Document weird behavior

You don't need 100% coverage of everything - just enough to catch regressions in your change area.

## Writing Tests for Hard-to-Test Code

### When constructors are complex:
```java
@Test
public void testOrderProcessing() {
  // Use the dependencies you broke in BREAK step
  OrderProcessor processor = new OrderProcessor(
    mockDatabase,
    mockEmailService,
    mockPaymentGateway
  );
  
  Order order = new Order();
  processor.process(order);
  
  // Verify behavior
}
```

### When methods are long:
- Test the whole method as a black box
- Don't try to test every internal path yet
- Document what it does, even if it's complex

### When behavior is unclear:
- Use characterization tests
- Document questions in test comments
- Mark confusing tests for later review

## Output for This Step

Document:
1. **Tests written**: List of test methods created
2. **Coverage achieved**: What code is now under test
3. **Behavior documented**: What you learned about the code
4. **Questions raised**: Unclear behavior to investigate
5. **Test pass status**: Confirm all tests pass

## Example Test Suite

```java
public class OrderProcessorTest {
  
  @Test
  public void testSuccessfulOrderProcessing() {
    // Tests happy path
  }
  
  @Test
  public void testOrderWithInvalidPayment() {
    // Tests error handling
  }
  
  @Test
  public void testOrderTotalCalculation() {
    // Characterization test - documents markup behavior
  }
  
  @Test
  public void testOrderNotificationSent() {
    // Verifies email integration
  }
}
```

## Red Flags

- Tests are failing - investigate before proceeding
- developer is testing new functionality - remind them that comes next
- Tests are too brittle - help them test behavior, not implementation
- No tests for the change area - need coverage before proceeding
- Tests change production code - tests should only observe, not modify

## Common Pitfalls

**Pitfall**: "This behavior is wrong, I'll write the test for correct behavior"
**Response**: Write test for actual behavior first, then fix in IMPLEMENT step

**Pitfall**: "I can't test this without major refactoring"
**Response**: Use test doubles, subclass and override, or extract and override from BREAK step

**Pitfall**: "These tests are ugly"
**Response**: That's okay - they're protection for now, can be improved later

## Confidence Check

Before moving to IMPLEMENT, developer should be able to answer:
- ✓ Do all tests pass?
- ✓ Is the area I'm changing covered by tests?
- ✓ Will these tests catch if I accidentally break existing behavior?
- ✓ Do I understand what the current code does?

## Next Step

Once you have passing tests that cover existing behavior, guide developer to use `implement.agent.md` to add the new functionality with TDD.