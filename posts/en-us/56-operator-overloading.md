# 🧠 C# Fundamentals: Operator Overloading

⏱️ Reading time: 8 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- How `foreach` works under the hood, via `IEnumerable<T>`  
- `yield return` and lazy evaluation in custom iterators  

You've used `+` to add up `int`s and `decimal`s your whole life. But did you know you can teach the compiler to understand `+`, `==`, `<`, and other operators for **your own types**? That's what operator overloading does.

👉 **Let's learn to overload operators in C#**

---

# 💡 The problem: operators don't make sense on custom types by default

```csharp
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
}

var a = new Money(10, "USD");
var b = new Money(20, "USD");

var total = a + b; // ❌ Compile error: operator + doesn't exist for Money
```

👉 The compiler doesn't know, by default, what "adding" two `Money` objects means — you need to teach it explicitly

---

# ➕ Overloading the `+` operator

```csharp
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Cannot add different currencies");

        return new Money(a.Amount + b.Amount, a.Currency);
    }
}

var a = new Money(10, "USD");
var b = new Money(20, "USD");
var total = a + b; // ✅ now it works: Money { Amount = 30, Currency = "USD" }
```

👉 **`operator +` = a special static method that defines what the `+` symbol means when applied to your type**

The body is regular C# code — nothing stops you from validating business rules (like preventing adding different currencies) inside the operator itself

---

# ⚖️ Overloading comparisons: `==` and `!=`

```csharp
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    // ... constructor ...

    public static bool operator ==(Money a, Money b)
    {
        if (a is null || b is null)
            return a is null && b is null;

        return a.Amount == b.Amount && a.Currency == b.Currency;
    }

    public static bool operator !=(Money a, Money b) => !(a == b);

    public override bool Equals(object obj) => obj is Money other && this == other;
    public override int GetHashCode() => HashCode.Combine(Amount, Currency);
}
```

👉 **Mandatory rule: whenever you overload `==` and `!=`, you also need to override `Equals` and `GetHashCode`** — all three need to agree with each other, or you end up with a type whose equality behavior is inconsistent depending on how it's compared

Remember `record`s? They do exactly this automatically — generating consistent `==`, `Equals`, and `GetHashCode` under the hood, without you writing any of it manually

---

# 🔢 Relational comparison operators: `<`, `>`, `<=`, `>=`

```csharp
public class Money : IComparable<Money>
{
    public decimal Amount { get; }

    // ...

    public static bool operator <(Money a, Money b) => a.Amount < b.Amount;
    public static bool operator >(Money a, Money b) => a.Amount > b.Amount;
    public static bool operator <=(Money a, Money b) => a.Amount <= b.Amount;
    public static bool operator >=(Money a, Money b) => a.Amount >= b.Amount;

    public int CompareTo(Money other) => Amount.CompareTo(other.Amount);
}

var list = new List<Money> { /* ... */ };
list.Sort(); // works because Money implements IComparable<Money>
```

👉 Implementing `IComparable<T>` alongside the relational operators lets your type work with `List<T>.Sort()`, LINQ's `OrderBy`, and any API that expects an ordering comparison

---

# 🎯 Custom conversions: `implicit` and `explicit`

```csharp
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency) { Amount = amount; Currency = currency; }

    public static implicit operator decimal(Money money) => money.Amount;
    public static explicit operator Money(decimal amount) => new Money(amount, "USD");
}

decimal amount = new Money(100, "USD"); // implicit: automatic conversion, no cast
Money money = (Money)50m;                // explicit: requires a cast, because it assumes "USD"
```

👉 **Practical rule: use `implicit` only when the conversion never loses information and can never fail. Use `explicit` when the conversion is "risky" or assumes something** — like in the example above, converting `decimal` to `Money` requires assuming a currency, which is risky enough to demand an explicit cast

---

# ⚠️ Common Mistakes

- Overloading `==` without overriding `Equals` and `GetHashCode`, creating a type with inconsistent equality behavior  
- Using `implicit` for conversions that can fail or lose precision, surprising whoever uses the type  
- Overloading operators on types that don't represent a natural mathematical or comparison concept, making the code more confusing, not clearer  
- Forgetting that `Money a = null` makes `a == b` call the overloaded operator — without a null check first, this throws `NullReferenceException`  

---

# 📌 Conclusion

- Overloaded operators are special static methods that teach the compiler how to handle `+`, `==`, `<` on your type  
- Overloading `==`/`!=` also requires overriding `Equals` and `GetHashCode`, to stay consistent  
- `IComparable<T>` alongside relational operators enables native ordering (`Sort`, `OrderBy`)  
- `implicit` for safe conversions; `explicit` for conversions that demand explicit attention from the caller  

👉 Custom operators make your types behave like first-class citizens of the language — the next feature enables something similar: making your types accessible with brackets, like an array

---

# 🔥 Next Step

Now that you know how to teach operators to your types, the next level is:

👉 **C# Fundamentals: Indexers on Custom Types**

Here you'll learn to use the `object[index]` syntax on your own types, the same way you already use it on arrays and lists.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
