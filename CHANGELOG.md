# Changelog

All notable changes to css-select-nv are recorded here. The format is
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this
package follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
with the pre-1.0 rule that a breaking change bumps the MINOR number.

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
