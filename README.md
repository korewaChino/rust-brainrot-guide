# Rust for brainrotted SWEs

This document is an informal lecture on Rust for junior SWEs who are brainrotted from using garbage, imperative-only languages like Python, JS, Java or C#.

It's not meant to be actually read in a professional setting, so if youre here to learn Rust for a job, you're in the wrong place.

It also serves as a crash course on functional programming concepts, with Rust examples as the language of choice.

If you would like me to explain something in a more detailed manner, please open an issue or a PR, and I'll be happy to add more content in this document.

## Monads

Monads are this kind of funny functional programming concept that wraps
a return value in a context enum thingy. You can use it to chain functions
conditionally rather than doing some long ass if-else chain

obligatory wikipedia article
https://en.wikipedia.org/wiki/Monad_(functional_programming)

For example, `Result`

```rs
enum Result<T,E> {
    Ok(T)
    Err(E)
}
```

Look at these funny generics

Let T be any type that we return if a function actually runs successfully

and E be any type that returns instead of it fucks up

So we can have something like
```rs
Result<i32, String>
```

We have here a Result monad that returns a 32bit integer if it runs
and returns a String (error message) if it fails

so when you do something that could fail you can return them in a Result
instead of the type directly so we don't crash

```rs
fn divide(num1: i32, num2: i32) -> Result<i32, String> {
    if num2 == 0 {
        return Result::Err("Cannot divide by zero".to_string());
    } else {
        return Ok(num1 / num2)
    }
}
```

As you can see a division can fail because someone may divide by zero
and we all know division by zero equals undefined, a monad helps you handle errors by letting you handle failures properly

some other languages have exceptions but we all know how bullshit exceptions are

Same applies for the Option type
https://en.wikipedia.org/wiki/Option_type

```rs
enum Option<T> {
    Some(T)
    None
}
```

So when you try to fetch data that may be nullable like a HashMap you can just do this

```rs
let opt: Option<String> = hashmap.get(key);
//  ^ see how we can just return None if the key doesn't exist

match opt {
    Some(value) => println!("Value is {}", value),
    None => println!("Key not found")
}
```

As you can see, we can handle nullable types without even actually
using nulls, which is a good thing because null references were a mistake


A note: Monads *must* return the same type as it's initially wrapped in, so you can't have a monad of `Result` that returns an `Option` or vice versa, unless you use `map` to... map the values to an another type.

For example, Let's say we got a `Result` that returns a `String`.

```rs
Result<String, String>
```

If we want to get the data of the `Result`, you must unwrap or map it first.
```rs
let result = Result::Ok("Hello, world!".to_string());
let result_data = result.unwrap(); // This will panic if the result is an error
```

But we can also use `map` to transform the data.

Hell, Rust has a helper for this called `ok()` that will return the data if it's an `Ok` variant, or `None` if it's an `Err` variant.

```rs
let result = Result::Ok("Hello, world!".to_string());
let result_data: Option<String> = result.ok(); // This will return Some("Hello, world!") if it's an Ok variant, or None if it's an Err variant
```

Map is just a function that lets you declaratively work on data inside monads without have to unwrap them first.

```rs
let result = Result::Ok("Hello, world!".to_string());
let result_data = result.map(|x| x.to_uppercase()); // This will return an Ok variant with the data transformed to uppercase *if* it's an Ok variant, otherwise it will just return the Err variant
```
https://doc.rust-lang.org/std/iter/struct.Map.html


## Higher Order Functions

Higher order functions are functions that can take more functions as arguments.

For example, the `map` function in Rust

`map` is a higher order function that takes another function (closure) as an argument, which can be used to transform elements in a collection like a monad
or an iterator/array/vector/whatever

So maybe you want to double all the elements in a vector

```rs
let vec = vec![1, 2, 3, 4, 5];

let doubled = vec.iter() // Turn into iterator so we can process the data
    .map(|x| // Now we create a lambda that takes x as an argument
        x * 2 // And then multiply x by 2
    ) // Now we have a new iterator with all the elements doubled thanks to map
    .collect::<Vec<i32>>(); // Now let's turn it back into a vector

assert_eq!(doubled, vec![2, 4, 6, 8, 10]);
```

HOFs don't even need to return the same type as the input, since it just executes a function to process the data

