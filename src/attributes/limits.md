# Limits

The following [attributes] affect compile-time limits.

## The `recursion_limit` attribute

The *`recursion_limit` attribute* may be applied at the [crate] level to set the
maximum depth for potentially infinitely-recursive compile-time operations
like macro expansion or name delegation. It uses the [_MetaNameValueStr_]
syntax to specify the recursion depth.

> Note: The default in the compiler is 128.

```ds,compile_fail
#![recursion_limit = "4"]

macro a {
    () => { a!(1); },
    (1) => { a!(2); },
    (2) => { a!(3); },
    (3) => { a!(4); },
    (4) => { },
}

// This fails to expand because it requires a recursion depth greater than 4.
a!{}
```

## The `type_length_limit` attribute

The *`type_length_limit` attribute* limits the maximum number of type
substitutions made when constructing a concrete type during monomorphization.
It is applied at the [crate] level, and uses the [_MetaNameValueStr_] syntax
to set the limit based on the number of type substitutions.

> Note: The default in the compiler is 1048576.

```ds,compile_fail
#![type_length_limit = "4"]

fn f<T>(x: T) {}

// This fails to compile because monomorphizing to
// `f::<((((i32,), i32), i32), i32)>` requires more than 4 type elements.
f(((((1,), 2), 3), 4));
```

[_MetaNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[attributes]: ../attributes.md
[crate]: ../crates-and-source-files.md
