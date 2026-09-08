# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

## 0.1.4 — 2026-09-08

- **Declares its layer**: `layer = "core"` in the manifest — the public API requires no effects, and `novo pkg publish` now checks the code against that budget.  No code changed.  The layers are described under Design in the [publishing guide](https://novo-lang.org/docs/publishing.html#design).

## 0.1.3

Documentation: the reference is generated from the code, and the
examples in it are doctests.  No code changed — every signature, and
every byte on the wire, is what 0.1.2 shipped.

- **Every `pub` item is documented under Go's rule**, the comment block
  directly above the declaration, its first sentence the summary a
  reader meets before opening anything.  `novo doc` turns that into
  [the package's page](https://novo-lang.org/packages/zigzag-nv); there
  is no hand-written API table left to go stale.
- **Five worked examples, and they run.**  A fenced `novo` block inside
  a documentation comment is compiled by `novo doc` and run by
  `novo test src/zigzag.nv`, so an example that stopped being true is a
  failing test rather than a reader's afternoon.

## 0.1.2

Developed in its own repository from this version.  `novolang/zigzag-nv` is
where the sources live, where CI runs and where releases are tagged,
and the novo-lang monorepo no longer carries a copy.  No code changed:
every signature, and every byte on the wire, is what 0.1.1 shipped.

- **`LICENSE` ships with the package.**  It is on the publish
  allow-list, so the tarball now carries the Apache-2.0 text rather
  than only naming it in the manifest.

## 0.1.1

A patch: the fold is the same fold and every signature is unchanged.
The Protocol Buffers table, both range ends and the interleaving
property all still pass, which is what says so.

- **The four bodies are written with the operators.**  `encode64` is
  `(n << 1) ^ (n >> 63)` and `decode64` is `(u >>> 1) ^ -(u & 1)`,
  where each was a nest of `bits.xor`, `bits.shl` and `bits.asr` calls
  that a reader had to unpick before checking it against the
  specification.  The 32-bit pair likewise, with its mask.  Twenty
  `bits.*` calls in all; `std.bits` is no longer imported.
- **The test module moved out of `src/`.**  A package's `src/` ships
  whole and a consumer compiles every module in it, so the suite is
  under `tests/` where it is not published.  Run it with
  `novo test tests/zigzag_tests.nv`.

## 0.1.0

First release: `encode64`, `decode64`, `encode32`, `decode32`,
`fits32`, `min32`, `max32`.

- **Branch-free and allocation-free.**  One shift, one arithmetic
  shift and one xor each way; the module is arithmetic on `Int` and
  needs no heap.
- **Exact over each width.**  The 64-bit pair covers the whole signed
  range, with the upper half of the folded values carried as the
  negative `Int` with that bit pattern; the 32-bit pair is defined over
  the signed 32-bit range and `fits32` names it.
- **The vectors are Protocol Buffers'** `sint32` / `sint64` table, so
  the suite is evidence about the transform rather than about this
  implementation.