For example, let's use the `filter` function

```rs
let vec = vec![1, 2, 3, 4, 5];

let evens = vec.iter()
    .filter(|x| x % 2 == 0) // Filter out all the odd numbers
    .collect::<Vec<i32>>();
```

The `filter` function takes a `FnMut(T) -> bool`, which means that it wants
a boolean return from the lambda, and if it's true, it keeps the element, otherwise it discards it

We can also chain these functions together as much as we want, but remember that the order of the functions matter, and also make sure what you write is readable so you don't confuse yourself

```rs
let vec = vec![1, 2, 3, 4, 5];

let doubled_evens = vec.iter()
    .filter(|x| x % 2 == 0) // Filter out all the odd numbers with the modulus
    .map(|x| x * 2) // Double all the even numbers
    .collect::<Vec<i32>>();
```

See how we're chaining functions together to build up the data we want to work with? While for some of you this may be confusing at first, just treat it like how you would declare a function in math, you just declare what operation you want to do with the data and then evaluate it at the end.

But in something like imperative JavaScript, it would look like this:

```js
let numbers = [1, 2, 3, 4, 5];
for (let i = 0; i < numbers.length; i++) {
    numbers[i] = numbers[i] * 2;
}
```

Or even imperative Rust:

```rs
let mut numbers = vec![1, 2, 3, 4, 5];
for i in 0..numbers.len() {
    numbers[i] = numbers[i] * 2;
}

// or...
// 
for i in numbers.iter_mut() {
    *i = *i * 2; // see how we're literally just modifying each entry step by step
}
```

Note how we have to modify the value in place, instead of just mapping them, which now means we have to modify the original data. In this example it's not a big deal, but in more complex applications like doing a very complex transformation on a large dataset, it can be a pain to debug and maintain.


## Closures/Lambdas/Anonymous Functions

Now you wonder: the HOF example above used a closure (`|x|`), what is a closure?

Closures are essentially lambda/anonymous functions that can capture variables from the scope they are defined in. The `map()` function above is a HOF that takes a function type of `FnMut(T)` where `T` is the type of the elements in the iterator.

And, anonymous functions are... functions. It's in the name. They're just functions that don't have names. They're just there to be used once and then thrown away.

The term "Lambda" comes from the fact that these kinds of functions are used in lambda calculus, which is a system that defines functions in a mathematical way.

https://en.wikipedia.org/wiki/Lambda_calculus

In Rust, closures are defined with the `|args| { body }` syntax, where `args` are the arguments the closure takes, and `body` is the code that the closure runs.

Actually, Rust doesn't even require to put curlies if it's a single expression, so you can just do `|args| expression`

In fact, you can literally declare a variable that is a closure if you want.

Lots of JavaScript developers do this.

```rs
let add = |x: i32, y: i32| x + y;

let result = add(1, 2);

assert_eq!(result, 3);
```

```js
const add = (x, y) => x + y;

const result = add(1, 2);

console.assert(result === 3);
```

We didn't even do a `fn` keyword, we just declared a variable and assigned a closure to it

### What is the `|x|` syntax?

So you ask: where did `x` come from?

x comes from the fact that the iterator, well, iterates over each element in the function, and the parameter here is actually a positional argument that the closure takes, so you can actually name it whatever you want

### Bonus: function reference in... a Fn closure type?

While you may think, "do I have to always write out a closure every time I want to use a HOF?" The answer is actually no! You can actually reference a named function in a closure, and Rust will just call that instead! As long as the return type and positional arguments match, you can just reference a function instead of writing a new closure every time.

```rs
fn double(x: i32) -> i32 {
    x * 2
}

let vec = vec![1, 2, 3, 4, 5];

let doubled = vec.iter()
    .map(double) // Reference the function instead of writing a closure
    .collect::<Vec<i32>>();

assert_eq!(doubled, vec![2, 4, 6, 8, 10]);

// And optionally, instead of positional args you can declare the specific type and argument names too

fn add(x: i32, y: i32) -> i32 {
    x + y
}

let vec = vec![1, 2, 3, 4, 5];

let summed = vec.iter()
    .map(|number| add(number, 5)) // Reference the function instead of writing a closure
    .collect::<Vec<i32>>();

assert_eq!(summed, vec![6, 7, 8, 9, 10]);

```

