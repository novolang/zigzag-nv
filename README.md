# zigzag-nv

ZigZag folding of signed integers onto unsigned ones, at 32 and 64
bits. A varint encoder spends one byte per seven bits, so it is cheap
for small numbers — but two's complement makes every negative number
large, and `-1` costs ten bytes. ZigZag renumbers the integers as 0,
-1, 1, -2, 2, … so that a number's **magnitude**, not its sign, decides
its size. It is what Protocol Buffers applies before writing an
`sint32` or `sint64`.

```novo
use zigzag

fn main() [io]
    println("${zigzag.encode64(0 - 1)}")            // 1
    println("${zigzag.decode64(1)}")                // -1
```

```
novo pkg add zigzag-nv
```

## What it gives you

| Function | |
|---|---|
| `zigzag.encode64(n: Int) -> Int` | `n` folded onto `0 .. 2^64 - 1`, as that value's bit pattern |
| `zigzag.decode64(u: Int) -> Int` | the inverse, reading `u` as unsigned 64-bit |
| `zigzag.encode32(n: Int) -> Int` | `n` folded onto `0 .. 2^32 - 1` |
| `zigzag.decode32(u: Int) -> Int` | the inverse, reading `u` as unsigned 32-bit |
| `zigzag.fits32(n: Int) -> Bool` | whether `n` is in the range the 32-bit pair is defined over |
| `zigzag.min32() / max32() -> Int` | that range's ends |

## Folding before a varint

The whole point of the fold is what happens next, so the pairing worth
showing is with a varint encoder:

```novo
use leb128
use zigzag

// A signed value on the wire, in as few bytes as its magnitude needs.
fn put_signed(dst: Cursor, n: Int) -> Int
    leb128.encode_into(dst, zigzag.encode64(n))

fn take_signed(src: Cursor) -> Result<Int, Leb128Error>
    Ok(zigzag.decode64(leb128.decode_from(src)!))
```

`-1` costs one byte through that pair and ten bytes without it.

## The two widths

`encode64` answers the bit pattern of a `u64`, so the upper half of its
range comes back as a negative `Int`: 2^63 − 1 folds to 2^64 − 2, which
is `Int`'s `-2`. That is not a loss — it is the same 64 bits every
other implementation writes, and `decode64` maps it back exactly.

`encode32` answers a value in `0 .. 2^32 − 1`, which fits an `Int` with
room to spare and is therefore always non-negative. The pair is exact
over the signed 32-bit range and undefined outside it; `fits32` is the
check to make once, where the number arrives.

Inside the signed 32-bit range the two widths produce the same number,
because they are the same renumbering — one just has more room above
it.

## What it costs

One left shift, one arithmetic right shift and one xor each way, plus a
mask at 32 bits. No branch, so every input costs the same. No buffer,
no allocation and no state: the module is arithmetic on `Int`, which is
why it runs unchanged on a target with no heap.

## What it does not do

It does not encode anything. ZigZag is a renumbering, not a wire
format: the result still has to be written down, and `leb128-nv` is
what does that. There is no ZigZag for 8 or 16 bits here, because no
format asks for one — a value that narrow is written as a fixed byte.

## Tests

```
novo test tests/zigzag_tests.nv
```

The vectors are the ones Protocol Buffers publishes for `sint32` and
`sint64`, including both ends of each range, plus the interleaving
property (`-k` on `2k − 1`, `k` on `2k`) that a sign-extending fold
would break.
