class: center, middle

Welcome to another

# You can do FP in any language

talk

---
# Obligatory "What is FP" slide

--
- In functional programming, we care a lot about pure functions

--
- Functions that don't perform side-effects

--
- But how do we do things?

---
# Some imperative code

```
stdout.printLine("What is your name?")
name = stdin.getLine
stdout.printLine("Hello $name")
```

--

How do we test this?

---

Dependency injection, the OO way.

```java
public interface ConsoleService {
  public void printLine(String s);
  public String getLine();
}

class MyApp {
  private ConsoleService console;

  public MyApp(ConsoleService console) {
    this.console = console;
  }

  public void run() {
    console.printLine("What is your name?");
    String name = console.readLine();
    console.printLine("Hello, " + name);
  }
}
```

---

```java
class MyApp {
  private ConsoleService console;
  private DatabaseService database;
  private AuthenticationService authenticator;
  private MetricsService metrics;
  private LoggingService logger;

  public MyApp(
    ConsoleService console,
    DatabaseService database,
    AuthenticationService authenticator,
    MetricsService metrics,
    LoggingService logger,
  ) {
    this.console = console;
    this.database = database;
    this.authenticator = authenticator;
    this.metrics = metrics;
    this.logger = logger;
  }

  public void run() {
    ...
  }
}
```

---
name: refactoring-is-hard
# Refactoring is hard

--

```java
public void run() {
  console.printLine("What is your given name?");
  String givenName = console.readLine().capitalize();
  console.printLine("What is your surname?");
  String surname = console.readLine().capitalize();
  console
    .printLine("Hello, " + givenName + " " + surname);
}
```

---
template: refactoring-is-hard

```java
public void run() {
  String readName = console.readLine().capitalize();

  console.printLine("What is your given name?");
  String givenName = readName;
  console.printLine("What is your surname?");
  String surname = readName;
  console
    .printLine("Hello, " + givenName + " " + surname);
}
```

--
Spot the bug.

---
# What is the easist function to test?

--

```java
OutputType myFunction(InputType input) { ... }
```

--

A function that just transforms data

--

```java
assertEqual(expectedOutput, myFunction(someInput))
```

--

- No mocking, stubbing, spying

---
class: center, middle

# What if side-effects were just data?

---

```java
// "A" represents the return type produced from running
// the effect
public abstract class Effect<A> { ... }
```

--

```java
// "Void" means that this effect doesn't return any
// useful value
public class WriteLine extends Effect<Void> {
  // Constructor parameters become the inputs to our
  // side-effect
  public WriteLine(String s) { ... }
}
```

--

```java
// Running this effect would produce a string
public class ReadLine extends Effect<String> { ... }
```

---
class: center, middle

# They're just descriptions

---

```java
Effect<String> readName = new ReadLine();
```

--

How can we manipulate the value inside:

```java
Effect<Integer> nameLength =
  readName + (String name -> name.length());
```

--

How can we chain multiple effects together

```java
Effect<Void> writeHello = new WriteLine("Hello");
Effect<Void> twice = writeHello + writeHello
```

--

```java
Effect<String> readName = new ReadLine();
Effect<Void> program =
  readName + (name -> new WriteLine("Hello, " + name));
```

---
class: center, middle

Not side-effects, just descriptions

---
class: center, middle

Now we need to <em>interpret</em> them

---

<div class="description-vs-execution">
<div>
Most of your code
<div class="description">
Program description
</div>
</div>
<div class="execution">
End of the world
<div class="interpreter">
App Interpreter
</div>
<div class="interpreter">
Test Interpreter
</div>
<div class="interpreter">
Interpreter
</div>
</div>
</div>

---

# Visitor Pattern

https://en.wikipedia.org/wiki/Visitor_pattern

---
