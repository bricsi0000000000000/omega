- [`IEnumerable` vs `List`](#ienumerable-vs-list)
- [Round a Decimal Value to 2 Decimal Places](#round-a-decimal-value-to-2-decimal-places)
- [Double - ToString() formatting with two decimal places but no rounding](#double---tostring-formatting-with-two-decimal-places-but-no-rounding)
- [Change symbol for decimal point in `double.ToString()`](#change-symbol-for-decimal-point-in-doubletostring)
- [Operators](#operators)
  - [Negation operator `!`](#negation-operator-)
  - [Null-forgiving operator `!`](#null-forgiving-operator-)
  - [Logical AND operator `&`](#logical-and-operator-)
  - [Logical exclusive OR operator `^`](#logical-exclusive-or-operator-)
  - [Logical OR operator `|`](#logical-or-operator-)
  - [Conditional logical AND operator `&&`](#conditional-logical-and-operator-)
  - [Conditional logical OR operator `||`](#conditional-logical-or-operator-)
  - [Nullable Boolean logical operators](#nullable-boolean-logical-operators)
  - [Compound assignment](#compound-assignment)
  - [Operator precedence](#operator-precedence)
  - [Null-coalescing operator `??`](#null-coalescing-operator-)
  - [Index from end operator `^`](#index-from-end-operator-)
  - [Range operator `..`](#range-operator-)
- [Attributes](#attributes)
  - [NotNullWhenAttribute](#notnullwhenattribute)
- [Interesting things](#interesting-things)

# `IEnumerable` vs `List`

> [Source](https://stackoverflow.com/questions/3628425/ienumerable-vs-list-what-to-use-how-do-they-work#3628705)

`IEnumerable` describes behavior, while **`List` is an implementation** of that behavior.

When you use `IEnumerable`, you give the compiler a chance to **defer work until later**, possibly optimizing along the way. If you use `ToList()` you force the compiler to reify the results right away.

*"Stacking"* **LINQ** expressions, use `IEnumerable`, because by only specifying the behavior you give **LINQ** a chance to defer evaluation and possibly optimize the program.

**LINQ** doesn't generate the SQL to query the database until you enumerate it.

```cs
public IEnumerable<Animals> AllSpotted()
{
  return from a in Zoo.Animals
         where a.coat.HasSpots == true
         select a;
}

public IEnumerable<Animals> Feline(IEnumerable<Animals> sample)
{
  return from a in sample
         where a.race.Family == "Felidae"
         select a;
}

public IEnumerable<Animals> Canine(IEnumerable<Animals> sample)
{
  return from a in sample
         where a.race.Family == "Canidae"
         select a;
}
```

Now you have a method that selects an initial sample (`AllSpotted`), plus some filters. So now you can do this:

```cs
var Leopards = Feline(AllSpotted());
var Hyenas = Canine(AllSpotted());
```

Is it faster to use `List` over `IEnumerable`?

- Only if you want to prevent a query from being executed more than once.

But is it better overall?

- In the above, Leopards and Hyenas get converted into **single SQL queries each**, and the **database only returns the rows that are relevant**.
- If we had returned a `List` from `AllSpotted()`, then **it may run slower** because **the database could return far more data than is actually needed**, and we waste cycles doing the filtering in the client.

In a program, **it may be better to defer converting your query to a list until the very end**, so if I'm going to enumerate through Leopards and Hyenas more than once, I'd do this:

```cs
List<Animals> Leopards = Feline(AllSpotted()).ToList();
List<Animals> Hyenas = Canine(AllSpotted()).ToList();
```

# Round a Decimal Value to 2 Decimal Places

> [Source](https://www.c-sharpcorner.com/UploadFile/9b86d4/how-to-round-a-decimal-value-to-2-decimal-places-in-C-Sharp/)

```cs
decimal decimalVar = 123.45M;
string decimalString = "";

decimalString = decimalVar.ToString("#.##");
decimalString = decimalVar.ToString("C");
decimalString = String.Format("{0:C}", decimalvar);
decimalString = decimalVar.ToString("F");
decimalString = String.Format("{0:0.00}", decimalVar);
```

```cs
decimal decimalVar = 123.45M;

decimalVar = decimal.Round(decimalVar, 2, MidpointRounding.AwayFromZero);
decimalVar = Math.Round(decimalVar, 2);
```

# Double - ToString() formatting with two decimal places but no rounding

> [Source](https://stackoverflow.com/questions/2453951/c-sharp-double-tostring-formatting-with-two-decimal-places-but-no-rounding)

```cs
double x = Math.Truncate(myDoubleValue * 100) / 100;
string s = string.Format("{0:N2}%", x); // No fear of rounding and takes the default number format
```

# Change symbol for decimal point in `double.ToString()`

> [Source](https://stackoverflow.com/questions/3135569/how-to-change-symbol-for-decimal-point-in-double-tostring)

```cs
using System.Globalization;

NumberFormatInfo nfi = new NumberFormatInfo();
nfi.NumberDecimalSeparator = ".";

value.ToString(nfi);
```

# Operators

> [Source](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/boolean-logical-operators)

## Negation operator `!`

The unary prefix `!` operator computes logical negation of its operand. That is, it produces `true`, if the operand evaluates to `false`, and `false`, if the operand evaluates to `true`.

```cs
bool passed = false;
Console.WriteLine(!passed);  // output: True
Console.WriteLine(!true);    // output: False
```

*Beginning with C# 8.0, the unary postfix ! operator is the **null-forgiving** operator.*

## Null-forgiving operator `!`

Null-forgiving, or null-suppression, operator.

Use the null-forgiving operator to declare that expression `x` of a reference type isn't null: `x!`. 

The null-forgiving operator has no effect at run time. It only affects the compiler's static flow analysis by changing the null state of the expression. At run time, expression x! evaluates to the result of the underlying expression `x`.

Testing the argument validation logic.

```cs
#nullable enable
public class Person
{
    public Person(string name) => Name = name ?? throw new ArgumentNullException(nameof(name));

    public string Name { get; }
}
```

Without the null-forgiving operator, the compiler generates the following warning for the preceding code: `Warning CS8625: Cannot convert null literal to non-nullable reference type`. By using the null-forgiving operator, you inform the compiler that passing `null` is expected and shouldn't be warned about.

You can also use the null-forgiving operator when you definitely know that an expression cannot be `null` but the compiler doesn't manage to recognize that.

If the `IsValid` method returns `true`, its argument is `not null` and you can safely dereference it:

```cs
public static void Main()
{
    Person? p = Find("John");
    if (IsValid(p))
    {
        Console.WriteLine($"Found {p!.Name}");
    }
}

public static bool IsValid(Person? person)
    => person is not null && person.Name is not null;
```

Without the null-forgiving operator, the compiler generates the following warning for the `p.Name` code: `Warning CS8602: Dereference of a possibly null reference`.

If you can modify the `IsValid` method, you can use the `NotNullWhen` attribute to inform the compiler that an argument of the `IsValid` method cannot be `null` when the method returns `true`:

```cs
public static void Main()
{
    Person? p = Find("John");
    if (IsValid(p))
    {
        Console.WriteLine($"Found {p.Name}");
    }
}

public static bool IsValid([NotNullWhen(true)] Person? person)
    => person is not null && person.Name is not null;
```

In the preceding example, you don't need to use the null-forgiving operator because the compiler has enough information to find out that `p` cannot be `null` inside the `if` statement. 

## Logical AND operator `&`

The `&` operator computes the logical AND of its operands.

The result of `x & y` is `true` if both `x` and `y` evaluate to `true`. Otherwise, the result is `false`.

The `&` operator **evaluates both operands** even if the left-hand operand evaluates to `false`.

For operands of the **integral numeric types**, the `&` operator computes the **bitwise logical AND** of its operands.

## Logical exclusive OR operator `^`

The `^` operator computes the **logical exclusive OR**, also known as the **logical XOR**, of its operands.

The result of `x ^ y` is `true` if `x` evaluates to `true` and `y` evaluates to `false`, or `x` evaluates to `false` and `y` evaluates to `true`. Otherwise, the result is `false`.

That is, for the `bool` operands, the `^` operator computes the same result as the **inequality operator** `!=`.

```cs
Console.WriteLine(true ^ true);    // output: False
Console.WriteLine(true ^ false);   // output: True
Console.WriteLine(false ^ true);   // output: True
Console.WriteLine(false ^ false);  // output: False
```

## Logical OR operator `|`

The `|` operator computes the **logical OR** of its operands.

The result of `x | y` is `true` if either `x` or `y` evaluates to `true`. Otherwise, the result is `false`.

The `|` operator **evaluates both operands** even if the left-hand operand evaluates to `true`.

For operands of the **integral numeric types**, the `|` operator computes the **bitwise logical OR** of its operands.

## Conditional logical AND operator `&&`

The **conditional logical AND** operator `&&`, also known as the *"short-circuiting"* logical AND operator, computes the logical AND of its operands.

The result of `x && y` is `true` if both `x` and `y` evaluate to `true`. Otherwise, the result is `false`.

**If `x` evaluates to false, `y` is not evaluated**.

## Conditional logical OR operator `||`

The **conditional logical OR** operator `||`, also known as the *"short-circuiting"* logical OR operator, computes the logical OR of its operands.

The result of `x || y` is `true` if either `x` or `y` evaluates to `true`. Otherwise, the result is `false`.

**If `x` evaluates to `true`, `y` is not evaluated**.

## Nullable Boolean logical operators

For `bool?` operands, the `&` *(logical AND)* and `|` *(logical OR)* operators support the **three-valued logic** as follows:

- **The `&` operator produces `true` only if both its operands evaluate to `true`**. If either `x` or `y` evaluates to `false`, `x & y` produces `false` *(even if another operand evaluates to null)*. Otherwise, the result of x & y is null.
- **The `|` operator produces `false` only if both its operands evaluate to `false`**. If either `x` or `y` evaluates to `true`, `x | y` produces `true` *(even if another operand evaluates to null)*. Otherwise, the result of `x | y` is `null`.

|    x    |    y    |   x&y   |  x\| y  |
| :-----: | :-----: | :-----: | :-----: |
| `true`  | `true`  | `true`  | `true`  |
| `true`  | `false` | `false` | `true`  |
| `true`  | `null`  | `null`  | `true`  |
| `false` | `true`  | `false` | `true`  |
| `false` | `false` | `false` | `false` |
| `false` | `null`  | `false` | `null`  |
| `null`  | `true`  | `null`  | `true`  |
| `null`  | `false` | `false` | `null`  |
| `null`  | `null`  | `null`  | `null`  |

`&` and `|` operators can **produce non-null even if one of the operands evaluates to `null`**.

You can also use the `!` and `^` operators with `bool?` operands:

```cs
bool? test = null;
Display(!test);         // output: null
Display(test ^ false);  // output: null
Display(test ^ null);   // output: null
Display(true ^ null);   // output: null

void Display(bool? b) => Console.WriteLine(b is null ? "null" : b.Value.ToString());
```

**The conditional logical operators `&&` and `||` don't support `bool?` operands**.

## Compound assignment

For a binary operator `op`, a compound assignment expression of the form `x op= y` is equivalent to `x = x op y` except that `x` is only evaluated once.

The `&`, `|`, and `^` operators support compound assignment.

```cs
bool test = true;
test &= false;
Console.WriteLine(test);  // output: False

test |= true;
Console.WriteLine(test);  // output: True

test ^= false;
Console.WriteLine(test);  // output: True
```

The conditional logical operators `&&` and `||` don't support compound assignment.

## Operator precedence

The following list orders logical operators **starting from the highest** precedence **to the lowest**:

- Logical negation operator `!`
- Logical AND operator `&`
- Logical exclusive OR operator `^`
- Logical OR operator `|`
- Conditional logical AND operator `&&`
- Conditional logical OR operator `||`

## Null-coalescing operator `??`

The **null-coalescing operator** `??` returns the value of its left-hand operand if it isn't `null`.

Otherwise, it evaluates the right-hand operand and returns its result.

The `??` operator **doesn't evaluate its right-hand operand if the left-hand operand evaluates to non-null**.

Available in C# 8.0 and later, the **null-coalescing assignment operator** `??=` assigns the value of its right-hand operand to its left-hand operand only if the left-hand operand evaluates to `null`.

The `??=` operator doesn't evaluate its right-hand operand if the left-hand operand evaluates to non-null.

```cs
List<int> numbers = null;
int? a = null;

(numbers ??= new List<int>()).Add(5);
Console.WriteLine(string.Join(" ", numbers));  // output: 5

numbers.Add(a ??= 0);
Console.WriteLine(string.Join(" ", numbers));  // output: 5 0
Console.WriteLine(a);  // output: 0
```

The left-hand operand of the `??=` operator **must be** a **variable**, a **property**, or an **indexer** element.

The null-coalescing operators are **right-associative**

- `a ?? b ?? c` is evaluated as `a ?? (b ?? c)`
- `d ??= e ??= f` is evaluated as `d ??= (e ??= f)`

## Index from end operator `^`

The `^` operator indicates the **element position from the end of a sequence**. For a sequence of length `length`, `^n` points to the element with offset `length - n` from the start of a sequence. For example, `^1` points to the **last element** of a sequence and `^length` points to the **first element** of a sequence.

```cs
int[] xs = new[] { 0, 10, 20, 30, 40 };
int last = xs[^1];
Console.WriteLine(last);  // output: 40

var lines = new List<string> { "one", "two", "three", "four" };
string prelast = lines[^2];
Console.WriteLine(prelast);  // output: three

string word = "Twenty";
Index toFirst = ^word.Length;
char first = word[toFirst];
Console.WriteLine(first);  // output: T
```

## Range operator `..`

The `..` operator specifies the **start** **and** end of a **range** of indices as its operands. The **left-hand operand is an inclusive** start of a range. The **right-hand operand is an exclusive** end of a range. Either of operands can be an index from the start or from the end of a sequence.

```cs
int[] numbers = new[] { 0, 10, 20, 30, 40, 50 };
int start = 1;
int amountToTake = 3;
int[] subset = numbers[start..(start + amountToTake)];
Display(subset);  // output: 10 20 30

int margin = 1;
int[] inner = numbers[margin..^margin];
Display(inner);  // output: 10 20 30 40

string line = "one two three";
int amountToTakeFromEnd = 5;
Range endIndices = ^amountToTakeFromEnd..^0;
string end = line[endIndices];
Console.WriteLine(end);  // output: three

void Display<T>(IEnumerable<T> xs) => Console.WriteLine(string.Join(" ", xs));
```

```cs
string[] words = new string[]
{
                // index from start    index from end
    "The",      // 0                   ^9
    "quick",    // 1                   ^8
    "brown",    // 2                   ^7
    "fox",      // 3                   ^6
    "jumped",   // 4                   ^5
    "over",     // 5                   ^4
    "the",      // 6                   ^3
    "lazy",     // 7                   ^2
    "dog"       // 8                   ^1
};  
```

```cs
string[] allWords = words[..];     // contains "The" through "dog".
string[] firstPhrase = words[..4]; // contains "The" through "fox"
string[] lastPhrase = words[6..];  // contains "the, "lazy" and "dog"
```

# Attributes

## NotNullWhenAttribute

Specifies that when a method returns `ReturnValue`, the parameter will not be `null` even if the corresponding type allows it.

# Interesting things

`null` becomes before anything else.

```cs
int? a = 10;
int? b = null;

a = a + b;  // a is null
```
