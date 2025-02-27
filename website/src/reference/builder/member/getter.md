# `getter` :microscope:

**Applies to:** <Badge type="warning" text="struct fields"/> <Badge type="warning" text="function arguments"/> <Badge type="warning" text="method arguments"/>

Generates a getter method for a member. The method is callable only after the value for the member is set using any of its setters.

::: danger 🔬 **Experimental**

This attribute is available under the cargo feature `experimental-getter`. Breaking changes may occur between **minor** releases but not between patch releases.

The fate of this feature depends on your feedback in the tracking issue [#225](https://github.com/elastio/bon/issues/225). Please, let us know if you have any ideas on improving this attribute, or if it's already good enough!

:::

The getter for a required member returns `&T`.

::: code-group

```rust [Struct]
use bon::Builder;

#[derive(Builder)]
struct Example {
    #[builder(getter)] // [!code highlight]
    x: u32,
}

let builder = Example::builder().x(1);

let x: &u32 = builder.get_x(); // [!code highlight]

assert_eq!(*x, 1);
```

```rust [Function]
use bon::builder;

#[builder]
fn example(
    #[builder(getter)] // [!code highlight]
    x: u32,
) {}

let builder = example().x(1);

let x: &u32 = builder.get_x(); // [!code highlight]

assert_eq!(*x, 1);
```

```rust [Method]
use bon::bon;

struct Example;

#[bon]
impl Example {
    #[builder]
    fn example(
        #[builder(getter)] // [!code highlight]
        x: u32,
    ) {}
}

let builder = Example::example().x(1);

let x: &u32 = builder.get_x(); // [!code highlight]

assert_eq!(*x, 1);
```

:::

The getter for an [optional member](../../../guide/basics/optional-members) returns `Option<&T>`.

```rust
use bon::Builder;

#[derive(Builder)]
struct Example {
    #[builder(getter)] // [!code highlight]
    x1: Option<u32>,

    // Getters for `default` members are the same // [!code highlight]
    // as for members of type `Option<T>`         // [!code highlight]
    #[builder(getter, default = 99)]              // [!code highlight]
    x2: u32,

}

let builder = Example::builder().x1(1).x2(2);

let x1: Option<&u32> = builder.get_x1(); // [!code highlight]
let x2: Option<&u32> = builder.get_x2(); // [!code highlight]

assert_eq!(x1, Some(&1));
assert_eq!(x2, Some(&2));
```

::: tip

The getter for the member with `#[builder(default)]` returns `Option<T>` because the default value is never stored in the builder. It's calculated [lazily in the finishing function](./default#evaluation-context).

:::

## Config

You can additionally override the name, visibility and docs for the getter method.

```attr
#[builder(
    getter(
        name = custom_name,
        vis = "pub(crate)",
        doc {
            /// Custom docs
        }
    )
)]
```

## `name`

The default name for getter is `get_{member}`.

## `vis`

The visibility must be enclosed with quotes. Use `""` or [`"pub(self)"`](https://doc.rust-lang.org/reference/visibility-and-privacy.html#pubin-path-pubcrate-pubsuper-and-pubself) for private visibility.

The default visibility is the same as the visibility of the [`builder_type`](../top-level/builder_type#vis), which in turn, defaults to the visibility of the underlying `struct` or `fn`.

## `doc`

Simple documentation is generated by default. The syntax of this attribute expects a block with doc comments.

```attr
doc {
    /// Doc comments
}
```
