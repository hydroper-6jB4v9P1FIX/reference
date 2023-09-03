# Conditional compilation

> **<sup>Syntax</sup>**\
> _ConfigurationPredicate_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _ConfigurationOption_\
> &nbsp;&nbsp; | _ConfigurationAll_\
> &nbsp;&nbsp; | _ConfigurationAny_\
> &nbsp;&nbsp; | _ConfigurationNot_
>
> _ConfigurationOption_ :\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;(`=` ([STRING_LITERAL] | [RAW_STRING_LITERAL]))<sup>?</sup>
>
> _ConfigurationAll_\
> &nbsp;&nbsp; `all` `(` _ConfigurationPredicateList_<sup>?</sup> `)`
>
> _ConfigurationAny_\
> &nbsp;&nbsp; `any` `(` _ConfigurationPredicateList_<sup>?</sup> `)`
>
> _ConfigurationNot_\
> &nbsp;&nbsp; `not` `(` _ConfigurationPredicate_ `)`
>
> _ConfigurationPredicateList_\
> &nbsp;&nbsp; _ConfigurationPredicate_ (`,` _ConfigurationPredicate_)<sup>\*</sup> `,`<sup>?</sup>

*Conditionally compiled source code* is source code that may or may not be
considered a part of the source code depending on certain conditions. <!-- This
definition is sort of vacuous --> Source code can be conditionally compiled
using the [attributes] [`cfg`] and [`cfg_attr`] and the built-in [`cfg` macro].
These conditions are based on the target architecture of the compiled crate,
arbitrary values passed to the compiler, and a few other miscellaneous things
further described below in detail.

Each form of conditional compilation takes a _configuration predicate_ that
evaluates to true or false. The predicate is one of the following:

* A configuration option. It is true if the option is set and false if it is
  unset.
* `all()` with a comma separated list of configuration predicates. It is false
  if at least one predicate is false. If there are no predicates, it is true.
* `any()` with a comma separated list of configuration predicates. It is true
  if at least one predicate is true. If there are no predicates, it is false.
* `not()` with a configuration predicate. It is true if its predicate is false
  and false if its predicate is true.

_Configuration options_ are names and key-value pairs that are either set or
unset. Names are written as a single identifier.
Key-value pairs are written as an identifier, `=`, and then a string. For
example, `feature = "foo"` is a configuration option.

> **Note**: Whitespace around the `=` is ignored. `foo="bar"` and `foo = "bar"`
> are equivalent configuration options.

Keys are not unique in the set of key-value configuration options. For example,
both `feature = "foo"` and `feature = "qux"` can be set at the same time.

## Set Configuration Options

Which configuration options are set is determined statically during the
compilation of the crate. Certain options are _compiler-set_ based on data
about the compilation. Other options are _arbitrarily-set_, set based on input
passed to the compiler outside of the code. It is not possible to set a
configuration option from within the source code of the crate being compiled.

> **Note**: Configuration options with the key `feature` are a convention used
> by the DS package manager for specifying compile-time options and optional
> dependencies.

### `test`

Enabled when compiling the test harness. See [Testing] for more on testing support.

### `debug_assertions`

Enabled by default when compiling without optimizations.
This can be used to enable extra debugging code in development but not in
production.  For example, it controls the behavior of the standard library's
`debug_assert!` macro.

## Forms of conditional compilation

### The `cfg` attribute

> **<sup>Syntax</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg` `(` _ConfigurationPredicate_ `)`

<!-- should we say they're active attributes here? -->

The `cfg` [attribute] conditionally includes the thing it is attached to based
on a configuration predicate.

It is written as `cfg`, `(`, a configuration predicate, and finally `)`.

If the predicate is true, the thing is rewritten to not have the `cfg` attribute
on it. If the predicate is false, the thing is removed from the source code.

When a crate-level `cfg` has a false predicate, the behavior is slightly
different: any crate attributes preceding the `cfg` are kept, and any crate
attributes following the `cfg` are removed.

Some examples on functions:

```ds
// The function is only included in the build when compiling with feature `foo`
#[cfg(feature = "foo")]
fn foo_feature_only() {
  // ...
}

// This function is only included when either foo or bar is defined
#[cfg(any(foo, bar))]
fn needs_foo_or_bar() {
  // ...
}
```

The `cfg` attribute is allowed anywhere attributes are allowed.

### The `cfg_attr` attribute

> **<sup>Syntax</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg_attr` `(` _ConfigurationPredicate_ `,` _CfgAttrs_<sup>?</sup> `)`
>
> _CfgAttrs_ :\
> &nbsp;&nbsp; [_Attr_]&nbsp;(`,` [_Attr_])<sup>\*</sup> `,`<sup>?</sup>

The `cfg_attr` [attribute] conditionally includes [attributes] based on a
configuration predicate.

When the configuration predicate is true, this attribute expands out to the
attributes listed after the predicate. For example, the following module will
either be found at `foo.ds` or `qux.ds` based on the feature `foo`.

```ds
#[cfg_attr(feature = "foo", path = "foo.ds")]
#[cfg_attr(not(feature = "foo"), path = "qux.ds")]
mod q;
```

Zero, one, or more attributes may be listed. Multiple attributes will each be
expanded into separate attributes. For example:

```ds
#[cfg_attr(feature = "magic", sparkles, crackles)]
fn bewitched() {}

// When the `magic` feature flag is enabled, the above will expand to:
#[sparkles]
#[crackles]
fn bewitched() {}
```

> **Note**: The `cfg_attr` can expand to another `cfg_attr`. For example,
> `#[cfg_attr(feature = "foo", cfg_attr(feature = "qux", some_other_attribute))]`
> is valid. This example would be equivalent to
> `#[cfg_attr(all(feature = "foo", feature ="qux"), some_other_attribute)]`.

The `cfg_attr` attribute is allowed anywhere attributes are allowed.

### The `cfg` macro

The built-in `cfg` macro takes in a single configuration predicate and evaluates
to the `true` literal when the predicate is true and the `false` literal when
it is false.

For example:

```ds
let ft = if cfg!(feature = "foo") {
    "foo"
} else if cfg!(feature = "qux") {
    "qux"
} else {
    "none"
};

print!("Feature: {ft}");
```

[IDENTIFIER]: identifiers.md
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[STRING_LITERAL]: tokens.md#string-literals
[Testing]: attributes/testing.md
[_Attr_]: attributes.md
[`--cfg`]: ../dsc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[`--test`]: ../dsc/command-line-arguments.html#--test-build-a-test-harness
[`cfg`]: #the-cfg-attribute
[`cfg` macro]: #the-cfg-macro
[`cfg_attr`]: #the-cfg_attr-attribute
[`debug_assert!`]: ../std/macro.debug_assert.html
[attribute]: attributes.md
[attributes]: attributes.md
[crate type]: linkage.md
