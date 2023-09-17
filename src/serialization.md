# Serialization

DS includes general serialization by use of the standard library's `ds::ser` module. "ser" is an abbreviation for serialization.

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

_Discriminant enumeration:_ The discriminant type of an enumeration that is serialized by discriminant must be one of (`str`, `f64`, `f32`, `u32`, `i32`, `u16`, `i16`, `u8`, `i8`, `bigint`). `bigint` maps to a string in serialized form.

```ds
#[derive(Serializable)]
#[discriminant(str)]
pub enum E {
    Variant1 = "variant1",
}
```

_Tagged enumeration:_ Tagged enumerations must specify a tag property and their discriminant type must be `str`. The tag property is specified through the `#[ser(tag = "type")]` attribute.

```ds
#[derive(Serializable)]
#[discriminant(str)]
#[ser(tag = "type")]
pub enum Enum {
    Point { x: f64, y: f64 } = "point",
}
```