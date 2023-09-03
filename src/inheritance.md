# Inheritance

This section describes operations used when working with struct inheritance.

## Cast

A cast from an object to a type `T` in terms of inheritance succeeds if the object is equals to or is a subtype of `T`.

There are two operations for casting:

- The `cast!(node, T)`: casts `node` into `T` by panicking on failure.
- The `try_cast!(node, T)`: casts `node` into `T` by returning `Result<T, CastError>`.

Examples:

```ds
if let Ok(button) = try_cast!(object, Button) {
    // button: Button
}
```

## Relationship

The `is_type!(node, T)` operator determines if `node` is of type `T` or a subtype of `T`.

## Restrictions

The `cast!`, `try_cast!` and `is_type!` operators do not accept the following types as second argument:

- `Option<T>`
- `[T]`
- `fn`
- `Map<T>`
- `WeakMap<T>`
- `Set<T>`
- Generic external structs
- Generic external traits

For `Option<T>` in particular, these operators treat `Option<T>` as `T`, as described in [Special Behavior](#special-behavior).

## Special Behavior

The `cast!`, `try_cast!` and `is_type!` operators have the following special behaviors:

- They naturally take an `Option<T>` value as `T`.