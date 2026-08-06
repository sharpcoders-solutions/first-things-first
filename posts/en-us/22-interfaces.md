# 🧠 C# Fundamentals: Interfaces and Contracts

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Classes, objects, and encapsulation  
- Inheritance and polymorphism with `virtual`/`override`  
- Abstract classes as contracts within a hierarchy  

But not every relationship between classes is an inheritance relationship. Sometimes you just want to guarantee that **completely different classes** follow the same rule.

👉 **That's exactly what interfaces are for**

---

# 💡 What is an interface?

👉 **Interface = a contract that defines what a class must do, without saying how**

```csharp
interface IAnimal
{
    void MakeSound();
}
```

```csharp
class Dog : IAnimal
{
    public void MakeSound() => Console.WriteLine("Woof!");
}

class Robot : IAnimal
{
    public void MakeSound() => Console.WriteLine("Beep boop!");
}
```

👉 `Dog` and `Robot` have no inheritance relationship whatsoever — but both **fulfill the `IAnimal` contract**, committing to provide an implementation for the MakeSound() method.

Notice the convention: interfaces in C# start with a capital `I`. It's not mandatory, but it's practically universal across the .NET ecosystem.

---

# 🔀 Interface vs Abstract Class: the most common question

This is the question everyone asks when learning interfaces. Both look like "contracts," but they solve different problems.

## 🔹 Abstract class
- Can have **implementation** and **state** (fields, ready-made properties)  
- A class can only inherit from **one** abstract class  
- Makes sense when there's a strong "is-a" relationship with shared behavior  

## 🔹 Interface
- Only defines **what** must exist (by default, with no implementation of its own)  
- A class can implement **multiple** interfaces  
- Makes sense when unrelated classes need to follow the same rule  

```csharp
// Abstract class: EMPLOYEE is the base of an "is-a" relationship
abstract class Employee
{
    public string Name { get; set; }
    public abstract decimal CalculateSalary();
}

// Interfaces: capabilities with no hierarchical relationship
interface IAuditable
{
    void RecordAudit();
}

interface INotifiable
{
    void SendNotification(string message);
}

class Manager : Employee, IAuditable, INotifiable
{
    public override decimal CalculateSalary() => 8000;
    public void RecordAudit() => Console.WriteLine("Audit recorded");
    public void SendNotification(string message) => Console.WriteLine(message);
}
```

👉 `Manager` inherits from **one** class (`Employee`) and implements **multiple** interfaces (`IAuditable`, `INotifiable`) — something that wouldn't be possible with inheritance alone

---

# 🧩 Multiple interfaces: working around the lack of multiple inheritance

C# doesn't let a class inherit from two classes at once, but it does let you implement as many interfaces as you need:

```csharp
interface IFlyer
{
    void Fly();
}

interface ISwimmer
{
    void Swim();
}

class Duck : IFlyer, ISwimmer
{
    public void Fly() => Console.WriteLine("Duck flying");
    public void Swim() => Console.WriteLine("Duck swimming");
}
```

👉 This solves a real problem: what if an object needs to behave as two things at once, without those things having a hierarchical relationship?

---

# 📐 Interfaces as contracts: decoupling code

An interface's biggest value shows up when you program **against the contract**, not the concrete implementation:

```csharp
interface IPayment
{
    bool Process(decimal amount);
}

class CreditCardPayment : IPayment
{
    public bool Process(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} on credit card");
        return true;
    }
}

class PixPayment : IPayment
{
    public bool Process(decimal amount)
    {
        Console.WriteLine($"Processing ${amount} via Pix");
        return true;
    }
}

class Checkout
{
    private readonly IPayment _payment;

    public Checkout(IPayment payment)
    {
        _payment = payment;
    }

    public void Complete(decimal total)
    {
        _payment.Process(total);
    }
}
```

```csharp
var cardCheckout = new Checkout(new CreditCardPayment());
cardCheckout.Complete(150);

var pixCheckout = new Checkout(new PixPayment());
pixCheckout.Complete(150);
```

👉 `Checkout` doesn't know (and doesn't need to know) whether it's dealing with a credit card, Pix, or a bank slip — it only knows the `IPayment` contract

This is the foundation of a principle widely used on professional teams: **program to interfaces, not implementations**. It's this decoupling that makes code easier to test and extend without breaking what already exists.

---

# 🔌 Default interface methods (C# 8+)

Modern versions of C# let an interface have a default implementation:

```csharp
interface INotifiable
{
    void SendNotification(string message);

    void SendAlert() // method with a default implementation
    {
        SendNotification("⚠️ Default alert");
    }
}
```

👉 Useful for evolving interfaces without breaking classes that already implement them — but use it sparingly, an interface's main goal is still to define a contract, not behavior

---

# ⚠️ Common Mistakes

- Creating giant interfaces with too many methods ("god interface") — prefer small, focused interfaces  
- Confusing interfaces with abstract classes and using the wrong one for the problem  
- Forgetting to implement **all** the interface's members (the code simply won't compile)  
- Depending on the concrete implementation instead of the interface, losing the decoupling  

---

# 📌 Conclusion

- An interface defines **what** a class must do, without imposing **how**  
- A class can implement multiple interfaces, but inherit from only one class  
- Interfaces decouple code: consumers only need to know the contract  
- Abstract classes share implementation within an "is-a" relationship; interfaces guarantee behavior across unrelated classes  

👉 With interfaces, your code is ready to grow and change without breaking what already works

---

# 🔥 Next Step

Now that you understand contracts between classes, the next level is:

👉 **C# Fundamentals: Exception Handling (try, catch, finally, and Custom Exceptions)**

Here you'll learn to handle errors in a safe, professional way.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
