# Structs

> **<sup>Syntax</sup>**\
> _Struct_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructStruct_\
> &nbsp;&nbsp; | _TupleStruct_
>
> _StructQualifiers_ :\
> &nbsp;&nbsp; `extern`<sup>?</sup>
>
> _StructStruct_ :\
> &nbsp;&nbsp; _StructQualifiers_ `struct`
>   [IDENTIFIER]&nbsp;
>   [_GenericParams_]<sup>?</sup>
>   [_WhereClause_]<sup>?</sup>
>   ( `{` _StructFields_<sup>?</sup> `}` | `;` )
>
> _TupleStruct_ :\
> &nbsp;&nbsp; _StructQualifiers_ `struct`
>   [IDENTIFIER]&nbsp;
>   [_GenericParams_]<sup>?</sup>
>   `(` _TupleFields_<sup>?</sup> `)`
>   [_WhereClause_]<sup>?</sup>
>   `;`
>
> _StructFields_ :\
> &nbsp;&nbsp; _StructField_ (`,` _StructField_)<sup>\*</sup> `,`<sup>?</sup>
>
> _StructField_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; `mut`<sup>?</sup>\
> &nbsp;&nbsp; [IDENTIFIER] `:` [_Type_]\
> &nbsp;&nbsp; (`=` [_Expression_])<sup>?</sup>
>
> _TupleFields_ :\
> &nbsp;&nbsp; _TupleField_ (`,` _TupleField_)<sup>\*</sup> `,`<sup>?</sup>
>
> _TupleField_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; `mut`<sup>?</sup>\
> &nbsp;&nbsp; [_Type_]

A _struct_ is a nominal [struct type] defined with the keyword `struct`.

An example of a `struct` item and its use:

```ds
struct Point {x: i32, y: i32}
let p = Point {x: 10, y: 11};
let px: i32 = p.x;
```

A _tuple struct_ is a nominal [tuple type], also defined with the keyword
`struct`. For example:

[struct type]: ../types/struct.md
[tuple type]: ../types/tuple.md

```ds
struct Point(i32, i32);
let p = Point(10, 11);
let px: i32 = match p { Point(x, _) => x };
```

A _unit-like struct_ is a struct without any fields, defined by leaving off the
list of fields entirely. For example:

```ds
struct Cookie;
let c = [Cookie, Cookie {}, Cookie, Cookie {}];
```

## Mutability

A field is mutable by using the `mut` keyword.

## Representation

Like tuples and enums, structs are reference types. Reference equality is tested for using `.ref_eq` or `.ref_ne`.

## `rc` attribute

Adding the `rc` attribute to a struct makes it a reference-counted struct and auto implements `Hash` based on the reference.

```ds
#[rc]
struct S {}
```

The user may need to consider the `ref_eq!` and `ref_ne!` macros when comparing references or simply use `#[derive(RcEq)]`.

## External struct

When a struct includes the `extern` qualifier, it is an external struct imported from a platform. External structs have the following restrictions:

- They must not specify fields.
- They must not be initialized by code.
- They must not contain the attribute `rc`; that is, they cannot be reference-counted.

## Field default

A field can have a default initializer, allowing it to be omitted when constructing a struct using the struct initializer.

```ds
struct S {
    x: f64 = 0.0,
}
let _ = S {};
```

[_OuterAttribute_]: ../attributes.md
[IDENTIFIER]: ../identifiers.md
[_Expressions_]: ../expressions.md
[_GenericParams_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_Visibility_]: ../visibility-and-privacy.md
[_Type_]: ../types.md#type-expressions
