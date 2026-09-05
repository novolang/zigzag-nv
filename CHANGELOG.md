# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

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
