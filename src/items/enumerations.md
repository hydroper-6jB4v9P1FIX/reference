# Enumerations

> **<sup>Syntax</sup>**\
> _Enumeration_ :\
> &nbsp;&nbsp; `enum`
>    [IDENTIFIER]&nbsp;
>    [_GenericParams_]<sup>?</sup>
>    [_WhereClause_]<sup>?</sup>
>    `{` _EnumItems_<sup>?</sup> `}`
>
> _EnumItems_ :\
> &nbsp;&nbsp; _EnumItem_ ( `,` _EnumItem_ )<sup>\*</sup> `,`<sup>?</sup>
>
> _EnumItem_ :\
> &nbsp;&nbsp; _OuterAttribute_<sup>\*</sup> [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;( _EnumItemTuple_ | _EnumItemStruct_ )<sup>?</sup>
>                                _EnumItemDiscriminant_<sup>?</sup>
>
> _EnumItemTuple_ :\
> &nbsp;&nbsp; `(` [_TupleFields_]<sup>?</sup> `)`
>
> _EnumItemStruct_ :\
> &nbsp;&nbsp; `{` [_StructFields_]<sup>?</sup> `}`
>
> _EnumItemDiscriminant_ :\
> &nbsp;&nbsp; `=` [_Expression_]

An *enumeration*, also referred to as an *enum*, is a simultaneous definition of a
nominal [enumerated type] as well as a set of *constructors*, that can be used
to create or pattern-match values of the corresponding enumerated type.

Enumerations are declared with the keyword `enum`.

An example of an `enum` item and its use:

```ds
enum Animal {
    Dog,
    Cat,
}

let mut a: Animal = Animal::Dog;
a = Animal::Cat;
```

Enum constructors can have either named or unnamed fields:

```ds
enum Animal {
    Dog(str, f64),
    Cat { name: str, weight: f64 },
}

let mut a: Animal = Animal::Dog("Cocoa", 37.2);
a = Animal::Cat { name: "Spotty", weight: 2.7 };
```

In this example, `Cat` is a _struct-like enum variant_, whereas `Dog` is simply
called an enum variant.

An enum where no constructors contain fields are called a
*<span id="field-less-enum">field-less enum</span>*. For example, this is a fieldless enum:

```ds
enum Fieldless {
    Tuple(),
    Struct{},
    Unit,
}
```

If a field-less enum only contains unit variants, the enum is called an
*<span id="unit-only-enum">unit-only enum</span>*. For example:

```ds
enum Enum {
    Foo = 3,
    Bar = 2,
    Baz = 1,
}
```

## Discriminants

Each enum instance has a _discriminant_. The discriminant type is `()` by default. A discriminant type `T` can be specified by using `#[discriminant(T)]` and `T` must implement `Eq | PartialEq`; for example:

```ds
#[discriminant(str)]
enum E {
    Foo = "foo",
    Bar(f64) = "bar",
}
```

A discriminant value is specified by using an `=` assignment following the variant. If the discriminant type is any other than `()`, then all variants must specify their discriminant.

Variants with duplicate discriminant are not allowed.

### Accessing discriminant

The discriminant of an enumeration can be accessed via the `Into<D>` trait, where `D` is the discriminant type, if the enum derives `PartialEq`. `Into<D>` is not implemented if `PartialEq` is not derived.

Example:

```ds
#[discriminant(str)]
#[derive(PartialEq)]
enum E {
    X = "x",
    Y = "y",
}
assert_eq!("x", E::X.into());
```

### Retrieving variant from discriminant

Retrieving a variant from a discriminant value is possible for unit-only enums. `TryFrom<D>` is implemented for unit-only `PartialEq` enums where `D` is the discriminant type.

```ds
#[discriminant(str)]
#[derive(PartialEq)]
enum E {
    X = "x",
    Y = "y",
}
assert_eq!(Ok(E::X), E::try_from("x"));
```

## Variant visibility

Enum variants syntactically allow a [_Visibility_] annotation, but this is
rejected when the enum is validated. This allows items to be parsed with a
unified syntax across different contexts where they are used.

```ds
macro mac_variant {
    ($vis:vis $name:ident) => {
        enum $name {
            $vis Unit,

            $vis Tuple(f64, u32),

            $vis Struct { f: f64 },
        }
    }
}

// Empty `vis` is allowed.
mac_variant! { E }

// This is allowed, since it is removed before being validated.
#[cfg(FALSE)]
enum E {
    pub U,
    pub(crate) T(u32),
    pub(super) T { f: str }
}
```

## Representation

Enums are passed by value by default. The `rc` attribute indicates the enum is passed by reference.

## `rc` attribute

Adding the `rc` attribute to an enum makes it a reference-counted enum and auto implements `Hash` based on the reference.

```ds
#[rc]
enum E {}
```

The user may need to consider the `rc_eq!` and `rc_ne!` macros when comparing references or simply use `#[derive(RcEq)]`.

[IDENTIFIER]: ../identifiers.md
[_GenericParams_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_Expression_]: ../expressions.md
[_TupleFields_]: structs.md
[_StructFields_]: structs.md
[_Visibility_]: ../visibility-and-privacy.md
[enumerated type]: ../types/enum.md
[never type]: ../types/never.md
[unit-only]: #unit-only-enum
[numeric cast]: ../expressions/operator-expr.md#semantics
[constant expression]: ../const_eval.md#constant-expressions
[default representation]: ../type-layout.md#the-default-representation
[primitive representation]: ../type-layout.md#primitive-representations
[`C` representation]: ../type-layout.md#the-c-representation
[Field-less enums]: #field-less-enum