## Iterators

Iterators are a Rust trait that lets you run functions *iterating* over a collection of elements, like an array

You can use the `iter()` function to turn a collection into an iterator

```rs
let vec = vec![1, 2, 3, 4, 5];

let iter = vec.iter();
```

Now that you have an iterator, you can run any of the HOFs we talked about earlier to declaratively process each element in the collection.

The `iter()` function is literally just a declarative way to do a for loop, but it's more powerful because you can chain functions together and it's more readable

You can do the multiplication example with a for loop too, but it's ugly

```rs
let vec = vec![1, 2, 3, 4, 5];

// See, we are creating a new mutable reference just for this loop
let mut doubled = Vec::new();

for x in vec.iter() {
    doubled.push(x * 2); // Imperatively multiply each element then append
}
```

This might look simple, but the more complex the logic gets, the more unreadable it becomes and the more bugs you introduce.


## Traits

This is pretty much outside of the functional programming paradigm, but it's still important to know.

If you're from the JVM world, these are also known as mixins or interfaces. Especially if you're from the TypeScript world, it's called an interface.

Basically, traits are a set of functions that you can patch onto a struct or enum that lets you define behavior for that struct or enum.

It's kind of like Python's duck typing but you can actually enforce it with the type system.

So if you have structs like this

```rs
struct Dog {
    name: String,
    bite: bool
}

struct Cat {
    name: String,
    scratches: bool,
}
```

There's 2 separate structs, they may have different fields but with traits you can define a common behavior for them with functions

Let's create a trait called `Animal` that has a getter for the name

```rs

trait Animal {
    fn get_name(&self) -> &str;
}
```

Now we can implement this trait for both `Dog` and `Cat`

```rs
impl Animal for Dog {
    fn get_name(&self) -> &str {
        &self.name
    }
}

impl Animal for Cat {
    fn get_name(&self) -> &str {
        &self.name
    }
}
```

But as you can see, there's also the `bites` and `scratches` fields that are different between the two structs

If we want to check if the animal is dangerous, we can create another trait called `Dangerous` that has a function that returns a boolean

```rs
trait Dangerous {
    fn is_dangerous(&self) -> bool;
}
```

Now we can implement this trait for `Dog` and `Cat` as well

```rs
impl Dangerous for Dog {
    fn is_dangerous(&self) -> bool {
        self.bite
    }
}

impl Dangerous for Cat {
    fn is_dangerous(&self) -> bool {
        self.scratches
    }
}
```

The dog is dangerous if it bites, and the cat is dangerous if it scratches.

And maybe, you want to euthanize the animal so let's create a generic function
to put them down

```rs
fn euthanize<T: Dangerous>(animal: T) {
    if animal.is_dangerous() {
        println!("{} is dangerous, euthanizing", animal.get_name());
        drop(animal); // Drop the animal from memory
    } else {
        println!("{} is not dangerous, letting it live", animal.get_name());
    }
}
```

See, we can now kill the animal. But only if it's dangerous.

The `<T: Dangerous>` part is the generic type constraint, it's saying that the type `T` must implement the `Dangerous` trait.

There's also a more verbose way to do this with `where` clauses

```rs
fn euthanize<T>(animal: T)
where T: Dangerous,
{
    if animal.is_dangerous() {
        println!("{} is dangerous, euthanizing", animal.get_name());
        drop(animal); // Drop the animal from memory
    } else {
        println!("{} is not dangerous, letting it live", animal.get_name());
    }
}
```

You can even combine constraints with `+` if you want to enforce multiple traits

```rs
fn euthanize<T>(animal: T)
where T: Dangerous + Animal,
{
    if animal.is_dangerous() {
        println!("{} is dangerous, euthanizing", animal.get_name());
        drop(animal); // Drop the animal from memory
    } else {
        println!("{} is not dangerous, letting it live", animal.get_name());
    }
}
```

This is a very powerful feature of Rust, and it's what makes the language so safe and expressive.

# Rust pointers: Ownership, Borrowing, and Lifetimes

Now that we've covered the basics of functional Rust, let's actually go back to
what makes Rust Rust: Ownership, Borrowing, and Lifetimes.

Unlike some other languages, Rust has a very strict system of pointer references.

