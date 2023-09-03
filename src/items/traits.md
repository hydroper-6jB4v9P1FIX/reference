# Traits

> **<sup>Syntax</sup>**\
> _Trait_ :\
> &nbsp;&nbsp; _TraitQualifiers_<sup>?</sup> `trait` [IDENTIFIER]&nbsp;
>              [_GenericParams_]<sup>?</sup>
>              ( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup>
>              [_WhereClause_]<sup>?</sup> `{`\
> &nbsp;&nbsp;&nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp;&nbsp;&nbsp; [_AssociatedItem_]<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _TraitQualifiers_ :\
> &nbsp;&nbsp; `extern`<sup>?</sup>

A _trait_ describes an abstract interface type that other types can implement. This
interface consists of [associated items], which come in three varieties:

- [functions](associated-items.md#associated-functions-and-methods)
- [types](associated-items.md#associated-types)
- [constants](associated-items.md#associated-constants)

All traits define an implicit type parameter `Self` that refers to "the type
that is implementing this interface". Traits may also contain additional type
parameters. These type parameters, including `Self`, may be constrained by
other traits and so forth [as usual][generics].

Traits are implemented for specific types through separate [implementations].

Trait functions may omit the function body by replacing it with a semicolon.
This indicates that the implementation must define the function. If the trait
function defines a body, this definition acts as a default for any
implementation which does not override it. Similarly, associated constants may
omit the equals sign and expression to indicate implementations must define
the constant value. Associated types must never define the type, the type may
only be specified in an implementation.

```ds
// Examples of associated trait items with and without definitions.
trait Example {
    const CONST_NO_DEFAULT: i32;
    const CONST_WITH_DEFAULT: i32 = 99;
    type TypeNoDefault;
    fn method_without_default(self);
    fn method_with_default(self) {}
}
```

Trait functions are not allowed to be [`async`] or [`extern`].

## Trait objects

A trait can be used normally as an opaque type.

```ds
trait Tr {}
struct S;
impl Tr for S {}

fn main() {
    let o: S = S;
    let o: Tr = S;
}
```

## Trait bounds

Generic items may use traits as [bounds] on their type parameters.

## Generic Traits

Type parameters can be specified for a trait to make it generic. These appear
after the trait name, using the same syntax used in [generic functions].

```ds
trait Seq<T> {
    fn len(self) -> u32;
    fn elt_at(self, n: u32) -> T;
}
```

## Supertraits

**Supertraits** are traits that are required to be implemented for a type to
implement a specific trait. Furthermore, anywhere a [generic][generics] is bounded by a trait, it has access to the associated items of its supertraits.

Supertraits are declared by trait bounds on the `Self` type of a trait and
transitively the supertraits of the traits declared in those trait bounds. It is
an error for a trait to be its own supertrait.

The trait with a supertrait is called a **subtrait** of its supertrait.

The following is an example of declaring `Shape` to be a supertrait of `Circle`.

```ds
trait Shape { fn area(self) -> f64; }
trait Circle : Shape { fn radius(self) -> f64; }
```

And the following is the same example, except using [where clauses].

```ds
trait Shape { fn area(self) -> f64; }
trait Circle where Self: Shape { fn radius(self) -> f64; }
```

This next example gives `radius` a default implementation using the `area`
function from `Shape`.

```ds
trait Circle where Self: Shape {
    fn radius(self) -> f64 {
        // A = pi * r^2
        // so algebraically,
        // r = sqrt(A / pi)
        (self.area() / PI).sqrt()
    }
}
```

This next example calls a supertrait method on a generic parameter.

```ds
fn print_area_and_radius<C: Circle>(c: C) {
    // Here we call the area method from the supertrait `Shape` of `Circle`.
    print!("Area: {}", c.area());
    print!("Radius: {}", c.radius());
}
```

## Parameter patterns

Function or method declarations without a body only allow [IDENTIFIER] or
`_` [wild card][WildcardPattern] patterns. `mut` [IDENTIFIER] is not allowed.

The kinds of patterns for parameters is limited to one of the following:

* [IDENTIFIER]
* `mut` [IDENTIFIER]
* [`_`][WildcardPattern]

All irrefutable patterns are allowed as long as there
is a body. Without a body, the limitations listed above are still in effect.

```ds
trait T {
    fn f1((a, b): (i32, i32)) {}
    fn f2(_: (i32, i32));  // Cannot use tuple pattern without a body.
}
```

## Item visibility

Trait items syntactically allow a [_Visibility_] annotation, but this is
rejected when the trait is validated. This allows items to be parsed with a
unified syntax across different contexts where they are used. As an example,
an empty `vis` macro fragment specifier can be used for trait items, where the
macro rule may be used in other situations where visibility is allowed.

```ds
macro create_method {
    ($vis:vis $name:ident) => {
        $vis fn $name(self) {}
    };
}

trait T1 {
    // Empty `vis` is allowed.
    create_method! { method_of_t1 }
}

struct S;

impl S {
    // Visibility is allowed here.
    create_method! { pub method_of_s }
}

impl T1 for S {}

fn main() {
    let s = S;
    s.method_of_t1();
    s.method_of_s();
}
```

## External trait

When a trait includes the `extern` qualifier, it is an external interface imported from ActionScript.

An external trait may use the `#[actionscript]` attribute with the following possible key-value pairs and names:

- `package = "q1.q2"`: indicates the ActionScript package that qualifies the interface. The `public` visibility is used.
- `name = "name"`: indicates the ActionScript property name.

Example of an external trait:

```ds
#[actionscript(package = "com.q", name = "Itrfc")]
extern trait Tr;
```

[IDENTIFIER]: ../identifiers.md
[WildcardPattern]: ../patterns.md#wildcard-pattern
[_AssociatedItem_]: associated-items.md
[_GenericParams_]: generics.md
[_InnerAttribute_]: ../attributes.md
[_TypeParamBounds_]: ../trait-bounds.md
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where-clauses
[bounds]: ../trait-bounds.md
[trait object]: ../types/trait-object.md
[RFC 255]: https://github.com/ds-lang/rfcs/blob/master/text/0255-object-safety.md
[associated items]: associated-items.md
[method]: associated-items.md#methods
[supertraits]: #supertraits
[implementations]: implementations.md
[generics]: generics.md
[where clauses]: generics.md#where-clauses
[generic functions]: functions.md#generic-functions
[unsafe]: ../unsafety.md
[trait implementation]: implementations.md#trait-implementations
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[`async`]: functions.md#async-functions
[`extern`]: functions.md#external-functions
