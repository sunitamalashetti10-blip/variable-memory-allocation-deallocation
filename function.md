"""
JavaScript, Node.js, Python, and Java: variables, functions, and memory
========================================================================

This file is a reading guide. Its examples are included as text so that
JavaScript and Java snippets can sit beside Python explanations in one place.

1. Variable declarations
-----------------------

JavaScript declares bindings with let, const, or (in older code) var.
Use let when a binding must be reassigned and const otherwise. const prevents
reassigning the binding; it does not make an object immutable. var has
function-level rather than block-level scope and is generally avoided in new
code.

    let count = 1;
    const names = ["Ada"];
    names.push("Lin");       // allowed: the array is still mutable
    // names = [];            // TypeError: const binding cannot be reassigned

Python binds names to objects; a name does not have a required declared type.
A name can be rebound to a value of another type:

    count = 1
    count = "one"

Java requires declared types for local variables and fields (with `var`,
Java 10+, the compiler infers a local variable's type from its initializer;
the type is still fixed).

    int count = 1;
    String name = "Ada";
    var total = 3;           // inferred as int; cannot later hold a String

2. Static and dynamic typing
----------------------------

Java is statically typed: the compiler checks declared and inferred types
before a program runs, and a variable's type does not change at runtime.
JavaScript and Python are dynamically typed: values have types at runtime,
and a binding can refer to values of different types at different times.
Both still perform runtime checks and can raise errors. "Dynamic" does not
mean "no types" or "no checking."

    # Python
    value = 10
    value = "ten"            # valid rebinding

    // Java
    int value = 10;
    // value = "ten";        // compile-time type error

3. Primitive values and objects
-------------------------------

JavaScript has primitive values (including number, bigint, string, boolean,
undefined, symbol, and null) and objects. Arrays and functions are objects.
Primitives are immutable values; objects can have mutable properties.

Python's data model treats values as objects. For example, integers and
strings are objects too; the distinction between "primitive" and "object"
from Java or JavaScript does not map directly onto Python.

Java has primitive types such as int and boolean, and reference types such as
String, arrays, and class instances. A reference variable holds a reference
to an object, or null. Generic type parameters such as List<Integer> use
reference types; Java boxes primitive values when needed.

4. Names, references, assignment, and copying
----------------------------------------------

Assignment normally binds a name or variable to a value; it does not
automatically make an independent copy of a mutable object.

    # Python
    a = [1, 2]
    b = a
    b.append(3)
    print(a)                 # [1, 2, 3]: both names refer to the same list

    // JavaScript
    const a = [1, 2];
    const b = a;
    b.push(3);
    console.log(a);          // [1, 2, 3]: both variables refer to the array

    // Java
    int[] a = {1, 2};
    int[] b = a;
    b[0] = 9;
    System.out.println(a[0]); // 9: both variables refer to the same array

For an immutable value such as a string, rebinding one name does not change
the value seen through another name. A shallow copy makes a new outer
collection but may still share nested objects. A deep copy recursively
duplicates objects where supported; copying behavior depends on the language
and the object type.

    # Python shallow copy: new list, same nested list
    a = [[1], [2]]
    b = a.copy()
    b[0].append(9)
    print(a)                 # [[1, 9], [2]]

5. Mutability examples
----------------------

Strings are immutable in Python, JavaScript, and Java. Operations that seem
to modify a string produce a new string/value instead.

Lists in Python, arrays in JavaScript, and Java arrays are mutable. Python
lists and JavaScript objects/arrays can grow or shrink; Java arrays have a
fixed length, though their elements can be reassigned. Java's ArrayList is a
resizable collection.

JavaScript objects, Python dictionaries/lists, and most ordinary Java
instances can be mutated through their references. In Java, a final reference
cannot be reassigned, but the referenced object may still be mutable.

    # Python
    text = "cat"
    text = text + "s"        # binds text to a new string
    items = ["tea"]
    items.append("coffee")   # mutates the existing list

    // JavaScript
    let text = "cat";
    text += "s";             // binds text to a new string
    const items = ["tea"];
    items.push("coffee");    // mutates the existing array

    // Java
    String text = "cat";
    text = text + "s";       // String is immutable
    var items = new java.util.ArrayList<String>();
    items.add("tea");        // mutates the list

6. Stack and heap: a useful but simplified picture
--------------------------------------------------

The call stack tracks active function/method calls. A call has an execution
context or frame containing such things as the return location and local
state. The heap is commonly used for dynamically allocated objects whose
lifetime can outlast one call.

It is misleading to conclude that "every simple variable is on the stack and
every object is on the heap." Language specifications generally do not
promise this physical layout. Compilers and runtimes can optimize, inline,
eliminate, or move values. A local variable may hold a primitive value, an
object reference, or be optimized away. In Java, objects are conceptually
created as objects, but a JIT compiler may optimize allocations when safe.
In Python and JavaScript, implementation details also depend on the runtime.

Use stack and heap as a starting conceptual model, not as a complete rule for
predicting memory use or lifetime.

7. What a function or method call does
--------------------------------------

On a call, the program evaluates arguments, establishes a new active call
context, makes parameters available, executes the body, and then returns a
value (or returns with no useful value). The call stack represents nested
calls; a call that never returns can keep growing the stack and may cause a
stack overflow.

Local names are normally usable only in their scope. Returning an object
reference lets the caller keep using that object after the function returns.
The returned value is not necessarily copied.

    # Python
    def make_list():
        local_name = [1, 2]
        return local_name

    result = make_list()     # result refers to the returned list

8. Functions and methods in the four environments
-------------------------------------------------

JavaScript functions are first-class values: they can be assigned to
variables, passed as arguments, and returned. Node.js runs JavaScript and
uses the same language function model.

Python functions are also first-class values and can be passed, stored, and
returned.

Java methods are associated with classes or instances rather than being
standalone function values in the same way. Java supports behavior values
through functional interfaces (such as Runnable or Function) and lambdas,
which can be passed to methods.

    // Java lambda implementing a functional interface
    java.util.function.Function<Integer, Integer> doubleIt =
        number -> number * 2;

9. Argument passing: value and reference semantics
--------------------------------------------------

An important distinction is whether a function can reassign the caller's
variable or mutate a shared object:

* JavaScript passes arguments by value. For an object argument, that value
  is a reference to the object. The function gets a copy of that reference:
  it can mutate the shared object, but reassigning its parameter does not
  reassign the caller's variable.
* Java also passes arguments by value. For a reference-type argument, the
  copied value is a reference. A method can mutate the referenced object,
  but assigning a new object to the parameter does not change the caller's
  reference variable.
* Python's argument passing is often described as "call by sharing" or
  "object-reference passing": the parameter is bound to the same object the
  argument evaluated to. Mutating a mutable object is visible to the caller;
  rebinding the parameter is not.

None of these examples means that the function receives a magical alias
that can replace the caller's variable. Mutating the shared object and
reassigning a local parameter are different operations.

    # Python
    def change(items):
        items.append("new")  # caller sees this mutation
        items = []            # only rebinds the local parameter

    values = ["old"]
    change(values)
    print(values)             # ["old", "new"]

    // JavaScript
    function change(items) {
      items.push("new");      // caller sees this mutation
      items = [];             // only rebinds the local parameter
    }
    const values = ["old"];
    change(values);
    console.log(values);      // ["old", "new"]

    // Java
    static void change(java.util.List<String> items) {
        items.add("new");     // caller sees this mutation
        items = new java.util.ArrayList<>(); // local rebind only
    }

Passing an immutable value such as a number or string does not let a
function change the caller's binding. It can return a new value for the
caller to assign.

10. Closures and captured variables
-----------------------------------

A closure is a function together with access to variables from its lexical
environment. If the function escapes the scope where it was created, the
runtime preserves the captured state for as long as the closure needs it.
The original function call can finish while the captured data remains
reachable through the closure.

    # Python
    def make_counter():
        count = 0

        def increment():
            nonlocal count
            count += 1
            return count

        return increment

    counter = make_counter()
    print(counter())          # 1
    print(counter())          # 2

JavaScript closures capture access to lexical bindings. Python closures can
also access enclosing-scope bindings; `nonlocal` allows rebinding an
enclosing function's local name. Java lambdas can capture local variables
only when they are final or effectively final. For a captured local, Java
captures its value for use by the lambda; a referenced mutable object may
still be mutated through that reference.

11. Garbage collection and object lifetime
-------------------------------------------

Garbage collection reclaims memory used by objects that the program can no
longer reach, reducing the need for most application code to manually free
each object. An object becomes eligible for collection when it is no longer
reachable from the runtime's roots (subject to the particular runtime).

Eligibility is not a promise that collection happens immediately or that
memory is immediately returned to the operating system. `del name` in
Python removes a name binding; JavaScript `delete` removes an object
property (it does not directly free an object); neither is a command to
immediately collect arbitrary memory.

12. Garbage-collection comparison
----------------------------------

JavaScript engines such as V8 use tracing garbage collection. Node.js uses
V8's JavaScript garbage collector.

In CPython, reference counting reclaims many objects when their reference
count reaches zero. A cyclic garbage collector supplements it by finding
unreachable reference cycles. Other Python implementations may use different
strategies.

The JVM specifies garbage-collected memory management, not one mandatory
collector. Java runtimes commonly use tracing collectors, with collector
choices and details depending on the JVM and its configuration.

13. Why garbage-collected programs can still leak memory
--------------------------------------------------------

An object that is no longer useful to the application can still be reachable,
so a collector must retain it. Common causes include:

* An unbounded cache or long-lived collection that keeps old entries.
* A global or static reference that was never cleared.
* Event listeners or callbacks that remain registered and retain their
  associated objects.
* A closure that unintentionally retains a large object graph.

Garbage collection handles unreachable objects; it cannot infer that a
reachable object is no longer useful.

14. Runtime comparison
----------------------

JavaScript is a language. Node.js is a runtime for executing JavaScript
outside a browser; it uses V8 by default and provides APIs and an event loop
for server-side and other applications. libuv supplies cross-platform
asynchronous I/O and event-loop support for Node.js.

Python is a language with multiple implementations. CPython is the
widely-used reference implementation; Python programs may also run on
different runtimes with different implementation details.

Java source is compiled to JVM bytecode and run by a Java Virtual Machine.
The JVM may interpret bytecode and compile frequently executed code at
runtime.

15. Compilation, interpretation, and JIT compilation
-----------------------------------------------------

"Compiled versus interpreted" is not a reliable one-word classification.
Implementations can combine stages:

* V8 parses and compiles JavaScript, uses interpreter/compiler machinery,
  and can JIT-compile hot code. Exact strategies change between versions.
* CPython compiles source to bytecode, which its virtual machine executes.
  This does not mean ordinary CPython automatically JIT-compiles all code.
  Other Python implementations may use different strategies.
* Java compiles source to bytecode. A JVM can interpret bytecode and use
  just-in-time compilation to produce optimized machine code while running.

16. Event loops, asynchronous I/O, threads, and CPU work
-------------------------------------------------------

Node.js commonly uses an event loop with non-blocking I/O. This is useful
for many I/O-bound tasks, but synchronous CPU-heavy work can block the event
loop. Node.js also supports worker threads and child processes for parallel
or isolated work.

Python's `asyncio` provides cooperative asynchronous programming with an
event loop; `async`/`await` does not automatically make CPU-bound code run in
parallel. Threads and processes are also available. CPython's GIL affects
parallel execution of Python bytecode in many builds, but I/O, native
extensions, Python versions, and alternative implementations can change
the practical picture.

Java provides threads and higher-level concurrency tools. Asynchronous and
non-blocking APIs are also available. Workload, libraries, and resource
limits determine which approach fits best.

I/O-bound work often spends time waiting for a network, disk, or database;
asynchronous I/O or threads can help manage waiting efficiently. CPU-bound
work spends time computing; parallel execution, native code, algorithms, and
work partitioning may matter more.

17. A request's path through an application
-------------------------------------------

A typical web request may travel through these stages:

    HTTP request
        -> server/runtime receives it
        -> route handler or controller runs
        -> local variables and objects represent input and state
        -> application calls a database or another API
        -> result is checked and converted into a response
        -> HTTP response is sent

Each stage can allocate objects and extend their lifetimes. A request-local
object may become unreachable after the response, while a cached result or
registered callback may remain reachable much longer.

18. Performance
---------------

There is no useful universal answer to "which language is fastest?" Measure
the actual workload. Consider algorithmic complexity, CPU time, I/O waits,
memory use, garbage-collection pauses, runtime warm-up, libraries, and
architecture. Profile before optimizing and compare equivalent work under
realistic inputs.

19. Variable scope and object lifetime
--------------------------------------

The lifetime of a name and the lifetime of the object it refers to are
related but not identical. A local name can stop being accessible when a
function returns, while an object it referenced can remain alive if it was
returned, stored globally, captured by a closure, or added to another live
object.

An object becomes eligible for collection once it is unreachable according
to its runtime. The collector's timing and when memory is returned to the
operating system are separate matters.

20. What happens for `result = a + b`?
--------------------------------------

The exact steps depend on the language, the types of `a` and `b`, and the
runtime. At a high level, the program evaluates the operands, applies the
language's addition rules, and binds or assigns the resulting value to
`result`.

Python:

    result = a + b

Python looks up `a` and `b`, applies the `+` operation supported by their
types (which can involve special methods such as `__add__`), and binds the
resulting object to the name `result`. For integers, addition produces an
integer value. The name does not declare a fixed type.

JavaScript:

    const result = a + b;

JavaScript evaluates both operands and applies its `+` rules. Depending on
the values this can perform numeric addition or string concatenation;
objects may be converted to primitives first. `const` creates a binding
that cannot be reassigned, but does not imply a physical memory location.

Java:

    int result = a + b;

The compiler checks that the operands and result are compatible with the
declared types. For `int` operands, integer addition is performed and the
result is assigned to the local variable; integer overflow wraps according
to Java's integer arithmetic rules. For reference types, `+` is not
generally defined, with String concatenation being a notable language
feature.

In every language, a simple source line does not tell you exactly where each
value lives in physical memory. The implementation may optimize storage
while preserving the language's observable behavior.
"""


