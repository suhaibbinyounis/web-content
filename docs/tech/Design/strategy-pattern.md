---
categories:
- Software Design
- Best Practices
- Object-Oriented Programming
comments: true
cover:
  image: https://images.pexels.com/photos/18069488/pexels-photo-18069488.png?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-18 15:56:31.477000
description: A concise and practical guide to the Strategy Design Pattern, explaining
  how to define a family of algorithms, encapsulate each one, and make them interchangeable.
math: true
tags:
- Design Patterns
- Strategy Pattern
- Software Design
- Polymorphism
- Behavioral Patterns
title: Strategy Pattern Encapsulating Interchangeable Algorithms
---

![Abstract representation of AI ethics with pills on a clear pathway, symbolizing data sorting.](https://images.pexels.com/photos/18069488/pexels-photo-18069488.png?auto=compress&cs=tinysrgb&h=650&w=940 "Abstract representation of AI ethics with pills on a clear pathway, symbolizing data sorting.")


## Introduction

The Strategy Pattern is a behavioral design pattern that enables an algorithm's behavior to be selected at runtime. It defines a family of algorithms, encapsulates each one, and makes them interchangeable. This pattern allows the client to choose an algorithm from a family of algorithms without exposing the implementation details of those algorithms.

It is particularly useful in scenarios where:
*   A class has varying behaviors that can be selected dynamically.
*   An algorithm needs to be interchangeable at runtime.
*   Conditional statements (e.g., `if-else` or `switch-case`) are used to select between multiple related behaviors.
*   You want to avoid tightly coupling the client to specific algorithm implementations.

## Implementation

The Strategy Pattern typically involves three key components:

1.  **Strategy Interface/Abstract Class**: Declares an interface common to all supported algorithms. The Context uses this interface to call the algorithm defined by a Concrete Strategy.
2.  **Concrete Strategy**: Implements the Strategy interface, defining a specific algorithm.
3.  **Context**: Maintains a reference to a Concrete Strategy object and interacts with it through the Strategy interface. The Context does not know which concrete strategy it is using.

Consider a payment processing system where different payment methods (e.g., Credit Card, PayPal) need to be handled.

```csharp
// 1. Strategy Interface
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}

// 2. Concrete Strategies
public class CreditCardPayment : IPaymentStrategy
{
    private string _cardNumber;
    private string _expiryDate;

    public CreditCardPayment(string cardNumber, string expiryDate)
    {
        _cardNumber = cardNumber;
        _expiryDate = expiryDate;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paying {amount:C} with Credit Card (ending in {_cardNumber.Substring(_cardNumber.Length - 4)}).");
        // Simulate credit card processing logic
    }
}

public class PayPalPayment : IPaymentStrategy
{
    private string _email;

    public PayPalPayment(string email)
    {
        _email = email;
    }

    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paying {amount:C} with PayPal (account: {_email}).");
        // Simulate PayPal processing logic
    }
}

// 3. Context
public class PaymentProcessor
{
    private IPaymentStrategy _paymentStrategy;

    public void SetPaymentStrategy(IPaymentStrategy strategy)
    {
        _paymentStrategy = strategy;
    }

    public void ProcessPayment(decimal amount)
    {
        if (_paymentStrategy == null)
        {
            throw new InvalidOperationException("Payment strategy not set.");
        }
        Console.WriteLine($"Initiating payment process for {amount:C}...");
        _paymentStrategy.Pay(amount);
        Console.WriteLine("Payment process completed.");
    }
}

// Client Usage
public class Client
{
    public static void Main(string[] args)
    {
        PaymentProcessor processor = new PaymentProcessor();

        // Use Credit Card strategy
        Console.WriteLine("--- Using Credit Card ---");
        processor.SetPaymentStrategy(new CreditCardPayment("1234-5678-9012-3456", "12/25"));
        processor.ProcessPayment(100.00m);

        Console.WriteLine("\n--- Using PayPal ---");
        // Use PayPal strategy
        processor.SetPaymentStrategy(new PayPalPayment("user@example.com"));
        processor.ProcessPayment(50.50m);
    }
}
```

## Mermaid Diagram

```mermaid
graph TD;
    A[Client] --> B[PaymentProcessor (Context)];
    B -- sets/uses --> C[IPaymentStrategy];
```

## Pros & Cons

### Advantages

*   **Open/Closed Principle (OCP)**: New strategies can be added without modifying the Context class.
*   **Encapsulation of Algorithms**: Each algorithm is self-contained within its own class, leading to cleaner code.
*   **Runtime Flexibility**: Allows algorithms to be selected or changed dynamically during program execution.
*   **Eliminates Conditional Logic**: Replaces bulky conditional statements (`if/else`, `switch`) with polymorphic behavior.
*   **Improved Testability**: Individual strategies can be tested independently.

### Disadvantages

*   **Increased Number of Objects**: Introducing the pattern often results in a higher number of classes/objects in the design.
*   **Client Responsibility**: The client must be aware of the different concrete strategies to select the appropriate one, unless a factory pattern is used to abstract this.
*   **Overhead for Simple Algorithms**: For very simple algorithms, the overhead of creating a new class for each strategy might outweigh the benefits.

## References

*   Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.
*   Refactoring.Guru. (n.d.). *Strategy*. Retrieved from [https://refactoring.guru/design-patterns/strategy](https://refactoring.guru/design-patterns/strategy)