Instead of nerfing your memory management with garbage collection or making you
manually manage pointers like in C, Rust has a system of ownership that enforces
memory safety at compile time.

This is the most important aspect of Rust, and it's what makes it so efficient.

## Pointers

In Rust, there are 3 types of pointers: references, smart pointers, and raw pointers.

### References

References are the most common type of pointer in Rust. They are a way to refer to a value without taking ownership of it.

There are two types of references: immutable and mutable.

To reference a value, prefix a variable with `&`. To create a mutable reference, prefix a variable with `&mut`.

```rs
let x = 5;

let y = &x; // Immutable reference

let mut z = 10;

let w = &mut z; // Mutable reference
```


### Smart Pointers

Most of the time variables are stored on the stack, but sometimes you need to store data on the heap, maybe let it live longer than the function scope, or maybe you want to share it between threads. This is where smart pointers come in.

Smart pointers are just structs that implement the `Deref` and `Drop` traits. They are a way to wrap a value and provide additional functionality.

Rust comes with some built-in smart pointers like `Box`, `Rc`, and `Arc`.

#### Box

`Box` is simply a pointer that just wraps a value in the heap. Most of the time people use it to restore recursive data or some kind of generic data that you don't know the size of at compile time, like types that only implement certain traits for example.

```rs

trait Animal {
    fn make_sound(&self);
}

struct Dog;
struct Cat;

struct Kennel {
    inner: Box<dyn Animal>, // Box wraps the trait object here,
    // the dyn keyword just says that the type inside here is dynamic and
    // may not be known at compile time
    // https://doc.rust-lang.org/nightly/std/keyword.dyn.html
}

...

```

The `Kennel` struct here is a smart pointer that wraps an `Animal` trait object. This is useful when you want to store different types of animals in the same collection

> [!NOTE]
> The above code might not be complete so check what you're actually doing

#### Rc

`Rc` stands for reference counting. It's a smart pointer that allows you to share ownership of a value between multiple pointers.

If you come from any garbage-collected language, this is similar to a garbage-collected reference. In fact, Apple brags about this being their main thing in Swift.

```rs
use std::rc::Rc;

let x = Rc::new(5);

let y = x.clone();
```

The `clone` method here is what makes `Rc` special. It doesn't actually clone the value, it just increments the reference count.

When the reference count goes to zero, the value is finally dropped.

#### Arc

`Arc` is just like `Rc`, but it's thread-safe. It stands for **atomic reference counting**.

It's useful when you want to share data between threads.

```rs

use std::sync::Arc;

let x = Arc::new(5);

thread::spawn(move || {
    let y = x.clone();
});
```


### Raw Pointers

Raw pointers are just like pointers in C. They are unsafe and should be avoided unless you really know what you're doing, so you're not going to see them that often unless you're writing some FFI bindings or hand-optimizing code for performance.

We're not gonna cover them here, but you can read more about them in the Rust book and the Rustonomicon.

https://doc.rust-lang.org/nomicon/

## Ownership

Ownership is the most important concept in Rust. It's also usually what prevents your code from compiling.

But it's there for a good reason: to prevent you from creating footguns like dangling pointers, double frees, or ***MEMORY LEAKS***.

In Rust, every value has a single owner. When the owner goes out of scope, the value is dropped.

So when you try to write some code that consumes a value, you're actually transferring ownership of that value to the function, you'll then see that rustc complains that you can't use the value anymore.
So be careful when you're passing values around, you might want to use references instead, or try not to consume the value until you're done with it.

Let's see an example:

```rs

let mut s = String::from("hello");

let s2 = &mut s;
let s3 = &mut s; // !?
//  ^^ error[E0499]: cannot borrow `s` as mutable more than once at a time

println!("{}", s3); // !? No, you can't read the value here either

```

We just tried to borrow `s` as a mutable pointer twice, but you can't just edit the same value twice at the same time! ***This will cause a race condition!!!***

Rust prevents this by enforcing the ownership rules.

Only one mutable reference can exist at a time, and you can't have a mutable reference and an immutable reference at the same time, either.

So you can edit the value, but then you have give it back to the owner before you can edit it again.

