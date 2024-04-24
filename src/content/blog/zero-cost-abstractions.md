---
author: Davide Ceschia
pubDatetime: 2023-03-20
title: Zero-Cost Abstractions in Rust - Unlocking High Performance and Expressiveness
postSlug: zero-cost-abstractions
featured: true
draft: false
tags:
  - javascript
  - typescript
  - rust
  - go
  - software engineering
ogImage: ""
description: Let's dive into zero-cost abstractions in rust and other languages, and understand how they can improve the way you think about code.
---

Let's dive into zero-cost abstractions in rust and other languages, and understand how they can improve the way you think about code.

## Table of contents

## Zero-Cost Abstractions in Rust: Unlocking High Performance and Expressiveness

As a passionate Rustacean, I am constantly amazed by the power and elegance of Rust. It has a secret sauce that sets it apart from other languages, and that's its emphasis on zero-cost abstractions. Let’s dive into the world of zero-cost abstractions, illustrating how they enable high-level features without sacrificing performance, and share my experiences on how they have transformed the way I think about code.

Zero-cost abstractions refer to language features that empower developers to write expressive, high-level code without incurring runtime overhead. These abstractions let us create safe, efficient, and maintainable code without compromising on performance. This balance is vital where safety/ergonomics and performance go hand in hand, just like peanut butter and jelly.

## **Iterators: Serving Expressive yet Optimized Code**

Iterators are a fantastic example of Rust's zero-cost abstractions. These [higher-order functions](https://medium.com/javascript-scene/higher-order-functions-composing-software-5365cf2cbe99) operate on iterators, such as `map()`, `filter()`, and `collect()`. They enable concise and expressive code while the compiler optimizes away the overhead. It's like having your cake and eating it too!

Consider the following example, where we want to find the sum of all even numbers in a **`Vec<i32>`**:

```rust
fn sum_even_numbers(numbers: &[i32]) -> i32 {
    numbers.iter().filter(|&&x| x % 2 == 0).sum()
}
```

With iterators, we can write such expressive code effortlessly. The compiler optimizes the iterator chain, effectively transforming it into a simple loop. For comparison, here's the equivalent loop code:

```rust
fn sum_even_numbers(numbers: &[i32]) -> i32 {
    let mut sum = 0;
    for &number in numbers {
        if number % 2 == 0 {
            sum += number;
        }
    }
    sum
}
```

Both versions achieve the same result with the same output code, but the iterator adaptors make the code more concise and expressive.
We can double check this by trying to compile the following code:

```rust
fn sum_even_numbers(numbers: &[i32]) -> i32 {
    // Input either the iterator chain or the loop
}
fn main() { 
    let x = [1, 2, 3];

    println!("{}", sum_even_numbers(&x));
 }
 ```

 Then by running `cargo build --release` on a macbook M1 with rust `1.67.0`  and passing the resulting binary through a `md5` hash, both versions will result in `2900274e2a14296181e19f7b1774c5f0` as hash, which means the machine code for the 2 binaries is identical.
 Keep in mind that if you are using a different operating system, architecture or compiler version the hash might be different, but it will still match between the 2 versions.

## **Comparing Rust's Zero-Cost Abstractions to JavaScript's Non-Zero-Cost Abstractions**

To appreciate zero-cost abstractions further, it's helpful to contrast them with non-zero-cost abstractions in other languages like JavaScript. For instance, JavaScript's Array **`map()`** function offers a similar high-level abstraction as Rust's iterator adaptors but comes with a performance cost.

In JavaScript, using **`map()`** or other higher-order functions like **`filter()`** or **`reduce()`** often results in slower code execution compared to using traditional **`for`** loops or the more modern **`for of`**. This is because the JavaScript engine cannot optimize these functions to the same extent as the Rust compiler can with iterator adaptors. As a result, JavaScript's abstractions introduce runtime overhead, which may be negligible for small datasets but can become significant when processing large amounts of data or in performance-critical applications.

A benchmark comparison of different loop styles in JavaScript demonstrates the difference in performance.

**Chained Higher-Order Functions**

```tsx
const sumEvenNumbersHOF = numbers =>
  numbers.filter(n => n % 2 === 0).reduce((acc, n) => acc + n, 0);
```

**Traditional `for` Loop**

```tsx
function sumEvenNumbersFor(numbers) {
  let sum = 0;
  for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] % 2 === 0) {
      sum += numbers[i];
    }
  }
  return sum;
}
```

**`for of` Loop**

```tsx
function sumEvenNumbersForOf(numbers) {
  let sum = 0;
  for (const number of numbers) {
    if (number % 2 === 0) {
      sum += number;
    }
  }
  return sum;
}
```

The benchmark results indicate the following:

- **`Chained HOF`**: 11.24 ops/s (operations per second) ± 2.09%, which is 30.56% slower than the fastest method
- Traditional **`for`**: 16.19 ops/s ± 1.86%, the fastest method in this comparison
- **`for of`**: 14.19 ops/s ± 1.14%, which is 12.38% slower than the fastest method

## **Generics: Rust's Type-Safe and Zero-Cost Abstractions vs. Go's Non-Zero-Cost Approach**

Generics are a powerful feature that allow developers to write reusable and type-safe code. They enable the creation of data structures and functions that can work with multiple types without requiring duplicated code. Rust and Go handle generics very differently.

### **Rust's Zero-Cost Generics: Monomorphization Magic**

When you define a generic function or data structure in Rust, the compiler generates specialized and efficient code for each type used in the generic context. This process, known as monomorphization, allows Rust to retain the performance characteristics of a low-level language while providing the expressiveness and type safety of high-level abstractions. The Rust compiler optimizes the generated code for each specific type.

