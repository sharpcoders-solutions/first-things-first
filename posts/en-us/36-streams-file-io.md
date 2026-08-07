# 🧠 C# Fundamentals: Streams and File I/O in C#

⏱️ Reading time: 5 minutes  
✍️ Author: Vitor Santos + 🤖 Copilot

## 🚀 Introduction

So far, you've learned:

- IDisposable and the Dispose pattern  
- Collections (post 18) — arrays, lists, dictionaries in memory  

Everything you've stored in memory so far disappears when the application stops. Time to read and write data that survives beyond the process — files on disk.

👉 **Let's learn Streams and File I/O**

---

# 💡 What is a Stream?

👉 **Stream = a sequence of bytes, read or written incrementally, without needing to load everything into memory at once**

```csharp
using FileStream stream = new FileStream("data.txt", FileMode.Open);
```

👉 Remember the previous post about IDisposable? `FileStream` is the classic example of an unmanaged resource — always used inside a `using`

---

# 🏗️ The simple way: `File` for common cases

```csharp
// Write text
File.WriteAllText("log.txt", "Application started");

// Read text
string content = File.ReadAllText("log.txt");

// Read line by line, without loading the whole file
foreach (var line in File.ReadLines("log.txt"))
{
    Console.WriteLine(line);
}
```

👉 The static `File` class abstracts `FileStream` under the hood for simple operations — for small files, you rarely need to manipulate the Stream directly

---

# 🎯 `StreamReader` and `StreamWriter`: text over byte streams

```csharp
using var writer = new StreamWriter("orders.csv");
writer.WriteLine("Id,Customer,Amount");
foreach (var order in orders)
{
    writer.WriteLine($"{order.Id},{order.Customer},{order.Amount}");
}
```

```csharp
using var reader = new StreamReader("orders.csv");
string? line;
while ((line = reader.ReadLine()) is not null)
{
    Console.WriteLine(line);
}
```

👉 A file is fundamentally bytes — `StreamReader`/`StreamWriter` add the text-encoding layer (UTF-8, by default) on top of the raw `FileStream`

---

# 📦 Processing large files without blowing up memory

```csharp
// ❌ Loads the entire file into memory at once
var allLines = File.ReadAllLines("10gb-file.csv");

// ✅ Processes one line at a time, constant memory
using var reader = new StreamReader("10gb-file.csv");
string? line;
while ((line = reader.ReadLine()) is not null)
{
    ProcessLine(line);
}
```

👉 Remember the ArrayPool post (70)? The same principle of avoiding unnecessary allocation applies here — loading an entire 10GB file into memory to process it line by line is wasteful; streams process incrementally

---

# ⚡ Asynchronous operations in file I/O

```csharp
public async Task SaveLogAsync(string message)
{
    await using var writer = new StreamWriter("log.txt", append: true);
    await writer.WriteLineAsync(message);
}
```

👉 Remember the async/await post? Disk I/O, just like network I/O, is a natural candidate for `async` — the thread doesn't sit blocked waiting for the disk to respond, it's freed up for other work in the meantime

---

# 🔄 Copying streams: file to file, or between types

```csharp
using var source = new FileStream("source.zip", FileMode.Open);
using var destination = new FileStream("destination.zip", FileMode.Create);

await source.CopyToAsync(destination);
```

👉 `CopyToAsync` works between **any** combination of streams — file to file, file to `HttpResponse`, memory to file — because they all inherit from the same base `Stream` class, regardless of where the data comes from

---

# ⚠️ Common Mistakes

- Using `File.ReadAllText`/`ReadAllLines` on large files, unnecessarily loading gigabytes into memory  
- Forgetting `using` on `FileStream`, `StreamReader`, or `StreamWriter`, leaving the file locked for other processes  
- Using synchronous methods (`ReadLine()`, `WriteLine()`) in API code, blocking the thread unnecessarily  
- Not explicitly specifying the encoding when reading files from external sources, causing corrupted characters (accented letters, for example)  

---

# 📌 Conclusion

- Streams process data incrementally, without requiring everything to fit in memory at once  
- `StreamReader`/`StreamWriter` add the text layer on top of the byte-level `FileStream`  
- File I/O operations should be `async`, following the same principle as network I/O  
- `CopyToAsync` works between any combination of streams, not just files  

👉 With Streams, you process files of any size efficiently, without compromising the application's memory

---

# 🔥 Next Step

Now that you can handle files efficiently, the next level is:

👉 **C# Fundamentals: Structured Logging with Serilog**

Here you'll learn to record what happens in your application in a structured, searchable way.

## ✍️ Author's Note

This article was produced in collaboration between **Vitor Santos** and an AI assistant (Copilot), combining practical experience with technological support to create high-quality technical content.
