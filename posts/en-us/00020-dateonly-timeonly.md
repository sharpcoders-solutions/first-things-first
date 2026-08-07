# 🧠 C# Fundamentals: DateOnly and TimeOnly

⏱️ Reading time: 6 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- LINQ for querying and transforming collections declaratively  
- `DateTime`, used since the earliest posts whenever an entity needs to record when something happened  

You've probably modeled a "date of birth" or a "store opening time" using `DateTime`. But `DateTime` always carries date **and** time together — even when your domain only needs one of the two parts.

👉 **Let's get to know `DateOnly` and `TimeOnly`**

---

# 💡 `DateTime`'s historical problem

```csharp
public class Person
{
    public DateTime DateOfBirth { get; set; } // 1990-05-15 00:00:00 — the time means nothing here
}

public class Store
{
    public DateTime OpeningTime { get; set; } // 0001-01-01 09:00:00 — the date means nothing here
}
```

👉 Modeling a date of birth with `DateTime` always leaves a time component (`00:00:00`) with no real meaning. Modeling a store's opening time with `DateTime` always requires a "phantom" date (usually `0001-01-01`) nobody ever uses. This has caused real bugs: comparisons that fail because of the time part, time zones applied where they make no sense

---

# 📅 `DateOnly`: just the date

```csharp
public class Person
{
    public DateOnly DateOfBirth { get; set; }
}

var birth = new DateOnly(1990, 5, 15);
var today = DateOnly.FromDateTime(DateTime.Now);

int age = today.Year - birth.Year;
if (birth > today.AddYears(-age)) age--;

Console.WriteLine(birth.ToString("MM/dd/yyyy")); // 05/15/1990
```

👉 **`DateOnly` = a `struct` that exclusively represents year, month, and day — no time, no time zone, no ambiguity**

Comparisons between two dates of birth never fail again because of different seconds or milliseconds — the type simply has no room to carry that information

---

# ⏰ `TimeOnly`: just the time

```csharp
public class Store
{
    public TimeOnly OpeningTime { get; set; }
    public TimeOnly ClosingTime { get; set; }
}

var opening = new TimeOnly(9, 0);
var closing = new TimeOnly(18, 30);

var now = TimeOnly.FromDateTime(DateTime.Now);
bool isOpen = now >= opening && now <= closing;

var duration = closing - opening; // TimeSpan of 9h30
```

👉 **`TimeOnly` = a `struct` that exclusively represents hour, minute, second — with no associated date**

Perfect for recurring schedules (store hours, a daily reminder time) that repeat every day, without depending on which specific day it is

---

# 🔄 Converting between `DateTime`, `DateOnly`, and `TimeOnly`

```csharp
var now = DateTime.Now;

DateOnly justDate = DateOnly.FromDateTime(now);
TimeOnly justTime = TimeOnly.FromDateTime(now);

// Recombining both back into a DateTime
DateTime recombined = justDate.ToDateTime(justTime);
```

👉 Conversion in both directions is straightforward — you can keep using `DateTime` where it makes sense (audit records, complete timestamps, from the structured logging post) and use `DateOnly`/`TimeOnly` where your domain's semantics call for exactly one of the parts

---

# 🗄️ Mapping in Entity Framework Core

```csharp
public class Event
{
    public int Id { get; set; }
    public DateOnly Date { get; set; }
    public TimeOnly StartTime { get; set; }
}
```

👉 Remember EF Core (post 35)? Since EF Core 6, `DateOnly` and `TimeOnly` are natively mapped to SQL Server's `date` and `time` types — no need for custom converters, and no waste storing a date or time part that will never be used

---

# ⚠️ Common Mistakes

- Continuing to use `DateTime` with the time part always zeroed out "out of habit," when `DateOnly` communicates the intent far more clearly  
- Mixing comparisons between `DateTime` and `DateOnly`/`TimeOnly` without converting explicitly, causing avoidable compile errors  
- Ignoring `DateOnly`/`TimeOnly` in legacy projects just because "it's always been `DateTime`," missing the chance to eliminate an entire class of time-zone and ambiguity bugs  
- Using `TimeOnly` to store a duration — `TimeOnly` represents a point in time within a day, not an interval; for duration, `TimeSpan` remains the right type  

---

# 📌 Conclusion

- `DateOnly` exclusively represents year, month, and day, with no time component  
- `TimeOnly` exclusively represents hour, minute, and second, with no associated date  
- Both eliminate time-zone and comparison bugs caused by a `DateTime`'s unused part  
- EF Core natively maps both to the database's `date`/`time` types, since version 6  

👉 With more precise date and time types, the next step is looking at another category of numeric types that also deserves more care than it usually gets: floating-point and decimal types

---

# 🔥 Next Step

Now that you model dates and times with precision, the next level is:

👉 **C# Fundamentals: Numeric Types and Precision (float, double, and decimal)**

Here you'll understand why `0.1 + 0.2` doesn't exactly equal `0.3` with `double`, and when each numeric type should actually be used.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
