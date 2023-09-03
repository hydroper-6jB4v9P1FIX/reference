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
>   _InheritedStruct_<sup>?</sup>
>   [_WhereClause_]<sup>?</sup>
>   ( `{` _StructFields_<sup>?</sup> `}` | `;` )
>
> _TupleStruct_ :\
> &nbsp;&nbsp; _StructQualifiers_ `struct`
>   [IDENTIFIER]&nbsp;
>   [_GenericParams_]<sup>?</sup>
>   `(` _TupleFields_<sup>?</sup> `)`
>   _InheritedStruct_<sup>?</sup>
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
>
> _InheritedStruct_ :\
> &nbsp;&nbsp; `:` [_Type_]

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

## Constructor

When a struct directly implements a [constructor method](functions.md#constructor), the tuple initializer is replaced by a constructor call that executes additional code and returns an instance of the struct.

## Representation

A struct constructs managed object references. More generally, all types, excluding primitives, are managed object types.

## Reference equality

Deriving reference equality (`RefEq`) means a struct is compared by reference when using the equality operators.

```ds
#[derive(RefEq)]
struct S;
```

## External struct

When a struct includes the `extern` qualifier, it is an external struct imported from ActionScript.

An external struct may use the `#[actionscript]` attribute with the following possible key-value pairs and names:

- `package = "q1.q2"`: indicates the ActionScript package that qualifies the struct. The `public` visibility is used.
- `name = "name"`: indicates the ActionScript property name.

Example of an external struct:

```ds
#[actionscript(package = "com.q", name = "C")]
extern struct C;
```

## Inheritance

A struct `S2` may inherit another struct `S1` by using a colon (`struct S2 : S1;`). In that case, `S1` must be inheritable (`#[inheritable]`) and `S1` must not be a subtype of `S2` already.

Cast and relationship operators are described in [Inheritance](../inheritance.md).

Here is an example of inheritance:

```ds
use air::display::*;

struct MySprite: DisplayObject;

impl MySprite {
    fn new(self) {
        super();
    }
}
```

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
