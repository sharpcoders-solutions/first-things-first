# 🧠 C# Fundamentals: Classes and Objects (Introduction to Object-Oriented Programming)

⏱️ Reading time: 10 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Variables, types, conditionals, and loops  
- Methods, parameters, and return values  
- Collections, LINQ, date/time types, numeric precision, and regex  

All of that is tooling. But so far, your code organizes **actions** — it doesn't organize **real-world concepts**.

👉 **That's the role of Object-Oriented Programming (OOP), and this is where C# really shows its strength**

Since this is one of the most important topics in a C# developer's career, let's take it slow and go deep — it's worth reading carefully.

---

# 💡 What is Object-Oriented Programming?

👉 **OOP = organizing code around "objects" that combine data and behavior**

Instead of thinking in terms of "functions that process loose data," you start thinking in terms of **entities**: a `Customer`, an `Order`, a `BankAccount`. Each one has:

- **Data** that describes it (state)  
- **Actions** it can perform (behavior)  

👉 This brings code closer to how we naturally think about the real world

---

# 🏗️ Class vs Object: the most important concept in this post

This is the foundation of everything, and where most beginners get confused.

## 🔹 Class

👉 **Class = a mold, a blueprint**

```csharp
class Person
{
    public string Name;
    public int Age;
}
```

The class **is not** a person. It only defines **how** a person is structured: that every person has a name and an age.

## 🔹 Object

👉 **Object = a real instance, created from the class**

```csharp
Person person1 = new Person();
person1.Name = "Maria";
person1.Age = 25;

Person person2 = new Person();
person2.Name = "João";
person2.Age = 30;
```

👉 From **one** class, you can create **infinite** different objects, each with its own state

Think of it this way: `Person` is the blueprint for a house. `person1` and `person2` are two houses built from it — similar in structure, but with different occupants.

---

# 🧱 Anatomy of a class

A well-built class usually has four parts:

## 🔹 1. Fields

```csharp
class BankAccount
{
    private decimal balance;
}
```

👉 Fields hold the object's internal state. By convention, they're usually `private` — only the class itself touches them directly

## 🔹 2. Properties

```csharp
class BankAccount
{
    private decimal balance;

    public decimal Balance
    {
        get { return balance; }
        private set { balance = value; }
    }
}
```

👉 Properties control **how** the outside world reads and writes the object's state — without exposing the field directly

## 🔹 Auto-properties (the most common form day to day)

When there's no extra logic on read/write, C# allows a leaner version:

```csharp
class Person
{
    public string Name { get; set; }
    public int Age { get; private set; }
}
```

👉 `get; set;` generates the hidden field automatically — you don't need to declare it yourself

## 🔹 3. Constructors

The constructor defines how an object is **born**:

```csharp
class Person
{
    public string Name { get; }
    public int Age { get; }

    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

Person person = new Person("Maria", 25);
```

👉 If you don't declare any constructor, C# generates a default (parameterless) one automatically — but as soon as you declare your own, the default one disappears

## 🔹 The role of `this`

```csharp
public Person(string name, int age)
{
    this.Name = name; // "this" refers to the current object
    this.Age = age;
}
```

👉 `this` is mainly useful when the parameter name matches the field/property name

## 🔹 4. Methods

Methods define the object's **behavior**:

```csharp
class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Invalid deposit amount");

        Balance += amount;
    }

    public bool Withdraw(decimal amount)
    {
        if (amount > Balance) return false;

        Balance -= amount;
        return true;
    }
}
```

👉 Notice how the business rules (don't withdraw more than the balance, don't deposit a negative amount) live **inside** the class, close to the data they protect

---

# 🔒 Encapsulation: the most immediate pillar

👉 **Encapsulation = hiding internal details and exposing only what's necessary**

Compare the two versions of the bank account:

```csharp
// ❌ Without encapsulation
class BadBankAccount
{
    public decimal Balance;
}

account.Balance = -500; // nothing stops this
```

```csharp
// ✅ With encapsulation
class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentException("Invalid amount");
        Balance += amount;
    }
}

account.Balance = -500; // compile error — Balance only changes via Deposit/Withdraw
```

👉 Encapsulation isn't about hiding for its own sake — it's about **guaranteeing the object never ends up in an invalid state**

---

# ⚙️ Static members vs instance members

Not everything in a class needs a created object:

```csharp
class Calculator
{
    public static int Sum(int a, int b) => a + b; // static: belongs to the class
    public int History { get; set; }               // instance: belongs to the object
}

int result = Calculator.Sum(2, 3); // no "new" needed
```

## 🔹 When to use `static`

- Behavior that **doesn't depend** on a specific object's state  
- Utilities, constants, helper methods  

👉 If the method needs to access data that varies per object, it shouldn't be `static`

---

# 🔑 Access modifiers

They control **who** can see each member of the class:

- `public` → accessible from anywhere  
- `private` → accessible only inside the class itself  
- `protected` → accessible in the class and in classes that inherit from it  
- `internal` → accessible within the same project/assembly  

👉 General rule: start as restrictive as possible (`private`) and only open access (`public`) when actually necessary

---

# 🧩 The four pillars of OOP (overview)

Classes and objects are the entry point. From here, OOP rests on four pillars:

1. **Encapsulation** → protecting internal state (you just saw this today)  
2. **Abstraction** → exposing only what matters, hiding complexity  
3. **Inheritance** → reusing behavior across related classes  
4. **Polymorphism** → treating different objects uniformly  

👉 We'll dedicate entire posts to inheritance and polymorphism next — they deserve depth of their own

---

# 🏗️ Full example: putting it all together

```csharp
class BankAccount
{
    public string Owner { get; }
    public decimal Balance { get; private set; }

    public BankAccount(string owner, decimal initialBalance = 0)
    {
        Owner = owner;
        Balance = initialBalance;
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Invalid deposit amount");

        Balance += amount;
    }

    public bool Withdraw(decimal amount)
    {
        if (amount <= 0 || amount > Balance) return false;

        Balance -= amount;
        return true;
    }

    public override string ToString() => $"{Owner}: ${Balance}";
}

var account = new BankAccount("Vitor", 1000);
account.Deposit(500);
account.Withdraw(200);

Console.WriteLine(account); // Vitor: $1300
```

👉 Notice how every concept from this post shows up here: fields, properties, constructor, methods, encapsulation, and even `this` (implicit in the properties)

---

# ⚠️ Common Mistakes

- Leaving fields `public` instead of exposing controlled properties  
- Forgetting that without a custom constructor, C# generates an empty one automatically  
- Confusing `static` members with instance members  
- Placing business rules outside the class instead of close to the data they protect  
- Thinking "class" and "object" are synonyms — they're different concepts  

---

# 📌 Conclusion

- A class is the mold; an object is the real instance created from it  
- Fields hold state; properties control access to that state  
- Constructors define how an object is born  
- Methods define behavior, and should protect the integrity of the data  
- Encapsulation guarantees the object never ends up in an invalid state  

👉 You just took the most important step in a C# career: thinking in objects, not just loose instructions

---

# 🔥 Next Step

Now that you understand classes and objects, the next level is:

👉 **C# Fundamentals: Inheritance and Polymorphism**

Here you'll learn to reuse behavior across classes and treat different objects uniformly.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
