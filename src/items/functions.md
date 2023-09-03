# Functions

> **<sup>Syntax</sup>**\
> _Function_ :\
> &nbsp;&nbsp; _FunctionQualifiers_ `fn` [IDENTIFIER]&nbsp;[_GenericParams_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `(` _FunctionParameters_<sup>?</sup> `)`\
> &nbsp;&nbsp; &nbsp;&nbsp; _FunctionReturnType_<sup>?</sup> [_WhereClause_]<sup>?</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; ( [_BlockExpression_] | `;` )
>
> _FunctionQualifiers_ :\
> &nbsp;&nbsp; `async`<sup>?</sup> `extern`<sup>?</sup>
>
> _FunctionParameters_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _SelfParam_ `,`<sup>?</sup>\
> &nbsp;&nbsp; | (_SelfParam_ `,`)<sup>?</sup> _FunctionParam_ (`,` _FunctionParam_)<sup>\*</sup> `,`<sup>?</sup>
>
> _SelfParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( _ShorthandSelf_ | _TypedSelf_ )
>
> _ShorthandSelf_ :\
> &nbsp;&nbsp;  `self`
>
> _TypedSelf_ :\
> &nbsp;&nbsp; `self` `:` [_Type_]
>
> _FunctionParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (
>   _FunctionParamPattern_ | `...` | [_Type_]
> )
>
> _FunctionParamPattern_ :\
> &nbsp;&nbsp; [_PatternNoTopAlt_] `:` ( [_Type_] | `...` )
>
> _FunctionReturnType_ :\
> &nbsp;&nbsp; `->` [_Type_]

A _function_ consists of a [block], along with a name, a set of parameters, and an output type.
Other than a name, all these are optional.
Functions are declared with the keyword `fn`.
Functions may declare a set of *input* [*variables*][variables] as parameters, through which the caller passes arguments into the function, and the *output* [*type*][type] of the value the function will return to its caller on completion.
If the output type is not explicitly stated, it is the [unit type].

When referred to, a _function_ yields a first-class *value* of the corresponding [*function type*], which when called evaluates to a direct call to the function.

For example, this is a simple function:
```ds
fn answer_to_life_the_universe_and_everything() -> i32 {
    return 42;
}
```

## Function parameters

Function parameters are irrefutable [patterns], so any pattern that is valid in
an else-less `let` binding is also valid as a parameter:

```ds
fn first((value, _): (i32, i32)) -> i32 { value }
```

If the first parameter is a _SelfParam_, this indicates that the function is a
[method]. Functions with a self parameter may only appear as an [associated
function] in a [trait] or [implementation].

If the function is a method named `new` belonging to a `struct`, it is a [constructor](#constructor) method.

A parameter with the `...` token indicates a [variadic function], and may only
be used as the last parameter of an external function. The variadic
parameter may have an optional identifier, such as `args: ...`.

## Function body

The block of a function is conceptually wrapped in a block that binds the
argument patterns and then `return`s the value of the function's block. This
means that the tail expression of the block, if evaluated, ends up being
returned to the caller. As usual, an explicit return expression within
the body of the function will short-cut that implicit return, if reached.

For example, the function above behaves as if it was written as:

```ds
// argument_0 is the actual first argument passed from the caller
let (value, _) = argument_0;
return {
    value
};
```

Functions without a body block are terminated with a semicolon. This form
may only appear in a [trait] or when the function is qualified by `extern`.

## Constructor

A constructor method (a `new` method inside a `struct`) must have a signature that takes at least `self` and returns `()`. It is a compile error if the inherited structure is not directly `Object` and the constructor method is not external and does not contain a `super();` statement at the top of the block.

Example of a constructor:

```
struct S;
impl S {
    fn new(self) {}
}
```

## Generic functions

A _generic function_ allows one or more _parameterized types_ to appear in its
signature. Each type parameter must be explicitly declared in an
angle-bracket-enclosed and comma-separated list, following the function name.

```ds
// foo is generic over A and B

fn foo<A, B>(x: A, y: B) {}
```

Inside the function signature and body, the name of the type parameter can be
used as a type name. [Trait] bounds can be specified for type
parameters to allow methods with that trait to be called on values of that
type. This is specified using the `where` syntax:

```ds
use ds::fmt::Debug;
fn foo<T>(x: T) where T: Debug {}
```

When a generic function is referenced, its type is instantiated based on the
context of the reference. For example, calling the `foo` function here:

```ds
use ds::fmt::Debug;

fn foo<T>(x: [T]) where T: Debug {
    // details elided
}

foo([1, 2]);
```

will instantiate type parameter `T` with `i32`.

The type parameters can also be explicitly supplied in a trailing [path]
component after the function name. This might be necessary if there is not
sufficient context to determine the type parameters. For example,
`memory::size_of::<u32>() == 4`.

## External functions

The `extern` keyword indicates that a function is external and is imported from ActionScript. External functions must contain no body. An external function can be variadic.

An external function may use the `#[actionscript]` attribute with the following possible key-value pairs and names:

- `package = "q1.q2"`: indicates the ActionScript package that qualifies the function. The `public` visibility is used.
- `name = "name"`: indicates the ActionScript property name.
- `get`: indicates that the imported ActionScript item is a variable or virtual property and that the external function is equivalent to a get-property operation.
- `set`: indicates that the imported ActionScript item is a variable or virtual property and that the external function is equivalent to a set-property operation.

Examples of external functions:

```
#[actionscript(package = "com.q")]
extern fn some_function();

#[actionscript(package = "com.q", name = "camelCase")]
extern fn snake_case();

#[actionscript(get, package = "com.q", name = "x")]
extern fn x();

impl S {
    #[actionscript(get, name = "x")]
    extern fn x(self) -> f64;

    #[actionscript(set, name = "x")]
    extern fn set_x(self, value: f64);
}
```

## Async functions

Functions may be qualified as async:

```ds
async fn regular_example() { }
```

Async functions execute directly when called, returning the `Future<T>` structure.

```ds
// Source
async fn example(x: str) -> u32 {
    x.len()
}
```

is roughly equivalent to:

```ds
// Desugared
fn example(x: str) -> Future<u32> {
    async { x.len() }
}
```

For more information on the effect of async, see [`async` blocks][async-blocks].

[async-blocks]: ../expressions/block-expr.md#async-blocks

## Generator functions

Functions that contain the `yield` operator are generators. A generator returns `Iterator`.

```ds
// fn f() -> Iterator<Item = i32>
fn f() -> i32 {
    for i in 0..10 {
        yield i;
    }
}
```

## Asynchronous generators

A generator can be asynchronous, returning `Iterator<Item = Future<T>>`.

```ds
// fn f() -> Iterator<Item = Future<i32>>
async fn f() -> i32 {
    for i in 0..10 {
        yield fetch_integer().await;
    }
}
```

## Attributes on functions

[Outer attributes][attributes] are allowed on functions. [Inner
attributes][attributes] are allowed directly after the `{` inside its [block].

This example shows an inner attribute on a function. The function is documented
with just the word "Example".

```ds
fn documented() {
    #![doc = "Example"]
}
```

> Note: Except for lints, it is idiomatic to only use outer attributes on
> function items.

The attributes that have meaning on a function are `actionscript`, [`cfg`], [`cfg_attr`], [`deprecated`],
[`doc`], [the lint check
attributes], [the procedural macro attributes], [the testing
attributes], and [the optimization hint attributes]. Functions also accept
attributes macros.

## Attributes on function parameters

[Outer attributes][attributes] are allowed on function parameters and the
permitted [built-in attributes] are restricted to `cfg`, `cfg_attr`, `allow`,
`warn`, `deny`, and `forbid`.

```ds
fn f(
    #[cfg(foo)] a: [f64],
    #[cfg(not(foo))] a: [u32],
) {
}
```

Inert helper attributes used by procedural macro attributes applied to items are also
allowed but be careful to not include these inert attributes in your final `TokenStream`.

For example, the following code defines an inert `some_inert_attribute` attribute that
is not formally defined anywhere and the `some_proc_macro_attribute` procedural macro is
responsible for detecting its presence and removing it from the output token stream.

```ds,ignore
#[some_proc_macro_attribute]
fn foo_oof(#[some_inert_attribute] arg: u32) {
}
```

[IDENTIFIER]: ../identifiers.md
[RAW_STRING_LITERAL]: ../tokens.md#raw-string-literals
[STRING_LITERAL]: ../tokens.md#string-literals
[_BlockExpression_]: ../expressions/block-expr.md
[_GenericParams_]: generics.md
[_Lifetime_]: ../trait-bounds.md
[_PatternNoTopAlt_]: ../patterns.md
[_Type_]: ../types.md#type-expressions
[_WhereClause_]: generics.md#where-clauses
[_OuterAttribute_]: ../attributes.md
[const contexts]: ../const_eval.md#const-context
[const functions]: ../const_eval.md#const-functions
[tuple struct]: structs.md
[tuple variant]: enumerations.md
[`extern`]: #extern-function-qualifier
[external block]: external-blocks.md
[path]: ../paths.md
[block]: ../expressions/block-expr.md
[variables]: ../variables.md
[type]: ../types.md#type-expressions
[unit type]: ../types/tuple.md
[*function type*]: ../types/function.md
[Trait]: traits.md
[attributes]: ../attributes.md
[`cfg`]: ../conditional-compilation.md#the-cfg-attribute
[`cfg_attr`]: ../conditional-compilation.md#the-cfg_attr-attribute
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[the procedural macro attributes]: ../procedural-macros.md
[the testing attributes]: ../attributes/testing.md
[the optimization hint attributes]: ../attributes/codegen.md#optimization-hints
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../dsdoc/the-doc-attribute.html
[`must_use`]: ../attributes/diagnostics.md#the-must_use-attribute
[patterns]: ../patterns.md
[`export_name`]: ../abi.md#the-export_name-attribute
[`link_section`]: ../abi.md#the-link_section-attribute
[`no_mangle`]: ../abi.md#the-no_mangle-attribute
[built-in attributes]: ../attributes.html#built-in-attributes-index
[trait item]: traits.md
[method]: associated-items.md#methods
[associated function]: associated-items.md#associated-functions-and-methods
[implementation]: implementations.md
[variadic function]: external-blocks.md#variadic-functions
