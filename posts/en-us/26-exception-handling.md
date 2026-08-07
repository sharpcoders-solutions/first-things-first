# 🧠 C# Fundamentals: Exception Handling (try, catch, finally, and Custom Exceptions)

⏱️ Reading time: 7 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Classes, inheritance, polymorphism, and interfaces  
- How to model contracts and behavior between objects  

But no real program runs error-free. Files don't exist, APIs go down, users type invalid values.

👉 **The difference between an amateur app and a professional one lies in how it handles that**

---

# 💡 What is an exception?

👉 **Exception = an unexpected event that interrupts the normal flow of the program**

```csharp
int[] numbers = { 1, 2, 3 };
Console.WriteLine(numbers[5]); // IndexOutOfRangeException
```

Without handling, an exception **crashes the program**. With handling, you decide what to do when something goes wrong.

---

# 🏗️ `try`, `catch`, and `finally`

```csharp
try
{
    int result = 10 / int.Parse("0");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    Console.WriteLine("This always runs, with or without an error");
}
```

## 🔹 How it works

- `try` → where you put the code that might fail  
- `catch` → catches the exception and decides what to do  
- `finally` → runs **always**, whether there was an error or not (great for releasing resources)  

👉 `finally` runs even if there's a `return` inside the `try` or the `catch`

---

# 🎯 Catching specific types

You can have multiple `catch` blocks, from most specific to most generic:

```csharp
try
{
    string text = null;
    Console.WriteLine(text.Length);
}
catch (NullReferenceException ex)
{
    Console.WriteLine("Null reference: " + ex.Message);
}
catch (Exception ex)
{
    Console.WriteLine("Unexpected error: " + ex.Message);
}
```

👉 C# tests `catch` blocks **in the order they appear** — that's why the most specific one should always come before the generic one

## 🔹 The most common exceptions day to day

- `NullReferenceException` → accessing a member of something that's `null`  
- `IndexOutOfRangeException` → accessing an index that doesn't exist  
- `InvalidOperationException` → an invalid operation for the object's current state  
- `ArgumentException` / `ArgumentNullException` → an invalid argument passed to a method  
- `DivideByZeroException` → division by zero on integer types  

---

# ✋ Throwing your own exceptions

```csharp
void Withdraw(decimal amount, decimal balance)
{
    if (amount > balance)
    {
        throw new InvalidOperationException("Insufficient balance");
    }
}
```

👉 `throw` immediately stops execution and propagates the exception to whoever called the method

---

# 🧱 Custom exceptions

When .NET's built-in exceptions don't describe your domain's problem well, you can create your own:

```csharp
class InsufficientBalanceException : Exception
{
    public decimal Balance { get; }
    public decimal RequestedAmount { get; }

    public InsufficientBalanceException(decimal balance, decimal requestedAmount)
        : base($"Insufficient balance: available ${balance}, requested ${requestedAmount}")
    {
        Balance = balance;
        RequestedAmount = requestedAmount;
    }
}
```

```csharp
void Withdraw(decimal amount, decimal balance)
{
    if (amount > balance)
        throw new InsufficientBalanceException(balance, amount);
}

try
{
    Withdraw(500, 100);
}
catch (InsufficientBalanceException ex)
{
    Console.WriteLine(ex.Message);
    Console.WriteLine($"Missing: ${ex.RequestedAmount - ex.Balance}");
}
```

👉 Custom exceptions inherit from `Exception` and can carry extra data relevant to whoever handles the error

---

# 🔁 `throw` vs `throw ex`: a detail that changes everything

```csharp
catch (Exception ex)
{
    throw;      // ✅ preserves the original stack trace
    // throw ex; // ❌ resets the stack trace, making diagnosis harder
}
```

👉 Use bare `throw;` to re-throw the exception without losing information about where it actually happened

---

# 🧹 `using`: a safer alternative to `finally` for resources

For resources that implement `IDisposable` (files, connections, streams), there's a cleaner way to guarantee release:

```csharp
using (var file = new StreamReader("data.txt"))
{
    string content = file.ReadToEnd();
} // file is released automatically here, even if an error occurs
```

👉 `using` calls `Dispose()` automatically — equivalent to an implicit `try/finally`

---

# ⚠️ Best Practices (and Common Mistakes)

- **Don't swallow exceptions silently**: an empty `catch` hides problems instead of solving them  
- **Don't use exceptions for normal control flow**: they're expensive and should represent genuinely exceptional situations  
- **Avoid catching generic `Exception`** when you know which specific error might occur  
- **Always prefer `throw;`** over `throw ex;` when re-throwing  
- **Release resources with `using`**, instead of relying on remembering to call `Dispose()`  

```csharp
// ❌ Never do this
try
{
    ProcessOrder();
}
catch (Exception)
{
    // nothing here — the error silently disappears
}
```

---

# 📌 Conclusion

- `try/catch/finally` structures how your program reacts to errors  
- Catch specific exceptions before generic ones  
- `throw` creates and propagates an exception; bare `throw;` re-throws while preserving the stack trace  
- Custom exceptions describe your domain's errors more clearly  
- `using` guarantees resource release more safely than a manual `finally`  

👉 Handling exceptions well is what separates code that only "works on the happy path" from code that's ready for production

---

# 🔥 Next Step

Now that your code knows how to handle the unexpected, the next level is:

👉 **C# Fundamentals: Generics**

Here you'll learn to write reusable, type-safe code for any kind of data.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
