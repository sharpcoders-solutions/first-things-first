# 🧠 C# Fundamentals: Inheritance and Polymorphism

⏱️ Reading time: 9 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- Classes and objects  
- Fields, properties, constructors, and methods  
- Encapsulation as the first pillar of OOP  

Now let's move on to the two pillars that give OOP its real power for reuse and flexibility:

👉 **Inheritance and Polymorphism**

Just like in the previous post, take your time here — this is the kind of concept that separates someone who just "writes C#" from someone who truly understands object-oriented programming.

---

# 💡 What is Inheritance?

👉 **Inheritance = one class reusing fields, properties, and methods from another**

Instead of repeating code across similar classes, you extract what's common into a base class:

```csharp
class Animal
{
    public string Name { get; set; }

    public void Sleep()
    {
        Console.WriteLine($"{Name} is sleeping.");
    }
}

class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine($"{Name} is barking.");
    }
}
```

```csharp
Dog rex = new Dog();
rex.Name = "Rex";       // inherited from Animal
rex.Sleep();              // inherited from Animal
rex.Bark();                // specific to Dog
```

👉 `Dog` **is an** `Animal` — this "is-a" relationship is the simplest test to check whether inheritance makes sense

---

# 🔑 The base class constructor: `base`

When the base class has a constructor, the child class needs to call it:

```csharp
class Animal
{
    public string Name { get; }

    public Animal(string name)
    {
        Name = name;
    }
}

class Dog : Animal
{
    public Dog(string name) : base(name)
    {
        // Dog's constructor can add more logic here
    }
}
```

👉 `: base(name)` guarantees that `Animal`'s constructor runs before `Dog`'s

---

# 🔒 `protected`: access granted to whoever inherits

Remember `private` from the previous post? There's a middle ground:

```csharp
class Animal
{
    protected int Energy = 100;
}

class Dog : Animal
{
    public void Run()
    {
        Energy -= 10; // accessible because it's protected, not private
    }
}
```

👉 `protected` is invisible to the outside world, but visible to classes that inherit from it

---

# ⚙️ Overriding behavior: `virtual` and `override`

This is where one of the most common mistakes for people learning inheritance happens.

```csharp
class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Generic animal sound");
    }
}

class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}

class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Meow!");
    }
}
```

👉 `virtual` on the base class **allows** the child class to override the method with `override`

⚠️ Without `virtual` on the original method, `override` in the child class **won't compile** — and if you use `new` instead of `override`, the method is just "hidden," not truly overridden (more on this below).

---

# 🧩 Polymorphism: the magic of treating everything uniformly

👉 **Polymorphism = treating objects of different types through a common interface, while letting each behave its own way**

```csharp
List<Animal> animals = new List<Animal>
{
    new Dog(),
    new Cat(),
    new Animal()
};

foreach (Animal animal in animals)
{
    animal.MakeSound();
}

// Woof!
// Meow!
// Generic animal sound
```

👉 The `foreach` doesn't know (and doesn't need to know) whether it's dealing with a `Dog` or a `Cat` — each object knows how to run its own version of `MakeSound()`

This is called **runtime polymorphism**: C# decides which method to run based on the object's **actual type**, not the type of the variable referencing it.

---

# ⚠️ The trap: `override` vs `new`

This is the most silent mistake in C# inheritance:

```csharp
class Animal
{
    public virtual void MakeSound() => Console.WriteLine("Generic sound");
}

class Cat : Animal
{
    public new void MakeSound() => Console.WriteLine("Meow!"); // "new", not "override"
}
```

```csharp
Animal animal = new Cat();
animal.MakeSound(); // prints "Generic sound" — not "Meow!"
```

👉 With `new`, the method is just **hidden**, not overridden. The actual behavior depends on the **variable's type**, not the object's — breaking polymorphism

**Practical rule:** if the intent is to override behavior, always use `override` (and never `new`, unless you know exactly why you're doing it)

---

# 🚫 `sealed`: preventing further inheritance

```csharp
sealed class Robot
{
    // no one can inherit from Robot
}

class AdvancedRobot : Animal
{
    public sealed override void MakeSound() // no one can override this again
    {
        Console.WriteLine("Beep boop");
    }
}
```

👉 Use `sealed` when you want to guarantee a class (or a specific `override`) is the final version, with no further specialization

---

# 🧱 Abstract classes: molds that can't become objects

Sometimes the base class doesn't make sense on its own — it only exists to be inherited:

```csharp
abstract class Shape
{
    public abstract double CalculateArea(); // no body — each child implements it
}

class Circle : Shape
{
    public double Radius { get; set; }

    public override double CalculateArea() => Math.PI * Radius * Radius;
}

class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public override double CalculateArea() => Width * Height;
}
```

```csharp
// Shape shape = new Shape(); // ❌ error: cannot instantiate an abstract class

List<Shape> shapes = new List<Shape>
{
    new Circle { Radius = 2 },
    new Rectangle { Width = 3, Height = 4 }
};

foreach (Shape shape in shapes)
{
    Console.WriteLine(shape.CalculateArea());
}
```

👉 `abstract` forces every child class to implement that behavior — it's a way of enforcing a "contract" within inheritance itself

---

# ⚠️ Common Mistakes

- Forgetting `virtual` on the base method and not understanding why `override` won't compile  
- Using `new` instead of `override`, silently breaking polymorphism  
- Building overly deep inheritance chains (`A : B : C : D`), hard to understand and maintain  
- Inheriting just to reuse code, without a real "is-a" relationship  
- Trying to instantiate an `abstract` class directly  

---

# 📌 Conclusion

- Inheritance reuses fields, properties, and methods across related classes  
- `base` calls the parent class's constructor (or members)  
- `virtual` + `override` allow truly overriding behavior  
- Polymorphism lets each object run its own version of a behavior through a common type  
- `abstract` classes define contracts that child classes are required to fulfill  

👉 With inheritance and polymorphism, your code stops repeating logic and starts modeling real relationships between concepts

---

# 🔥 Next Step

Now that you understand inheritance and polymorphism, the next level is:

👉 **C# Fundamentals: Interfaces and Contracts**

Here you'll learn an even more flexible way to enforce behavior between classes that don't have a direct inheritance relationship.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
