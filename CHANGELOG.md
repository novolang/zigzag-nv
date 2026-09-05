# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

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
