# zigzag-nv

ZigZag is a mapping from the signed integers onto the unsigned ones. It
places each small negative number next to a small positive one, so that
a number's magnitude rather than its sign decides how many bytes an
encoder spends on it. It is defined in the
[Protocol Buffers encoding documentation](https://protobuf.dev/programming-guides/encoding/),
which applies it before writing an `sint32` or an `sint64`. This package
brings the mapping to novo-lang, at 32 and at 64 bits.
[varint-nv](https://novo-lang.org/packages/varint-nv) is built on it.

## What ZigZag is

A varint is a variable-length encoding of an integer that spends one
byte on every seven bits of it. Small numbers are therefore short and
large ones long. Two's complement makes every negative number large:
`-1` is sixty-four set bits, and a varint writes it in ten bytes.

ZigZag renumbers the integers before they are written. The order runs
0, -1, 1, -2, 2, and on, so that the two values of one magnitude land
next to each other. `-1` becomes 1 and costs one byte. That
interleaving is the whole of the mapping.

The fold is two shifts and one exclusive or. At 64 bits it is
`(n << 1) ^ (n >> 63)`, where `>>` is the arithmetic shift of a signed
operand. The shift is 0 for a non-negative `n` and all ones for a
negative one, so the exclusive or flips the doubled value's bits
exactly when `n` is negative. The unfold is `(u >>> 1) ^ -(u & 1)`,
where `>>>` is the logical shift. There is no branch, so every input
costs the same.

The mapping is a bijection over the 64-bit range. Every one of the
2^64 signed values folds to a distinct one of the 2^64 unsigned values,
and every unsigned value unfolds to the signed value it came from. No
input is refused and no value overflows.

The extremes fold to the top of the unsigned range. 2^63 − 1, the
largest `Int`, folds to 2^64 − 2. −2^63, the most negative `Int`, folds
to 2^64 − 1, which is the largest unsigned value there is. Both are
above 2^63, so `Int` carries them as the negative numbers with those
bit patterns, −2 and −1.

| Quantity | Value |
| --- | --- |
| The fold, at 64 bits | `(n << 1) ^ (n >> 63)` |
| The unfold, at 64 bits | `(u >>> 1) ^ -(u & 1)` |
| The fold, at 32 bits | `((n << 1) ^ (n >> 31)) & 0xffffffff` |
| `0` folded | 0 |
| `-1` folded | 1 |
| `1` folded | 2 |
| `-2` folded | 3 |
| `2147483647` folded | 4294967294 |
| `-2147483648` folded | 4294967295 |
| Bottom of the signed 32-bit range | −2147483648 |
| Top of the signed 32-bit range | 2147483647 |

## Install

```
novo pkg add zigzag-nv
```

## Example

```novo
use zigzag

fn main() [io]
    // Fold a signed number onto an unsigned one. -1 folds to 1.
    println("${zigzag.encode64(0 - 1)}")

    // Unfold it back. The mapping is exact in both directions.
    println("${zigzag.decode64(1)}")
```

Build and test with `novo pkg build` and `novo test tests/zigzag_tests.nv`.

## What the package contains

| Module | Contents |
| --- | --- |
| `zigzag` | The 64-bit pair, `encode64` and `decode64`. The 32-bit pair, `encode32` and `decode32`. The three calls that name the 32-bit range, `min32`, `max32` and `fits32`. |

The API reference is on
[the package's page](https://novo-lang.org/packages/zigzag-nv). `novo
doc` generates it from these sources: every `pub` declaration with its
signature, its effect row and the comment block written above it. Each
entry carries a worked example that is compiled and run as a test.

## How to choose an entry point

**`encode64` and `decode64` are the pair for any `Int`.** The whole
64-bit range is in range, nothing is refused, and the round trip is
exact. This is the pair to reach for unless the value is known to be a
32-bit one.

**`encode32` and `decode32` are the pair for a number the format calls
32 bits.** They are defined over the signed 32-bit range, and a folded
value is always non-negative because it has room above it in an `Int`.
Inside that range they answer what the 64-bit pair answers.

**`min32`, `max32` and `fits32` name the 32-bit range.** `fits32` is
the check a caller makes once, where the number arrives, before it
reaches `encode32`.

## The rules a user needs

1. **The fold is `(n << 1) ^ (n >> 63)` and the unfold is
   `(u >>> 1) ^ -(u & 1)`.** The 32-bit pair is the same arithmetic
   with 31 in place of 63 and a mask to 32 bits. The Protocol Buffers
   encoding documentation is where those lines come from.
2. **The 64-bit pair is a bijection over the 64-bit range.** Every
   `Int` folds to a distinct 64-bit pattern and every pattern unfolds
   to the `Int` it came from. Nothing is refused, and no value
   overflows.
3. **A folded value above 2^63 arrives as a negative `Int`.** 2^63 − 1
   folds to 2^64 − 2, which is `Int`'s −2, and −2^63 folds to
   2^64 − 1, which is `Int`'s −1. Those are the same 64 bits every
   other implementation writes, and `decode64` reads them back exactly.
4. **An even folded value is a non-negative number and an odd one is
   negative.** The low bit is the sign, which is how a reader tells the
   sign of a wire value by eye.
5. **`encode32` is defined over the signed 32-bit range and nowhere
   else.** Outside it the result is masked to 32 bits, so it is a
   well-formed number that means a different value to every other
   implementation. `fits32` is the question to ask first.
6. **The two widths agree wherever both are defined.** Inside the
   signed 32-bit range `encode32` and `encode64` answer the same
   number, because they are one renumbering and one of them has more
   room above it.
7. **`decode32` masks the bits above 32 away rather than trusting
   them.** A value that arrived sign-extended from another decoder
   unfolds to the number it meant rather than to a large negative one.
8. **Nothing here allocates and no function declares an effect.** The
   module is arithmetic on `Int`, with no buffer and no state, which is
   why the layer is `core`.

## What is not included

- **Writing the number down.** ZigZag is a renumbering rather than a
  wire format, and the folded value still has to be encoded.
  [leb128-nv](https://novo-lang.org/packages/leb128-nv) writes the
  bytes.
- **An 8-bit or a 16-bit form.** No format asks for one, because a
  value that narrow is written as a fixed byte.
- **A range check inside `encode32`.** An out-of-range input folds to a
  number nothing downstream would question, so the check belongs where
  the number arrives rather than at the fold. `fits32` is that check.
- **Any input or output.** Every function here is arithmetic on a
  number the caller already holds.

## Related packages

- [leb128-nv](https://novo-lang.org/packages/leb128-nv) writes an
  unsigned integer as groups of seven bits, least significant group
  first, which is the encoding a folded value is written with. It is
  not a dependency of this package.
- [varint-nv](https://novo-lang.org/packages/varint-nv) is this fold
  and that byte loop in one call, at 32 and at 64 bits. It is what
  Protocol Buffers calls `sint32` and `sint64`, and it depends on this
  package.

A caller that writes the pairing out rather than reaching for varint-nv
writes these two functions. leb128-nv is not a dependency here, so the
block below is an illustration and is not compiled.

```novo ignore
use leb128
use zigzag

// A signed value on the wire, in as few bytes as its magnitude needs.
fn put_signed(dst: Cursor, n: Int) -> Int
    leb128.encode_into(dst, zigzag.encode64(n))

fn take_signed(src: Cursor) -> Result<Int, Leb128Error>
    Ok(zigzag.decode64(leb128.decode_from(src)!))
```

`-1` costs one byte through that pair and ten bytes without it.

## Tests

```
novo test tests/zigzag_tests.nv
```

The vectors are the ones the Protocol Buffers encoding documentation
publishes for `sint32` and `sint64`, copied value for value. Testing
against the reference's numbers rather than against this code's own
output is what makes the suite evidence that the mapping is ZigZag and
not merely self-consistent.

Beside the vectors the suite asserts both ends of each range, the round
trip over every power of two at each width, that no 32-bit fold reaches
bit 32, that `fits32` names the range the 32-bit pair is defined over,
that the two widths agree wherever both are defined, and the
interleaving property, `-k` on `2k − 1` and `k` on `2k`, that a
sign-extending fold would break. Every example in a documentation
comment is compiled by `novo doc` and run by `novo test`, so an example
that stopped being true is a failing test.

## Licence

Apache-2.0. See `LICENSE`.

<!-- docs/writing-a-readme.md is the style guide for this page. -->
