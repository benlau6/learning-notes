# learning-notes

Notes on rust, sveltekit, design patterns, software architectures, and more

## Testing

- unit test vs integration test vs functional test
  - integration test: testing data flow between systems or modules, e.g. database interaction
- functional test: testing real life usage, e.g. login, the most important one

## Rust <-> Python

- Anonymous Function: `closure` <-> `lambda function`
- `enum` <-> `Enum`, `T1 | T2
- `Result` <-> `results.Result`
- `Optional` <-> `T | None`, `returns.Maybe`
- `impl` <-> `mixin`
- `struct` <-> `@dataclass`
- Interface: `trait` <-> `Protocol`
  - code reuse: default `trait` method <-> default `Protocol` method
  - polymorphism: generics and trait bounds <-> subclass
    - generics and trait bounds

        ```rust
        pub struct Screen<T: Draw> {
            pub components: Vec<T>,
        }

        impl<T> Screen<T>
        where
            T: Draw,
        {
            pub fn run(&self) {
                for component in self.components.iter() {
                    component.draw();
                }
            }
        }
        ```

    - duck typing - we never have to check whether a value implements a particular method at runtime or worry about getting errors if a value doesn’t implement a method but we call it anyway. Rust won’t compile our code if the values don’t implement the traits that the trait objects need.
    - trait bounds on generics
      - static dispatch - the compiler knows what method you’re calling at compile time.
        - the compiler generates nongeneric implementations of functions and methods for each concrete type that we use in place of a generic type parameter
    - trait objects
      - dynamic dispatch

### Building Rust library for Python

- [Building and distribution](https://pyo3.rs/main/building_and_distribution.html)

## Rust

### Readings

- [How to Become a Rust Super-developer](https://hashnode.com/post/how-to-become-a-rust-super-developer-cjpv1ee7e000buhs2aqrdw2ym)
- [Easy Rust](https://dhghomon.github.io/easy_rust/Chapter_1.html)
- [Rust By Practice](https://practice.rs/why-exercise.html)
- [Rust By Example](https://doc.rust-lang.org/rust-by-example/index.html)
- [Rust Performance Book](https://nnethercote.github.io/perf-book/introduction.html)
- [The Rust Reference](https://doc.rust-lang.org/reference/)

### General notes

- testing
  - <https://www.youtube.com/watch?v=JIvKgSyvtxI>
- lifetime
  - only relevance for reference
  - can be inferred, but sometimes not
  - `fn get_some<'a>(lhs: &'a SomeStruct, rhs: &'a SomeStruct) -> &'a SomeStruct` don't change lifetimes of the parameters
  - `struct SomeStruct<'a> { num: &'a i32, }` indicates that num should live with SomeStruct
- Interior Mutability (Don't learn it until you need it)
  - Cell, RefCell, RwLock and Mutex
  - <https://www.youtube.com/watch?v=HwupNf9iCJk&t=>
- ownership
  - T, &mut T, &T
  - T is owned, you are responsible of freeing it
  - &mut T is exclusive, a reference pointer with additional contract that nobody else can read or write T, i.e. exclusive access, but you have not responsibility to free it after, just borrowing
  - &T is shared, a reference pointer with additional contract that you are not allowed to modify T, cuz other people are also reading it
  - rust will check these contracts at compile time such that you cannot have data races, it is guaranteed there are either multiple readers or single writer
  - p.s. rust will also check lifetime at compile time
  - ref: <https://www.youtube.com/watch?v=s19G6n0UjsM&t=1518s>
- constant vs immutable variable
  - The value of an immutable variable is created at runtime, but the value for a const is created at compile time. This means that you can use things such as user input in an immutable variable.
- String vs &str in struct
  - As a rule of thumb, you should never put the &str type in a struct. Lifetimes on structs should only be used when the struct is a "view" or "cursor" that looks inside some other struct, which is not what is happening here.
  - a Struct should always (mostly) own it's member variables?

### Module

- [Modules Cheat Sheet](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#modules-cheat-sheet)
  - You can declare module to `crate` by writing `mod garden` in `src/lib.rs` or `src/main.rs`.  The compiler will look for the module’s code in the following places
    - Inline, within curly brackets that replace the semicolon following mod garden
    - In the file src/garden.rs
    - In the file src/garden/mod.rs (older style)
  - You can also declare submodule to `garden` by writing `mod vegetables`. The compiler will then look for
    - Inline
    - src/garden/vegetables.rs
    - src/garden/vegetables/mod.rs (older style)
  - Path to code in modules, e.g. to get an `Asparagus` type, we can get form
    - `crate::garden::vegetables::Asparagus`
- [Paths for Referring to an Item in the Module Tree](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html)
  - absolute path: starts from `crate`
  - relative path: starts from the current module and uses `self`, `super`, or an identifier in the current module.
    - We can construct relative paths that begin in the parent module, rather than the current module or the crate root, by using `super` at the start of the path. This is like starting a filesystem path with the `..` syntax
- Private vs public
  - Code within a module is private from its parent modules by default. To make a module public, declare it with pub mod instead of mod. But note that the contents inside are still private.
  - If we use pub before a struct definition, we make the struct public, but the struct’s fields will still be private
  - In contrast, if we make an enum public, all of its variants are then public. We only need the pub before the enum keyword.
    - reason: Enums aren’t very useful unless their variants are public; it would be annoying to have to annotate all enum variants with pub in every case, so the default for enum variants is to be public. Structs are often useful without their fields being public, so struct fields follow the general rule of everything being private by default unless annotated with pub.
- [The `use` keyword](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html)
  - It creates shortcuts to items to reduce repetition of long paths, you can actually use the items without `use`
  - Note that use only creates the shortcut for the particular scope in which the use occurs.
  - The idiomatic way to bring a function into scope with use i bringing the function’s parent module into scope with use. Specifying the parent module when calling the function makes it clear that the function isn’t locally defined while still minimizing repetition of the full path.
  - On the other hand, when bringing in structs, enums, and other items with use, it’s idiomatic to specify the full path
    - There’s no strong reason behind this idiom: it’s just the convention that has emerged, and folks have gotten used to reading and writing Rust code this way.
  - When we bring a name into scope with the use keyword, the name available in the new scope is private (e.g. external code). However, we can use `pub use` if we want to expose it.
    - Re-exporting is useful when the internal structure of your code is different from how programmers calling your code would think about the domain. For example, in this restaurant metaphor, the people running the restaurant think about “front of house” and “back of house.” But customers visiting a restaurant probably won’t think about the parts of the restaurant in those terms.
    - A deep nested module/items can be re-exported to root file by `pub use` for outsiders who are using the lib.
  - we can use self in the nested path. e.g.
    from

    ```rust
    use std::io;
    use std::io::Write;
    ```

    to

    ```rust
    use std::io::{self, Write};
    ```

- The glob operator `*` will bring all public items defined in a path into scope, which is often used when testing to bring everything under test into the tests module

### Error handling

- [Unrecoverable Errors with panic!](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html)
- [Recoverable Errors with Result](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html)
  - Most errors aren’t serious enough to require the program to stop entirely. Sometimes, when a function fails, it’s for a reason that you can easily interpret and respond to. For example, if you try to open a file and that operation fails because the file doesn’t exist, you might want to create the file instead of terminating the process.
- [To panic! or not to panic!](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html)
  - When code panics, there’s no way to recover. You could call panic! for any error situation, whether there’s a possible way to recover or not, but then you’re making the decision that a situation is unrecoverable on behalf of the calling code. When you choose to return a Result value, you give the calling code options
  - returning Result is a good default choice when you’re defining a function that might fail.
  - In situations such as examples, prototype code, and tests, it’s more appropriate to write code that panics instead of returning a Result
  - It’s advisable to have your code panic when it’s possible that your code could end up in a bad state. In this context, a bad state is when some assumption, guarantee, contract, or invariant has been broken, such as when invalid values, contradictory values, or missing values are passed to your code—plus one or more of the following:
    - The bad state is something that is unexpected, as opposed to something that will likely happen occasionally, like a user entering data in the wrong format.
    - Your code after this point needs to rely on not being in this bad state, rather than checking for the problem at every step.
    - There’s not a good way to encode this information in the types you use

### Generics

- Generic type parameters in a struct definition aren’t always the same as those you use in that same struct’s method signatures.

### Struct

- Unit struct: `struct QuitMessage;`
- Tuple struct: `struct WriteMessage(String);` or `struct ChangeColorMessage(i32, i32, i32)`

### Enum

- Zero-variant enum: `enum QuitMessage`
- [Why Rust enums are so cool](https://hashrust.com/blog/why-rust-enums-are-so-cool/)
- [Rust Examples - Enum](https://doc.rust-lang.org/rust-by-example/custom_types/enum.html)

### Trait

- [We can use trait as a parameter with `impl some_trait`](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters), which means that the parameter accepts any type that implements the specified trait. And then in the function body, we can call any methods on item that come from that trait
  - The impl Trait syntax works for straightforward cases but is actually syntax sugar for a longer form known as a trait bound. following codes are equivalent

    ```rust
    pub fn notify(item: &impl Summary) {
        println!("Breaking news! {}", item.summarize());
    }
    ```

    ```rust
    pub fn notify<T: Summary>(item: &T) {
        println!("Breaking news! {}", item.summarize());
    }
    ```

  - The impl Trait syntax is convenient and makes for more concise code in simple cases, while the fuller trait bound syntax can express more complexity in other cases
    - However, following codes are different that
      with impl Trait, item1 and item2 could be different

      ```rust
      pub fn notify(item1: &impl Summary, item2: &impl Summary) {
      ```

      but with trait bound, item1 and item2 have to be the same

      ```rust
      pub fn notify<T: Summary>(item1: &T, item2: &T) {
      ```

  - trait bound can also be written as following for readability

    ```rust
    fn some_function<T, U>(t: &T, u: &U) -> i32
    where
        T: Display + Clone,
        U: Clone + Debug,
    { ... }
    ```

- [We can also use trait as a return type with `impl some_trait`](https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits)
  - The impl Trait syntax lets you concisely specify that a function returns types that only the compiler knows or types that are very long to specify
  - However, you can only use impl Trait if you’re returning a single type
- A trait defines functionality a particular type has and can share with other types. We can use traits to define shared behavior in an abstract way. We can use trait bounds to specify that a generic type can be any type that has certain behavior.
- By using a trait bound with an impl block that uses generic type parameters, we can implement methods conditionally for types that implement the specified traits. E.g.

    ```rust
    impl<T: Display + PartialOrd> Pair<T> {
        fn cmp_display(&self) {
            if self.x >= self.y {
                println!("The largest member is x = {}", self.x);
            } else {
                println!("The largest member is y = {}", self.y);
            }
        }
    }
    ```

- You can also have an impl block that only applies to a struct with a particular concrete type for the generic type parameter T. But note that it is not a trait bound. Following is an example:

    ```rust
    impl Point<f32> {
        fn distance_from_origin(&self) -> f32 {
            (self.x.powi(2) + self.y.powi(2)).sqrt()
        }
    }
    ```

- [CS 242: Traits](https://stanford-cs242.github.io/f19/lectures/07-1-traits)

#### Trait Object

- [What is a trait object?](https://doc.rust-lang.org/reference/types/trait-object.html)
  - A trait object is an opaque value of another type that implements a set of traits.
  - Trait objects are written as the keyword `dyn` followed by a set of trait bounds
  - Due to the opaqueness of which concrete type the value is of, trait objects are dynamically sized types. Like all DSTs, trait objects are used behind some type of pointer; for example `&dyn SomeTrait` or `Box<dyn SomeTrait>`
  - The purpose of trait objects is to permit "late binding" of methods. Calling a method on a trait object results in virtual dispatch at runtime
- [Returning Traits with dyn](https://doc.rust-lang.org/rust-by-example/trait/dyn.html)
  - The Rust compiler needs to know how much space every function's return type requires.
  - This means all your functions have to return a concrete type.
  - there's an easy workaround. Instead of returning a trait object directly, our functions return a Box which contains some Animal. A box is just a reference to some memory in the heap. Because a reference has a statically-known size, and the compiler can guarantee it points to a heap-allocated Animal, we can return a trait from our function!
  - Rust tries to be as explicit as possible whenever it allocates memory on the heap. So if your function returns a pointer-to-trait-on-heap in this way, you need to write the return type with the dyn keyword, e.g. `Box<dyn Animal>`.
- [Box around traits](https://dhghomon.github.io/easy_rust/Chapter_54.html)
- [Don't use boxed trait objects](https://bennett.dev/dont-use-boxed-trait-objects-for-struct-internals/)

#### Impl Trait

- This is not a trait object.
- impl Trait provides ways to specify unnamed but concrete types that implement a specific trait. It can appear in two sorts of places: argument position (where it can act as an anonymous type parameter to functions), and return position (where it can act as an abstract return type)
- impl Trait in argument position is syntactic sugar for a generic type parameter like <T: Trait>, except that the type is anonymous and doesn't appear in the GenericParams list.
  - With a generic parameter such as <T: Trait>, the caller has the option to explicitly specify the generic argument for T at the call site using GenericArgs, for example, `foo::<usize>(1)`. If impl Trait is the type of any function parameter, then the caller can't ever provide any generic arguments when calling that function
  - impl Trait in return position allows a function to return an unboxed abstract type. This is particularly useful with closures and iterators
  - Similarly, the concrete types of iterators could become very complex, incorporating the types of all previous iterators in a chain. Returning impl Iterator means that a function only exposes the Iterator trait as a bound on its return type, instead of explicitly specifying all of the other iterator types involved.
  - In argument position, impl Trait is very similar in semantics to a generic type parameter. However, there are significant differences between the two in return position. With impl Trait, unlike with a generic type parameter, the function chooses the return type, and the caller cannot choose the return type.
  - impl Trait can only appear as a parameter or return type of a free or inherent function. It cannot appear inside implementations of traits, nor can it be the type of a let binding or appear inside a type alias.

#### Trait Bound

- [Trait and lifetime bounds](https://doc.rust-lang.org/reference/trait-bounds.html)

### Enum vs trait objects

- An enum is a closed set of types, with an arbitrary number of related properties.
- A trait is a closed set of properties, with an arbitrary number of related types.
- [why not enums?](https://doc.rust-lang.org/stable/book/ch17-03-oo-design-patterns.html#why-not-an-enum)
  - Enums represent a closed set of type, trait objects represent an open set.
  - In terms of code cleanness, enums cope better with lots of methods acting on a small set of variants, whereas trait objects cope better with lots of variants sharing a small set of methods. Conveniently enough, that's also the setting that minimizes the size of the jump tables used by each solution, increasing the odds that they fit in the cache.
  - Match statements allow you to write different code for each variant, a possibility which you do not exploit by using them as a vtable. Diverging code across match arms has a more complex performance trade-off (it effectively amounts to forcing the inlining of method code in your solution).
  - Even when living outside of a Box, trait objects require pointer-based access, which in Rust can reduce usability compared to enums since mutable data cannot be aliased. Enums do not require pointer indirection, for example they can be stored in containers as-is.
- Traits do not support downcasting - Rust is not inheritance/subtyping-based language, and it gives you another set of abstractions. Moreover, what you want to do is unsound - traits are open (everyone can implement them for anything), so even if in your case match *f covers all possible cases, in general the compiler can't know that.
- If you know the set of structures implementing your trait in advance, just use enum, it's a perfect tool for this. They allow you to statically match on a closed set of variants

### Typestate and builder

- rethink state pattern in rust
  - [Encoding States and Behavior as Types](https://doc.rust-lang.org/stable/book/ch17-03-oo-design-patterns.html#encoding-states-and-behavior-as-types)
    - which is the typical typestate without generics
- [The Typestate Pattern in Rust](https://cliffle.com/blog/rust-typestate/)
  - [comments](https://www.reddit.com/r/rust/comments/c33u9m/the_typestate_pattern_in_rust/)
  - first method: typical typestate with separate structs
  - second method: stateless generic typestates using PhantomData
    - the showcase is using the zero-variant enum, but we can also use the unit struct
  - third method: stateful generic typestates
- [The Case for the Typestate Pattern - The Typestate Pattern itself](https://www.novatec-gmbh.de/en/blog/the-case-for-the-typestate-pattern-the-typestate-pattern-itself/)
- [Builder with typestate in Rust](https://www.greyblake.com/blog/builder-with-typestate-in-rust/)
  - typestate
    - Enforce order of function calls
    - Forbid a function to be called twice
    - Mutually exclusive function calls
    - Require a function to be always called
  - lib: <https://github.com/idanarye/rust-typed-builder>
- [RFC: Evolving the new service builder API](https://awslabs.github.io/smithy-rs/design/rfcs/rfc0023_refine_builder.html)
- [Rusty Typestates - Starting Out](https://rustype.github.io/notes/notes/rust-typestate-series/rust-typestate-part-1)
  - [comments](https://www.reddit.com/r/rust/comments/k2bqn4/rusty_typestates_an_exploration_of_typestates_in/)
  - [typestate vs session types](https://www.reddit.com/r/rust/comments/p0xs9h/retrofitting_typestates_into_rust_paper/)
  - For communication protocol: Session types describe the communication between parties ("outer protocol")
  - For automata: Typestates allow you to describe how they're handled inside the party ("inner protocol")
- [CS 242: Typestate](https://stanford-cs242.github.io/f19/lectures/08-2-typestate.html)
- [CS 242: Session types](https://stanford-cs242.github.io/f19/lectures/09-1-session-types)
- [Builder pattern](https://rust-unofficial.github.io/patterns/patterns/creational/builder.html)
- [The Phantom Builder](https://freemasen.com/blog/phantom-builder/)
- [The Typestate Pattern in C# - Redesigning PactNet](https://adamrodger.com/post/2021-10-13-typestate-pattern-in-csharp/)
  - From builder to typestate
  - builder
    - you can potentially create invalid or nonsensical states
    - To catch these problems at runtime and try to enforce some kind of order and correctness, the implementation has to have lots of runtime checks to make sure you didn’t create an invalid state
    - This is a potentially frustrating development experience because the code compiles fine but then blows up at runtime.
  - typestate
    - invalid states can’t be expressed, and the error is moved to compile-time if you attempt it
    - IDE autocomplete guides you into the pit of success
- ["Type-Driven API Design in Rust" by Will Crichton](https://www.youtube.com/watch?v=bnnacleqg6k)
- [Type-Driven API Design in Rust - Typestate](https://willcrichton.net/rust-api-type-patterns/typestate.html)
- [Typestates in Rust](https://yoric.github.io/post/rust-typestate/)

### Finite State Machine

- [Pretty State Machine Patterns in Rust](https://hoverbear.org/blog/rust-state-machine-pattern/)
- [A Fistful of States: More State Machine Patterns in Rust](https://deislabs.io/posts/a-fistful-of-states/)
- [Regex Library in Rust from Scratch (Finite-State Machines)](https://www.youtube.com/watch?v=MH56D5M9xSQ)
- [RefactorGuru - State Pattern](https://refactoring.guru/design-patterns/state)

### From/Into

- [Rust By Example - From/Into](https://doc.rust-lang.org/rust-by-example/conversion/from_into.html)
- [Rust By Practice - From/Into](https://practice.rs/type-conversions/from-into.html)
- `impl From<i32> for Number` implements the From trait for custom type `Number` from `i32`
- The `From` trait allows for a type to define how to create itself from another type
- `let s = String::from("Hello")` creates `String` s from `&str` Hello
- `let s: String = "Hello".into()` converts `&str` Hello into `String` s
- When performing error handling it is often useful to implement From trait for our own error type. Then we can use ? to automatically convert the underlying error type to our own error type.

  ```rust
  use std::fs;
  use std::io;
  use std::num;

  enum CliError {
      IoError(io::Error),
      ParseError(num::ParseIntError),
  }

  impl From<io::Error> for CliError {
      // IMPLEMENT from method
  }

  impl From<num::ParseIntError> for CliError {
      // IMPLEMENT from method
  }

  fn open_and_parse_file(file_name: &str) -> Result<i32, CliError> {
      // ? automatically converts io::Error to CliError
      let contents = fs::read_to_string(&file_name)?;
      // num::ParseIntError -> CliError
      let num: i32 = contents.trim().parse()?;
      Ok(num)
  }

  fn main() {
      println!("Success!");
  }
  ```

- Unlike From/Into, TryFrom and TryInto are used for fallible conversions and return a Result instead of a plain value.

### API

- [demo-rust-axum](https://github.com/joelparkerhenderson/demo-rust-axum/blob/main/README.md)
- [Actors with Tokio](https://ryhl.io/blog/actors-with-tokio/)

#### readings

- [Object-Orientation in Rust](https://stevedonovan.github.io/rust-gentle-intro/object-orientation.html)
- [Object-Oriented Programming Features of Rust](https://doc.rust-lang.org/book/ch17-00-oop.html)
- [Understanding inheritance and other limitations in Rust](https://blog.logrocket.com/understanding-inheritance-other-limitations-rust/)
- [Behavioural Modelling](https://users.rust-lang.org/t/rust-koans/2408)
- [How to implement inheritance-like feature for Rust?](https://users.rust-lang.org/t/how-to-implement-inheritance-like-feature-for-rust/31159/20)

### Software Architecture

- [2021-08-21 - Hexagonal architecture in Rust #1 - Domain](https://alexis-lozano.com/hexagonal-architecture-in-rust-1/)

## Sveltekit

- fetch with credential
  - external api
    - component
      - no way, cuz you cannot use sveltekit fetch, gotta proxy it by sveltekit api
    - (page/layout).ts
      - sveltekit fetch with cookies in load function
    - (page/layout).server.ts
      - sveltekit fetch with locals.token
      - note: sveltekit fetch with cookies is unstable and should be avoided due to nondeterministic server cookies management in different platform
- how to avoid promise waterfall in load function?
  - Promise.allSettled
  - or wait until [SvelteKit Issue #7635](https://github.com/sveltejs/kit/issues/7635) being resolved

## Design Pattern

### OOP vs Composition

- [Inheritance and Composition: A Python OOP Guide](https://realpython.com/inheritance-composition-python/#choosing-between-inheritance-and-composition-in-python)
- [Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)
- [Single Inheritance Deep Dive: Examples + Code | C++ Tutorials for Beginners #38](https://www.youtube.com/watch?v=S1BR0xDdsyM)
- [The Flaws of Inheritance](https://www.youtube.com/watch?v=hxGOiiR9ZKg)
- [Composition Vs Inheritance - Why You Should Stop Using Inheritance](https://www.youtube.com/watch?v=nnwD5Lwwqdo)
- [Composition over Inheritance](https://www.youtube.com/watch?v=wfMtDGfHWpA)
- [Why COMPOSITION is better than INHERITANCE - detailed Python example](https://www.youtube.com/watch?v=0mcP8ZpUR38&t=1101s)

## Naming

- `get()` vs `find()`
  - [How and why to decide between naming methods with "get" and "find" prefixes](https://softwareengineering.stackexchange.com/questions/182113/how-and-why-to-decide-between-naming-methods-with-get-and-find-prefixes)
  - [find vs. get](https://tuhrig.de/find-vs-get/)
  - [The practical difference between “findBy” and “getBy” in repositories](https://szymonkrajewski.pl/the-practical-difference-between-findby-and-getby-in-repositories/)
