# bon

Batteries-included tools for building and reshaping Rust data structures.

// TODO: add docs about the associated methods syntax
```rust
use bon::bon;

struct Greeter {
    label: String,
    level: usize,
}

#[bon]
impl Greeter {
    #[builder]
    fn builder(label: String, level: usize) -> Self {
        Self { label, level }
    }

    #[builder]
    fn greet(&self, name: &str, surname: &str) -> String {
        let Self { label, level } = self;
        format!("{label}: Hello {name} {suname} at level {level}")
    }
}

let greeter = Greeter::builder().label("INFO").level(42).call();
let greeting = greeter.greet().name("John").surname("Doe").call();

assert_eq!(greeting, "INFO: Hello John Doe at level 42");
```

## Named function parameters via a type-safe builder

Turn functions with many positional parameters into a function with "named" parameters via a builder. It's as easy as placing `#[builder]` macro on top of your function.

**Example:**

```rust
use bon::builder;

#[builder]
fn greet(name: &str, surname: &str, age: u32) -> String {
    format!("Hello {name} {surname} with age {age}!")
}

let greeting = greet()
    .name("John")
    .surname("Doe")
    .age(32)
    .call();

assert_eq!(greeting, "Hello John Doe with age 32!");
```

### Type safe. No panics

The builders generated by `#[builder]` are all type-safe. They use the typestate pattern to make sure all required values are filled and setter methods aren't called repeatedly to prevent unintentional overwrites and typos. If something is wrong, a compile error will be created. There are no potential panics and `unwrap()` calls inside of the builder.

### Documentation for arguments

In regular Rust, it's not possible to place doc comments on function arguments. But with `#[builder]` it is. Documentation written on the arguments will be placed on the generated setter methods.

**Example:**

```rust
use bon::builder;

/// Function that returns a greeting special-tailored for a given person
#[builder]
fn greet(
    /// Name of the person to greet.
    ///
    /// **Example:**
    /// ```
    /// greet().name("John");
    /// ```
    name: &str,

    /// Age expressed in full years passed since the birth date.
    age: u32
) -> String {
    format!("Hello {name} with age {age}!")
}
```

<details>
<summary>How does this work?</summary>

This works because Rust compiler checks for invalid placement of `#[doc = ...]` attributes only after the macro expansion stage. `#[builder]` makes sure to remove the docs from the function's arguments in the expanded code, and instead moves them to the docs on setter methods.

</details>

### Supported syntax

`#[builder]` attribute works almost with any kind of function that uses any available Rust syntax.
All of the following is supported.

- Functions can return any values including `Result`, `Option`, etc.
- `impl Trait` syntax is supported both in function parameters and return type.
- `async` functions.
- `unsafe` functions.
- Generic type parameters.
- Generic const parameters (const generics).
- Generic lifetimes.
- `where` clauses.
- Anonymous lifetimes, i.e. `'_` or just regular references without explicit lifetimes like `&u32`.
- Nested functions defined inside of other items bodies. E.g.
  ```rust
  fn foo() {
      // Just works
      #[bon::builder]
      fn bar() {}
  }
  ```

### Limitations

- Functions must use simple named parameters. Complex pattern destructuring directly in function arguments is unsupported since it's not clear what name the setter method should have for such arguments. This limitation can be removed in the future with custom name overrides via attributes.
  ```rust
  fn foo_unsupported(
      // There isn't an obvious way to generate a setter method for this function parameter
      // because it doesn't have a single name.
      (x, y): (u32, u32)
  ) {}

  // If you need to destructure your arguments, then do it separately in the function body
  fn foo_supported(
      point: (u32, u32)
  ) {
      let (x, y) = point;
  }
  ```

## References

The design of the generated builders was heavily inspired by such awesome crates as [`buildstructor`](https://docs.rs/buildstructor) and [`typed-builder`](https://docs.rs/typed-builder). This crate was designed as an evolution of both of these with many lessons learned and a bunch more batteries provided.