def basic_calculator(a, b):
    if b == 0:
        raise ZeroDivisionError("The second number must not be zero.")

    return {
        "addition": a + b,
        "subtraction": a - b,
        "multiplication": a * b,
        "division": a / b,
        "floor division": a // b,
        "remainder": a % b,
    }


def check_number(number):
    return {
        "even_or_odd": "Even" if number % 2 == 0 else "Odd",
        "divisible_by_3": number % 3 == 0,
        "divisible_by_5": number % 5 == 0,
    }


def student_result(marks):
    if not 0 <= marks <= 100:
        raise ValueError("Marks must be between 0 and 100.")
    if marks >= 75:
        return "Distinction"
    if marks >= 35:
        return "Passed"
    return "Fail"


def check_eligibility(marks, attendance, has_backlog):
    eligible = marks >= 60 and attendance >= 75 and not has_backlog
    return "Eligible" if eligible else "Not eligible"


def is_valid_user(username, password):
    return username == "Admin" and password == "python123"


def calculate_discount(purchase_amount):
    if purchase_amount < 0:
        raise ValueError("Purchase amount cannot be negative.")

    if purchase_amount >= 5000:
        discount_rate = 0.20
    elif purchase_amount >= 3000:
        discount_rate = 0.10
    else:
        discount_rate = 0.05

    discount_amount = round(purchase_amount * discount_rate, 2)
    payable_amount = round(purchase_amount - discount_amount, 2)
    return discount_amount, payable_amount


