# Changelog

All notable changes to css-select-nv are recorded here. The format is
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this
package follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
with the pre-1.0 rule that a breaking change bumps the MINOR number.

## Unreleased

### Changed

- Internal: ladders rewritten as `match` with character literals, and the string-end and word-start lookups answer an optional instead of -1. No public signature changed.

### Fixed

- An `an+b` number of more than nine digits reads as 10^9, and `cssparse.nth_matches` holds its operands to 2^61, so a series of any size answers without overflowing.

## 0.1.0 — 2026-10-10

**The implementation.** Every body the 0.0.1 interface published is
written, all five suites are green, and the four modules under `src/`
are covered by them on 886 of their 895 instrumented lines.  No
signature, type or effect row changed, so a program compiled against
0.0.1 links against this release unchanged.

- **The parse is the specification's, refusal by refusal.** Section 5's
  grammar is read here and everything outside it is a value a caller
  can print: a byte where none may stand, a bracket or a quote that is
  never closed, an operator that is not one of the seven, a combinator
  with nothing on one side.
- **A name is decoded rather than copied.** Section 10.2's escape is
  read as one to six hex digits with the whitespace after them, or as
  any byte a newline does not precede, so `.\31 23` is the class `123`
  and `.\6a` is `j`; a hex escape of zero is the replacement character.
  A lone backslash begins no identifier and one at the end of a name is
  not part of it.
- **A namespace prefix is told apart from a dash-match**, because
  `[lang|=en]` and `svg|rect` begin the same way.  A prefix of `*` is
  read and dropped, an empty one is read and dropped, and a named one
  is refused: the tree records no namespace for a node, so there is
  nothing to compare one against.
- **An attribute value is an identifier or a quoted string and nothing
  else**, which is why `[href=/a]` is a syntax error and `[href=a]` is
  not.  The Level 4 case flags are named as Level 4 rather than called
  bad syntax, because they are valid and this package does not do them.
- **Matching walks upwards from the node**, so one node is tested
  without the tree being read from the root; a walk that runs off the
  end of a document answers `NO_NODE` or false rather than reading a
  node that is not there.
- **The structural pseudo-classes count element siblings only, on both
  sides of the node**, including the four `-of-type` kinds, whose
  siblings are the ones sharing the node's tag.
- **Specificity is section 9's three counts**, compared from the left,
  with the universal selector counting nothing and a `:not()` counting
  its contents and not itself.
- **Every refusal carries an offset, a code and a caret line**, and the
  catalogue tells a valid selector this package does not implement
  apart from a typo.

The new tests live in `tests/css_select_edge_tests.nv` — the decoded
escapes, the namespace prefixes, every attribute form and its refusals,
the structural pseudo-classes and the an+b micro-syntax, the canonical
spelling of each shape, the catalogue's three answers, and the counts
the matcher answers for a non-element or for an index past the end of
the document — because the API suite is the published oracle and is not
edited.

### Changed

- **One name in `src/cssspec.nv`'s header was wrong and is corrected.**
  It said "`specificity_of` takes a `CssComplex`", and no function of
  that name has ever existed in this package — the complex-level call
  is `of_complex`, alongside `of_compound` and `of_selector`.  The
  sentence's meaning was right and only the name was wrong, so a reader
  who grepped for it found nothing.  It was already wrong in 0.0.2, so
  it is not a sentence this release made false; it is one this release
  fixes, and it is the only change to any published doc comment here.

### Known

- Nine instrumented lines are excused, in eight regions, each with a
  `// cov: skip` marker carrying the reason, and the publish prints
  them.  In `cssmatch` they are the two out-of-range answers in
  `previous_element` and `next_element`, which every caller asks about
  an index it read from the node list, and the two guards that only a
  non-element could satisfy, which `matches_simple` and
  `pseudo_matches` turn away first.  In `cssparse` they are the
  literal-backslash branch of `ident_text` and the stop-at-the-close
  branch of `string_text`, which the scan that found the range makes
  unreachable, and the two refusals that the caller's own dispatch
  guards: `type_or_universal` is only called where a name or a `*` or a
  `|` begins, and `alternative` answers at a comma or at the end and
  never between the two.

## 0.0.2 — 2026-09-28

The dependency ranges move to the dependencies' current releases.  A
pre-1.0 caret range admits only the release it names, so the old
ranges held this package on interface releases, and a program could
not take this package beside those packages' current releases.  No
signature in this package changed.

- html-nv: `^0.0.1` to `^0.1.0`.

## [0.0.1] — 2026-09-17

**The interface, published before anyone implements it.** Every public
type and function carries its full signature, its effect row and its
doc comment; every body is `todo()`; the release is recorded
`implemented = false`.

### Added

- `cssparse` — the whole grammar of Selectors Level 3, parsed into a
  tree of public values: a selector holds alternatives, an alternative
  holds compounds and combinators, and a compound holds the simple
  selectors that must match and the ones inside `:not()` that must not.
  Level 3 admits exactly one simple selector per `:not()` and forbids
  nesting, so the parsed value is not recursive. The an+b micro-syntax
  is parsed into two integers by a function of its own, because it is
  the part of CSS most often reimplemented wrongly.
- `cssmatch` — matching against html-nv's `HtmlDoc`, right to left, so
  one node can be tested without walking the tree from the root. The
  sibling positions the structural pseudo-classes are answered from are
  public, and only element siblings are counted.
- `cssspec` — the three counts of section 9, the comparison from the
  left, and a threshold check. The universal selector counts nothing
  and a `:not()` counts its contents and not itself.
- `csserror` — a catalogue of what is deliberately not here, each entry
  with one of three reasons: a later level, a state a tree does not
  have, or a pseudo-element that is not a node.
  `is_unimplemented` tells a valid selector this package does not
  support apart from a typo, because telling a user to check a selector
  that is correct is the worst message a tool can produce.

### Known

- `novo test` is red, and that is the release's expected state: every
  assertion in the API suite reaches `not implemented:
  css-select-nv.<module>.<fn>`.
- **html-nv is itself an interface at 0.0.1.** This package cannot be
  implemented before its parser and tree have bodies.
- html-nv's `htmlquery` module compiles a named subset of CSS into an
  opaque value. The two are not alternatives: that one is for a
  selector written in a caller's own source, and this one is for a
  selector that came from a user or a stylesheet, for a refusal that
  has to be explained, and for everything other than matching.
