# Constant items

> **<sup>Syntax</sup>**\
> _ConstantItem_ :\
> &nbsp;&nbsp; `const` ( [IDENTIFIER] | `_` ) ( `:` [_Type_] )<sup>?</sup> ( `=` [_Expression_] )<sup>?</sup> `;`

The meaning of a *constant item* depends on the context it is used or attributes specified:

- If it has the attribute `#[inline]`, it is always an *inlined constant*, independent from the context it appears in.
- If it appears inside a module, it is a static item.
- If it appears inside a trait or inside an implementation for a trait, it is an *inlined constant*.
- If it appears inside a function, it is a let statement.

If the constant item is an inlined constant, its expression is expanded wherever used, similiar to a function call.

Examples:

```
const MAX: f64 = 10_000.0;

fn f() {
    let x = 10;
    const y = x * x;
}

trait Tr {
    const F;
}

static X: f64 = 0.0

#[inline]
const COMPUTED_X: f64 = X * X;
```

A constant item can omit its type only when it is a let statement.

## Unnamed constant

Unlike an [associated constant], a [free] constant may be unnamed by using
an underscore instead of the name. For example:

```ds
const _: () =  { struct _SameNameTwice; };

// OK although it is the same name as above:
const _: () =  { struct _SameNameTwice; };
```

As with [underscore imports], macros may safely emit the same unnamed constant in
the same scope more than once. For example, the following should not produce an error:

```ds
macro m {
    ($item: item) => { $item $item }
}

m!(const _: () = (););
// This expands to:
// const _: () = ();
// const _: () = ();
```

[associated constant]: ../items/associated-items.md#associated-constants
[constant value]: ../const_eval.md#constant-expressions
[free]: ../glossary.md#free-item
[trait definition]: traits.md
[IDENTIFIER]: ../identifiers.md
[underscore imports]: use-declarations.md#underscore-imports
[_Type_]: ../types.md#type-expressions
[_Expression_]: ../expressions.md
