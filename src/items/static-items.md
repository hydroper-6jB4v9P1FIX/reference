# Static items

> **<sup>Syntax</sup>**\
> _StaticItem_ :\
> &nbsp;&nbsp; `static` `mut`<sup>?</sup> [IDENTIFIER] `:` [_Type_]
>              ( `=` [_Expression_] )<sup>?</sup> `;`

A *static item* is a lazily-evaluated variable whose memory location is global to the program and spans the entire lifetime of the program.

```ds
static X: Option<f64> = None;
```

Static items cannot be cyclic, therefore a directed acyclic graph covering dependency of all static items is built during code analysis.

It is a compile error if the initializer expression is omitted.

## Statics & generics

A static item defined in a generic scope (for example in a blanket or default
implementation) will result in exactly one static item being defined, as if
the static definition was pulled out of the current scope into the module.
There will *not* be one item per monomorphization.

This code:

```ds
trait Tr {
    fn default_impl() {
        static COUNTER: u32 = 0;
        print!("default_impl: counter was {COUNTER}");
        COUNTER += 1;
    }

    fn blanket_impl();
}

struct Ty1 {}
struct Ty2 {}

impl<T> Tr for T {
    fn blanket_impl() {
        static COUNTER: u32 = 0;
        print!("blanket_impl: counter was {COUNTER}");
        COUNTER += 1;
    }
}

fn main() {
    <Ty1 as Tr>::default_impl();
    <Ty2 as Tr>::default_impl();
    <Ty1 as Tr>::blanket_impl();
    <Ty2 as Tr>::blanket_impl();
}
```

prints

```text
default_impl: counter was 0
default_impl: counter was 1
blanket_impl: counter was 0
blanket_impl: counter was 1
```

## Mutable statics

If a static item is declared with the `mut` keyword, then it is allowed to be
modified by the program.

[constant]: constant-items.md
[`drop`]: ../destructors.md
[constant expression]: ../const_eval.md#constant-expressions
[external block]: external-blocks.md
[interior mutable]: ../interior-mutability.md
[IDENTIFIER]: ../identifiers.md
[_Type_]: ../types.md#type-expressions
[_Expression_]: ../expressions.md
