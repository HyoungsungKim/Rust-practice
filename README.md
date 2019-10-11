# Rust-practice
Can't spell trust without rust

## Awesome rust list 

- Tutorial(KOR) : https://rinthel.github.io/rust-lang-book-ko/ch00-00-introduction.html

## What is ownership?

Rust uses a third approach: memory is managed through a system of ownership with a set of rules that the compiler checks at compile time. None of the ownership features slow down your program while it’s running.

### Ownership Rules

- Each value in Rust has a variable that’s called its *owner*.
- There can only be ***one owner at a time.***
- When the owner goes out of scope, the value will be dropped.

### Memory and Allocation

In the case of a `string literal`, we know the contents at compile time, so the text is hardcoded directly into the final executable. This is why string literals are fast and efficient. But these properties only come from the string literal’s immutability. Unfortunately, we can’t put a blob of memory into the binary for each piece of text whose size is unknown at compile time and whose size might change while running the program.

> string의 사이즈가 고정되어 있으면 컴파일 타임에 알 수 있음. 따라서 string 문자열이 빠르고 효율적임

With the `String` type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents. This means:

- The memory must be requested from the operating system at runtime.
- We need a way of returning this memory to the operating system when we’re done with our `String`.

Rust takes a different path: the memory is automatically returned once the variable that owns it goes out of scope. This function is called `drop`. Rust calls `drop` automatically at the closing curly bracket.

### Ways Variables and Data Interact: Move

```rust
let s1 = String::from("hello")
let s2 = s1;
```

When we assign `s1` to `s2`, the `String` data is copied, meaning ***we copy the pointer,*** the length, and the capacity that are on the stack. We do not copy the data on the heap that the pointer refers to. 

This is a problem: when `s2` and `s1` go out of scope, they will both try to free the same memory. This is known as a *double free* error and is one of the memory safety bugs we mentioned previously. Freeing memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.

Instead of trying to copy the allocated memory, Rust considers `s1` to no longer be valid and, therefore, Rust doesn’t need to free anything when `s1` goes out of scope. Check out what happens when you try to use `s1` after `s2` is created; it won’t work:

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world", s1);
// error s1 is already deallocated
```

Rust will never automatically create “deep” copies of your data. Therefore, ***any automatic copying can be assumed to be inexpensive in terms of runtime performance.***

### Ways variables and Data Interact: Clone

***If we do want to deeply copy the heap data of the `String`, not just the stack data,*** we can use a common method called `clone`. 

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

### Stack-Only Data: copy

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

The reason is that types such as ***integers that have a known size at compile time*** are stored entirely on the stack, so copies of the actual values are quick to make.  ***There’s no difference between deep and shallow copying here***

> move해도 x는 stack에 저장되어 heap에 저장되어있는게 없기 때문에 메모리 해제 되는게 없음. 따라서 `println!`에서 참조 가능

### Ownership and Functions

Passing a variable to a function will move or copy, just as assignment does.