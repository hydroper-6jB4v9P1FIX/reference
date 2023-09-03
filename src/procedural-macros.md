## Procedural Macros

*Procedural macros* allow creating syntax extensions as execution of a function.
Procedural macros come in one of three flavors:

* [Function-like macros] - `custom!(...)`
* [Derive macros] - `#[derive(CustomDerive)]`
* [Attribute macros] - `#[custom_attribute]`

Procedural macros allow you to run code at compile time that operates over Rust
syntax, both consuming and producing Rust syntax. You can sort of think of
procedural macros as functions from an AST to another AST.

Procedural macros are implemented as separate processes written in the Rust language.

As functions, they must either return syntax, panic, or loop endlessly. Returned
syntax either replaces or adds the syntax depending on the kind of procedural
macro. Panics are caught by the compiler and are turned into a compiler error.
Endless loops are not caught by the compiler which hangs the compiler.

Procedural macros run during compilation, and thus have the same resources that
the compiler has. For example, standard input, error, and output are the same
that the compiler has access to. Similarly, file access is the same.

Procedural macros have two ways of reporting errors. The first is to panic. The
second is to emit a `compile_error` macro invocation.

### The `ds_proc_macro` crate for Rust

Procedural macros are written in Rust by depending on the crate `ds_proc_macro`. The entry point for the Rust project is as follows:

```rust
use ds_proc_macro::{TokenStream, proc_macros};

proc_macros! {
    #[proc_macro]
    fn my_function_macro(input: TokenStream) -> TokenStream {
        output
    }

    #[proc_macro_attribute]
    fn my_attribute_macro(attribute: TokenStream, item: TokenStream) -> TokenStream {
        output
    }

    #[proc_macro_derive]
    fn my_derive_macro(item_input: TokenStream) -> TokenStream {
        // `output` is appended to module or block that the item is in.
        output
    }
}
```

This crate primarily contains a [`TokenStream`] type. Procedural macros operate
over *token streams* instead of AST nodes, which is a far more stable interface
over time for both the compiler and for procedural macros to target. A
*token stream* is roughly equivalent to `Vec<TokenTree>` where a `TokenTree`
can roughly be thought of as lexical token. For example `foo` is an `Ident`
token, `.` is a `Punct` token, and `1.2` is a `Literal` token. The `TokenStream`
type, unlike `Vec<TokenTree>`, is cheap to clone.

All tokens have an associated `Span`. A `Span` is an opaque value that cannot
be modified but can be manufactured. `Span`s represent an extent of source
code within a program and are primarily used for error reporting. While you
cannot modify a `Span` itself, you can always change the `Span` *associated*
with any token, such as through getting a `Span` from another token.

### Procedural macro hygiene

Procedural macros are *unhygienic*. This means they behave as if the output
token stream was simply written inline to the code it's next to. This means that
it's affected by external items and also affects external imports.

Macro authors need to be careful to ensure their macros work in as many contexts
as possible given this limitation. This often includes using absolute paths to
items in libraries (for example, `::ds::option::Option` instead of `Option`) or
by ensuring that generated functions have names that are unlikely to clash with
other functions (like `__internal_foo` instead of `foo`).

### The `proc_macros!` section

A series of procedural macros are defined within the `proc_macros!` macro. It refers to a Rust project by indicating its path relative to the current source's parent directory:

```
proc_macros! {
    lang = "rust";
    path = "./my_proc_macros";

    // procedural macros here...
}
```

### Function-like procedural macros

*Function-like procedural macros* are procedural macros that are invoked using
the macro invocation operator (`!`).

These macros are defined by a function with the `proc_macro`
[attribute] inside a `proc_macros!` section, using an empty signature.

```
proc_macros! {
    lang = "rust";
    path = "./my_proc_macros";

    #[proc_macro]
    fn my_function_macro();
}
```

Function-like procedural macros may be invoked in any macro invocation
position, which includes [statements], [expressions], [patterns], [type
expressions], [item] positions, including items in [`extern` blocks], inherent
and trait [implementations], and [trait definitions].

### Derive macros

*Derive macros* define new inputs for the [`derive` attribute]. These macros
can create new [items] given the token stream of a [struct] or [enum].
They can also define [derive macro helper attributes].

Custom derive macros are defined by a function with the
`proc_macro_derive` attribute and an empty signature inside a `proc_macros` section:

```
proc_macros! {
    lang = "rust";
    path = "./my_proc_macros";

    #[proc_macro_derive(DeriveMacro)]
    fn my_derive_macro();
}
```

#### Derive macro helper attributes

Derive macros can add additional [attributes] into the scope of the [item]
they are on. Said attributes are called *derive macro helper attributes*. These
attributes are [inert], and their only purpose is to be fed into the derive
macro that defined them. That said, they can be seen by all macros.

The way to define helper attributes is to put an `attributes` key in the
`proc_macro_derive` attribute with a comma separated list of identifiers that are
the names of the helper attributes.

For example, the following derive macro defines a helper attribute
`helper`, but ultimately doesn't do anything with it.

