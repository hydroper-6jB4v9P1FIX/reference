# Modules

> **<sup>Syntax:</sup>**\
> _Module_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `mod` [IDENTIFIER] `;`\
> &nbsp;&nbsp; | `mod` [IDENTIFIER] `{`\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_Item_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `}`

A module is a container for zero or more [items].

A _module item_ is a module, surrounded in braces, named, and prefixed with the
keyword `mod`. A module item introduces a new, named module into the tree of
modules making up a crate. Modules can nest arbitrarily.

An example of a module:

```ds
mod math {
    type Complex = (f64, f64);
    fn sin(f: f64) -> f64 {
        /* ... */
    }
    fn cos(f: f64) -> f64 {
        /* ... */
    }
    fn tan(f: f64) -> f64 {
        /* ... */
    }
}
```

Modules and types share the same namespace. Declaring a named type with the
same name as a module in scope is forbidden: that is, a type definition, trait,
struct, enumeration, type parameter or crate can't shadow the name of a
module in scope, or vice versa. Items brought into scope with `use` also have
this restriction.

## Module Source Filenames

A module without a body is loaded from an external file. When the module does
not have a `path` attribute, the path to the file mirrors the logical [module
path]. Ancestor module path components are directories, and the module's
contents are in a file with the name of the module plus the `.ds` extension.
For example, the following module structure can have this corresponding
filesystem structure:

Module Path               | Filesystem Path  | File Contents
------------------------- | ---------------  | -------------
`crate`                   | `lib.ds`         | `mod util;`
`crate::util`             | `util.ds`        | `mod config;`
`crate::util::config`     | `util/config.ds` |

### The `path` attribute

The directories and files used for loading external file modules can be
influenced with the `path` attribute.

For `path` attributes on modules not inside inline module blocks, the file
path is relative to the directory the source file is located. For example, the
following code snippet would use the paths shown based on where it is located:

```ds
#[path = "foo.ds"]
mod c;
```

Source File    | `c`'s File Location | `c`'s Module Path
-------------- | ------------------- | ----------------------
`src/a/b.ds`   | `src/a/foo.ds`      | `crate::a::b::c`
`src/a.ds`     | `src/a/foo.ds`      | `crate::a::c`

For `path` attributes inside inline module blocks, the relative location of
the file path depends on the kind of source file the `path` attribute is
located in. "mod-ds" source files are root modules (such as `lib.ds` or
`main.ds`). "non-mod-ds" source files are all other module files. Paths for `path` attributes inside inline module
blocks in a mod-ds file are relative to the directory of the mod-ds file
including the inline module components as directories. For non-mod-ds files,
it is the same except the path starts with a directory with the name of the
non-mod-ds module. For example, the following code snippet would use the paths
shown based on where it is located:

```ds
mod inline {
    #[path = "other.ds"]
    mod inner;
}
```

Source File    | `inner`'s File Location   | `inner`'s Module Path
-------------- | --------------------------| ----------------------------
`src/a/b.ds`   | `src/a/b/inline/other.ds` | `crate::a::b::inline::inner`
`src/a/mod.ds` | `src/a/inline/other.ds`   | `crate::a::inline::inner`

An example of combining the above rules of `path` attributes on inline modules
and nested modules within (applies to both mod-ds and non-mod-ds files):

```ds
#[path = "foo_files"]
mod foo {
    // Load the `qux` module from `foo_files/bar.ds` relative to
    // this source file's directory.
    #[path = "bar.ds"]
    mod qux;
}
```

## Attributes on Modules

Modules, like all items, accept outer attributes. They also accept inner
attributes: either after `{` for a module with a body, or at the beginning of the
source file, after the optional BOM.

The built-in attributes that have meaning on a module are [`cfg`],
[`deprecated`], [`doc`], [the lint check attributes], [`path`], and
[`no_implicit_prelude`]. Modules also accept macro attributes.

[_InnerAttribute_]: ../attributes.md
[_Item_]: ../items.md
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../dsdoc/the-doc-attribute.html
[`no_implicit_prelude`]: ../names/preludes.md#the-no_implicit_prelude-attribute
[`path`]: #the-path-attribute
[IDENTIFIER]: ../identifiers.md
[attribute]: ../attributes.md
[items]: ../items.md
[module path]: ../paths.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
