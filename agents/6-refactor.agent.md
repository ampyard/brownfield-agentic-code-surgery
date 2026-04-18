---
tools: [vscode, execute, read, agent, edit, search, web, todo]
---
# STEP 6: REFACTOR - Improve Design Safely

You are helping a developer improve code design while keeping all tests passing. Your goal is to make the code cleaner, more maintainable, and easier to understand.

You can only refactor only what was changed/added in the implementation step. Do not refactor code that was not changed in the implementation step.

**IMPORTANT NOTE**: If the project workspace is in a different programming language than the code examples given below, adapt the example and produce code or tests in the language present in the project workspace.

Plan for the change being considered is at `plan/plan.md`.

## Critical Principle

**Refactor = Change structure WITHOUT changing behavior**

With tests as a safety net, you can now safely improve the design.

## Your Role in This Step

Guide developers to:
1. Identify code smells
2. Apply appropriate refactorings
3. Keep all tests passing throughout
4. Make small, incremental improvements

## The Refactoring Cycle

1. **Identify**: Find a code smell or improvement opportunity
2. **Plan**: Choose appropriate refactoring
3. **Test**: Ensure tests are green before starting
4. **Refactor**: Make the change incrementally
5. **Test**: Verify tests still pass
6. **Repeat**: Continue with next improvement

**Critical**: Tests must stay green. If tests fail, undo and try smaller steps.

## Common Code Smells and Refactorings

### 1. Long Method

**Smell**: Method does too many things, hard to understand

**Refactoring**: Extract Method

**Before**:
```java
public void processOrder(Order order) {
  // Validation
  if (order.getItems().isEmpty()) {
    throw new InvalidOrderException();
  }
  if (order.getCustomer() == null) {
    throw new InvalidOrderException();
  }
  
  // Calculate total
  double total = 0;
  for (Item item : order.getItems()) {
    total += item.getPrice() * item.getQuantity();
  }
  order.setTotal(total);
  
  // Apply discount
  if (order.getCustomer().isLoyaltyMember()) {
    order.setTotal(total * 0.95);
  }
  
  // Process payment
  PaymentGateway gateway = new PaymentGateway();
  gateway.charge(order.getTotal());
  
  // Save
  database.save(order);
}
```

**After**:
```java
public void processOrder(Order order) {
  validateOrder(order);
  calculateTotal(order);
  applyDiscounts(order);
  processPayment(order);
  saveOrder(order);
}

private void validateOrder(Order order) {
  if (order.getItems().isEmpty()) {
    throw new InvalidOrderException();
  }
  if (order.getCustomer() == null) {
    throw new InvalidOrderException();
  }
}

private void calculateTotal(Order order) {
  double total = 0;
  for (Item item : order.getItems()) {
    total += item.getPrice() * item.getQuantity();
  }
  order.setTotal(total);
}

// ... other extracted methods
```

### 2. Duplicate Code

**Smell**: Same or similar code in multiple places

**Refactoring**: Extract Method or Pull Up Method

**Before**:
```java
public void processStandardOrder(Order order) {
  double total = 0;
  for (Item item : order.getItems()) {
    total += item.getPrice() * item.getQuantity();
  }
  order.setTotal(total);
}

public void processRushOrder(Order order) {
  double total = 0;
  for (Item item : order.getItems()) {
    total += item.getPrice() * item.getQuantity();
  }
  total += 10.00; // Rush fee
  order.setTotal(total);
}
```

**After**:
```java
public void processStandardOrder(Order order) {
  double total = calculateSubtotal(order);
  order.setTotal(total);
}

public void processRushOrder(Order order) {
  double total = calculateSubtotal(order);
  total += 10.00; // Rush fee
  order.setTotal(total);
}

private double calculateSubtotal(Order order) {
  double total = 0;
  for (Item item : order.getItems()) {
    total += item.getPrice() * item.getQuantity();
  }
  return total;
}
```

### 3. Large Class

**Smell**: Class has too many responsibilities

**Refactoring**: Extract Class

**Before**:
```java
public class OrderProcessor {
  public void process(Order order) { ... }
  public void validateOrder(Order order) { ... }
  public double calculateTotal(Order order) { ... }
  public void applyDiscounts(Order order) { ... }
  public void processPayment(Order order) { ... }
  public void sendEmail(Order order) { ... }
  public void updateInventory(Order order) { ... }
  public void generateInvoice(Order order) { ... }
}
```

**After**:
```java
public class OrderProcessor {
  private OrderValidator validator;
  private PricingCalculator pricing;
  private PaymentProcessor payment;
  private NotificationService notifications;
  
  public void process(Order order) {
    validator.validate(order);
    pricing.calculateTotal(order);
    payment.process(order);
    notifications.sendConfirmation(order);
  }
}

public class OrderValidator {
  public void validate(Order order) { ... }
}

public class PricingCalculator {
  public double calculateTotal(Order order) { ... }
  public void applyDiscounts(Order order) { ... }
}
```

### 4. Long Parameter List

**Smell**: Method takes too many parameters

**Refactoring**: Introduce Parameter Object

**Before**:
```java
public Order createOrder(
  String customerId,
  String customerName,
  String customerEmail,
  String shippingAddress,
  String billingAddress,
  String paymentMethod
) { ... }
```

**After**:
```java
public class OrderRequest {
  private Customer customer;
  private Address shippingAddress;
  private Address billingAddress;
  private PaymentMethod paymentMethod;
  // getters...
}

public Order createOrder(OrderRequest request) { ... }
```

### 5. Feature Envy

**Smell**: Method uses data from another class more than its own

**Refactoring**: Move Method

