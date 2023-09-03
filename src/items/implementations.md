# Implementations

> **<sup>Syntax</sup>**\
> _Implementation_ :\
> &nbsp;&nbsp; _InherentImpl_ | _TraitImpl_
>
> _InherentImpl_ :\
> &nbsp;&nbsp; `impl` [_GenericParams_]<sup>?</sup>&nbsp;[_Type_]&nbsp;[_WhereClause_]<sup>?</sup> `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; [_AssociatedItem_]<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _TraitImpl_ :\
> &nbsp;&nbsp; `impl` [_GenericParams_]<sup>?</sup>
>              [_TypePath_] `for` [_Type_]\
> &nbsp;&nbsp; [_WhereClause_]<sup>?</sup>\
> &nbsp;&nbsp; `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; [_AssociatedItem_]<sup>\*</sup>\
> &nbsp;&nbsp; `}`

An _implementation_ is an item that associates items with an _implementing type_.
Implementations are defined with the keyword `impl` and contain functions
that belong to an instance of the type that is being implemented or to the
type statically.

There are two types of implementations:

- inherent implementations
- [trait] implementations

## Inherent Implementations

An inherent implementation is defined as the sequence of the `impl` keyword,
generic type declarations, a path to a nominal type, a where clause, and a
bracketed set of associable items.

The nominal type is called the _implementing type_ and the associable items are
the _associated items_ to the implementing type.

Inherent implementations associate the contained items to the
implementing type.  Inherent implementations can contain [associated
functions] (including [methods]) and [associated constants]. They cannot
contain associated type aliases.

The [path] to an associated item is any path to the implementing type,
followed by the associated item's identifier as the final path
component.

A type can also have multiple inherent implementations. An implementing type
must be defined within the same crate as the original type definition.

``` ds
pub mod color {
    pub struct Color(pub u32, pub u32, pub u32);

    impl Color {
        pub const WHITE: Color = Color(255, 255, 255);
    }
}

mod values {
    use super::color::Color;
    impl Color {
        pub fn red() -> Color {
            Color(255, 0, 0)
        }
    }
}

pub use self::color::Color;
fn main() {
    // Actual path to the implementing type and impl in the same module.
    color::Color::WHITE;

    // Impl blocks in different modules are still accessed through a path to the type.
    color::Color::red();

    // Re-exported paths to the implementing type also work.
    Color::red();

    // Does not work, because use in `values` is not pub.
    // values::Color::red();
}
```

## Trait Implementations

A _trait implementation_ is defined like an inherent implementation except that
the optional generic type declarations are followed by a [trait], followed
by the keyword `for`, followed by a path to a nominal type.

<!-- To understand this, you have to back-reference to the previous section. :( -->

The trait is known as the _implemented trait_. The implementing type
implements the implemented trait.

A trait implementation must define all non-default associated items declared
by the implemented trait, may redefine default associated items defined by the
implemented trait, and cannot define any other items.

The path to the associated items is `<` followed by a path to the implementing
type followed by `as` followed by a path to the trait followed by `>` as a path
component followed by the associated item's path component.

```ds
struct Circle {
    radius: f64,
    center: Point,
}

impl Shape for Circle {
    fn draw(self, s: Surface) { do_draw_circle(s, self); }
    fn bounding_box(self) -> BoundingBox {
        let r = self.radius;
        BoundingBox {
            x: self.center.x - r,
            y: self.center.y - r,
            width: 2.0 * r,
            height: 2.0 * r,
        }
    }
}
```

### Trait Implementation Coherence

A trait implementation is considered incoherent if either the orphan rules check fails
or there are overlapping implementation instances.

Two trait implementations overlap when there is a non-empty intersection of the
traits the implementation is for, the implementations can be instantiated with
the same type. <!-- This is probably wrong? Source: No two implementations can
be instantiable with the same set of types for the input type parameters. -->

#### Orphan rules

Given `impl<P1..=Pn> Trait<T1..=Tn> for T0`, an `impl` is valid only if at
least one of the following is true:

- `Trait` is a [local trait]
- All of
  - At least one of the types `T0..=Tn` must be a [local type]. Let `Ti` be the
    first such type.
  - No [uncovered type] parameters `P1..=Pn` may appear in `T0..Ti` (excluding
    `Ti`)

## Generic Implementations

An implementation can take [generic parameters], which can be used in the rest
of the implementation. Implementation parameters are written directly after the
`impl` keyword.

```ds
impl<T> Seq<T> for [T] {
    /* ... */
}
impl Seq<bool> for u32 {
    /* Treat the integer as a sequence of bits */
}
```

Generic parameters *constrain* an implementation if the parameter appears at
least once in one of:

* The implemented trait, if it has one
* The implementing type
* As an [associated type] in the [bounds] of a type that contains another
  parameter that constrains the implementation

Type parameters must always constrain the implementation.

Examples of constraining situations:

```ds
// T constrains by being an argument to GenericTrait.
impl<T> GenericTrait<T> for i32 { /* ... */ }

// T constrains by being an argument to GenericStruct
impl<T> Trait for GenericStruct<T> { /* ... */ }

// T constrains by being in an associated type in a bound for type `U` which is
// itself a generic parameter constraining the trait.
impl<T, U> GenericTrait<U> for u32 where U: HasAssocType<Ty = T> { /* ... */ }

// Like previous, except the type is `(U, i32)`. `U` appears inside the type
// that includes `T`, and is not the type itself.
impl<T, U> GenericStruct<U> where (U, i32): HasAssocType<Ty = T> { /* ... */ }
```

Examples of non-constraining situations:

```ds
// The rest of these are errors, since they have type or const parameters that
// do not constrain.

// T does not constrain since it does not appear at all.
impl<T> Struct { /* ... */ }

// Usage of T inside the implementation does not constrain the impl.
impl<T> Struct {
    fn uses_t(t: T) { /* ... */ }
}

// T is used as an associated type in the bounds for U, but U does not constrain.
impl<T, U> Struct where U: HasAssocType<Ty = T> { /* ... */ }

// T is used in the bounds, but not as an associated type, so it does not constrain.
impl<T, U> GenericTrait<U> for u32 where U: GenericTrait<T> {}
```

## Attributes on Implementations

Implementations may contain outer [attributes] before the `impl` keyword and
inner [attributes] inside the brackets that contain the associated items. Inner
attributes must come before any associated items. The attributes that have
meaning here are [`cfg`], [`deprecated`], [`doc`], and [the lint check
attributes].

[_AssociatedItem_]: associated-items.md
[_GenericParams_]: generics.md
[_InnerAttribute_]: ../attributes.md
[_TypePath_]: ../paths.md#paths-in-types
[_Type_]: ../types.md#type-expressions
[_WhereClause_]: generics.md#where-clauses
[trait]: traits.md
[associated constants]: associated-items.md#associated-constants
[associated functions]: associated-items.md#associated-functions-and-methods
[associated type]: associated-items.md#associated-types
[attributes]: ../attributes.md
[bounds]: ../trait-bounds.md
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../dsdoc/the-doc-attribute.html
[generic parameters]: generics.md
[methods]: associated-items.md#methods
[path]: ../paths.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[Unsafe traits]: traits.md#unsafe-traits
[local trait]: ../glossary.md#local-trait
[local type]: ../glossary.md#local-type
[fundamental types]: ../glossary.md#fundamental-type-constructors
[uncovered type]: ../glossary.md#uncovered-type
