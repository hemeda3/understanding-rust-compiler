
# understanding-rust-compiler
walking through rustc source code  

rustc_lint->src->unused.rs


#### is_ty_must_use 

which is determining whether a given type should be used (i.e., whether it has a #[must_use] attribute). It does so by examining various kinds of types:

- ADT (Algebraic Data Type): Checked if it is a box or has a #[must_use] attribute.
- Opaque Type Alias: Checked against its trait bounds to see if any have a #[must_use] attribute.
- Dynamic Trait Object: Also checked for the #[must_use] attribute.
- Tuple: Each element of the tuple is checked recursively for the #[must_use] attribute.
- Array: If it’s not empty, the array's element type is checked for the #[must_use] attribute.
- Closure and Generator: These types have special handling, where async generators are considered implementers of Future and checked accordingly.


- ADT:
  ADTs, especially enums and structs, can be checked for the #[must_use] attribute. This attribute indicates that the value of that type is expected to be used, not ignored.
 
```rust
      #[must_use]
    enum Response {
        Success(String),
        Error(String),
    }
    fn function_returning_response() -> Response {
        // Imagine a real implementation here
        Response::Success("It worked!".to_string())
    }
    
    // Since `Response` has a `#[must_use]` attribute,
    // ignoring its value will produce a warning.
    let _ = function_returning_response();

```
### Opaque Type Alias

Opaque type aliases using the `impl Trait` syntax can have their underlying traits checked for the `#[must_use]` attribute.

```rust
    trait MustUseTrait {
        fn do_something(&self);
    }
    
    #[must_use]
    impl MustUseTrait for i32 {
        fn do_something(&self) {
            println!("{}", self);
        }
    }
    
    fn function_returning_opaque() -> impl MustUseTrait {
        42
    }
    
    // Ignoring the value will produce a warning because i32 implements a #[must_use] trait.
    let _ = function_returning_opaque();
```


### Dynamic Trait Object

Trait objects can also be checked for the `#[must_use]` attribute in their trait definitions.


```rust
#[must_use]
trait ImportantTrait {
    fn important_function(&self);
}

impl ImportantTrait for String {
    fn important_function(&self) {
        println!("Important: {}", self);
    }
}

let important: Box<dyn ImportantTrait> = Box::new("Hello, world!".to_string());

// We are expected to use `important` here, not ignore it.


```


### Tuple

Each element of a tuple is checked for the `#[must_use]` attribute.
```rust
#[must_use]
struct Important(i32);

let tuple = (Important(1), 2, 3);

// We are expected to use `tuple.0` here, not ignore it.

```

### Closure and Generator

Closures and generators, especially async generators, can be considered as implementers of Future and should be awaited or used.

```rust
let closure = || {
    #[must_use]
    struct Important;
    Important
};

// We are expected to use the closure here, not ignore it.

let generator = || async {
    yield "Hello, world!".to_string();
};

// As an async generator, this is considered as a Future and should be awaited or used.

```

### Array

For non-empty arrays, the array's element type is checked for the `#[must_use]` attribute.

```rust
let array: [Important; 5] = [Important(1),
 Important(2), Important(3), 
 Important(4), Important(5)
 ];

// We are expected to use elements of `array` here, not ignore them.
```


#### emit_must_use_untranslated and check_must_use_def 

are dedicated to emitting warnings if something that should be used (per the #[must_use] attribute) is not actually used.

#### declare_lint! macro

is dedicated to a different kind of lint: detecting unnecessary delimiters (like parentheses or braces) that don’t affect the code’s behavior.