```rs

let mut s = String::from("hello");

// Let's create a new scope here
{
    let s2 = &mut s;
    s2.push_str(", world");
    // drop(s2); // s2 goes out of scope here,
    // note that you don't need to call drop() here unlike in C
    // where you have to do free() or delete() manually
} // s2 goes out of scope here, it's safe to borrow `s` again

// the reference to the string has now returned to its rightful owner, `s`.

let s3 = &mut s;

println!("{}", s3); // "hello, world"


```

### Dangling Pointers

Dangling pointers are a common problem in C and C++. They occur when you try to access a pointer that has already been freed.

So, what if you try to access a value that has already been dropped in Rust?

```rs

fn get_string() -> &String {
    let s = String::from("hello");
    &s // !?????
}

fn main() {
    let s = get_string();
    println!("{}", s);
}

```

You just returned a reference to a value that only occurs in that function scope, which means that the time the function returns, the value is already dropped. This is a dangling pointer.

If you do this in C, you get segfaulted at runtime, or worse, you get a security vulnerability and then someone manages to figure out how to hack your system.

But in Rust, rustc will just complain at compile time.


This is how you properly return a value from a function:
```rs

fn get_string() -> String {
    let s = String::from("hello");
    s
}

fn main() {
    let s = get_string();
    println!("{}", s);
}
```

The difference here is that instead of returning some random reference to the value, we literally return the entire value itself, now owned by the caller.

See the omitted `&` in the return type of the function? That's the difference between returning a reference and returning the value itself.

Remember: Don't return the pointer, return the data.

#### Extras: Strings!?!

You may notice that there's a lot of different types of strings in Rust. This is because Rust is a systems programming language, so you have to be kind of explicit about what you're doing with strings.

Here's a quick rundown of the different types of strings in Rust:

- `&str`: This is a string slice. It's a reference to a string, and it's immutable. It's usually used for function arguments or when you want to read a string without modifying it. You can actually use this anywhere but you can't modify it. Just copy the string and make a new owned object.
- `String`: This is a heap-allocated string. It's mutable and you can modify it. This is what you usually use if you want to declare a field in a struct because it'll be owned by the struct. It's just a string of UTF-8 bytes.
- `OsStr`: This is a string that's in the operating system's native format. Its contents depends on what platform you're on so you're usually going to use this when you're dealing with filesystem paths or environment variables. You're gonna be converting this to a `String` or `&str` most of the time.


## (Explicit) Lifetimes

Lifetimes are a way to tell the Rust compiler how long a reference is valid for.

You usually don't have to worry about lifetimes because the Rust compiler is smart enough to figure it out most of the time, but sometimes it's just useful to know how they work.

Here's an example:

```rs

fn get_string() -> &str {
    let s = String::from("hello");
    &s
}

fn main() {
    let s = get_string();
    println!("{}", s);
}

```

This code won't compile because the reference to `s` is only valid for the duration of the function call, but you're trying to return it from the function.

You can fix this by adding a lifetime parameter to the function:

```rs

fn get_string<'a>() -> &'a str {
    let s = String::from("hello");
    &s
}

fn main() {
    let s = get_string();
    println!("{}", s);
}

```
The borrow checker uses explicit lifetime annotations to determine how long references should be valid for.

You can also use lifetime annotations to tell the compiler that two references should have the same lifetime:
```rs

fn get_string<'a>() -> &'a str {
    let s = String::from("hello");
    &s
}

fn get_string2<'a>(s: &'a str) -> &'a str {
    s
}

fn main() {
    let s = get_string();
    let s2 = get_string2(s);
    println!("{}", s2);
}
```

I suck at explaining lifetimes, so here's a better explanation: https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html

There's also a special lifetime called `'static` which means that the reference is valid for the entire duration of the program. You usually use this when you're dealing with static variables or constants.

### Extras: Working around lifetimes and borrowing

Sometimes you might run into a situation where you need to borrow a value, but you can't because of the borrow checker.

This is a common problem, but there's a hacky workaround that you can use to get around it if you don't care about performance.

Sometimes types implement the `Clone` trait, which allows you to make a copy of the value. You can use this to get around the borrow checker by literally just copying the value. It's also useful when you just want to actually copy the value and create a new owned object based on the last one.


```rs

fn get_string() -> String {
    let s = String::from("hello");
    s
}