For example, consider a generic **`sum()`** function:

```rust
fn sum<T: std::ops::Add<Output = T> + Copy + Default>(numbers: &[T]) -> T {
    let mut total = T::default();
    for &number in numbers {
        total = total + number;
    }
    total
}
```

When using this generic **`sum()`** function with different types, such as **`i32`** or **`f64`**, the compiler generates specialized code for each type.

### **Go's Non-Zero-Cost Generics: A Trade-off Between Abstraction and Performance**

Go, on the other hand, introduced support for generics in version 1.18, but they use runtime type assertions and dynamic dispatch.

Unlike Rust's monomorphization, Go's compiler doesn't generate specialized code for each type used in a generic context. Instead, it relies on interfaces and dynamic dispatch to achieve type abstraction. While this approach provides the benefits of code reusability and type safety, it comes at the cost of runtime performance.

For example, consider a generic **`sum()`** function:

```go
func sum[T constraints.Ordered](numbers []T) T {
    var total T
    for _, number := range numbers {
        total += number
    }
    return total
}
```

When using this generic **`sum()`** function with different types, such as **`int`** or **`float64`**, Go's compiler doesn't generate specialized code for each type.

## **Traits: A Foundation for Reusable and Efficient Code with Static and Dynamic Dispatch**

Traits in Rust are another powerful example of zero-cost abstractions. They define shared behavior across different types. When you use traits in Rust, the compiler generates efficient code that is specialized for each type implementing the trait. This ensures optimal performance, as if you had written the code manually for each specific type.

One of the main differences between Rust's traits and other languages' interfaces, such as Go, is the way they handle dispatching method calls. In Rust, traits allow both static and dynamic dispatch.

### **Static Dispatch**

Static dispatch is the default behavior when using traits in Rust. The compiler knows the concrete type at compile time and generates specialized code for each type implementing the trait, resulting in direct function calls. To use static dispatch in Rust, you can utilize generics with trait bounds:

```rust
fn some_function<T: SomeTrait>(item: &T) {
    item.some_method();
}
```

### **Dynamic Dispatch**

Dynamic dispatch, on the other hand, involves determining the concrete type at runtime. It's associated with a performance cost due to the use of virtual method tables (vtables) and indirect function calls. However, dynamic dispatch can be beneficial when dealing with heterogeneous collections or when the number of types implementing a trait is unknown at compile time. In Rust, you can use dynamic dispatch by employing trait objects:

```rust
fn some_function(item: &dyn SomeTrait) {
    item.some_method();
}
```

In contrast, Go only has dynamic dispatch through its interfaces. While Go's interfaces provide a similar level of abstraction, the absence of static dispatch makes them less suitable for performance-critical systems.

Here's an example of Go interface and its usage:

```go
type SomeInterface interface {
    SomeMethod()
}

type SomeType1 struct{}

func (t SomeType1) SomeMethod() {
    fmt.Println("SomeType1's method")
}

type SomeType2 struct{}

func (t SomeType2) SomeMethod() {
    fmt.Println("SomeType2's method")
}

func someFunction(item SomeInterface) {
    item.SomeMethod()
}

func main() {
    t1 := SomeType1{}
    t2 := SomeType2{}

    someFunction(t1) // Output: SomeType1's method
    someFunction(t2) // Output: SomeType2's method
}
```

Rust's support for both static and dynamic dispatch with its trait system allows developers to choose the most appropriate dispatch mechanism for their specific use cases. This flexibility, combined with zero-cost abstractions, helps Rust achieve a perfect balance between abstraction, reusability, and performance.

## **Smart Pointers: Advanced Memory Management**

Smart pointers in Rust, such as **`Box`**, **`Rc`**, and **`Arc`**, are another example of zero-cost abstractions working their magic. They enable advanced memory management techniques while abstracting away underlying complexity. This abstraction comes with no runtime cost, as the Rust compiler optimizes the usage of smart pointers in the generated code.

For example, when using the **`Box`** smart pointer, the Rust compiler ensures that the allocated memory is deallocated when the **`Box`** goes out of scope. This automatic memory management provides safety guarantees similar to those in garbage-collected languages but without a garbage collector.

## **Macros: Compile-Time Expansion for Runtime Efficiency**

Macros enable the definition of reusable chunks of code that can be invoked like functions. However, unlike functions, macros are expanded at compile time.
That means that instead of calling a function at runtime and adding it to the call stack, the compiler replaces the macro invocation with the code generated by the macro.

One of the most commonly used macros is **`println!`**, which is used to print formatted text to the console. The **`println!`** macro ensures type safety and correct formatting at compile time while generating efficient code with no additional runtime cost.

## **Main Takeaway**

Rust's zero-cost abstractions offer significant advantages in terms of both expressiveness and performance. While Rust is a powerful and versatile language, it's essential to remember that it may not always be the best tool for every job. For instance, as explained in **[this](https://zackoverflow.dev/writing/unsafe-rust-vs-zig/)** article, Zig can be a better alternative for writing unsafe code, providing superior syntax and tooling for specific use cases. Nevertheless, Rust excels in many areas and is a valuable language to learn and master.

Understanding zero-cost abstractions and their implications is beneficial for developers, regardless of whether or not they actively use these abstractions. Being aware of how these features work and the performance characteristics of the code you're writing helps you make more informed decisions when choosing the right tool for the job. By recognizing the strengths and limitations of each language, you can effectively leverage their capabilities to create high-performance and maintainable software.