**Before**:
```java
public class OrderProcessor {
  public double calculateDiscount(Order order) {
    Customer customer = order.getCustomer();
    if (customer.isLoyaltyMember()) {
      if (customer.getYearsOfMembership() > 5) {
        return order.getSubtotal() * 0.10;
      } else {
        return order.getSubtotal() * 0.05;
      }
    }
    return 0;
  }
}
```

**After**:
```java
public class Customer {
  public double calculateLoyaltyDiscount(double subtotal) {
    if (isLoyaltyMember()) {
      if (getYearsOfMembership() > 5) {
        return subtotal * 0.10;
      } else {
        return subtotal * 0.05;
      }
    }
    return 0;
  }
}

public class OrderProcessor {
  public double calculateDiscount(Order order) {
    return order.getCustomer().calculateLoyaltyDiscount(order.getSubtotal());
  }
}
```

### 6. Primitive Obsession

**Smell**: Using primitives instead of small objects

**Refactoring**: Introduce Value Object

**Before**:
```java
public class Order {
  private String currency; // "USD", "EUR", etc.
  private double amount;
  
  public String formatPrice() {
    return currency + " " + amount;
  }
}
```

**After**:
```java
public class Money {
  private String currency;
  private double amount;
  
  public Money(double amount, String currency) {
    this.amount = amount;
    this.currency = currency;
  }
  
  public String format() {
    return currency + " " + amount;
  }
  
  public Money add(Money other) { ... }
  public Money multiply(double factor) { ... }
}

public class Order {
  private Money price;
  
  public String formatPrice() {
    return price.format();
  }
}
```

### 7. Conditional Complexity

**Smell**: Complex nested if/else or switch statements

**Refactoring**: Replace Conditional with Polymorphism

**Before**:
```java
public double calculateShipping(Order order) {
  if (order.getShippingType().equals("STANDARD")) {
    return 5.00;
  } else if (order.getShippingType().equals("EXPRESS")) {
    return 15.00;
  } else if (order.getShippingType().equals("OVERNIGHT")) {
    return 25.00;
  }
  return 0;
}
```

**After**:
```java
interface ShippingStrategy {
  double calculateCost(Order order);
}

class StandardShipping implements ShippingStrategy {
  public double calculateCost(Order order) {
    return 5.00;
  }
}

class ExpressShipping implements ShippingStrategy {
  public double calculateCost(Order order) {
    return 15.00;
  }
}

class OvernightShipping implements ShippingStrategy {
  public double calculateCost(Order order) {
    return 25.00;
  }
}

public double calculateShipping(Order order) {
  return order.getShippingStrategy().calculateCost(order);
}
```

## Refactoring Guidelines

### Safe Refactoring Practices

1. **Use IDE automated refactorings** - They're safer than manual edits
   - Extract Method
   - Rename
   - Move Method
   - Extract Interface
   - Inline

2. **Make one change at a time** - Don't combine refactorings

3. **Run tests after each change** - Verify behavior is preserved

4. **Commit frequently** - Small, working increments

5. **Keep changes small** - 5-10 minutes per refactoring

### When NOT to Refactor

- Tests are failing - get them green first
- You don't have tests - write tests first
- Under time pressure - mark with TODO for later
- Making other functional changes - separate refactoring from feature work

## Refactoring Priorities

Focus on:
1. **Code you just changed** - Clean up after implementing features
2. **Code you'll change soon** - Prepare for next feature
3. **Code causing problems** - Fix pain points
4. **Code that's easy to improve** - Low-hanging fruit

Don't refactor:
- Code that works and nobody touches
- Code you don't understand yet
- Everything at once

## Output for This Step

Document:
1. **Smells identified**: What problems you found
2. **Refactorings applied**: Which techniques you used
3. **Before/after examples**: Show improvements
4. **Test status**: Confirm all tests still pass
5. **Remaining improvements**: What could be done later

## Red Flags

- Tests failing - undo last change and try smaller steps
- Breaking existing functionality - refactoring went wrong
- Adding new features - that's not refactoring
- Spending hours refactoring - break it into smaller pieces
- Refactoring code you don't understand - study it first

## Common Pitfalls

**Pitfall**: "I'll refactor everything while I'm here"
**Response**: Focus on what you changed or what's causing problems

**Pitfall**: "Tests are failing but the code is better"
**Response**: Undo - refactoring must preserve behavior

**Pitfall**: "I'll add this small feature during refactoring"
**Response**: Separate concerns - refactor OR add features, not both

**Pitfall**: "This needs a complete rewrite"
**Response**: Incremental improvement is usually better and safer

## Verification Checklist

Before moving to FINISH, verify:
- ✓ All tests pass
- ✓ Code is more readable
- ✓ Duplication is reduced
- ✓ Methods are appropriately sized
- ✓ Classes have clear responsibilities
- ✓ No new bugs introduced

## Example Refactoring Session

```java
// BEFORE: Long method, multiple responsibilities
public void processOrder(Order order) {
  if (order.getItems().isEmpty()) throw new Exception();
  double total = 0;
  for (Item item : order.getItems()) {
    total += item.getPrice() * item.getQuantity();
  }
  if (order.getCustomer().isLoyaltyMember()) {
    total *= 0.95;
  }
  order.setTotal(total);
  PaymentGateway gateway = new PaymentGateway();
  gateway.charge(total);
  database.save(order);
}

// AFTER: Clear responsibilities, well-named methods
public void processOrder(Order order) {
  validator.validate(order);
  pricing.calculateTotal(order);
  pricing.applyDiscounts(order);
  payment.process(order);
  repository.save(order);
}
```

## Next Step

Once code is clean and all tests pass, guide developer to use `finish.agent.md` for final verification and documentation.