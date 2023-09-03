# Serialization

DS includes general serialization by use of the standard library's `ds::ser` module. "ser" is an abbreviation for serialization.

The `ser` attribute is included in the prelude.

## Serializable

The `Serializable` trait indicates that a type is serializable and deserializable. The language does not allow the user to implement this trait; it must be derived on a struct or enum instead.

Example:

```ds
#[derive(Serializable)]
struct S {
    x: f64,
    y: f64,
}
```

A struct field can declare a serialized name using `#[ser(name = "serialized_name")]`:

```ds
#[derive(Serializable)]
struct S {
    x: f64,
    #[ser(name = "serialized_name")]
    y: f64,
}
```

## Serialization for enumerations

_Discriminant enumeration:_ The discriminant type of an enumeration that is serialized by discriminant must be either `str`, `f64`, `u32` or `i32`.

```ds
#[derive(Serializable)]
#[discriminant(str)]
pub enum E {
    Variant1 = "variant1",
}
```

_Structural enumeration:_ Structural enumerations must specify a type property and their discriminant type must be `str`. The type property is specified through the `#[ser(type_property = "type")]` attribute.

```ds
#[derive(Serializable)]
#[ser(type_property = "type")]
#[discriminant(str)]
pub enum Enum {
    Point { x: f64, y: f64 } = "point",
}
```