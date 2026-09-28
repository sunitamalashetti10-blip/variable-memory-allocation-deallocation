# variable-memory-allocation-deallocation

## Variables and Memory in Node.js and Python

This guide explains what variables are used for, how they relate to memory, how long their values remain available, and how Node.js and Python reclaim memory.

### What variables are for

A variable gives a program a name it can use to refer to a value. Programs use variables to keep data between steps, make decisions, repeat work, and pass information to functions. For example, an order-processing program might keep a customer's name, a list of items, and the order total.

A useful distinction is that a variable is not necessarily a box that contains an entire object. In both languages, a variable name is associated with a value; for many objects, that value acts like a reference to an object stored elsewhere in memory. Assigning an object to another variable usually makes both names refer to the same object, rather than copying the object.

### One real-world example: processing an order

The following examples represent a small online order. The `order` object contains a customer's name, purchased items, and a total. A function applies a discount and returns the updated total.

**Node.js (JavaScript):**

```js
function applyDiscount(order, discount) {
	order.total = order.total - discount;
	return order.total;
}

function processOrder() {
	const customerName = "Maya";
	const order = {
		customer: customerName,
		items: ["notebook", "pen"],
		total: 25,
	};

	const amountDue = applyDiscount(order, 5);
	console.log(`${customerName} owes $${amountDue}`);
}

processOrder();
```

**Python:**

```python
def apply_discount(order, discount):
		order["total"] -= discount
		return order["total"]


def process_order():
		customer_name = "Maya"
		order = {
				"customer": customer_name,
				"items": ["notebook", "pen"],
				"total": 25,
		}

		amount_due = apply_discount(order, 5)
		print(f"{customer_name} owes ${amount_due}")


process_order()
```

In each version, `order` refers to a mutable object. The function receives a reference to that same object, so changing its `total` is visible to the caller. `customerName`/`customer_name` and `amountDue`/`amount_due` are local names used while their function is running. The strings and numbers are values too; exactly how an implementation represents them internally is a runtime detail.

### How allocation works

When code creates a value or object, the runtime arranges memory for it. A variable name is bound to the value, or to a reference to the value. Scope rules determine where that name can be used; reachability determines whether an object is still needed.

**Node.js:** Node.js runs JavaScript on the V8 engine by default. V8 allocates memory for JavaScript values and objects, commonly using a managed heap for objects. It may represent or optimize values differently internally, so the simple rule that "primitives are on the stack and objects are on the heap" is not a reliable description of JavaScript engines. Local bindings belong to their function or block scope, while an object can remain alive after a function returns if some reachable reference still points to it (for example, a returned object or a closure).

**Python:** In the common CPython implementation, names are references to Python objects. Assigning `second = first` binds another name to the same object; it does not normally duplicate that object. Objects are dynamically allocated and have implementation-managed lifetimes. Python's language specification does not require CPython's exact memory-management strategy, so details can differ in other Python implementations.

### Validity and when memory is released

There is no single expiry time for a variable or object. Two separate ideas matter:

- **Name visibility (scope):** A local name is generally usable only within its function or block according to the language's rules. Module-level or global names may remain available for as long as the module or program remains loaded.
- **Object lifetime (reachability):** An object can be reclaimed only when the runtime determines it is no longer needed. If another live variable, data structure, closure, or global still refers to it, it can remain alive even after the original local name goes out of scope.

In the order example, after `processOrder` finishes, its local names are no longer usable from outside the function. If nothing else refers to the order object, it becomes eligible for reclamation. If the program saved it in a global order history, that reference keeps the object reachable, so it must remain available.

**Node.js garbage collection:** V8's garbage collector finds objects that are no longer reachable from program roots and reclaims their memory. Collection runs automatically when the engine decides it is appropriate; JavaScript code generally cannot choose an exact collection time. `const` prevents rebinding a name, but does not make the referenced object immutable or determine how long it lives. Setting a reference to `null` or letting a scope end can remove a reference, but does not force immediate collection.

**Python garbage collection:** In CPython, reference counting usually frees an object when its reference count reaches zero. CPython also has a cyclic garbage collector to find unreachable groups of objects that refer to one another. As with JavaScript, Python code should not depend on an exact moment when memory is returned to the operating system. Other Python implementations may use different strategies. `del name` removes that name binding; it does not necessarily destroy the object if another reference remains.

### Common causes of unexpectedly high memory use

Memory use can keep growing when a program accidentally retains references to data it no longer needs. For example, storing every processed order in a long-lived list, keeping unused data in a cache without limits, or registering callbacks that are never removed can keep objects reachable. These are often called memory leaks, even in garbage-collected languages: the collector cannot reclaim objects that the program still references.

For long-running Node.js or Python services, remove unneeded references and bound caches or collections. Do not rely on a local variable going out of scope as proof that memory will immediately be returned to the operating system; runtimes may keep reclaimed memory available for later allocations.

### Summary

Variables are names used to access values. Scope controls where a name is usable; references control whether an object is still reachable; and the runtime manages allocation and reclamation. Node.js/V8 uses tracing garbage collection, while CPython primarily uses reference counting plus cyclic garbage collection. In neither language should application logic depend on a precise memory-deletion time.