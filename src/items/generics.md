# Generic parameters

> **<sup>Syntax</sup>**\
> _GenericParams_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `<` `>`\
> &nbsp;&nbsp;  | `<` (_GenericParam_ `,`)<sup>\*</sup> _GenericParam_ `,`<sup>?</sup> `>`
>
> _GenericParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> _TypeParam_
>
> _TypeParam_ :\
> &nbsp;&nbsp; [IDENTIFIER]( `:` [_TypeParamBounds_]<sup>?</sup> )<sup>?</sup> ( `=` [_Type_] )<sup>?</sup>

[Functions], [type aliases], [structs], [enumerations], [traits], and
[implementations] may be *parameterized* by types. These
parameters are listed in angle <span class="parenthetical">brackets (`<...>`)</span>,
usually immediately after the name of the item and before its definition. For
implementations, which don't have a name, they come directly after `impl`.

Some examples of items with type parameters:

```ds
fn foo<T>() {}
trait A<U> {}
struct S<T> { array: [T] }
```

Generic parameters are in scope within the item definition where they are
declared. They are not in scope for items declared within the body of a
function as described in [item declarations].

[Arrays], [tuples] and [function types] have type parameters as well, but are not
referred to with path syntax.

## Where clauses

> **<sup>Syntax</sup>**\
> _WhereClause_ :\
> &nbsp;&nbsp; `where` ( _WhereClauseItem_ `,` )<sup>\*</sup> _WhereClauseItem_ <sup>?</sup>
>
> _WhereClauseItem_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _TypeBoundWhereClauseItem_
>
> _TypeBoundWhereClauseItem_ :\
> &nbsp;&nbsp; [_Type_] `:` [_TypeParamBounds_]<sup>?</sup>

*Where clauses* provide another way to specify bounds on type parameters as well as a way to specify bounds on types that aren't type
parameters.

## Attributes

Generic type parameters allow [attributes] on them. There are no
built-in attributes that do anything in this position, although custom derive
attributes may give meaning to it.

[IDENTIFIER]: ../identifiers.md

[_OuterAttribute_]: ../attributes.md
[_Type_]: ../types.md#type-expressions
[_TypeParamBounds_]: ../trait-bounds.md

[array repeat expression]: ../expressions/array-expr.md
[arrays]: ../types/array.md
[slices]: ../types/slice.md
[associated const]: associated-items.md#associated-constants
[associated type]: associated-items.md#associated-types
[block]: ../expressions/block-expr.md
[const contexts]: ../const_eval.md#const-context
[const expression]: ../const_eval.md#constant-expressions
[const item]: constant-items.md
[enumerations]: enumerations.md
[functions]: functions.md
[function pointers]: ../types/function-pointer.md
[generic implementations]: implementations.md#generic-implementations
[implementations]: implementations.md
[item declarations]: ../statements.md#item-declarations
[item]: ../items.md
[literal]: ../expressions/literal-expr.md
[path]: ../paths.md
[path expression]: ../expressions/path-expr.md
[raw pointers]: ../types/pointer.md#raw-pointers-const-and-mut
[references]: ../types/pointer.md#shared-references-
[structs]: structs.md
[tuples]: ../types/tuple.md
[trait object]: ../types/trait-object.md
[traits]: traits.md
[type aliases]: type-aliases.md
[type]: ../types.md
[attributes]: ../attributes.md
