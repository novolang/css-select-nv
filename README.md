# css-select-nv

A CSS selector is a pattern that names a set of elements in a document.
The version every tool implements is
[Selectors Level 3](https://www.w3.org/TR/selectors-3/), a W3C
Recommendation. This package parses a Level 3 selector into a value,
matches it against the tree
[html-nv](https://novo-lang.org/packages/html-nv) builds, and computes
its specificity.

**Status: NOT IMPLEMENTED — interface only.** Every function is
declared with its full signature, but every body is a `todo()` that
panics when called. The package is published so its design can be
reviewed and depended on before it is implemented. Version 0.1.0 will
be the first working release.

## What a selector is

A **simple selector** is one condition on one element: `*`, a tag name,
`#id`, `.class`, an attribute test, or a structural pseudo-class.

A **compound selector** is a run of simple selectors that all apply to
the same element, written with nothing between them: `a.row[href]`.

A **complex selector** is compound selectors joined by combinators:
`main > .row a`. There are four combinators.

| Written | Meaning |
| --- | --- |
| a blank | The right side is a descendant of the left, at any depth |
| `>` | The right side is a child of the left |
| `+` | The right side is the next element sibling of the left |
| `~` | The right side is a later element sibling of the left |

A **selector list** is complex selectors separated by commas: an
element matches the list if it matches any of them.

An **attribute selector** compares the value of an attribute. Level 3
section 6.3 defines seven forms.

| Written | True when |
| --- | --- |
| `[attr]` | The attribute is present |
| `[attr=v]` | The value is exactly `v` |
| `[attr~=v]` | The value is a whitespace-separated list holding `v` |
| `[attr\|=v]` | The value is `v`, or begins with `v` and a hyphen |
| `[attr^=v]` | The value begins with `v` |
| `[attr$=v]` | The value ends with `v` |
| `[attr*=v]` | The value holds `v` anywhere |

A **structural pseudo-class** is a question about an element's position
among its siblings: `:root`, `:empty`, `:first-child`, `:last-child`,
`:only-child`, the three `-of-type` forms, and the four `:nth-*()`
forms. The body of an `:nth-*()` is the **an+b micro-syntax** of
section 6.6.5: `odd` means `2n+1`, `even` means `2n`, `3` means the
third, `-n+3` means the first three.

`:not()` takes exactly one simple selector and may not be nested.
Anything larger inside it is Selectors Level 4.

**Specificity** is three counts — id selectors, then class, attribute
and pseudo-class selectors, then type selectors — compared from the
left. It decides which of two rules that both match an element sets a
value. Section 9.

## Install

```
novo pkg add css-select-nv
```

## Example

```novo
use cssmatch
use cssparse
use cssspec
use htmlparse
use htmltree

fn main() [io]
    let doc = htmlparse.parse("<ul class=\"rows\"><li>a</li><li class=\"on\">b</li></ul>")

    match cssparse.parse(".rows > li:nth-child(odd):not(.on)")
        Err(fault)   => println(fault.message())
        Ok(selector) =>
            // Every matching node, in document order.
            let found = cssmatch.select_all(doc, htmltree.root(doc), selector)
            println("${list.len(found)} row(s)")

            // The parsed value is public, so a tool can read the
            // selector rather than only run it.
            println("classes named: ${list.len(cssparse.class_names(selector))}")
            println("specificity: ${cssspec.specificity_text(cssspec.of_selector(selector))}")
```

Build and test with `novo pkg build` and `novo test`. Today `novo test`
fails on purpose: every test reaches a
`not implemented: css-select-nv.<module>.<fn>` panic. The tests are the
specification the implementation will have to satisfy.

## What the package contains

| Module | Contents |
| --- | --- |
| `csserror` | Why a selector would not compile, and the catalogue of what is deliberately not implemented, each entry with its reason. |
| `cssparse` | The grammar of Level 3, parsed into a tree of public values, and the an+b micro-syntax. |
| `cssspec` | Specificity: the three counts, the comparison, and a threshold check. |
| `cssmatch` | A parsed selector against html-nv's tree, and the sibling positions the structural pseudo-classes are answered from. |

## How to choose an entry point

**`cssparse.parse` takes a selector list.** It is what a stylesheet
rule and a `select_all` call both hold.

**`cssparse.parse_one` takes a single complex selector** and refuses a
comma. Use it where a list would be a mistake.

**`cssmatch.matches` tests one node.** Matching runs right to left, so
it costs that node's ancestors and not the document.

**`cssmatch.select_all` walks a subtree.** `cssmatch.select_first`
stops after a count, for a document with fifty thousand rows in it.

**`cssmatch.closest` walks upwards from a node.** It is what a scraper
does when it has found a piece of text and wants the row it sits in.

**`html-nv`'s own `htmlquery` is the other way in.** It matches a named
subset of CSS and parses nothing for `by_tag`, `by_id` and `by_class`.
Use it when the selector is a constant in your own source. Use this
package when the selector came from a user, a configuration file or a
stylesheet, when a refusal must be explained, or when anything other
than matching is wanted.

## The rules a user needs

1. **The parsed selector is public and is meant to be read.** A
   `CssSelector` holds alternatives, an alternative holds compounds and
   combinators, and a compound holds simple selectors. A linter, a
   template checker, a class-renaming tool and an editor all need that,
   and none of them can be written against an opaque value.
2. **`:not()` is two lists on a compound, not a nested selector.**
   Level 3 section 6.6.7 admits exactly one simple selector inside and
   forbids nesting, so a compound is completely described by the simple
   selectors that must match and the ones that must not. Nothing in the
   parsed value is recursive.
3. **A selector this package does not implement is refused by name,
   with a reason.** `csserror.is_unimplemented` tells that apart from a
   syntax error. Matching nothing would send a caller to look at their
   document when the problem is in their selector.
4. **Three things have no answer in a tree, and each is refused rather
   than matched as false.** Level 4 selectors are a later
   specification. `:hover`, `:checked` and their family describe the
   state of a live document. `::before` and the other pseudo-elements
   describe things a stylesheet creates and a document does not
   contain.
5. **A namespace prefix is refused.** Level 3 section 6.1 resolves
   `svg|rect` against a namespace declaration a stylesheet carries, and
   html-nv's tree records no namespace for a node.
6. **A typo is not the same refusal as an unimplemented selector.**
   `:hovr` is `CssUnknownPseudo` and `:hover` is `CssNotImplemented`.
   The first sends a user to look at their selector and the second does
   not.
7. **Positions count from one.** `:nth-child(1)` is the first element
   child. This is CSS's convention and not most languages'.
8. **Only element siblings are counted.** Text nodes and comments do
   not affect `:nth-child()` or `:first-child`. Section 6.6.5.
9. **`:empty` is about content, and a comment is not content.** An
   element holding only a comment is empty; one holding a space is not.
10. **Tag and attribute names are compared case-insensitively; id
    values, class names and attribute values are not.** That is what
    HTML specifies, and html-nv's parser already lower-cases the first
    two.
11. **A node appears once in a result** however many of a selector's
    alternatives it matched. `cssmatch.matching_alternative` says which
    one did, because the alternatives have different specificities.
12. **`cssparse.selector_text` prints canonically, not verbatim.** One
    space around each combinator, `*` written where it was implied,
    attribute values quoted. Two selectors that mean the same thing
    print the same. `CssSelector.source` is what was written.
13. **The universal selector counts nothing towards specificity, and a
    `:not()` counts its contents but not itself.** Section 9, and the
    two halves of the rule people get wrong.
14. **A selector list has no specificity of its own.**
    `cssspec.of_selector` answers the largest among its alternatives,
    which is what a caller ranking a whole rule wants.

## What is not included

- **Selectors Level 4**: `:is()`, `:where()`, `:has()`, a `:not()`
  holding more than one simple selector, `:nth-child(An+B of S)`, and
  the `[attr=v i]` case-sensitivity flags. Each is named by
  `csserror.CssUnsupported` with its reason.
- **The state pseudo-classes** and **the pseudo-elements**. See rule 4.
- **Namespaces.** See rule 5.
- **Parsing a stylesheet.** This package parses a selector. The rules,
  the declarations and the at-rules around one are a different grammar.
- **The cascade.** Specificity is one of the inputs; the origin of a
  rule, `!important` and document order are the others, and they belong
  to whatever holds the stylesheet.
- **Fetching anything.** A document's stylesheets are never loaded.
  This package performs no input or output.

## Related packages

- [html-nv](https://novo-lang.org/packages/html-nv) parses the document
  and owns the tree. Its `htmlquery` module matches a named subset of
  CSS with no parsing at all. This package depends on it.
- [xml-nv](https://novo-lang.org/packages/xml-nv) has `find` and
  `findall` for XML documents, which are a different pattern language.

## Tests

```bash
novo test tests/cssparse_tests.nv   # the grammar, and the parsed tree
novo test tests/cssmatch_tests.nv   # matching against html-nv's tree
novo test tests/cssspec_tests.nv    # the three counts and the comparison
novo test tests/csserror_tests.nv   # refusals, and what is not here
```

The normative source is Selectors Level 3, with section 6.6.5 for the
structural pseudo-classes and section 9 for specificity. The reference
implementations are the Rust crate `selectors`, which Servo uses, and
the Python package `soupsieve`.

The suite asserts that the four combinators behave as section 8
describes, that the structural pseudo-classes count only element
siblings in a document that deliberately holds a comment and a text
node among its list items, that `:empty` ignores a comment and not a
space, that a node appears once however many alternatives matched it,
that a universal selector adds nothing to specificity, and that a
`:not()` counts its contents and not itself.

The tests compile today and fail at run, each on the
`not implemented: css-select-nv.<module>.<fn>` panic that is its body.
That is the expected state of an interface release. They turn green one
at a time as bodies land.

## Implementation status

| Item | Implemented |
| --- | --- |
| `cssparse.CssSelector`, `.CssComplex`, `.CssCompound`, `.CssSimple`, `.CssCombinator`, `.CssAttrOp`, `.CssPseudoKind`, `.CssNth` | the types are declared |
| `csserror.CssFault`, `.CssUnsupported`, `cssspec.CssSpecificity` | the types are declared |
| `cssparse.parse`, `.parse_one`, `.is_valid` | no |
| `cssparse.parse_nth`, `.nth_matches` | no |
| `cssparse.selector_text`, `.simple_text`, `.combinator_text`, `.operator_text`, `.pseudo_text` | no |
| `cssparse.class_names`, `.type_names` | no |
| `cssmatch.matches`, `.matches_complex`, `.matches_compound`, `.matches_simple` | no |
| `cssmatch.select`, `.select_all`, `.select_first`, `.closest`, `.matching_alternative` | no |
| `cssmatch.child_position`, `.type_position`, `.sibling_count` | no |
| `cssspec.zero`, `.of_complex`, `.of_compound`, `.of_selector` | no |
| `cssspec.compare`, `.specificity_text`, `.is_over` | no |
| `csserror.unsupported_name`, `.unsupported_reason`, `.unsupported_code`, `.unsupported_for` | no |
| `csserror.offset`, `.code`, `.is_unimplemented`, `.caret`, `CssFault.message` | no |

## Licence

Apache-2.0. See `LICENSE`.

<!-- docs/writing-a-readme.md is the style guide for this page. -->