def check_access(age, has_id, is_employee):
    if is_employee or (age >= 18 and has_id):
        return "Access granted"
    return "Access denied"


REQUIRED_SKILLS = ["Python", "SQL", "Git", "HTML"]


def check_skill(skill_name):
    available_skills = {skill.casefold() for skill in REQUIRED_SKILLS}
    if skill_name.casefold() in available_skills:
        return "Skill available"
    return "Skill not available"


def calculate(a, operator, b):
    operations = {
        "+": lambda: a + b,
        "-": lambda: a - b,
        "*": lambda: a * b,
        "/": lambda: a / b,
        "//": lambda: a // b,
        "%": lambda: a % b,
        "**": lambda: a**b,
    }

    if operator not in operations:
        raise ValueError(f"Invalid operator: {operator}")
    if operator in ("/", "//", "%") and b == 0:
        raise ZeroDivisionError("Cannot divide by zero.")

    return operations[operator]()


def placement_result(age, marks, attendance, experience, has_backlog):
    if experience < 0:
        raise ValueError("Experience cannot be negative.")

    eligible = marks >= 60 and attendance >= 75 and not has_backlog

    if experience == 0:
        category = "Fresher"
    elif experience <= 2:
        category = "Junior"
    else:
        category = "Experienced"

    return {
        "age": age,
        "placement_eligible": "Yes" if eligible else "No",
        "candidate_category": category,
    }


if __name__ == "__main__":
    print("Task 1:", basic_calculator(17, 5))
    print("Task 2:", check_number(15))
    print("Task 3:", student_result(82))
    print("Task 4:", check_eligibility(72, 80, False))
    print("Task 5:", is_valid_user("Admin", "python123"))
    print("Task 6:", calculate_discount(4500))
    print("Task 7:", check_access(17, False, True))
    print("Task 8:", check_skill("python"))
    print("Task 9:", calculate(2, "**", 3))
    print("Task 10:", placement_result(22, 78, 82, 1, False))