fn main() {
    let s = get_string();
    let mut ss = s.clone(); // We just copied the value
    ss.push_str(", world");
    println!("{}", s); // "hello"
    println!("{}", ss); // "hello, world"
    
    // You can also use the `to_owned()` method to do the same thing as `clone()`
}
```

It's a hack, but it works.


# Rust Best Practices

Here are some best practices that you should follow when writing Rust code:

## Error handling

Rust has a very powerful error handling system that allows you to handle errors in a very elegant way, see the Monad section for what I mean.

Beginners usually use `unwrap()` to handle errors, but this is a bad practice because if the function fails, it will panic and crash the program.

If you don't care about crashing but still want information on why the function failed, you can use `expect()` instead of `unwrap()`. This will let you input
what you expect in the correct value

```rs
let x: Result<i32, &str> = Ok(2);

x.expect("A number"); // We expect a number from x

```

But when you actually want to handle errors, there's 3 ways to do it:

- `match` statements: This is how you match enums in Rust. You can use this to match the `Result` enum and handle the error if it fails.

```rs
let x: Result<i32, &str> = Ok(2);

match x {
    Ok(v) => println!("Number: {}", v),
    Err(e) => println!("Error: {}", e),
}
```

- `if let` statements: This is a shorthand for `match` statements. You can use this to match the `Result` enum and handle the error if it fails, it also lets you unwrap the value safely and even name it if it works

```rs
// Let's switch it up and use Option
let x: Option<i32> = Some(2);

if let Some(v) = x {
    println!("Number: {}", v);
} else {
    println!("Error: None");
}
```

- `try!`/`?` operator: This is the most elegant way to handle errors in Rust. You can use this to automatically return the error if the function fails. You should use this if you're writing a function that returns a monad.

```rs
fn get_number() -> Result<i32, &str> {
    let x: Result<i32, &str> = Ok(2);
    x
}

fn res() -> Result<(), &str> {
    let x = get_number()?;
    println!("Number: {}", x);
    Ok(())
}

// It also works with Option!
fn get_number_opt() -> Option<i32> {
    let x: Option<i32> = Some(2);
    x
}

fn opt() -> Option<()> {
    let x = get_number_opt()?;
    println!("Number: {}", x);
    Some(())
}
```

Please note that you can only use the `?` operator in functions that return the same monad type as the function you're calling, but you can always remap the error type if you need to.

## Clippy and linting

Clippy is a linter for Rust that helps you write better code. You should always run Clippy on your code to catch any potential issues because rustc doesn't catch everything, only the most important stuff like syntax errors and dead code.

Clippy lints and warns you about potential issues and style problems in your code. It's a great tool to use when you're learning Rust because it helps you write idiomatic Rust code.

In fact, you can customize the Clippy hints to your liking in your source code by adding clippy macros to your code.

```rs
#![warn(clippy::all)]
```

This will warn you about all the Clippy hints in your code.

If you're really strict about unsafe code, you can even just forbid all unsafe code in your codebase by adding this to your source code:

```rs
#![forbid(unsafe_code)]
```

This will forbid all unsafe code in your codebase and prevent compilation if you try to write unsafe blocks.

## Documentation

You can actually embed documentation in your code using comments in Rust. Doc comments are written using `///` (or `//!` if you're documenting an entire module) and are used to generate documentation for your code.

You can use Markdown in your doc comments to format the text and add links, code blocks, and other formatting.

```rs
/// This function adds two numbers together.
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

You can even include tests in the documentation if you add an example block to your doc comments. This is a great way to ensure that your code is correct and that it works as expected.

```rs
/// This function adds two numbers together.
///
/// # Examples
/// ```rs
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

You can generate documentation for your code using the `cargo doc` command. This will generate HTML documentation for your code using mdbook and open it in your browser. You can also publish your documentation to docs.rs if you publish your crate to crates.io.

## Testing

While you can test your code using doc comments, you can also write dedicated unit tests for your code using the `#[test]` attribute. You can write tests in a separate module in your codebase and run them using the `cargo test` command.

```rs
#[test]
fn test_add() {
    assert_eq!(add(2, 3), 5);
}
```

Most of the time tests are in a separate module called `tests` in the same file as the code you're testing.

```rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }
}
```

Note that `#[cfg(test)]` is a conditional compilation attribute that tells the compiler to only compile the code in the module when running tests, so you don't waste time compiling test code when you're building for production.