```ds
proc_macros! {
    lang = "rust";
    path = "./my_proc_macros";

    #[proc_macro_derive(HelperAttr, attributes(helper))]
    fn derive_helper_attr();
}
```

The process:

```rust
use ds_proc_macro::{TokenStream, proc_macros};

proc_macros! {
    #[proc_macro_derive]
    fn derive_helper_attr(_item_input: TokenStream) -> TokenStream {
        TokenStream::new()
    }
}
```

And then usage on the derive macro on a struct:

```ds
#[derive(HelperAttr)]
struct Struct {
    #[helper] field: ()
}
```

### Attribute macros

*Attribute macros* define new [outer attributes][attributes] which can be
attached to [items], inherent and trait
[implementations], and [trait definitions].

Attribute macros are defined by a function with the
`proc_macro_attribute` [attribute] and an empty signature:

```ds
proc_macros! {
    lang = "rust";
    path = "./my_proc_macros";

    #[proc_macro_attribute]
    fn attribute_macro();
}
```

The process referred by the `proc_macros!` section is based on the following Rust code:

```rust
use ds_proc_macro::{TokenStream, proc_macros};

proc_macros! {
    /// # Parameters
    ///
    /// - `attribute` is the delimited token following the attribute's
    /// name, not including the outer delimiters.
    /// - `item` is the rest of the item, including other attributes
    /// on the item.
    ///
    /// # Return
    ///
    /// The returned `TokenStream` replaces the item with an
    /// arbitrary number of items.
    ///
    #[proc_macro_attribute]
    fn attribute_macro(attribute: TokenStream, item: TokenStream) -> TokenStream {
        output
    }
}
```

### Declarative macro tokens and procedural macro tokens

Declarative `macro`s and procedural macros use similar, but
different definitions for tokens (or rather [`TokenTree`s].)

Token trees in `macro` (corresponding to `tt` matchers) are defined as
- Delimited groups (`(...)`, `{...}`, etc)
- All operators supported by the language, both single-character and
  multi-character ones (`+`, `+=`).
    - Note that this set doesn't include the single quote `'`.
- Literals (`"string"`, `1`, etc)
    - Note that negation (e.g. `-1`) is never a part of such literal tokens,
      but a separate operator token.
- Identifiers, including keywords (`ident`, `r#ident`, `fn`)
- Labels (`'ident`)
- Metavariable substitutions in `macro` (e.g. `$my_expr` in
  `macro mac { ($my_expr: expr) => { $my_expr } }` after the `mac`'s
  expansion, which will be considered a single token tree regardless of the
  passed expression)

Token trees in procedural macros are defined as
- Delimited groups (`(...)`, `{...}`, etc)
- All operators supported by the language, both single-character and
  multi-character ones (`+`, `+=`).
- Literals (`"string"`, `1`, etc)
    - Negation (e.g. `-1`) is supported as a part of integer
      and floating point literals.
- Identifiers, including keywords (`ident`, `r#ident`, `fn`)
- Labels

Mismatches between these two definitions are accounted for when token streams
are passed to and from procedural macros. \
Note that the conversions below may happen lazily, so they might not happen if
the tokens are not actually inspected.

When passed to a proc-macro
- All multi-character operators are broken into single characters.
- All metavariable substitutions are represented as their underlying token
  streams.
    - Such token streams may be wrapped into delimited groups ([`Group`]) with
      implicit delimiters (`Delimiter::None`) when it's necessary for
      preserving parsing priorities.
    - `tt` and `ident` substitutions are never wrapped into such groups and
      always represented as their underlying token trees.

When emitted from a proc macro
- Negative literals are converted into two tokens (the `-` and the literal)
  possibly wrapped into a delimited group ([`Group`]) with implicit delimiters
  (`Delimiter::None`) when it's necessary for preserving parsing priorities.

Note that neither declarative nor procedural macros support doc comment tokens
(e.g. `/// Doc`), so they are always converted to token streams representing
their equivalent `#[doc = r"str"]` attributes when passed to macros.

[Attribute macros]: #attribute-macros
[Derive macros]: #derive-macros
[Function-like macros]: #function-like-procedural-macros
[`Group`]: ../proc_macro/struct.Group.html
[`TokenStream`]: ../proc_macro/struct.TokenStream.html
[`TokenStream`s]: ../proc_macro/struct.TokenStream.html
[`TokenTree`s]: ../proc_macro/enum.TokenTree.html
[`derive` attribute]: attributes/derive.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`proc_macro` crate]: ../proc_macro/index.html
[attribute]: attributes.md
[attributes]: attributes.md
[block]: expressions/block-expr.md
[crate type]: linkage.md
[derive macro helper attributes]: #derive-macro-helper-attributes
[enum]: items/enumerations.md
[expressions]: expressions.md
[function]: items/functions.md
[implementations]: items/implementations.md
[inert]: attributes.md#active-and-inert-attributes
[item]: items.md
[items]: items.md
[module]: items/modules.md
[patterns]: patterns.md
[public]: visibility-and-privacy.md
[statements]: statements.md
[struct]: items/structs.md
[trait definitions]: items/traits.md
[type expressions]: types.md#type-expressions
[type]: types.md
[union]: items/unions.md
