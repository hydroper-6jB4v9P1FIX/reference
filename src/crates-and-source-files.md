# Crates and source files

> **<sup>Syntax</sup>**\
> _Crate_ :\
> &nbsp;&nbsp; UTF8BOM<sup>?</sup>\
> &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Item_]<sup>\*</sup>

> **<sup>Lexer</sup>**\
> UTF8BOM : `\uFEFF`


> Note: Although DS, like any other language, can be implemented by an
> interpreter as well as a compiler, the only existing implementation is a
> compiler, and the language has always been designed to be compiled. For these
> reasons, this section assumes a compiler.

DS's semantics obey a *phase distinction* between compile-time and
run-time.[^phase-distinction] Semantic rules that have a *static
interpretation* govern the success or failure of compilation, while
semantic rules that have a *dynamic interpretation* govern the behavior of the
program at run-time.

The compilation model centers on artifacts called _crates_. Each compilation
processes a single crate in source form, and if successful, produces a single
crate in binary form: either an executable or some sort of
library.[^cratesourcefile]

A _crate_ is a unit of compilation and linking, as well as versioning,
distribution, and runtime loading. A crate contains a _tree_ of nested
[module] scopes. The top level of this tree is a module that is
anonymous (from the point of view of paths within the module) and any item
within a crate has a canonical [module path] denoting its location
within the crate's module tree.

The DS compiler is always invoked with a single source file as input, and
always produces a single output crate. The processing of that source file may
result in other source files being loaded as modules. Source files have the
extension `.ds`.

A DS source file describes a module, the name and location of which &mdash;
in the module tree of the current crate &mdash; are defined from outside the
source file: either by an explicit [_Module_][module] item in a referencing
source file, or by the name of the crate itself. Every source file is a
module, but not every module needs its own source file: [module
definitions][module] can be nested within one file.

Each source file contains a sequence of zero or more [_Item_] definitions, and
may optionally begin with any number of [attributes]
that apply to the containing module, most of which influence the behavior of
the compiler. The anonymous crate module can have additional attributes that
apply to the crate as a whole.

```ds
// Specify the crate name.
#![crate_name = "projx"]

// Specify the type of output artifact.
#![crate_type = "lib"]

// Turn on a warning.
// This can be done in any module, not just the anonymous crate module.
#![warn(non_camel_case_types)]
```

## Byte order mark

The optional [_UTF8 byte order mark_] (UTF8BOM production) indicates that the
file is encoded in UTF8. It can only occur at the beginning of the file and
is ignored by the compiler.

## Main Functions

A crate that contains a `main` [function] can be compiled to an executable. If a
`main` function is present, it must take no arguments, must not be generic, must not declare any
[trait bounds], must not have any [where clauses], and its return
type must be `()`.

```ds
fn main() {}
```

## The `crate_name` attribute

The *`crate_name` [attribute]* may be applied at the crate level to specify the
name of the crate with the [_MetaNameValueStr_] syntax.

```ds
#![crate_name = "mycrate"]
```

The crate name must not be empty, and must only contain [Unicode alphanumeric]
or `_` (U+005F) characters.

[^phase-distinction]: This distinction would also exist in an interpreter.
    Static checks like syntactic analysis, type checking, and lints should
    happen before the program is executed regardless of when it is executed.

[^cratesourcefile]: A crate is somewhat analogous to an *assembly* in the
    ECMA-335 CLI model, a *library* in the SML/NJ Compilation Manager, a *unit*
    in the Owens and Flatt module system, or a *configuration* in Mesa.

[Unicode alphanumeric]: ../std/primitive.char.html#method.is_alphanumeric
[`!`]: types/never.md
[_InnerAttribute_]: attributes.md
[_Item_]: items.md
[_MetaNameValueStr_]: attributes.md#meta-item-attribute-syntax
[_shebang_]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[_utf8 byte order mark_]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[attribute]: attributes.md
[attributes]: attributes.md
[comments]: comments.md
[function]: items/functions.md
[module]: items/modules.md
[module path]: paths.md
[trait bounds]: trait-bounds.md
[where clauses]: items/generics.md#where-clauses
[whitespace]: whitespace.md
