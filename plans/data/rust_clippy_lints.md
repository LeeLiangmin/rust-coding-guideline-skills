# Rust Clippy Lints 完整规则参考

> 来源：https://rust-lang.github.io/rust-clippy/master/
> 抓取时间：2026-03-20
> 总计：823 条 lint 规则

---

## 概览

### 按分组统计

| 分组 | 说明 | 数量 |
|------|------|------|
| correctness | 正确性 - 完全错误或非常无用的代码，默认 deny | 68 |
| suspicious | 可疑 - 很可能错误或无用的代码，默认 warn | 82 |
| style | 风格 - 应该以更惯用方式编写的代码，默认 warn | 157 |
| complexity | 复杂度 - 做简单事情的复杂代码，默认 warn | 137 |
| perf | 性能 - 可以用更快方式编写的代码，默认 warn | 36 |
| pedantic | 严格 - 非常严格的 lint，默认 allow | 140 |
| restriction | 限制 - 不一定是坏的但可能需要限制的代码，默认 allow | 130 |
| nursery | 实验性 - 仍在开发中的新 lint，默认 allow | 52 |
| cargo | Cargo - Cargo manifest 相关的 lint，默认 allow | 5 |

---

## Correctness (68)

*正确性 - 完全错误或非常无用的代码，默认 deny*

### `absurd_extreme_comparisons`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for comparisons where one side of the relation is
either the minimum or maximum value for its type and warns if it involves a
case that is always true or always false. Only integer and boolean types are
checked.

#### Why is this bad?

An expression like `min <= x` may misleadingly imply
that it is possible for `x` to be less than the minimum. Expressions like
`max < x` are probably mistakes.

#### Known problems

For `usize` the size of the current compile target will
be assumed (e.g., 64 bits on 64 bit systems). This means code that uses such
a comparison to detect target pointer width will trigger this lint. One can
use `mem::sizeof` and compare its value or conditional compilation
attributes
like `#[cfg(target_pointer_width = "64")] ..` instead.

#### Example

```rust
let vec: Vec<isize> = Vec::new();
if vec.len() <= 0 {}
if 100 > i32::MAX {}
```

---

### `almost_swapped`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `foo = bar; bar = foo` sequences.

#### Why is this bad?

This looks like a failed attempt to swap.

#### Example

```rust
a = b;
b = a;
```

If swapping is intended, use `swap()` instead:

```rust
std::mem::swap(&mut a, &mut b);
```

---

### `approx_constant`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for floating point literals that approximate
constants which are defined in
[std::f32::consts](https://doc.rust-lang.org/stable/std/f32/consts/#constants)
or
[std::f64::consts](https://doc.rust-lang.org/stable/std/f64/consts/#constants),
respectively, suggesting to use the predefined constant.

#### Why is this bad?

Usually, the definition in the standard library is more
precise than what people come up with. If you find that your definition is
actually more precise, please [file a Rust
issue](https://github.com/rust-lang/rust/issues).

#### Example

```rust
let x = 3.14;
let y = 1_f64 / x;
```

Use instead:

```rust
let x = std::f32::consts::PI;
let y = std::f64::consts::FRAC_1_PI;
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `async_yields_async`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.48.0 |

#### What it does

Checks for async blocks that yield values of types
that can themselves be awaited.

#### Why is this bad?

An await is likely missing.

#### Example

```rust
async fn foo() {}

fn bar() {
  let x = async {
    foo()
  };
}
```

Use instead:

```rust
async fn foo() {}

fn bar() {
  let x = async {
    foo().await
  };
}
```

---

### `bad_bit_mask`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for incompatible bit masks in comparisons.

The formula for detecting if an expression of the type `_ <bit_op> m <cmp_op> c` (where `<bit_op>` is one of {`&`, `|`} and `<cmp_op>` is one of
{`!=`, `>=`, `>`, `!=`, `>=`, `>`}) can be determined from the following
table:

| Comparison | Bit Op | Example | is always | Formula |
| --- | --- | --- | --- | --- |
| == or != | & | x & 2 == 3 | false | c & m != c |
| <  or >= | & | x & 2 < 3 | true | m < c |
| >  or <= | & | x & 1 > 1 | false | m <= c |
| == or != | \| | x \| 1 == 0 | false | c \| m != c |
| <  or >= | \| | x \| 1 < 1 | false | m >= c |
| <= or > | \| | x \| 1 > 0 | true | m > c |

#### Why is this bad?

If the bits that the comparison cares about are always
set to zero or one by the bit mask, the comparison is constant `true` or
`false` (depending on mask, compared value, and operators).

So the code is actively misleading, and the only reason someone would write
this intentionally is to win an underhanded Rust contest or create a
test-case for this lint.

#### Example

```rust
if (x & 1 == 2) { }
```

---

### `cast_slice_different_sizes`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | HasPlaceholders |
| 引入版本 | 1.61.0 |

#### What it does

Checks for `as` casts between raw pointers to slices with differently sized elements.

#### Why is this bad?

The produced raw pointer to a slice does not update its length metadata. The produced
pointer will point to a different number of bytes than the original pointer because the
length metadata of a raw slice pointer is in elements rather than bytes.
Producing a slice reference from the raw pointer will either create a slice with
less data (which can be surprising) or create a slice with more data and cause Undefined Behavior.

#### Example

// Missing data

```rust
let a = [1_i32, 2, 3, 4];
let p = &a as *const [i32] as *const [u8];
unsafe {
    println!("{:?}", &*p);
}
```

// Undefined Behavior (note: also potential alignment issues)

```rust
let a = [1_u8, 2, 3, 4];
let p = &a as *const [u8] as *const [u32];
unsafe {
    println!("{:?}", &*p);
}
```

Instead use `ptr::slice_from_raw_parts` to construct a slice from a data pointer and the correct length

```rust
let a = [1_i32, 2, 3, 4];
let old_ptr = &a as *const [i32];
// The data pointer is cast to a pointer to the target `u8` not `[u8]`
// The length comes from the known length of 4 i32s times the 4 bytes per i32
let new_ptr = core::ptr::slice_from_raw_parts(old_ptr as *const u8, 16);
unsafe {
    println!("{:?}", &*new_ptr);
}
```

---

### `char_indices_as_byte_indices`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.88.0 |

#### What it does

Checks for usage of a character position yielded by `.chars().enumerate()` in a context where a **byte index** is expected,
such as an argument to a specific `str` method or indexing into a `str` or `String`.

#### Why is this bad?

A character (more specifically, a Unicode scalar value) that is yielded by `str::chars` can take up multiple bytes,
so a character position does not necessarily have the same byte index at which the character is stored.
Thus, using the character position where a byte index is expected can unexpectedly return wrong values
or panic when the string consists of multibyte characters.

For example, the character `a` in `äa` is stored at byte index 2 but has the character position 1.
Using the character position 1 to index into the string will lead to a panic as it is in the middle of the first character.

Instead of `.chars().enumerate()`, the correct iterator to use is `.char_indices()`, which yields byte indices.

This pattern is technically fine if the strings are known to only use the ASCII subset,
though in those cases it would be better to use `bytes()` directly to make the intent clearer,
but there is also no downside to just using `.char_indices()` directly and supporting non-ASCII strings.

You may also want to read the [chapter on strings in the Rust Book](https://doc.rust-lang.org/book/ch08-02-strings.html)
which goes into this in more detail.

#### Example

```rust
for (idx, c) in s.chars().enumerate() {
    let _ = s[idx..]; // ⚠️ Panics for strings consisting of multibyte characters
}
```

Use instead:

```rust
for (idx, c) in s.char_indices() {
    let _ = s[idx..];
}
```

---

### `deprecated_semver`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `#[deprecated]` annotations with a `since`
field that is not a valid semantic version. Also allows “TBD” to signal
future deprecation.

#### Why is this bad?

For checking the version of the deprecation, it must be
a valid semver. Failing that, the contained information is useless.

#### Example

```rust
#[deprecated(since = "forever")]
fn something_else() { /* ... */ }
```

---

### `derive_ord_xor_partial_ord`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.47.0 |

#### What it does

Lints against manual `PartialOrd` and `Ord` implementations for types with a derived `Ord`
or `PartialOrd` implementation.

#### Why is this bad?

The implementation of these traits must agree (for
example for use with `sort`) so it’s probably a bad idea to use a
default-generated `Ord` implementation with an explicitly defined
`PartialOrd`. In particular, the following must hold for any type
implementing `Ord`:

```text
k1.cmp(&k2) == k1.partial_cmp(&k2).unwrap()
```

#### Example

```rust
#[derive(Ord, PartialEq, Eq)]
struct Foo;

impl PartialOrd for Foo {
    ...
}
```

Use instead:

```rust
#[derive(PartialEq, Eq)]
struct Foo;

impl PartialOrd for Foo {
    fn partial_cmp(&self, other: &Foo) -> Option<Ordering> {
       Some(self.cmp(other))
    }
}

impl Ord for Foo {
    ...
}
```

or, if you don’t need a custom ordering:

```rust
#[derive(Ord, PartialOrd, PartialEq, Eq)]
struct Foo;
```

---

### `derived_hash_with_manual_eq`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Lints against manual `PartialEq` implementations for types with a derived `Hash`
implementation.

#### Why is this bad?

The implementation of these traits must agree (for
example for use with `HashMap`) so it’s probably a bad idea to use a
default-generated `Hash` implementation with an explicitly defined
`PartialEq`. In particular, the following must hold for any type:

```text
k1 == k2 ⇒ hash(k1) == hash(k2)
```

#### Example

```rust
#[derive(Hash)]
struct Foo;

impl PartialEq for Foo {
    ...
}
```

#### Past names

- derive_hash_xor_eq

---

### `eager_transmute`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.77.0 |

#### What it does

Checks for integer validity checks, followed by a transmute that is (incorrectly) evaluated
eagerly (e.g. using `bool::then_some`).

#### Why is this bad?

Eager evaluation means that the `transmute` call is executed regardless of whether the condition is true or false.
This can introduce unsoundness and other subtle bugs.

#### Example

Consider the following function which is meant to convert an unsigned integer to its enum equivalent via transmute.

```rust
#[repr(u8)]
enum Opcode {
    Add = 0,
    Sub = 1,
    Mul = 2,
    Div = 3
}

fn int_to_opcode(op: u8) -> Option<Opcode> {
    (op < 4).then_some(unsafe { std::mem::transmute(op) })
}
```

This may appear fine at first given that it checks that the `u8` is within the validity range of the enum,
*however* the transmute is evaluated eagerly, meaning that it executes even if `op >= 4`!

This makes the function unsound, because it is possible for the caller to cause undefined behavior
(creating an enum with an invalid bitpattern) entirely in safe code only by passing an incorrect value,
which is normally only a bug that is possible in unsafe code.

One possible way in which this can go wrong practically is that the compiler sees it as:

```rust
let temp: Foo = unsafe { std::mem::transmute(op) };
(0 < 4).then_some(temp)
```

and optimizes away the `(0 < 4)` check based on the assumption that since a `Foo` was created from `op` with the validity range `0..3`,
it is **impossible** for this condition to be false.

In short, it is possible for this function to be optimized in a way that makes it [never return None](https://godbolt.org/z/ocrcenevq),
even if passed the value `4`.

This can be avoided by instead using lazy evaluation. For the example above, this should be written:

```rust
fn int_to_opcode(op: u8) -> Option<Opcode> {
    (op < 4).then(|| unsafe { std::mem::transmute(op) })
             ^^^^ ^^ `bool::then` only executes the closure if the condition is true!
}
```

---

### `enum_clike_unportable_variant`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for C-like enumerations that are
`repr(isize/usize)` and have values that don’t fit into an `i32`.

#### Why is this bad?

This will truncate the variant value on 32 bit
architectures, but works fine on 64 bit.

#### Example

```rust
#[repr(usize)]
enum NonPortable {
    X = 0x1_0000_0000,
    Y = 0,
}
```

---

### `eq_op`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for equal operands to comparison, logical and
bitwise, difference and division binary operators (`==`, `>`, etc., `&&`,
`||`, `&`, `|`, `^`, `-` and `/`).

#### Why is this bad?

This is usually just a typo or a copy and paste error.

#### Known problems

False negatives: We had some false positives regarding
calls (notably [racer](https://github.com/phildawes/racer) had one instance
of `x.pop() && x.pop()`), so we removed matching any function or method
calls. We may introduce a list of known pure functions in the future.

#### Example

```rust
if x + 1 == x + 1 {}

// or

assert_eq!(a, a);
```

---

### `erasing_op`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for erasing operations, e.g., `x * 0`.

#### Why is this bad?

The whole expression can be replaced by zero.
This is most likely not the intended outcome and should probably be
corrected

#### Example

```rust
let x = 1;
0 / x;
0 * x;
x & 0;
```

---

### `if_let_mutex`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.45.0 |

#### What it does

Checks for `Mutex::lock` calls in `if let` expression
with lock calls in any of the else blocks.

#### Disabled starting in Edition 2024

This lint is effectively disabled starting in
Edition 2024 as `if let ... else` scoping was reworked
such that this is no longer an issue. See
[Proposal: stabilize if_let_rescope for Edition 2024](https://github.com/rust-lang/rust/issues/131154)

#### Why is this bad?

The Mutex lock remains held for the whole
`if let ... else` block and deadlocks.

#### Example

```rust
if let Ok(thing) = mutex.lock() {
    do_thing();
} else {
    mutex.lock();
}
```

Should be written

```rust
let locked = mutex.lock();
if let Ok(thing) = locked {
    do_thing(thing);
} else {
    use_locked(locked);
}
```

---

### `ifs_same_cond`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for consecutive `if`s with the same condition.

#### Why is this bad?

This is probably a copy & paste error.

#### Example

```rust
if a == b {
    …
} else if a == b {
    …
}
```

Note that this lint ignores all conditions with a function call as it could
have side effects:

```rust
if foo() {
    …
} else if foo() { // not linted
    …
}
```

#### Configuration

- `ignore-interior-mutability`:  A list of paths to types that should be treated as if they do not contain interior mutability

(default: `["bytes::Bytes"]`)

---

### `impl_hash_borrow_with_str_and_bytes`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.76.0 |

#### What it does

This lint is concerned with the semantics of `Borrow` and `Hash` for a
type that implements all three of `Hash`, `Borrow<str>` and `Borrow<[u8]>`
as it is impossible to satisfy the semantics of Borrow and `Hash` for
both `Borrow<str>` and `Borrow<[u8]>`.

#### Why is this bad?

When providing implementations for `Borrow<T>`, one should consider whether the different
implementations should act as facets or representations of the underlying type. Generic code
typically uses `Borrow<T>` when it relies on the identical behavior of these additional trait
implementations. These traits will likely appear as additional trait bounds.

In particular `Eq`, `Ord` and `Hash` must be equivalent for borrowed and owned values:
`x.borrow() == y.borrow()` should give the same result as `x == y`.
It follows then that the following equivalence must hold:
`hash(x) == hash((x as Borrow<[u8]>).borrow()) == hash((x as Borrow<str>).borrow())`

Unfortunately it doesn’t hold as `hash("abc") != hash("abc".as_bytes())`.
This happens because the `Hash` impl for str passes an additional `0xFF` byte to
the hasher to avoid collisions. For example, given the tuples `("a", "bc")`, and `("ab", "c")`,
the two tuples would have the same hash value if the `0xFF` byte was not added.

#### Example

```rust
use std::borrow::Borrow;
use std::hash::{Hash, Hasher};

struct ExampleType {
    data: String
}

impl Hash for ExampleType {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.data.hash(state);
    }
}

impl Borrow<str> for ExampleType {
    fn borrow(&self) -> &str {
        &self.data
    }
}

impl Borrow<[u8]> for ExampleType {
    fn borrow(&self) -> &[u8] {
        self.data.as_bytes()
    }
}
```

As a consequence, hashing a `&ExampleType` and hashing the result of the two
borrows will result in different values.

---

### `impossible_comparisons`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Checks for double comparisons that can never succeed

#### Why is this bad?

The whole expression can be replaced by `false`,
which is probably not the programmer’s intention

#### Example

```rust
if status_code <= 400 && status_code > 500 {}
```

---

### `ineffective_bit_mask`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for bit masks in comparisons which can be removed
without changing the outcome. The basic structure can be seen in the
following table:

| Comparison | Bit Op | Example | equals |
| --- | --- | --- | --- |
| > / <= | \| / ^ | x \| 2 > 3 | x > 3 |
| < / >= | \| / ^ | x ^ 1 < 4 | x < 4 |

#### Why is this bad?

Not equally evil as [bad_bit_mask](#bad_bit_mask),
but still a bit misleading, because the bit mask is ineffective.

#### Known problems

False negatives: This lint will only match instances
where we have figured out the math (which is for a power-of-two compared
value). This means things like `x | 1 >= 7` (which would be better written
as `x >= 6`) will not be reported (but bit masks like this are fairly
uncommon).

#### Example

```rust
if (x | 1 > 3) {  }
```

Use instead:

```rust
if (x >= 2) {  }
```

---

### `infinite_iter`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for iteration that is guaranteed to be infinite.

#### Why is this bad?

While there may be places where this is acceptable
(e.g., in event streams), in most cases this is simply an error.

#### Example

```rust
use std::iter;

iter::repeat(1_u8).collect::<Vec<_>>();
```

---

### `inherent_to_string_shadow_display`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.38.0 |

#### What it does

Checks for the definition of inherent methods with a signature of `to_string(&self) -> String` and if the type implementing this method also implements the `Display` trait.

#### Why is this bad?

This method is also implicitly defined if a type implements the `Display` trait. The less versatile inherent method will then shadow the implementation introduced by `Display`.

#### Example

```rust
use std::fmt;

pub struct A;

impl A {
    pub fn to_string(&self) -> String {
        "I am A".to_string()
    }
}

impl fmt::Display for A {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "I am A, too")
    }
}
```

Use instead:

```rust
use std::fmt;

pub struct A;

impl fmt::Display for A {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "I am A")
    }
}
```

---

### `inline_fn_without_body`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `#[inline]` on trait methods without bodies

#### Why is this bad?

Only implementations of trait methods may be inlined.
The inline attribute is ignored for trait methods without bodies.

#### Example

```rust
trait Animal {
    #[inline]
    fn name(&self) -> &'static str;
}
```

---

### `invalid_regex`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks [regex](https://crates.io/crates/regex) creation
(with `Regex::new`, `RegexBuilder::new`, or `RegexSet::new`) for correct
regex syntax.

#### Why is this bad?

This will lead to a runtime panic.

#### Example

```rust
Regex::new("(")
```

Use instead:

```rust
Regex::new("\(")
```

---

### `inverted_saturating_sub`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.83.0 |

#### What it does

Checks for comparisons between integers, followed by subtracting the greater value from the
lower one.

#### Why is this bad?

This could result in an underflow and is most likely not what the user wants. If this was
intended to be a saturated subtraction, consider using the `saturating_sub` method directly.

#### Example

```rust
let a = 12u32;
let b = 13u32;

let result = if a > b { b - a } else { 0 };
```

Use instead:

```rust
let a = 12u32;
let b = 13u32;

let result = a.saturating_sub(b);
```

---

### `invisible_characters`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Checks for invisible Unicode characters in the code.

#### Why is this bad?

Having an invisible character in the code makes for all
sorts of April fools, but otherwise is very much frowned upon.

#### Example

You don’t see it, but there may be a zero-width space or soft hyphen
some­where in this text.

#### Past names

- zero_width_space

---

### `iter_next_loop`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for loops on `x.next()`.

#### Why is this bad?

`next()` returns either `Some(value)` if there was a
value, or `None` otherwise. The insidious thing is that `Option<_>`
implements `IntoIterator`, so that possibly one value will be iterated,
leading to some hard to find bugs. No one will want to write such code
[except to win an Underhanded Rust
Contest](https://www.reddit.com/r/rust/comments/3hb0wm/underhanded_rust_contest/cu5yuhr).

#### Example

```rust
for x in y.next() {
    ..
}
```

---

### `iter_skip_zero`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.73.0 |

#### What it does

Checks for usage of `.skip(0)` on iterators.

#### Why is this bad?

This was likely intended to be `.skip(1)` to skip the first element, as `.skip(0)` does
nothing. If not, the call should be removed.

#### Example

```rust
let v = vec![1, 2, 3];
let x = v.iter().skip(0).collect::<Vec<_>>();
let y = v.iter().collect::<Vec<_>>();
assert_eq!(x, y);
```

---

### `iterator_step_by_zero`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for calling `.step_by(0)` on iterators which panics.

#### Why is this bad?

This very much looks like an oversight. Use `panic!()` instead if you
actually intend to panic.

#### Example

```rust
for x in (0..100).step_by(0) {
    //..
}
```

---

### `let_underscore_lock`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.43.0 |

#### What it does

Checks for `let _ = sync_lock`. This supports `mutex` and `rwlock` in
`parking_lot`. For `std` locks see the `rustc` lint
[let_underscore_lock](https://doc.rust-lang.org/nightly/rustc/lints/listing/deny-by-default.html#let-underscore-lock)

#### Why is this bad?

This statement immediately drops the lock instead of
extending its lifetime to the end of the scope, which is often not intended.
To extend lock lifetime to the end of the scope, use an underscore-prefixed
name instead (i.e. _lock). If you want to explicitly drop the lock,
`std::mem::drop` conveys your intention better and is less error-prone.

#### Example

```rust
let _ = mutex.lock();
```

Use instead:

```rust
let _lock = mutex.lock();
```

---

### `lint_groups_priority`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.78.0 |

#### What it does

Checks for lint groups with the same priority as lints in the `Cargo.toml`
[[lints] table](https://doc.rust-lang.org/cargo/reference/manifest.html#the-lints-section).

This lint will be removed once [cargo#12918](https://github.com/rust-lang/cargo/issues/12918)
is resolved.

#### Why is this bad?

The order of lints in the `[lints]` is ignored, to have a lint override a group the
`priority` field needs to be used, otherwise the sort order is undefined.

#### Known problems

Does not check lints inherited using `lints.workspace = true`

#### Example

```toml
[lints.clippy]
pedantic = "warn"
similar_names = "allow"
```

Use instead:

```toml
[lints.clippy]
pedantic = { level = "warn", priority = -1 }
similar_names = "allow"
```

---

### `match_str_case_mismatch`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MachineApplicable |
| 引入版本 | 1.58.0 |

#### What it does

Checks for `match` expressions modifying the case of a string with non-compliant arms

#### Why is this bad?

The arm is unreachable, which is likely a mistake

#### Example

```rust
match &*text.to_ascii_lowercase() {
    "foo" => {},
    "Bar" => {},
    _ => {},
}
```

Use instead:

```rust
match &*text.to_ascii_lowercase() {
    "foo" => {},
    "bar" => {},
    _ => {},
}
```

---

### `mem_replace_with_uninit`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MachineApplicable |
| 引入版本 | 1.39.0 |

#### What it does

Checks for `mem::replace(&mut _, mem::uninitialized())`
and `mem::replace(&mut _, mem::zeroed())`.

#### Why is this bad?

This will lead to undefined behavior even if the
value is overwritten later, because the uninitialized value may be
observed in the case of a panic.

#### Example

```rust
use std::mem;

#[allow(deprecated, invalid_value)]
fn myfunc (v: &mut Vec<i32>) {
    let taken_v = unsafe { mem::replace(v, mem::uninitialized()) };
    let new_v = may_panic(taken_v); // undefined behavior on panic
    mem::forget(mem::replace(v, new_v));
}
```

The [take_mut](https://docs.rs/take_mut) crate offers a sound solution,
at the cost of either lazily creating a replacement value or aborting
on panic, to ensure that the uninitialized value cannot be observed.

---

### `min_max`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for expressions where `std::cmp::min` and `max` are
used to clamp values, but switched so that the result is constant.

#### Why is this bad?

This is in all probability not the intended outcome. At
the least it hurts readability of the code.

#### Example

```rust
min(0, max(100, x))

// or

x.max(100).min(0)
```

It will always be equal to `0`. Probably the author meant to clamp the value
between 0 and 100, but has erroneously swapped `min` and `max`.

---

### `mistyped_literal_suffixes`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.30.0 |

#### What it does

Warns for mistyped suffix in literals

#### Why is this bad?

This is most probably a typo

#### Known problems

- Does not match on integers too large to fit in the corresponding unsigned type
- Does not match on `_127` since that is a valid grouping for decimal and octal numbers

#### Example

```rust
`2_32` => `2_i32`
`250_8 => `250_u8`
```

---

### `modulo_one`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for getting the remainder of integer division by one or minus
one.

#### Why is this bad?

The result for a divisor of one can only ever be zero; for
minus one it can cause panic/overflow (if the left operand is the minimal value of
the respective integer type) or results in zero. No one will write such code
deliberately, unless trying to win an Underhanded Rust Contest. Even for that
contest, it’s probably a bad idea. Use something more underhanded.

#### Example

```rust
let a = x % 1;
let a = x % -1;
```

---

### `mut_from_ref`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint checks for functions that take immutable references and return
mutable ones. This will not trigger if no unsafe code exists as there
are multiple safe functions which will do this transformation

To be on the conservative side, if there’s at least one mutable
reference with the output lifetime, this lint will not trigger.

#### Why is this bad?

Creating a mutable reference which can be repeatably derived from an
immutable reference is unsound as it allows creating multiple live
mutable references to the same object.

This [error](https://github.com/rust-lang/rust/issues/39465) actually
lead to an interim Rust release 1.15.1.

#### Known problems

This pattern is used by memory allocators to allow allocating multiple
objects while returning mutable references to each one. So long as
different mutable references are returned each time such a function may
be safe.

#### Example

```rust
fn foo(&Foo) -> &mut Bar { .. }
```

---

### `never_loop`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for loops that will always `break`, `return` or
`continue` an outer loop.

#### Why is this bad?

This loop never loops, all it does is obfuscating the
code.

#### Example

```rust
loop {
    ..;
    break;
}
```

---

### `non_octal_unix_permissions`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MachineApplicable |
| 引入版本 | 1.53.0 |

#### What it does

Checks for non-octal values used to set Unix file permissions.

#### Why is this bad?

They will be converted into octal, creating potentially
unintended file permissions.

#### Example

```rust
use std::fs::OpenOptions;
use std::os::unix::fs::OpenOptionsExt;

let mut options = OpenOptions::new();
options.mode(644);
```

Use instead:

```rust
use std::fs::OpenOptions;
use std::os::unix::fs::OpenOptionsExt;

let mut options = OpenOptions::new();
options.mode(0o644);
```

---

### `nonsensical_open_options`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for duplicate open options as well as combinations
that make no sense.

#### Why is this bad?

In the best case, the code will be harder to read than
necessary. I don’t know the worst case.

#### Example

```rust
use std::fs::OpenOptions;

OpenOptions::new().read(true).truncate(true);
```

---

### `not_unsafe_ptr_arg_deref`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for public functions that dereference raw pointer
arguments but are not marked `unsafe`.

#### Why is this bad?

The function should almost definitely be marked `unsafe`, since for an
arbitrary raw pointer, there is no way of telling for sure if it is valid.

In general, this lint should **never be disabled** unless it is definitely a
false positive (please submit an issue if so) since it breaks Rust’s
soundness guarantees, directly exposing API users to potentially dangerous
program behavior. This is also true for internal APIs, as it is easy to leak
unsoundness.

#### Context

In Rust, an `unsafe {...}` block is used to indicate that the code in that
section has been verified in some way that the compiler can not. For a
function that accepts a raw pointer then accesses the pointer’s data, this is
generally impossible as the incoming pointer could point anywhere, valid or
not. So, the signature should be marked `unsafe fn`: this indicates that the
function’s caller must provide some verification that the arguments it sends
are valid (and then call the function within an `unsafe` block).

#### Known problems

- It does not check functions recursively so if the pointer is passed to a
private non-`unsafe` function which does the dereferencing, the lint won’t
trigger (false negative).
- It only checks for arguments whose type are raw pointers, not raw pointers
got from an argument in some other way (`fn foo(bar: &[*const u8])` or
`some_argument.get_raw_ptr()`) (false negative).

#### Example

```rust
pub fn foo(x: *const u8) {
    println!("{}", unsafe { *x });
}

// this call "looks" safe but will segfault or worse!
// foo(invalid_ptr);
```

Use instead:

```rust
pub unsafe fn foo(x: *const u8) {
    println!("{}", unsafe { *x });
}

// this would cause a compiler error for calling without `unsafe`
// foo(invalid_ptr);

// sound call if the caller knows the pointer is valid
unsafe { foo(valid_ptr); }
```

---

### `option_env_unwrap`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.43.0 |

#### What it does

Checks for usage of `option_env!(...).unwrap()` and
suggests usage of the `env!` macro.

#### Why is this bad?

Unwrapping the result of `option_env!` will panic
at run-time if the environment variable doesn’t exist, whereas `env!`
catches it at compile-time.

#### Example

```rust
let _ = option_env!("HOME").unwrap();
```

Is better expressed as:

```rust
let _ = env!("HOME");
```

---

### `out_of_bounds_indexing`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for out of bounds array indexing with a constant
index.

#### Why is this bad?

This will always panic at runtime.

#### Example

```rust
let x = [1, 2, 3, 4];

x[9];
&x[2..9];
```

Use instead:

```rust
// Index within bounds

x[0];
x[3];
```

---

### `overly_complex_bool_expr`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for boolean expressions that contain terminals that
can be eliminated.

#### Why is this bad?

This is most likely a logic bug.

#### Known problems

Ignores short circuiting behavior.

#### Example

```rust
// The `b` is unnecessary, the expression is equivalent to `if a`.
if a && b || a { ... }
```

Use instead:

```rust
if a {}
```

#### Past names

- logic_bug

---

### `panicking_overflow_checks`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Detects C-style underflow/overflow checks.

#### Why is this bad?

These checks will, by default, panic in debug builds rather than check
whether the operation caused an overflow.

#### Example

```rust
if a + b < a {
    // handle overflow
}
```

Use instead:

```rust
if a.checked_add(b).is_none() {
    // handle overflow
}
```

Or:

```rust
if a.overflowing_add(b).1 {
    // handle overflow
}
```

#### Past names

- overflow_check_conditional

---

### `panicking_unwrap`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for calls of `unwrap[_err]()` that will always fail.

#### Why is this bad?

If panicking is desired, an explicit `panic!()` should be used.

#### Known problems

This lint only checks `if` conditions not assignments.
So something like `let x: Option<()> = None; x.unwrap();` will not be recognized.

#### Example

```rust
if option.is_none() {
    do_something_with(option.unwrap())
}
```

This code will always panic. The if condition should probably be inverted.

---

### `possible_missing_comma`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for possible missing comma in an array. It lints if
an array element is a binary operator expression and it lies on two lines.

#### Why is this bad?

This could lead to unexpected results.

#### Example

```rust
let a = &[
    -1, -2, -3 // <= no comma here
    -4, -5, -6
];
```

---

### `read_line_without_trim`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MachineApplicable |
| 引入版本 | 1.73.0 |

#### What it does

Looks for calls to [`Stdin::read_line`] to read a line from the standard input
into a string, then later attempting to use that string for an operation that will never
work for strings with a trailing newline character in it (e.g. parsing into a `i32`).

#### Why is this bad?

The operation will always fail at runtime no matter what the user enters, thus
making it a useless operation.

#### Example

```rust
let mut input = String::new();
std::io::stdin().read_line(&mut input).expect("Failed to read a line");
let num: i32 = input.parse().expect("Not a number!");
assert_eq!(num, 42); // we never even get here!
```

Use instead:

```rust
let mut input = String::new();
std::io::stdin().read_line(&mut input).expect("Failed to read a line");
let num: i32 = input.trim_end().parse().expect("Not a number!");
//                  ^^^^^^^^^^^ remove the trailing newline
assert_eq!(num, 42);
```

---

### `recursive_format_impl`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.48.0 |

#### What it does

Checks for format trait implementations (e.g. `Display`) with a recursive call to itself
which uses `self` as a parameter.
This is typically done indirectly with the `write!` macro or with `to_string()`.

#### Why is this bad?

This will lead to infinite recursion and a stack overflow.

#### Example

```rust
use std::fmt;

struct Structure(i32);
impl fmt::Display for Structure {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.to_string())
    }
}
```

Use instead:

```rust
use std::fmt;

struct Structure(i32);
impl fmt::Display for Structure {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.0)
    }
}
```

#### Past names

- to_string_in_display

---

### `redundant_comparisons`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Checks for ineffective double comparisons against constants.

#### Why is this bad?

Only one of the comparisons has any effect on the result, the programmer
probably intended to flip one of the comparison operators, or compare a
different value entirely.

#### Example

```rust
if status_code <= 400 && status_code < 500 {}
```

---

### `reversed_empty_ranges`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.45.0 |

#### What it does

Checks for range expressions `x..y` where both `x` and `y`
are constant and `x` is greater to `y`. Also triggers if `x` is equal to `y` when they are conditions to a `for` loop.

#### Why is this bad?

Empty ranges yield no values so iterating them is a no-op.
Moreover, trying to use a reversed range to index a slice will panic at run-time.

#### Example

```rust
fn main() {
    (10..=0).for_each(|x| println!("{}", x));

    let arr = [1, 2, 3, 4, 5];
    let sub = &arr[3..1];
}
```

Use instead:

```rust
fn main() {
    (0..=10).rev().for_each(|x| println!("{}", x));

    let arr = [1, 2, 3, 4, 5];
    let sub = &arr[1..3];
}
```

#### Past names

- reverse_range_loop

---

### `self_assignment`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.48.0 |

#### What it does

Checks for explicit self-assignments.

#### Why is this bad?

Self-assignments are redundant and unlikely to be
intentional.

#### Known problems

If expression contains any deref coercions or
indexing operations they are assumed not to have any side effects.

#### Example

```rust
struct Event {
    x: i32,
}

fn copy_position(a: &mut Event, b: &Event) {
    a.x = a.x;
}
```

Should be:

```rust
struct Event {
    x: i32,
}

fn copy_position(a: &mut Event, b: &Event) {
    a.x = b.x;
}
```

---

### `serde_api_misuse`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for misuses of the serde API.

#### Why is this bad?

Serde is very finicky about how its API should be
used, but the type system can’t be used to enforce it (yet?).

#### Example

Implementing `Visitor::visit_string` but not
`Visitor::visit_str`.

---

### `size_of_in_element_count`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.50.0 |

#### What it does

Detects expressions where
`size_of::<T>` or `size_of_val::<T>` is used as a
count of elements of type `T`

#### Why is this bad?

These functions expect a count
of `T` and not a number of bytes

#### Example

```rust
const SIZE: usize = 128;
let x = [2u8; SIZE];
let mut y = [2u8; SIZE];
unsafe { copy_nonoverlapping(x.as_ptr(), y.as_mut_ptr(), size_of::<u8>() * SIZE) };
```

---

### `suspicious_splitn`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.54.0 |

#### What it does

Checks for calls to [`splitn`]
(https://doc.rust-lang.org/std/primitive.str.html#method.splitn) and
related functions with either zero or one splits.

#### Why is this bad?

These calls don’t actually split the value and are
likely to be intended as a different number.

#### Example

```rust
for x in s.splitn(1, ":") {
    // ..
}
```

Use instead:

```rust
for x in s.splitn(2, ":") {
    // ..
}
```

---

### `transmute_null_to_fn`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.68.0 |

#### What it does

Checks for null function pointer creation through transmute.

#### Why is this bad?

Creating a null function pointer is undefined behavior.

More info: https://doc.rust-lang.org/nomicon/ffi.html#the-nullable-pointer-optimization

#### Known problems

Not all cases can be detected at the moment of this writing.
For example, variables which hold a null pointer and are then fed to a `transmute`
call, aren’t detectable yet.

#### Example

```rust
let null_fn: fn() = unsafe { std::mem::transmute( std::ptr::null::<()>() ) };
```

Use instead:

```rust
let null_fn: Option<fn()> = None;
```

---

### `transmuting_null`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.35.0 |

#### What it does

Checks for transmute calls which would receive a null pointer.

#### Why is this bad?

Transmuting a null pointer is undefined behavior.

#### Known problems

Not all cases can be detected at the moment of this writing.
For example, variables which hold a null pointer and are then fed to a `transmute`
call, aren’t detectable yet.

#### Example

```rust
let null_ref: &u64 = unsafe { std::mem::transmute(0 as *const u64) };
```

---

### `uninit_assumed_init`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.39.0 |

#### What it does

Checks for `MaybeUninit::uninit().assume_init()`.

#### Why is this bad?

For most types, this is undefined behavior.

#### Known problems

For now, we accept empty tuples and tuples / arrays
of `MaybeUninit`. There may be other types that allow uninitialized
data, but those are not yet rigorously defined.

#### Example

```rust
// Beware the UB
use std::mem::MaybeUninit;

let _: usize = unsafe { MaybeUninit::uninit().assume_init() };
```

Note that the following is OK:

```rust
use std::mem::MaybeUninit;

let _: [MaybeUninit<bool>; 5] = unsafe {
    MaybeUninit::uninit().assume_init()
};
```

---

### `uninit_vec`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.58.0 |

#### What it does

Checks for `set_len()` call that creates `Vec` with uninitialized elements.
This is commonly caused by calling `set_len()` right after allocating or
reserving a buffer with `new()`, `default()`, `with_capacity()`, or `reserve()`.

#### Why is this bad?

It creates a `Vec` with uninitialized data, which leads to
undefined behavior with most safe operations. Notably, uninitialized
`Vec<u8>` must not be used with generic `Read`.

Moreover, calling `set_len()` on a `Vec` created with `new()` or `default()`
creates out-of-bound values that lead to heap memory corruption when used.

#### Known Problems

This lint only checks directly adjacent statements.

#### Example

```rust
let mut vec: Vec<u8> = Vec::with_capacity(1000);
unsafe { vec.set_len(1000); }
reader.read(&mut vec); // undefined behavior!
```

#### How to fix?

1. Use an initialized buffer:

```rust
let mut vec: Vec<u8> = vec![0; 1000];
reader.read(&mut vec);
```
2. Wrap the content in `MaybeUninit`:

```rust
let mut vec: Vec<MaybeUninit<T>> = Vec::with_capacity(1000);
vec.set_len(1000);  // `MaybeUninit` can be uninitialized
```
3. If you are on 1.60.0 or later, `Vec::spare_capacity_mut()` is available:

```rust
let mut vec: Vec<u8> = Vec::with_capacity(1000);
let remaining = vec.spare_capacity_mut();  // `&mut [MaybeUninit<u8>]`
// perform initialization with `remaining`
vec.set_len(...);  // Safe to call `set_len()` on initialized part
```

---

### `unit_cmp`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for comparisons to unit. This includes all binary
comparisons (like `==` and `<`) and asserts.

#### Why is this bad?

Unit is always equal to itself, and thus is just a
clumsily written constant. Mostly this happens when someone accidentally
adds semicolons at the end of the operands.

#### Example

```rust
if {
    foo();
} == {
    bar();
} {
    baz();
}
```

is equal to

```rust
{
    foo();
    bar();
    baz();
}
```

For asserts:

```rust
assert_eq!({ foo(); }, { bar(); });
```

will always succeed

---

### `unit_hash`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.58.0 |

#### What it does

Detects `().hash(_)`.

#### Why is this bad?

Hashing a unit value doesn’t do anything as the implementation of `Hash` for `()` is a no-op.

#### Example

```rust
match my_enum {
	Empty => ().hash(&mut state),
	WithValue(x) => x.hash(&mut state),
}
```

Use instead:

```rust
match my_enum {
	Empty => 0_u8.hash(&mut state),
	WithValue(x) => x.hash(&mut state),
}
```

---

### `unit_return_expecting_ord`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.47.0 |

#### What it does

Checks for functions that expect closures of type
Fn(…) -> Ord where the implemented closure returns the unit type.
The lint also suggests to remove the semi-colon at the end of the statement if present.

#### Why is this bad?

Likely, returning the unit type is unintentional, and
could simply be caused by an extra semi-colon. Since () implements Ord
it doesn’t cause a compilation error.
This is the same reasoning behind the unit_cmp lint.

#### Known problems

If returning unit is intentional, then there is no
way of specifying this without triggering needless_return lint

#### Example

```rust
let mut twins = vec![(1, 1), (2, 2)];
twins.sort_by_key(|x| { x.1; });
```

---

### `unsound_collection_transmute`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks for transmutes between collections whose
types have different ABI, size or alignment.

#### Why is this bad?

This is undefined behavior.

#### Known problems

Currently, we cannot know whether a type is a
collection, so we just lint the ones that come with `std`.

#### Example

```rust
// different size, therefore likely out-of-bounds memory access
// You absolutely do not want this in your code!
unsafe {
    std::mem::transmute::<_, Vec<u32>>(vec![2_u16])
};
```

You must always iterate, map and collect the values:

```rust
vec![2_u16].into_iter().map(u32::from).collect::<Vec<_>>();
```

---

### `unused_io_amount`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for unused written/read amount.

#### Why is this bad?

`io::Write::write(_vectored)` and
`io::Read::read(_vectored)` are not guaranteed to
process the entire buffer. They return how many bytes were processed, which
might be smaller
than a given buffer’s length. If you don’t need to deal with
partial-write/read, use
`write_all`/`read_exact` instead.

When working with asynchronous code (either with the `futures`
crate or with `tokio`), a similar issue exists for
`AsyncWriteExt::write()` and `AsyncReadExt::read()` : these
functions are also not guaranteed to process the entire
buffer.  Your code should either handle partial-writes/reads, or
call the `write_all`/`read_exact` methods on those traits instead.

#### Known problems

Detects only common patterns.

#### Examples

```rust
use std::io;
fn foo<W: io::Write>(w: &mut W) -> io::Result<()> {
    w.write(b"foo")?;
    Ok(())
}
```

Use instead:

```rust
use std::io;
fn foo<W: io::Write>(w: &mut W) -> io::Result<()> {
    w.write_all(b"foo")?;
    Ok(())
}
```

---

### `useless_attribute`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `extern crate` and `use` items annotated with
lint attributes.

This lint permits lint attributes for lints emitted on the items themself.
For `use` items these lints are:

- ambiguous_glob_reexports
- dead_code
- deprecated
- hidden_glob_reexports
- unreachable_pub
- unused
- unused_braces
- unused_import_braces
- clippy::disallowed_types
- clippy::enum_glob_use
- clippy::macro_use_imports
- clippy::module_name_repetitions
- clippy::redundant_pub_crate
- clippy::single_component_path_imports
- clippy::unsafe_removed_from_name
- clippy::wildcard_imports

For `extern crate` items these lints are:

- `unused_imports` on items with `#[macro_use]`

#### Why is this bad?

Lint attributes have no effect on crate imports. Most
likely a `!` was forgotten.

#### Example

```rust
#[deny(dead_code)]
extern crate foo;
#[forbid(dead_code)]
use foo::bar;
```

Use instead:

```rust
#[allow(unused_imports)]
use foo::baz;
#[allow(unused_imports)]
#[macro_use]
extern crate baz;
```

---

### `vec_resize_to_zero`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.46.0 |

#### What it does

Finds occurrences of `Vec::resize(0, an_int)`

#### Why is this bad?

This is probably an argument inversion mistake.

#### Example

```rust
vec![1, 2, 3, 4, 5].resize(0, 5)
```

Use instead:

```rust
vec![1, 2, 3, 4, 5].clear()
```

---

### `while_immutable_condition`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks whether variables used within while loop condition
can be (and are) mutated in the body.

#### Why is this bad?

If the condition is unchanged, entering the body of the loop
will lead to an infinite loop.

#### Known problems

If the `while`-loop is in a closure, the check for mutation of the
condition variables in the body can cause false negatives. For example when only `Upvar` `a` is
in the condition and only `Upvar` `b` gets mutated in the body, the lint will not trigger.

#### Example

```rust
let i = 0;
while i > 10 {
    println!("let me loop forever!");
}
```

---

### `wrong_transmute`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for transmutes that can’t ever be correct on any
architecture.

#### Why is this bad?

It’s basically guaranteed to be undefined behavior.

#### Known problems

When accessing C, users might want to store pointer
sized objects in `extradata` arguments to save an allocation.

#### Example

```rust
let ptr: *const T = core::intrinsics::transmute('x')
```

---

### `zst_offset`

| 属性 | 值 |
|------|----|
| 分组 | correctness |
| 默认级别 | deny |
| Applicability | Unspecified |
| 引入版本 | 1.41.0 |

#### What it does

Checks for `offset(_)`, `wrapping_`{`add`, `sub`}, etc. on raw pointers to
zero-sized types

#### Why is this bad?

This is a no-op, and likely unintended

#### Example

```rust
unsafe { (&() as *const ()).offset(1) };
```

---

## Suspicious (82)

*可疑 - 很可能错误或无用的代码，默认 warn*

### `almost_complete_range`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.68.0 |

#### What it does

Checks for ranges which almost include the entire range of letters from ‘a’ to ‘z’
or digits from ‘0’ to ‘9’, but don’t because they’re a half open range.

#### Why is this bad?

This (`'a'..'z'`) is almost certainly a typo meant to include all letters.

#### Example

```rust
let _ = 'a'..'z';
```

Use instead:

```rust
let _ = 'a'..='z';
```

#### Past names

- almost_complete_letter_range

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `arc_with_non_send_sync`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does.

This lint warns when you use `Arc` with a type that does not implement `Send` or `Sync`.

#### Why is this bad?

`Arc<T>` is a thread-safe `Rc<T>` and guarantees that updates to the reference counter
use atomic operations. To send an `Arc<T>` across thread boundaries and
share ownership between multiple threads, `T` must be [both Send and Sync](https://doc.rust-lang.org/std/sync/struct.Arc.html#thread-safety),
so either `T` should be made `Send + Sync` or an `Rc` should be used instead of an `Arc`.

#### Example

```rust
fn main() {
    // This is fine, as `i32` implements `Send` and `Sync`.
    let a = Arc::new(42);

    // `RefCell` is `!Sync`, so either the `Arc` should be replaced with an `Rc`
    // or the `RefCell` replaced with something like a `RwLock`
    let b = Arc::new(RefCell::new(42));
}
```

---

### `await_holding_invalid_type`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.62.0 |

#### What it does

Allows users to configure types which should not be held across await
suspension points.

#### Why is this bad?

There are some types which are perfectly safe to use concurrently from
a memory access perspective, but that will cause bugs at runtime if
they are held in such a way.

#### Example

```toml
await-holding-invalid-types = [
  # You can specify a type name
  "CustomLockType",
  # You can (optionally) specify a reason
  { path = "OtherCustomLockType", reason = "Relies on a thread local" }
]
```

```rust
struct CustomLockType;
struct OtherCustomLockType;
async fn foo() {
  let _x = CustomLockType;
  let _y = OtherCustomLockType;
  baz().await; // Lint violation
}
```

#### Configuration

- `await-holding-invalid-types`:  The list of types which may not be held across an await point.

(default: `[]`)

---

### `await_holding_lock`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.45.0 |

#### What it does

Checks for calls to `await` while holding a non-async-aware
`MutexGuard`.

#### Why is this bad?

The Mutex types found in [std::sync](https://doc.rust-lang.org/stable/std/sync/) and
[parking_lot](https://docs.rs/parking_lot/latest/parking_lot/) are
not designed to operate in an async context across await points.

There are two potential solutions. One is to use an async-aware `Mutex`
type. Many asynchronous foundation crates provide such a `Mutex` type.
The other solution is to ensure the mutex is unlocked before calling
`await`, either by introducing a scope or an explicit call to
[Drop::drop](https://doc.rust-lang.org/std/ops/trait.Drop.html).

#### Known problems

Will report false positive for explicitly dropped guards
([#6446](https://github.com/rust-lang/rust-clippy/issues/6446)). A
workaround for this is to wrap the `.lock()` call in a block instead of
explicitly dropping the guard.

#### Example

```rust
async fn foo(x: &Mutex<u32>) {
  let mut guard = x.lock().unwrap();
  *guard += 1;
  baz().await;
}

async fn bar(x: &Mutex<u32>) {
  let mut guard = x.lock().unwrap();
  *guard += 1;
  drop(guard); // explicit drop
  baz().await;
}
```

Use instead:

```rust
async fn foo(x: &Mutex<u32>) {
  {
    let mut guard = x.lock().unwrap();
    *guard += 1;
  }
  baz().await;
}

async fn bar(x: &Mutex<u32>) {
  {
    let mut guard = x.lock().unwrap();
    *guard += 1;
  } // guard dropped here at end of scope
  baz().await;
}
```

---

### `await_holding_refcell_ref`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.49.0 |

#### What it does

Checks for calls to `await` while holding a `RefCell`, `Ref`, or `RefMut`.

#### Why is this bad?

`RefCell` refs only check for exclusive mutable access
at runtime. Holding a `RefCell` ref across an await suspension point
risks panics from a mutable ref shared while other refs are outstanding.

#### Known problems

Will report false positive for explicitly dropped refs
([#6353](https://github.com/rust-lang/rust-clippy/issues/6353)). A workaround for this is
to wrap the `.borrow[_mut]()` call in a block instead of explicitly dropping the ref.

#### Example

```rust
async fn foo(x: &RefCell<u32>) {
  let mut y = x.borrow_mut();
  *y += 1;
  baz().await;
}

async fn bar(x: &RefCell<u32>) {
  let mut y = x.borrow_mut();
  *y += 1;
  drop(y); // explicit drop
  baz().await;
}
```

Use instead:

```rust
async fn foo(x: &RefCell<u32>) {
  {
     let mut y = x.borrow_mut();
     *y += 1;
  }
  baz().await;
}

async fn bar(x: &RefCell<u32>) {
  {
    let mut y = x.borrow_mut();
    *y += 1;
  } // y dropped here at end of scope
  baz().await;
}
```

---

### `blanket_clippy_restriction_lints`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.47.0 |

#### What it does

Checks for `warn`/`deny`/`forbid` attributes targeting the whole clippy::restriction category.

#### Why is this bad?

Restriction lints sometimes are in contrast with other lints or even go against idiomatic rust.
These lints should only be enabled on a lint-by-lint basis and with careful consideration.

#### Example

```rust
#![deny(clippy::restriction)]
```

Use instead:

```rust
#![deny(clippy::as_conversions)]
```

---

### `cast_abs_to_unsigned`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

Checks for usage of the `abs()` method that cast the result to unsigned.

#### Why is this bad?

The `unsigned_abs()` method avoids panic when called on the MIN value.

#### Example

```rust
let x: i32 = -42;
let y: u32 = x.abs() as u32;
```

Use instead:

```rust
let x: i32 = -42;
let y: u32 = x.unsigned_abs();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `cast_enum_constructor`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.61.0 |

#### What it does

Checks for casts from an enum tuple constructor to an integer.

#### Why is this bad?

The cast is easily confused with casting a c-like enum value to an integer.

#### Example

```rust
enum E { X(i32) };
let _ = E::X as usize;
```

---

### `cast_enum_truncation`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.61.0 |

#### What it does

Checks for casts from an enum type to an integral type that will definitely truncate the
value.

#### Why is this bad?

The resulting integral value will not match the value of the variant it came from.

#### Example

```rust
enum E { X = 256 };
let _ = E::X as u8;
```

---

### `cast_nan_to_int`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.66.0 |

#### What it does

Checks for a known NaN float being cast to an integer

#### Why is this bad?

NaNs are cast into zero, so one could simply use this and make the
code more readable. The lint could also hint at a programmer error.

#### Example

```rust
let _ = (0.0_f32 / 0.0) as u64;
```

Use instead:

```rust
let _ = 0_u64;
```

---

### `cast_slice_from_raw_parts`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.65.0 |

#### What it does

Checks for a raw slice being cast to a slice pointer

#### Why is this bad?

This can result in multiple `&mut` references to the same location when only a pointer is
required.
`ptr::slice_from_raw_parts` is a safe alternative that doesn’t require
the same [safety requirements](https://doc.rust-lang.org/std/slice/fn.from_raw_parts.html#safety) to be upheld.

#### Example

```rust
let _: *const [u8] = std::slice::from_raw_parts(ptr, len) as *const _;
let _: *mut [u8] = std::slice::from_raw_parts_mut(ptr, len) as *mut _;
```

Use instead:

```rust
let _: *const [u8] = std::ptr::slice_from_raw_parts(ptr, len);
let _: *mut [u8] = std::ptr::slice_from_raw_parts_mut(ptr, len);
```

---

### `confusing_method_to_numeric_cast`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.89.0 |

#### What it does

Checks for casts of a primitive method pointer like `max`/`min` to any integer type.

#### Why restrict this?

Casting a function pointer to an integer can have surprising results and can occur
accidentally if parentheses are omitted from a function call. If you aren’t doing anything
low-level with function pointers then you can opt out of casting functions to integers in
order to avoid mistakes. Alternatively, you can use this lint to audit all uses of function
pointer casts in your code.

#### Example

```rust
let _ = u16::max as usize;
```

Use instead:

```rust
let _ = u16::MAX as usize;
```

---

### `const_is_empty`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.79.0 |

#### What it does

It identifies calls to `.is_empty()` on constant values.

#### Why is this bad?

String literals and constant values are known at compile time. Checking if they
are empty will always return the same value. This might not be the intention of
the expression.

#### Example

```rust
let value = "";
if value.is_empty() {
    println!("the string is empty");
}
```

Use instead:

```rust
println!("the string is empty");
```

---

### `crate_in_macro_def`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

Checks for usage of `crate` as opposed to `$crate` in a macro definition.

#### Why is this bad?

`crate` refers to the macro call’s crate, whereas `$crate` refers to the macro definition’s
crate. Rarely is the former intended. See:
https://doc.rust-lang.org/reference/macros-by-example.html#hygiene

#### Example

```rust
#[macro_export]
macro_rules! print_message {
    () => {
        println!("{}", crate::MESSAGE);
    };
}
pub const MESSAGE: &str = "Hello!";
```

Use instead:

```rust
#[macro_export]
macro_rules! print_message {
    () => {
        println!("{}", $crate::MESSAGE);
    };
}
pub const MESSAGE: &str = "Hello!";
```

Note that if the use of `crate` is intentional, an `allow` attribute can be applied to the
macro definition, e.g.:

```rust
#[allow(clippy::crate_in_macro_def)]
macro_rules! ok { ... crate::foo ... }
```

---

### `crosspointer_transmute`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for transmutes between a type `T` and `*T`.

#### Why is this bad?

It’s easy to mistakenly transmute between a type and a
pointer to that type.

#### Example

```rust
core::intrinsics::transmute(t) // where the result type is the same as
                               // `*t` or `&t`'s
```

---

### `declare_interior_mutable_const`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the declaration of named constant which contain interior mutability.

#### Why is this bad?

Named constants are copied at every use site which means any change to their value
will be lost after the newly created value is dropped. e.g.

```rust
use core::sync::atomic::{AtomicUsize, Ordering};
const ATOMIC: AtomicUsize = AtomicUsize::new(0);
fn add_one() -> usize {
    // This will always return `0` since `ATOMIC` is copied before it's used.
    ATOMIC.fetch_add(1, Ordering::AcqRel)
}
```

If shared modification of the value is desired, a `static` item is needed instead.
If that is not desired, a `const fn` constructor should be used to make it obvious
at the use site that a new value is created.

#### Known problems

Prior to `const fn` stabilization this was the only way to provide a value which
could initialize a `static` item (e.g. the `std::sync::ONCE_INIT` constant). In
this case the use of `const` is required and this lint should be suppressed.

There also exists types which contain private fields with interior mutability, but
no way to both create a value as a constant and modify any mutable field using the
type’s public interface (e.g. `bytes::Bytes`). As there is no reasonable way to
scan a crate’s interface to see if this is the case, all such types will be linted.
If this happens use the `ignore-interior-mutability` configuration option to allow
the type.

#### Example

```rust
use std::sync::atomic::{AtomicUsize, Ordering::SeqCst};

const CONST_ATOM: AtomicUsize = AtomicUsize::new(12);
CONST_ATOM.store(6, SeqCst); // the content of the atomic is unchanged
assert_eq!(CONST_ATOM.load(SeqCst), 12); // because the CONST_ATOM in these lines are distinct
```

Use instead:

```rust
static STATIC_ATOM: AtomicUsize = AtomicUsize::new(15);
STATIC_ATOM.store(9, SeqCst);
assert_eq!(STATIC_ATOM.load(SeqCst), 9); // use a `static` item to refer to the same instance
```

#### Configuration

- `ignore-interior-mutability`:  A list of paths to types that should be treated as if they do not contain interior mutability

(default: `["bytes::Bytes"]`)

---

### `deprecated_clippy_cfg_attr`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.78.0 |

#### What it does

Checks for `#[cfg_attr(feature = "cargo-clippy", ...)]` and for
`#[cfg(feature = "cargo-clippy")]` and suggests to replace it with
`#[cfg_attr(clippy, ...)]` or `#[cfg(clippy)]`.

#### Why is this bad?

This feature has been deprecated for years and shouldn’t be used anymore.

#### Example

```rust
#[cfg(feature = "cargo-clippy")]
struct Bar;
```

Use instead:

```rust
#[cfg(clippy)]
struct Bar;
```

---

### `doc_nested_refdefs`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.85.0 |

#### What it does

Warns if a link reference definition appears at the start of a
list item or quote.

#### Why is this bad?

This is probably intended as an intra-doc link. If it is really
supposed to be a reference definition, it can be written outside
of the list item or quote.

#### Example

```rust
//! - [link]: description
```

Use instead:

```rust
//! - [link][]: description (for intra-doc link)
//!
//! [link]: destination (for link reference definition)
```

---

### `doc_suspicious_footnotes`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | 1.89.0 |

#### What it does

Detects syntax that looks like a footnote reference.

Rustdoc footnotes are compatible with GitHub-Flavored Markdown (GFM).
GFM does not parse a footnote reference unless its definition also
exists. This lint checks for footnote references with missing
definitions, unless it thinks you’re writing a regex.

#### Why is this bad?

This probably means that a footnote was meant to exist,
but was not written.

#### Example

```rust
/// This is not a footnote[^1], because no definition exists.
fn my_fn() {}
```

Use instead:

```rust
/// This is a footnote[^1].
///
/// [^1]: defined here
fn my_fn() {}
```

---

### `drop_non_drop`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.62.0 |

#### What it does

Checks for calls to `std::mem::drop` with a value that does not implement `Drop`.

#### Why is this bad?

Calling `std::mem::drop` is no different than dropping such a type. A different value may
have been intended.

#### Example

```rust
struct Foo;
let x = Foo;
std::mem::drop(x);
```

---

### `duplicate_mod`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.63.0 |

#### What it does

Checks for files that are included as modules multiple times.

#### Why is this bad?

Loading a file as a module more than once causes it to be compiled
multiple times, taking longer and putting duplicate content into the
module tree.

#### Example

```rust
// lib.rs
mod a;
mod b;
```

```rust
// a.rs
#[path = "./b.rs"]
mod b;
```

Use instead:

```rust
// lib.rs
mod a;
mod b;
```

```rust
// a.rs
use crate::b;
```

---

### `duplicated_attributes`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.79.0 |

#### What it does

Checks for attributes that appear two or more times.

#### Why is this bad?

Repeating an attribute on the same item (or globally on the same crate)
is unnecessary and doesn’t have an effect.

#### Example

```rust
#[allow(dead_code)]
#[allow(dead_code)]
fn foo() {}
```

Use instead:

```rust
#[allow(dead_code)]
fn foo() {}
```

---

### `empty_docs`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.78.0 |

#### What it does

Detects documentation that is empty.

#### Why is this bad?

Empty docs clutter code without adding value, reducing readability and maintainability.

#### Example

```rust
///
fn returns_true() -> bool {
    true
}
```

Use instead:

```rust
fn returns_true() -> bool {
    true
}
```

---

### `empty_line_after_doc_comments`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.70.0 |

#### What it does

Checks for empty lines after doc comments.

#### Why is this bad?

The doc comment may have meant to be an inner doc comment, regular
comment or applied to some old code that is now commented out. If it was
intended to be a doc comment, then the empty line should be removed.

#### Example

```rust
/// Some doc comment with a blank line after it.

fn f() {}

/// Docs for `old_code`
// fn old_code() {}

fn new_code() {}
```

Use instead:

```rust
//! Convert it to an inner doc comment

// Or a regular comment

/// Or remove the empty line
fn f() {}

// /// Docs for `old_code`
// fn old_code() {}

fn new_code() {}
```

---

### `empty_line_after_outer_attr`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for empty lines after outer attributes

#### Why is this bad?

The attribute may have meant to be an inner attribute (`#![attr]`). If
it was meant to be an outer attribute (`#[attr]`) then the empty line
should be removed

#### Example

```rust
#[allow(dead_code)]

fn not_quite_good_code() {}
```

Use instead:

```rust
// Good (as inner attribute)
#![allow(dead_code)]

fn this_is_fine() {}

// or

// Good (as outer attribute)
#[allow(dead_code)]
fn this_is_fine_too() {}
```

---

### `empty_loop`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for empty `loop` expressions.

#### Why is this bad?

These busy loops burn CPU cycles without doing
anything. It is *almost always* a better idea to `panic!` than to have
a busy loop.

If panicking isn’t possible, think of the environment and either:

- block on something
- sleep the thread for some microseconds
- yield or pause the thread

For `std` targets, this can be done with
[std::thread::sleep](https://doc.rust-lang.org/std/thread/fn.sleep.html)
or [std::thread::yield_now](https://doc.rust-lang.org/std/thread/fn.yield_now.html).

For `no_std` targets, doing this is more complicated, especially because
`#[panic_handler]`s can’t panic. To stop/pause the thread, you will
probably need to invoke some target-specific intrinsic. Examples include:

- [x86_64::instructions::hlt](https://docs.rs/x86_64/0.12.2/x86_64/instructions/fn.hlt.html)
- [cortex_m::asm::wfi](https://docs.rs/cortex-m/0.6.3/cortex_m/asm/fn.wfi.html)

#### Example

```rust
loop {}
```

---

### `float_equality_without_abs`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.48.0 |

#### What it does

Checks for statements of the form `(a - b) < f32::EPSILON` or
`(a - b) < f64::EPSILON`. Note the missing `.abs()`.

#### Why is this bad?

The code without `.abs()` is more likely to have a bug.

#### Known problems

If the user can ensure that b is larger than a, the `.abs()` is
technically unnecessary. However, it will make the code more robust and doesn’t have any
large performance implications. If the abs call was deliberately left out for performance
reasons, it is probably better to state this explicitly in the code, which then can be done
with an allow.

#### Example

```rust
pub fn is_roughly_equal(a: f32, b: f32) -> bool {
    (a - b) < f32::EPSILON
}
```

Use instead:

```rust
pub fn is_roughly_equal(a: f32, b: f32) -> bool {
    (a - b).abs() < f32::EPSILON
}
```

---

### `forget_non_drop`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.62.0 |

#### What it does

Checks for calls to `std::mem::forget` with a value that does not implement `Drop`.

#### Why is this bad?

Calling `std::mem::forget` is no different than dropping such a type. A different value may
have been intended.

#### Example

```rust
struct Foo;
let x = Foo;
std::mem::forget(x);
```

---

### `four_forward_slashes`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.73.0 |

#### What it does

Checks for outer doc comments written with 4 forward slashes (`////`).

#### Why is this bad?

This is (probably) a typo, and results in it not being a doc comment; just a regular
comment.

#### Example

```rust
//// My amazing data structure
pub struct Foo {
    // ...
}
```

Use instead:

```rust
/// My amazing data structure
pub struct Foo {
    // ...
}
```

---

### `from_raw_with_void_ptr`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.67.0 |

#### What it does

Checks if we’re passing a `c_void` raw pointer to `{Box,Rc,Arc,Weak}::from_raw(_)`

#### Why is this bad?

When dealing with `c_void` raw pointers in FFI, it is easy to run into the pitfall of calling `from_raw` with the `c_void` pointer.
The type signature of `Box::from_raw` is `fn from_raw(raw: *mut T) -> Box<T>`, so if you pass a `*mut c_void` you will get a `Box<c_void>` (and similarly for `Rc`, `Arc` and `Weak`).
For this to be safe, `c_void` would need to have the same memory layout as the original type, which is often not the case.

#### Example

```rust
let ptr = Box::into_raw(Box::new(42usize)) as *mut c_void;
let _ = unsafe { Box::from_raw(ptr) };
```

Use instead:

```rust
let _ = unsafe { Box::from_raw(ptr as *mut usize) };
```

---

### `incompatible_msrv`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.78.0 |

#### What it does

This lint checks that no function newer than the defined MSRV (minimum
supported rust version) is used in the crate.

#### Why is this bad?

It would prevent the crate to be actually used with the specified MSRV.

#### Example

```rust
// MSRV of 1.3.0
use std::thread::sleep;
use std::time::Duration;

// Sleep was defined in `1.4.0`.
sleep(Duration::new(1, 0));
```

To fix this problem, either increase your MSRV or use another item
available in your current MSRV.

You can also locally change the MSRV that should be checked by Clippy,
for example if a feature in your crate (e.g., `modern_compiler`) should
allow you to use an item:

```rust
//! This crate has a MSRV of 1.3.0, but we also have an optional feature
//! `sleep_well` which requires at least Rust 1.4.0.

// When the `sleep_well` feature is set, do not warn for functions available
// in Rust 1.4.0 and below.
#![cfg_attr(feature = "sleep_well", clippy::msrv = "1.4.0")]

use std::time::Duration;

#[cfg(feature = "sleep_well")]
fn sleep_for_some_time() {
    std::thread::sleep(Duration::new(1, 0)); // Will not trigger the lint
}
```

You can also increase the MSRV in tests, by using:

```rust
// Use a much higher MSRV for tests while keeping the main one low
#![cfg_attr(test, clippy::msrv = "1.85.0")]

#[test]
fn my_test() {
    // The tests can use items introduced in Rust 1.85.0 and lower
    // without triggering the `incompatible_msrv` lint.
}
```

#### Configuration

- `check-incompatible-msrv-in-tests`:  Whether to check MSRV compatibility in `#[test]` and `#[cfg(test)]` code.

(default: `false`)

---

### `ineffective_open_options`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.76.0 |

#### What it does

Checks if both `.write(true)` and `.append(true)` methods are called
on a same `OpenOptions`.

#### Why is this bad?

`.append(true)` already enables `write(true)`, making this one
superfluous.

#### Example

```rust
let _ = OpenOptions::new()
           .write(true)
           .append(true)
           .create(true)
           .open("file.json");
```

Use instead:

```rust
let _ = OpenOptions::new()
           .append(true)
           .create(true)
           .open("file.json");
```

---

### `infallible_try_from`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.89.0 |

#### What it does

Finds manual impls of `TryFrom` with infallible error types.

#### Why is this bad?

Infallible conversions should be implemented via `From` with the blanket conversion.

#### Example

```rust
use std::convert::Infallible;
struct MyStruct(i16);
impl TryFrom<i16> for MyStruct {
    type Error = Infallible;
    fn try_from(other: i16) -> Result<Self, Infallible> {
        Ok(Self(other.into()))
    }
}
```

Use instead:

```rust
struct MyStruct(i16);
impl From<i16> for MyStruct {
    fn from(other: i16) -> Self {
        Self(other)
    }
}
```

---

### `iter_out_of_bounds`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.74.0 |

#### What it does

Looks for iterator combinator calls such as `.take(x)` or `.skip(x)`
where `x` is greater than the amount of items that an iterator will produce.

#### Why is this bad?

Taking or skipping more items than there are in an iterator either creates an iterator
with all items from the original iterator or an iterator with no items at all.
This is most likely not what the user intended to do.

#### Example

```rust
for _ in [1, 2, 3].iter().take(4) {}
```

Use instead:

```rust
for _ in [1, 2, 3].iter() {}
```

---

### `join_absolute_paths`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.76.0 |

#### What it does

Checks for calls to `Path::join` that start with a path separator (`\\` or `/`).

#### Why is this bad?

If the argument to `Path::join` starts with a separator, it will overwrite
the original path. If this is intentional, prefer using `Path::new` instead.

Note the behavior is platform dependent. A leading `\\` will be accepted
on unix systems as part of the file name

See [Path::join](https://doc.rust-lang.org/std/path/struct.Path.html#method.join)

#### Example

```rust
let path = Path::new("/bin");
let joined_path = path.join("/sh");
assert_eq!(joined_path, PathBuf::from("/sh"));
```

Use instead;

```rust
let path = Path::new("/bin");

// If this was unintentional, remove the leading separator
let joined_path = path.join("sh");
assert_eq!(joined_path, PathBuf::from("/bin/sh"));

// If this was intentional, create a new path instead
let new = Path::new("/sh");
assert_eq!(new, PathBuf::from("/sh"));
```

---

### `let_underscore_future`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.67.0 |

#### What it does

Checks for `let _ = <expr>` where the resulting type of expr implements `Future`

#### Why is this bad?

Futures must be polled for work to be done. The original intention was most likely to await the future
and ignore the resulting value.

#### Example

```rust
async fn foo() -> Result<(), ()> {
    Ok(())
}
let _ = foo();
```

Use instead:

```rust
async fn foo() -> Result<(), ()> {
    Ok(())
}
let _ = foo().await;
```

---

### `lines_filter_map_ok`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.70.0 |

#### What it does

Checks for usage of `lines.filter_map(Result::ok)` or `lines.flat_map(Result::ok)`
when `lines` has type `std::io::Lines`.

#### Why is this bad?

`Lines` instances might produce a never-ending stream of `Err`, in which case
`filter_map(Result::ok)` will enter an infinite loop while waiting for an
`Ok` variant. Calling `next()` once is sufficient to enter the infinite loop,
even in the absence of explicit loops in the user code.

This situation can arise when working with user-provided paths. On some platforms,
`std::fs::File::open(path)` might return `Ok(fs)` even when `path` is a directory,
but any later attempt to read from `fs` will return an error.

#### Known problems

This lint suggests replacing `filter_map()` or `flat_map()` applied to a `Lines`
instance in all cases. There are two cases where the suggestion might not be
appropriate or necessary:

- If the `Lines` instance can never produce any error, or if an error is produced
only once just before terminating the iterator, using `map_while()` is not
necessary but will not do any harm.
- If the `Lines` instance can produce intermittent errors then recover and produce
successful results, using `map_while()` would stop at the first error.

#### Example

```rust
let mut lines = BufReader::new(File::open("some-path")?).lines().filter_map(Result::ok);
// If "some-path" points to a directory, the next statement never terminates:
let first_line: Option<String> = lines.next();
```

Use instead:

```rust
let mut lines = BufReader::new(File::open("some-path")?).lines().map_while(Result::ok);
let first_line: Option<String> = lines.next();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `macro_metavars_in_unsafe`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.80.0 |

#### What it does

Looks for macros that expand metavariables in an unsafe block.

#### Why is this bad?

This hides an unsafe block and allows the user of the macro to write unsafe code without an explicit
unsafe block at callsite, making it possible to perform unsafe operations in seemingly safe code.

The macro should be restructured so that these metavariables are referenced outside of unsafe blocks
and that the usual unsafety checks apply to the macro argument.

This is usually done by binding it to a variable outside of the unsafe block
and then using that variable inside of the block as shown in the example, or by referencing it a second time
in a safe context, e.g. `if false { $expr }`.

#### Known limitations

Due to how macros are represented in the compiler at the time Clippy runs its lints,
it’s not possible to look for metavariables in macro definitions directly.

Instead, this lint looks at expansions of macros.
This leads to false negatives for macros that are never actually invoked.

By default, this lint is rather conservative and will only emit warnings on publicly-exported
macros from the same crate, because oftentimes private internal macros are one-off macros where
this lint would just be noise (e.g. macros that generate `impl` blocks).
The default behavior should help with preventing a high number of such false positives,
however it can be configured to also emit warnings in private macros if desired.

#### Example

```rust
/// Gets the first element of a slice
macro_rules! first {
    ($slice:expr) => {
        unsafe {
            let slice = $slice; // ⚠️ expansion inside of `unsafe {}`

            assert!(!slice.is_empty());
            // SAFETY: slice is checked to have at least one element
            slice.first().unwrap_unchecked()
        }
    }
}

assert_eq!(*first!(&[1i32]), 1);

// This will compile as a consequence (note the lack of `unsafe {}`)
assert_eq!(*first!(std::hint::unreachable_unchecked() as &[i32]), 1);
```

Use instead:

```rust
macro_rules! first {
    ($slice:expr) => {{
        let slice = $slice; // ✅ outside of `unsafe {}`
        unsafe {
            assert!(!slice.is_empty());
            // SAFETY: slice is checked to have at least one element
            slice.first().unwrap_unchecked()
        }
    }}
}

assert_eq!(*first!(&[1]), 1);

// This won't compile:
assert_eq!(*first!(std::hint::unreachable_unchecked() as &[i32]), 1);
```

#### Configuration

- `warn-unsafe-macro-metavars-in-private-macros`:  Whether to also emit warnings for unsafe blocks with metavariable expansions in **private** macros.

(default: `false`)

---

### `manual_unwrap_or_default`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.79.0 |

#### What it does

Checks if a `match` or `if let` expression can be simplified using
`.unwrap_or_default()`.

#### Why is this bad?

It can be done in one call with `.unwrap_or_default()`.

#### Example

```rust
let x: Option<String> = Some(String::new());
let y: String = match x {
    Some(v) => v,
    None => String::new(),
};

let x: Option<Vec<String>> = Some(Vec::new());
let y: Vec<String> = if let Some(v) = x {
    v
} else {
    Vec::new()
};
```

Use instead:

```rust
let x: Option<String> = Some(String::new());
let y: String = x.unwrap_or_default();

let x: Option<Vec<String>> = Some(Vec::new());
let y: Vec<String> = x.unwrap_or_default();
```

---

### `misnamed_getters`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.67.0 |

#### What it does

Checks for getter methods that return a field that doesn’t correspond
to the name of the method, when there is a field’s whose name matches that of the method.

#### Why is this bad?

It is most likely that such a method is a bug caused by a typo or by copy-pasting.

#### Example

```rust
struct A {
    a: String,
    b: String,
}

impl A {
    fn a(&self) -> &str{
        &self.b
    }
}
```

Use instead:

```rust
struct A {
    a: String,
    b: String,
}

impl A {
    fn a(&self) -> &str{
        &self.a
    }
}
```

---

### `misrefactored_assign_op`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `a op= a op b` or `a op= b op a` patterns.

#### Why is this bad?

Most likely these are bugs where one meant to write `a op= b`.

#### Known problems

Clippy cannot know for sure if `a op= a op b` should have
been `a = a op a op b` or `a = a op b`/`a op= b`. Therefore, it suggests both.
If `a op= a op b` is really the correct behavior it should be
written as `a = a op a op b` as it’s less confusing.

#### Example

```rust
let mut a = 5;
let b = 2;
// ...
a += a + b;
```

---

### `missing_transmute_annotations`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.79.0 |

#### What it does

Checks if transmute calls have all generics specified.

#### Why is this bad?

If not, one or more unexpected types could be used during `transmute()`, potentially leading
to Undefined Behavior or other problems.

This is particularly dangerous in case a seemingly innocent/unrelated change causes type
inference to result in a different type. For example, if `transmute()` is the tail
expression of an `if`-branch, and the `else`-branch type changes, the compiler may silently
infer a different type to be returned by `transmute()`. That is because the compiler is
free to change the inference of a type as long as that inference is technically correct,
regardless of the programmer’s unknown expectation.

Both type-parameters, the input- and the output-type, to any `transmute()` should
be given explicitly: Setting the input-type explicitly avoids confusion about what the
argument’s type actually is. Setting the output-type explicitly avoids type-inference
to infer a technically correct yet unexpected type.

#### Example

```rust
let mut x: i32 = 0;
// Avoid "naked" calls to `transmute()`!
x = std::mem::transmute([1u16, 2u16]);

// `first_answers` is intended to transmute a slice of bool to a slice of u8.
// But the programmer forgot to index the first element of the outer slice,
// so we are actually transmuting from "pointers to slices" instead of
// transmuting from "a slice of bool", causing a nonsensical result.
let the_answers: &[&[bool]] = &[&[true, false, true]];
let first_answers: &[u8] = std::mem::transmute(the_answers);
```

Use instead:

```rust
let x = std::mem::transmute::<[u16; 2], i32>([1u16, 2u16]);

// The explicit type parameters on `transmute()` makes the intention clear,
// and cause a type-error if the actual types don't match our expectation.
let the_answers: &[&[bool]] = &[&[true, false, true]];
let first_answers: &[u8] = std::mem::transmute::<&[bool], &[u8]>(the_answers[0]);
```

---

### `multi_assignments`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.65.0 |

#### What it does

Checks for nested assignments.

#### Why is this bad?

While this is in most cases already a type mismatch,
the result of an assignment being `()` can throw off people coming from languages like python or C,
where such assignments return a copy of the assigned value.

#### Example

```rust
a = b = 42;
```

Use instead:

```rust
b = 42;
a = b;
```

---

### `mut_range_bound`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for loops with a range bound that is a mutable variable.

#### Why is this bad?

One might think that modifying the mutable variable changes the loop bounds. It doesn’t.

#### Known problems

False positive when mutation is followed by a `break`, but the `break` is not immediately
after the mutation:

```rust
let mut x = 5;
for _ in 0..x {
    x += 1; // x is a range bound that is mutated
    ..; // some other expression
    break; // leaves the loop, so mutation is not an issue
}
```

False positive on nested loops ([#6072](https://github.com/rust-lang/rust-clippy/issues/6072))

#### Example

```rust
let mut foo = 42;
for i in 0..foo {
    foo -= 1;
    println!("{i}"); // prints numbers from 0 to 41, not 0 to 21
}
```

---

### `mutable_key_type`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.42.0 |

#### What it does

Checks for sets/maps with mutable key types.

#### Why is this bad?

All of `HashMap`, `HashSet`, `BTreeMap` and
`BtreeSet` rely on either the hash or the order of keys be unchanging,
so having types with interior mutability is a bad idea.

#### Known problems

#### False Positives

It’s correct to use a struct that contains interior mutability as a key when its
implementation of `Hash` or `Ord` doesn’t access any of the interior mutable types.
However, this lint is unable to recognize this, so it will often cause false positives in
these cases.

#### False Negatives

This lint does not follow raw pointers (`*const T` or `*mut T`) as `Hash` and `Ord`
apply only to the **address** of the contained value. This can cause false negatives for
custom collections that use raw pointers internally.

#### Example

```rust
use std::cmp::{PartialEq, Eq};
use std::collections::HashSet;
use std::hash::{Hash, Hasher};
use std::sync::atomic::AtomicUsize;

struct Bad(AtomicUsize);
impl PartialEq for Bad {
    fn eq(&self, rhs: &Self) -> bool {
         ..
    }
}

impl Eq for Bad {}

impl Hash for Bad {
    fn hash<H: Hasher>(&self, h: &mut H) {
        ..
    }
}

fn main() {
    let _: HashSet<Bad> = HashSet::new();
}
```

#### Configuration

- `ignore-interior-mutability`:  A list of paths to types that should be treated as if they do not contain interior mutability

(default: `["bytes::Bytes"]`)

---

### `needless_character_iteration`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.81.0 |

#### What it does

Checks if an iterator is used to check if a string is ascii.

#### Why is this bad?

The `str` type already implements the `is_ascii` method.

#### Example

```rust
"foo".chars().all(|c| c.is_ascii());
```

Use instead:

```rust
"foo".is_ascii();
```

---

### `needless_maybe_sized`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.81.0 |

#### What it does

Lints `?Sized` bounds applied to type parameters that cannot be unsized

#### Why is this bad?

The `?Sized` bound is misleading because it cannot be satisfied by an
unsized type

#### Example

```rust
// `T` cannot be unsized because `Clone` requires it to be `Sized`
fn f<T: Clone + ?Sized>(t: &T) {}
```

Use instead:

```rust
fn f<T: Clone>(t: &T) {}

// or choose alternative bounds for `T` so that it can be unsized
```

---

### `no_effect_replace`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.63.0 |

#### What it does

Checks for `replace` statements which have no effect.

#### Why is this bad?

It’s either a mistake or confusing.

#### Example

```rust
"1234".replace("12", "12");
"1234".replacen("12", "12", 1);
```

---

### `non_canonical_clone_impl`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.72.0 |

#### What it does

Checks for non-canonical implementations of `Clone` when `Copy` is already implemented.

#### Why is this bad?

If both `Clone` and `Copy` are implemented, they must agree. This can done by dereferencing
`self` in `Clone`’s implementation, which will avoid any possibility of the implementations
becoming out of sync.

#### Example

```rust
#[derive(Eq, PartialEq)]
struct A(u32);

impl Clone for A {
    fn clone(&self) -> Self {
        Self(self.0)
    }
}

impl Copy for A {}
```

Use instead:

```rust
#[derive(Eq, PartialEq)]
struct A(u32);

impl Clone for A {
    fn clone(&self) -> Self {
        *self
    }
}

impl Copy for A {}
```

#### Past names

- incorrect_clone_impl_on_copy_type

---

### `non_canonical_partial_ord_impl`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Checks for non-canonical implementations of `PartialOrd` when `Ord` is already implemented.

#### Why is this bad?

If both `PartialOrd` and `Ord` are implemented, they must agree. This is commonly done by
wrapping the result of `cmp` in `Some` for `partial_cmp`. Not doing this may silently
introduce an error upon refactoring.

#### Known issues

Code that calls the `.into()` method instead will be flagged, despite `.into()` wrapping it
in `Some`.

#### Example

```rust
#[derive(Eq, PartialEq)]
struct A(u32);

impl Ord for A {
    fn cmp(&self, other: &Self) -> Ordering {
        // ...
    }
}

impl PartialOrd for A {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        // ...
    }
}
```

Use instead:

```rust
#[derive(Eq, PartialEq)]
struct A(u32);

impl Ord for A {
    fn cmp(&self, other: &Self) -> Ordering {
        // ...
    }
}

impl PartialOrd for A {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        Some(self.cmp(other))   // or self.cmp(other).into()
    }
}
```

#### Past names

- incorrect_partial_ord_impl_on_ord_type

---

### `octal_escapes`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.59.0 |

#### What it does

Checks for `\0` escapes in string and byte literals that look like octal
character escapes in C.

#### Why is this bad?

C and other languages support octal character escapes in strings, where
a backslash is followed by up to three octal digits. For example, `\033`
stands for the ASCII character 27 (ESC). Rust does not support this
notation, but has the escape code `\0` which stands for a null
byte/character, and any following digits do not form part of the escape
sequence. Therefore, `\033` is not a compiler error but the result may
be surprising.

#### Known problems

The actual meaning can be the intended one. `\x00` can be used in these
cases to be unambiguous.

The lint does not trigger for format strings in `print!()`, `write!()`
and friends since the string is already preprocessed when Clippy lints
can see it.

#### Example

```rust
let one = "\033[1m Bold? \033[0m";  // \033 intended as escape
let two = "\033\0";                 // \033 intended as null-3-3
```

Use instead:

```rust
let one = "\x1b[1mWill this be bold?\x1b[0m";
let two = "\x0033\x00";
```

---

### `path_ends_with_ext`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.74.0 |

#### What it does

Looks for calls to `Path::ends_with` calls where the argument looks like a file extension.

By default, Clippy has a short list of known filenames that start with a dot
but aren’t necessarily file extensions (e.g. the `.git` folder), which are allowed by default.
The `allowed-dotfiles` configuration can be used to allow additional
file extensions that Clippy should not lint.

#### Why is this bad?

This doesn’t actually compare file extensions. Rather, `ends_with` compares the given argument
to the last **component** of the path and checks if it matches exactly.

#### Known issues

File extensions are often at most three characters long, so this only lints in those cases
in an attempt to avoid false positives.
Any extension names longer than that are assumed to likely be real path components and are
therefore ignored.

#### Example

```rust
fn is_markdown(path: &Path) -> bool {
    path.ends_with(".md")
}
```

Use instead:

```rust
fn is_markdown(path: &Path) -> bool {
    path.extension().is_some_and(|ext| ext == "md")
}
```

#### Configuration

- `allowed-dotfiles`:  Additional dotfiles (files or directories starting with a dot) to allow

(default: `[]`)

---

### `permissions_set_readonly_false`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.68.0 |

#### What it does

Checks for calls to `std::fs::Permissions.set_readonly` with argument `false`.

#### Why is this bad?

On Unix platforms this results in the file being world writable,
equivalent to `chmod a+w <file>`.

#### Example

```rust
use std::fs::File;
let f = File::create("foo.txt").unwrap();
let metadata = f.metadata().unwrap();
let mut permissions = metadata.permissions();
permissions.set_readonly(false);
```

---

### `pointers_in_nomem_asm_block`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.81.0 |

#### What it does

Checks if any pointer is being passed to an asm! block with `nomem` option.

#### Why is this bad?

`nomem` forbids any reads or writes to memory and passing a pointer suggests
that either of those will happen.

#### Example

```rust
fn f(p: *mut u32) {
    unsafe { core::arch::asm!("mov [{p}], 42", p = in(reg) p, options(nomem, nostack)); }
}
```

Use instead:

```rust
fn f(p: *mut u32) {
    unsafe { core::arch::asm!("mov [{p}], 42", p = in(reg) p, options(nostack)); }
}
```

---

### `possible_missing_else`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.91.0 |

#### What it does

Checks for an `if` expression followed by either a block or another `if` that
looks like it should have an `else` between them.

#### Why is this bad?

This is probably some refactoring remnant, even if the code is correct, it
might look confusing.

#### Example

```rust
if foo {
} { // looks like an `else` is missing here
}

if foo {
} if bar { // looks like an `else` is missing here
}
```

---

### `print_in_format_impl`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | 1.61.0 |

#### What it does

Checks for usage of `println`, `print`, `eprintln` or `eprint` in an
implementation of a formatting trait.

#### Why is this bad?

Using a print macro is likely unintentional since formatting traits
should write to the `Formatter`, not stdout/stderr.

#### Example

```rust
use std::fmt::{Display, Error, Formatter};

struct S;
impl Display for S {
    fn fmt(&self, f: &mut Formatter) -> Result<(), Error> {
        println!("S");

        Ok(())
    }
}
```

Use instead:

```rust
use std::fmt::{Display, Error, Formatter};

struct S;
impl Display for S {
    fn fmt(&self, f: &mut Formatter) -> Result<(), Error> {
        writeln!(f, "S");

        Ok(())
    }
}
```

---

### `rc_clone_in_vec_init`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | 1.63.0 |

#### What it does

Checks for reference-counted pointers (`Arc`, `Rc`, `rc::Weak`, and `sync::Weak`)
in `vec![elem; len]`

#### Why is this bad?

This will create `elem` once and clone it `len` times - doing so with `Arc`/`Rc`/`Weak`
is a bit misleading, as it will create references to the same pointer, rather
than different instances.

#### Example

```rust
let v = vec![std::sync::Arc::new("some data".to_string()); 100];
// or
let v = vec![std::rc::Rc::new("some data".to_string()); 100];
```

Use instead:

```rust
// Initialize each value separately:
let mut data = Vec::with_capacity(100);
for _ in 0..100 {
    data.push(std::rc::Rc::new("some data".to_string()));
}

// Or if you want clones of the same reference,
// Create the reference beforehand to clarify that
// it should be cloned for each value
let data = std::rc::Rc::new("some data".to_string());
let v = vec![data; 100];
```

---

### `redundant_locals`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Checks for redundant redefinitions of local bindings.

#### Why is this bad?

Redundant redefinitions of local bindings do not change behavior other than variable’s lifetimes and are likely to be unintended.

These rebindings can be intentional to shorten the lifetimes of variables because they affect when the `Drop` implementation is called. Other than that, they do not affect your code’s meaning but they *may* affect `rustc`’s stack allocation.

#### Example

```rust
let a = 0;
let a = a;

fn foo(b: i32) {
    let b = b;
}
```

Use instead:

```rust
let a = 0;
// no redefinition with the same name

fn foo(b: i32) {
  // no redefinition with the same name
}
```

---

### `repeat_vec_with_capacity`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.76.0 |

#### What it does

Looks for patterns such as `vec![Vec::with_capacity(x); n]` or `iter::repeat(Vec::with_capacity(x))`.

#### Why is this bad?

These constructs work by cloning the element, but cloning a `Vec<_>` does not
respect the old vector’s capacity and effectively discards it.

This makes `iter::repeat(Vec::with_capacity(x))` especially suspicious because the user most certainly
expected that the yielded `Vec<_>` will have the requested capacity, otherwise one can simply write
`iter::repeat(Vec::new())` instead and it will have the same effect.

Similarly for `vec![x; n]`, the element `x` is cloned to fill the vec.
Unlike `iter::repeat` however, the vec repeat macro does not have to clone the value `n` times
but just `n - 1` times, because it can reuse the passed value for the last slot.
That means that the last `Vec<_>` gets the requested capacity but all other ones do not.

#### Example

```rust
let _: Vec<Vec<u8>> = vec![Vec::with_capacity(42); 123];
let _: Vec<Vec<u8>> = iter::repeat(Vec::with_capacity(42)).take(123).collect();
```

Use instead:

```rust
let _: Vec<Vec<u8>> = iter::repeat_with(|| Vec::with_capacity(42)).take(123).collect();
//                                      ^^^ this closure executes 123 times
//                                          and the vecs will have the expected capacity
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `repr_packed_without_abi`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.85.0 |

#### What it does

Checks for items with `#[repr(packed)]`-attribute without ABI qualification

#### Why is this bad?

Without qualification, `repr(packed)` implies `repr(Rust)`. The Rust-ABI is inherently unstable.
While this is fine as long as the type is accessed correctly within Rust-code, most uses
of `#[repr(packed)]` involve FFI and/or data structures specified by network-protocols or
other external specifications. In such situations, the unstable Rust-ABI implied in
`#[repr(packed)]` may lead to future bugs should the Rust-ABI change.

In case you are relying on a well defined and stable memory layout, qualify the type’s
representation using the `C`-ABI. Otherwise, if the type in question is only ever
accessed from Rust-code according to Rust’s rules, use the `Rust`-ABI explicitly.

#### Example

```rust
#[repr(packed)]
struct NetworkPacketHeader {
    header_length: u8,
    header_version: u16
}
```

Use instead:

```rust
#[repr(C, packed)]
struct NetworkPacketHeader {
    header_length: u8,
    header_version: u16
}
```

---

### `single_range_in_vec_init`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.72.0 |

#### What it does

Checks for `Vec` or array initializations that contain only one range.

#### Why is this bad?

This is almost always incorrect, as it will result in a `Vec` that has only one element.
Almost always, the programmer intended for it to include all elements in the range or for
the end of the range to be the length instead.

#### Example

```rust
let x = [0..200];
```

Use instead:

```rust
// If it was intended to include every element in the range...
let x = (0..200).collect::<Vec<i32>>();
// ...Or if 200 was meant to be the len
let x = [0; 200];
```

---

### `size_of_ref`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.68.0 |

#### What it does

Checks for calls to `size_of_val()` where the argument is
a reference to a reference.

#### Why is this bad?

Calling `size_of_val()` with a reference to a reference as the argument
yields the size of the reference-type, not the size of the value behind
the reference.

#### Example

```rust
struct Foo {
    buffer: [u8],
}

impl Foo {
    fn size(&self) -> usize {
        // Note that `&self` as an argument is a `&&Foo`: Because `self`
        // is already a reference, `&self` is a double-reference.
        // The return value of `size_of_val()` therefore is the
        // size of the reference-type, not the size of `self`.
        size_of_val(&self)
    }
}
```

Use instead:

```rust
struct Foo {
    buffer: [u8],
}

impl Foo {
    fn size(&self) -> usize {
        // Correct
        size_of_val(self)
    }
}
```

---

### `suspicious_arithmetic_impl`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Lints for suspicious operations in impls of arithmetic operators, e.g.
subtracting elements in an Add impl.

#### Why is this bad?

This is probably a typo or copy-and-paste error and not intended.

#### Example

```rust
impl Add for Foo {
    type Output = Foo;

    fn add(self, other: Foo) -> Foo {
        Foo(self.0 - other.0)
    }
}
```

---

### `suspicious_assignment_formatting`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of the non-existent `=*`, `=!` and `=-`
operators.

#### Why is this bad?

This is either a typo of `*=`, `!=` or `-=` or
confusing.

#### Example

```rust
a =- 42; // confusing, should it be `a -= 42` or `a = -42`?
```

---

### `suspicious_command_arg_space`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.69.0 |

#### What it does

Checks for `Command::arg()` invocations that look like they
should be multiple arguments instead, such as `arg("-t ext2")`.

#### Why is this bad?

`Command::arg()` does not split arguments by space. An argument like `arg("-t ext2")`
will be passed as a single argument to the command,
which is likely not what was intended.

#### Example

```rust
std::process::Command::new("echo").arg("-n hello").spawn().unwrap();
```

Use instead:

```rust
std::process::Command::new("echo").args(["-n", "hello"]).spawn().unwrap();
```

---

### `suspicious_doc_comments`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.70.0 |

#### What it does

Detects the use of outer doc comments (`///`, `/**`) followed by a bang (`!`): `///!`

#### Why is this bad?

Triple-slash comments (known as “outer doc comments”) apply to items that follow it.
An outer doc comment followed by a bang (i.e. `///!`) has no specific meaning.

The user most likely meant to write an inner doc comment (`//!`, `/*!`), which
applies to the parent item (i.e. the item that the comment is contained in,
usually a module or crate).

#### Known problems

Inner doc comments can only appear before items, so there are certain cases where the suggestion
made by this lint is not valid code. For example:

```rust
fn foo() {}
///!
fn bar() {}
```

This lint detects the doc comment and suggests changing it to `//!`, but an inner doc comment
is not valid at that position.

#### Example

In this example, the doc comment is attached to the *function*, rather than the *module*.

```rust
pub mod util {
    ///! This module contains utility functions.

    pub fn dummy() {}
}
```

Use instead:

```rust
pub mod util {
    //! This module contains utility functions.

    pub fn dummy() {}
}
```

---

### `suspicious_else_formatting`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for formatting of `else`. It lints if the `else`
is followed immediately by a newline or the `else` seems to be missing.

#### Why is this bad?

This is probably some refactoring remnant, even if the
code is correct, it might look confusing.

#### Example

```rust
if foo {
} { // looks like an `else` is missing here
}

if foo {
} if bar { // looks like an `else` is missing here
}

if foo {
} else

{ // this is the `else` block of the previous `if`, but should it be?
}

if foo {
} else

if bar { // this is the `else` block of the previous `if`, but should it be?
}
```

---

### `suspicious_map`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.39.0 |

#### What it does

Checks for calls to `map` followed by a `count`.

#### Why is this bad?

It looks suspicious. Maybe `map` was confused with `filter`.
If the `map` call is intentional, this should be rewritten
using `inspect`. Or, if you intend to drive the iterator to
completion, you can just use `for_each` instead.

#### Example

```rust
let _ = (0..3).map(|x| x + 2).count();
```

---

### `suspicious_op_assign_impl`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Lints for suspicious operations in impls of OpAssign, e.g.
subtracting elements in an AddAssign impl.

#### Why is this bad?

This is probably a typo or copy-and-paste error and not intended.

#### Example

```rust
impl AddAssign for Foo {
    fn add_assign(&mut self, other: Foo) {
        *self = *self - other;
    }
}
```

---

### `suspicious_open_options`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.77.0 |

#### What it does

Checks for the suspicious use of `OpenOptions::create()`
without an explicit `OpenOptions::truncate()`.

#### Why is this bad?

`create()` alone will either create a new file or open an
existing file. If the file already exists, it will be
overwritten when written to, but the file will not be
truncated by default.
If less data is written to the file
than it already contains, the remainder of the file will
remain unchanged, and the end of the file will contain old
data.
In most cases, one should either use `create_new` to ensure
the file is created from scratch, or ensure `truncate` is
called so that the truncation behaviour is explicit. `truncate(true)`
will ensure the file is entirely overwritten with new data, whereas
`truncate(false)` will explicitly keep the default behavior.

#### Example

```rust
use std::fs::OpenOptions;

OpenOptions::new().create(true);
```

Use instead:

```rust
use std::fs::OpenOptions;

OpenOptions::new().create(true).truncate(true);
```

---

### `suspicious_to_owned`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.65.0 |

#### What it does

Checks for the usage of `_.to_owned()`, on a `Cow<'_, _>`.

#### Why is this bad?

Calling `to_owned()` on a `Cow` creates a clone of the `Cow`
itself, without taking ownership of the `Cow` contents (i.e.
it’s equivalent to calling `Cow::clone`).
The similarly named `into_owned` method, on the other hand,
clones the `Cow` contents, effectively turning any `Cow::Borrowed`
into a `Cow::Owned`.

Given the potential ambiguity, consider replacing `to_owned`
with `clone` for better readability or, if getting a `Cow::Owned`
was the original intent, using `into_owned` instead.

#### Example

```rust
let s = "Hello world!";
let cow = Cow::Borrowed(s);

let data = cow.to_owned();
assert!(matches!(data, Cow::Borrowed(_)))
```

Use instead:

```rust
let s = "Hello world!";
let cow = Cow::Borrowed(s);

let data = cow.clone();
assert!(matches!(data, Cow::Borrowed(_)))
```

or

```rust
let s = "Hello world!";
let cow = Cow::Borrowed(s);

let _data: String = cow.into_owned();
```

---

### `suspicious_unary_op_formatting`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks the formatting of a unary operator on the right hand side
of a binary operator. It lints if there is no space between the binary and unary operators,
but there is a space between the unary and its operand.

#### Why is this bad?

This is either a typo in the binary operator or confusing.

#### Example

```rust
// &&! looks like a different operator
if foo &&! bar {}
```

Use instead:

```rust
if foo && !bar {}
```

---

### `swap_ptr_to_ref`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.63.0 |

#### What it does

Checks for calls to `core::mem::swap` where either parameter is derived from a pointer

#### Why is this bad?

When at least one parameter to `swap` is derived from a pointer it may overlap with the
other. This would then lead to undefined behavior.

#### Example

```rust
unsafe fn swap(x: &[*mut u32], y: &[*mut u32]) {
    for (&x, &y) in x.iter().zip(y) {
        core::mem::swap(&mut *x, &mut *y);
    }
}
```

Use instead:

```rust
unsafe fn swap(x: &[*mut u32], y: &[*mut u32]) {
    for (&x, &y) in x.iter().zip(y) {
        core::ptr::swap(x, y);
    }
}
```

---

### `test_attr_in_doctest`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.76.0 |

#### What it does

Checks for `#[test]` in doctests unless they are marked with
either `ignore`, `no_run` or `compile_fail`.

#### Why is this bad?

Code in examples marked as `#[test]` will somewhat
surprisingly not be run by `cargo test`. If you really want
to show how to test stuff in an example, mark it `no_run` to
make the intent clear.

#### Examples

```rust
/// An example of a doctest with a `main()` function
///
/// # Examples
///
/// ```
/// #[test]
/// fn equality_works() {
///     assert_eq!(1_u8, 1);
/// }
/// ```
fn test_attr_in_doctest() {
    unimplemented!();
}
```

---

### `type_id_on_box`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.73.0 |

#### What it does

Looks for calls to `.type_id()` on a `Box<dyn _>`.

#### Why is this bad?

This almost certainly does not do what the user expects and can lead to subtle bugs.
Calling `.type_id()` on a `Box<dyn Trait>` returns a fixed `TypeId` of the `Box` itself,
rather than returning the `TypeId` of the underlying type behind the trait object.

For `Box<dyn Any>` specifically (and trait objects that have `Any` as its supertrait),
this lint will provide a suggestion, which is to dereference the receiver explicitly
to go from `Box<dyn Any>` to `dyn Any`.
This makes sure that `.type_id()` resolves to a dynamic call on the trait object
and not on the box.

If the fixed `TypeId` of the `Box` is the intended behavior, it’s better to be explicit about it
and write `TypeId::of::<Box<dyn Trait>>()`:
this makes it clear that a fixed `TypeId` is returned and not the `TypeId` of the implementor.

#### Example

```rust
use std::any::{Any, TypeId};

let any_box: Box<dyn Any> = Box::new(42_i32);
assert_eq!(any_box.type_id(), TypeId::of::<i32>()); // ⚠️ this fails!
```

Use instead:

```rust
use std::any::{Any, TypeId};

let any_box: Box<dyn Any> = Box::new(42_i32);
assert_eq!((*any_box).type_id(), TypeId::of::<i32>());
//          ^ dereference first, to call `type_id` on `dyn Any`
```

---

### `unconditional_recursion`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.77.0 |

#### What it does

Checks that there isn’t an infinite recursion in trait
implementations.

#### Why is this bad?

Infinite recursion in trait implementation will either cause crashes
or result in an infinite loop, and it is hard to detect.

#### Example

```rust
enum Foo {
    A,
    B,
}

impl PartialEq for Foo {
    fn eq(&self, other: &Self) -> bool {
        self == other // bad!
    }
}
```

Use instead:

```rust
#[derive(PartialEq)]
enum Foo {
    A,
    B,
}
```

As an alternative, rewrite the logic without recursion:

```rust
enum Foo {
    A,
    B,
}

impl PartialEq for Foo {
    fn eq(&self, other: &Self) -> bool {
        matches!((self, other), (Foo::A, Foo::A) | (Foo::B, Foo::B))
    }
}
```

---

### `unnecessary_clippy_cfg`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.78.0 |

#### What it does

Checks for `#[cfg_attr(clippy, allow(clippy::lint))]`
and suggests to replace it with `#[allow(clippy::lint)]`.

#### Why is this bad?

There is no reason to put clippy attributes behind a clippy `cfg` as they are not
run by anything else than clippy.

#### Example

```rust
#![cfg_attr(clippy, allow(clippy::deprecated_cfg_attr))]
```

Use instead:

```rust
#![allow(clippy::deprecated_cfg_attr)]
```

---

### `unnecessary_get_then_check`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.78.0 |

#### What it does

Checks the usage of `.get().is_some()` or `.get().is_none()` on std map types.

#### Why is this bad?

It can be done in one call with `.contains()`/`.contains_key()`.

#### Example

```rust
let s: HashSet<String> = HashSet::new();
if s.get("a").is_some() {
    // code
}
```

Use instead:

```rust
let s: HashSet<String> = HashSet::new();
if s.contains("a") {
    // code
}
```

---

### `unnecessary_option_map_or_else`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.92.0 |

#### What it does

Checks for usage of `.map_or_else()` “map closure” for `Option` type.

#### Why is this bad?

This can be written more concisely by using `unwrap_or_else()`.

#### Example

```rust
let k = 10;
let x: Option<u32> = Some(4);
let y = x.map_or_else(|| 2 * k, |n| n);
```

Use instead:

```rust
let k = 10;
let x: Option<u32> = Some(4);
let y = x.unwrap_or_else(|| 2 * k);
```

---

### `unnecessary_result_map_or_else`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.78.0 |

#### What it does

Checks for usage of `.map_or_else()` “map closure” for `Result` type.

#### Why is this bad?

This can be written more concisely by using `unwrap_or_else()`.

#### Example

```rust
let x: Result<u32, ()> = Ok(0);
let y = x.map_or_else(|err| handle_error(err), |n| n);
```

Use instead:

```rust
let x: Result<u32, ()> = Ok(0);
let y = x.unwrap_or_else(|err| handle_error(err));
```

---

### `zero_repeat_side_effects`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.79.0 |

#### What it does

Checks for array or vec initializations which contain an expression with side effects,
but which have a repeat count of zero.

#### Why is this bad?

Such an initialization, despite having a repeat length of 0, will still call the inner function.
This may not be obvious and as such there may be unintended side effects in code.

#### Example

```rust
fn side_effect() -> i32 {
    println!("side effect");
    10
}
let a = [side_effect(); 0];
```

Use instead:

```rust
fn side_effect() -> i32 {
    println!("side effect");
    10
}
side_effect();
let a: [i32; 0] = [];
```

---

### `zombie_processes`

| 属性 | 值 |
|------|----|
| 分组 | suspicious |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.83.0 |

#### What it does

Looks for code that spawns a process but never calls `wait()` on the child.

#### Why is this bad?

As explained in the [standard library documentation](https://doc.rust-lang.org/stable/std/process/struct.Child.html#warning),
calling `wait()` is necessary on Unix platforms to properly release all OS resources associated with the process.
Not doing so will effectively leak process IDs and/or other limited global resources,
which can eventually lead to resource exhaustion, so it’s recommended to call `wait()` in long-running applications.
Such processes are called “zombie processes”.

To reduce the rate of false positives, if the spawned process is assigned to a binding, the lint actually works the other way around; it
conservatively checks that all uses of a variable definitely don’t call `wait()` and only then emits a warning.
For that reason, a seemingly unrelated use can get called out as calling `wait()` in help messages.

#### Control flow

If a `wait()` call exists in an if/then block but not in the else block (or there is no else block),
then this still gets linted as not calling `wait()` in all code paths.
Likewise, when early-returning from the function, `wait()` calls that appear after the return expression
are also not accepted.
In other words, the `wait()` call must be unconditionally reachable after the spawn expression.

#### Example

```rust
use std::process::Command;

let _child = Command::new("ls").spawn().expect("failed to execute child");
```

Use instead:

```rust
use std::process::Command;

let mut child = Command::new("ls").spawn().expect("failed to execute child");
child.wait().expect("failed to wait on child");
```

---

## Style (157)

*风格 - 应该以更惯用方式编写的代码，默认 warn*

### `assertions_on_constants`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.34.0 |

#### What it does

Checks for `assert!(true)` and `assert!(false)` calls.

#### Why is this bad?

Will be optimized out by the compiler or should probably be replaced by a
`panic!()` or `unreachable!()`

#### Example

```rust
assert!(false)
assert!(true)
const B: bool = false;
assert!(B)
```

---

### `assign_op_pattern`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `a = a op b` or `a = b commutative_op a`
patterns.

#### Why is this bad?

These can be written as the shorter `a op= b`.

#### Known problems

While forbidden by the spec, `OpAssign` traits may have
implementations that differ from the regular `Op` impl.

#### Example

```rust
let mut a = 5;
let b = 0;
// ...

a = a + b;
```

Use instead:

```rust
let mut a = 5;
let b = 0;
// ...

a += b;
```

---

### `blocks_in_conditions`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.45.0 |

#### What it does

Checks for `if` and `match` conditions that use blocks containing an
expression, statements or conditions that use closures with blocks.

#### Why is this bad?

Style, using blocks in the condition makes it hard to read.

#### Examples

```rust
if { true } { /* ... */ }

if { let x = somefunc(); x } { /* ... */ }

match { let e = somefunc(); e } {
    // ...
}
```

Use instead:

```rust
if true { /* ... */ }

let res = { let x = somefunc(); x };
if res { /* ... */ }

let res = { let e = somefunc(); e };
match res {
    // ...
}
```

#### Past names

- block_in_if_condition_expr
- block_in_if_condition_stmt
- blocks_in_if_conditions

---

### `bool_assert_comparison`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.53.0 |

#### What it does

This lint warns about boolean comparisons in assert-like macros.

#### Why is this bad?

It is shorter to use the equivalent.

#### Example

```rust
assert_eq!("a".is_empty(), false);
assert_ne!("a".is_empty(), true);
```

Use instead:

```rust
assert!(!"a".is_empty());
```

---

### `borrow_interior_mutable_const`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for a borrow of a named constant with interior mutability.

#### Why is this bad?

Named constants are copied at every use site which means any change to their value
will be lost after the newly created value is dropped. e.g.

```rust
use core::sync::atomic::{AtomicUsize, Ordering};
const ATOMIC: AtomicUsize = AtomicUsize::new(0);
fn add_one() -> usize {
    // This will always return `0` since `ATOMIC` is copied before it's borrowed
    // for use by `fetch_add`.
    ATOMIC.fetch_add(1, Ordering::AcqRel)
}
```

#### Known problems

This lint does not, and cannot in general, determine if the borrow of the constant
is used in a way which causes a mutation. e.g.

```rust
use core::cell::Cell;
const CELL: Cell<usize> = Cell::new(0);
fn get_cell() -> Cell<usize> {
    // This is fine. It borrows a copy of `CELL`, but never mutates it through the
    // borrow.
    CELL.clone()
}
```

There also exists types which contain private fields with interior mutability, but
no way to both create a value as a constant and modify any mutable field using the
type’s public interface (e.g. `bytes::Bytes`). As there is no reasonable way to
scan a crate’s interface to see if this is the case, all such types will be linted.
If this happens use the `ignore-interior-mutability` configuration option to allow
the type.

#### Example

```rust
use std::sync::atomic::{AtomicUsize, Ordering::SeqCst};
const CONST_ATOM: AtomicUsize = AtomicUsize::new(12);

CONST_ATOM.store(6, SeqCst); // the content of the atomic is unchanged
assert_eq!(CONST_ATOM.load(SeqCst), 12); // because the CONST_ATOM in these lines are distinct
```

Use instead:

```rust
use std::sync::atomic::{AtomicUsize, Ordering::SeqCst};
const CONST_ATOM: AtomicUsize = AtomicUsize::new(12);

static STATIC_ATOM: AtomicUsize = CONST_ATOM;
STATIC_ATOM.store(9, SeqCst);
assert_eq!(STATIC_ATOM.load(SeqCst), 9); // use a `static` item to refer to the same instance
```

#### Configuration

- `ignore-interior-mutability`:  A list of paths to types that should be treated as if they do not contain interior mutability

(default: `["bytes::Bytes"]`)

---

### `box_default`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.66.0 |

#### What it does

checks for `Box::new(Default::default())`, which can be written as
`Box::default()`.

#### Why is this bad?

`Box::default()` is equivalent and more concise.

#### Example

```rust
let x: Box<String> = Box::new(Default::default());
```

Use instead:

```rust
let x: Box<String> = Box::default();
```

---

### `builtin_type_shadow`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if a generic shadows a built-in type.

#### Why is this bad?

This gives surprising type errors.

#### Example

```rust
impl<u32> Foo<u32> {
    fn impl_func(&self) -> u32 {
        42
    }
}
```

---

### `byte_char_slices`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.81.0 |

#### What it does

Checks for hard to read slices of byte characters, that could be more easily expressed as a
byte string.

#### Why is this bad?

Potentially makes the string harder to read.

#### Example

```rust
&[b'H', b'e', b'l', b'l', b'o'];
```

Use instead:

```rust
b"Hello"
```

---

### `bytes_nth`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Checks for the use of `.bytes().nth()`.

#### Why is this bad?

`.as_bytes().get()` is more efficient and more
readable.

#### Example

```rust
"Hello".bytes().nth(3);
```

Use instead:

```rust
"Hello".as_bytes().get(3);
```

---

### `chars_last_cmp`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `_.chars().last()` or
`_.chars().next_back()` on a `str` to check if it ends with a given char.

#### Why is this bad?

Readability, this can be written more concisely as
`_.ends_with(_)`.

#### Example

```rust
name.chars().last() == Some('_') || name.chars().next_back() == Some('-');
```

Use instead:

```rust
name.ends_with('_') || name.ends_with('-');
```

---

### `chars_next_cmp`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `.chars().next()` on a `str` to check
if it starts with a given char.

#### Why is this bad?

Readability, this can be written more concisely as
`_.starts_with(_)`.

#### Example

```rust
let name = "foo";
if name.chars().next() == Some('_') {};
```

Use instead:

```rust
let name = "foo";
if name.starts_with('_') {};
```

---

### `cmp_null`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint checks for equality comparisons with `ptr::null` or `ptr::null_mut`

#### Why is this bad?

It’s easier and more readable to use the inherent
`.is_null()`
method instead

#### Example

```rust
use std::ptr;

if x == ptr::null() {
    // ..
}
```

Use instead:

```rust
if x.is_null() {
    // ..
}
```

---

### `collapsible_if`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for nested `if` statements which can be collapsed
by `&&`-combining their conditions.

#### Why is this bad?

Each `if`-statement adds one level of nesting, which
makes code look more complex than it really is.

#### Example

```rust
if x {
    if y {
        // …
    }
}
```

Use instead:

```rust
if x && y {
    // …
}
```

#### Configuration

- `lint-commented-code`:  Whether collapsible `if` and `else if` chains are linted if they contain comments inside the parts
that would be collapsed.

(default: `false`)

---

### `collapsible_match`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.50.0 |

#### What it does

Finds nested `match` or `if let` expressions where the patterns may be “collapsed” together
without adding any branches.

Note that this lint is not intended to find *all* cases where nested match patterns can be merged, but only
cases where merging would most likely make the code more readable.

#### Why is this bad?

It is unnecessarily verbose and complex.

#### Example

```rust
fn func(opt: Option<Result<u64, String>>) {
    let n = match opt {
        Some(n) => match n {
            Ok(n) => n,
            _ => return,
        }
        None => return,
    };
}
```

Use instead:

```rust
fn func(opt: Option<Result<u64, String>>) {
    let n = match opt {
        Some(Ok(n)) => n,
        _ => return,
    };
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `comparison_to_empty`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Checks for comparing to an empty slice such as `""` or `[]`,
and suggests using `.is_empty()` where applicable.

#### Why is this bad?

Some structures can answer `.is_empty()` much faster
than checking for equality. So it is good to get into the habit of using
`.is_empty()`, and having it is cheap.
Besides, it makes the intent clearer than a manual comparison in some contexts.

#### Example

```rust
if s == "" {
    ..
}

if arr == [] {
    ..
}
```

Use instead:

```rust
if s.is_empty() {
    ..
}

if arr.is_empty() {
    ..
}
```

---

### `default_instead_of_iter_empty`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

It checks for `std::iter::Empty::default()` and suggests replacing it with
`std::iter::empty()`.

#### Why is this bad?

`std::iter::empty()` is the more idiomatic way.

#### Example

```rust
let _ = std::iter::Empty::<usize>::default();
let iter: std::iter::Empty<usize> = std::iter::Empty::default();
```

Use instead:

```rust
let _ = std::iter::empty::<usize>();
let iter: std::iter::Empty<usize> = std::iter::empty();
```

---

### `disallowed_fields`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.93.0 |

#### What it does

Denies the configured fields in clippy.toml

Note: Even though this lint is warn-by-default, it will only trigger if
fields are defined in the clippy.toml file.

#### Why is this bad?

Some fields are undesirable in certain contexts, and it’s beneficial to
lint for them as needed.

#### Example

An example clippy.toml configuration:

```toml
disallowed-fields = [
    # Can use a string as the path of the disallowed field.
    "std::ops::Range::start",
    # Can also use an inline table with a `path` key.
    { path = "std::ops::Range::start" },
    # When using an inline table, can add a `reason` for why the field
    # is disallowed.
    { path = "std::ops::Range::start", reason = "The start of the range is not used" },
]
```

```rust
use std::ops::Range;

let range = Range { start: 0, end: 1 };
println!("{}", range.start); // `start` is disallowed in the config.
```

Use instead:

```rust
use std::ops::Range;

let range = Range { start: 0, end: 1 };
println!("{}", range.end); // `end` is _not_ disallowed in the config.
```

#### Configuration

- `disallowed-fields`:  The list of disallowed fields, written as fully qualified paths.

**Fields:**

- `path` (required): the fully qualified path to the field that should be disallowed
- `reason` (optional): explanation why this field is disallowed
- `replacement` (optional): suggested alternative method
- `allow-invalid` (optional, `false` by default): when set to `true`, it will ignore this entry
if the path doesn’t exist, instead of emitting an error

(default: `[]`)

---

### `disallowed_macros`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.66.0 |

#### What it does

Denies the configured macros in clippy.toml

Note: Even though this lint is warn-by-default, it will only trigger if
macros are defined in the clippy.toml file.

#### Why is this bad?

Some macros are undesirable in certain contexts, and it’s beneficial to
lint for them as needed.

#### Example

An example clippy.toml configuration:

```toml
disallowed-macros = [
    # Can use a string as the path of the disallowed macro.
    "std::print",
    # Can also use an inline table with a `path` key.
    { path = "std::println" },
    # When using an inline table, can add a `reason` for why the macro
    # is disallowed.
    { path = "serde::Serialize", reason = "no serializing" },
    # This would normally error if the path is incorrect, but with `allow-invalid` = `true`,
    # it will be silently ignored
    { path = "std::invalid_macro", reason = "use alternative instead", allow-invalid = true }
]
```

```rust
use serde::Serialize;

println!("warns");

// The diagnostic will contain the message "no serializing"
#[derive(Serialize)]
struct Data {
    name: String,
    value: usize,
}
```

#### Configuration

- `disallowed-macros`:  The list of disallowed macros, written as fully qualified paths.

**Fields:**

- `path` (required): the fully qualified path to the macro that should be disallowed
- `reason` (optional): explanation why this macro is disallowed
- `replacement` (optional): suggested alternative macro
- `allow-invalid` (optional, `false` by default): when set to `true`, it will ignore this entry
if the path doesn’t exist, instead of emitting an error

(default: `[]`)

---

### `disallowed_methods`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Denies the configured methods and functions in clippy.toml

Note: Even though this lint is warn-by-default, it will only trigger if
methods are defined in the clippy.toml file.

#### Why is this bad?

Some methods are undesirable in certain contexts, and it’s beneficial to
lint for them as needed.

#### Example

An example clippy.toml configuration:

```toml
disallowed-methods = [
    # Can use a string as the path of the disallowed method.
    "std::boxed::Box::new",
    # Can also use an inline table with a `path` key.
    { path = "std::time::Instant::now" },
    # When using an inline table, can add a `reason` for why the method
    # is disallowed.
    { path = "std::vec::Vec::leak", reason = "no leaking memory" },
    # Can also add a `replacement` that will be offered as a suggestion.
    { path = "std::sync::Mutex::new", reason = "prefer faster & simpler non-poisonable mutex", replacement = "parking_lot::Mutex::new" },
    # This would normally error if the path is incorrect, but with `allow-invalid` = `true`,
    # it will be silently ignored
    { path = "std::fs::InvalidPath", reason = "use alternative instead", allow-invalid = true },
]
```

```rust
let xs = vec![1, 2, 3, 4];
xs.leak(); // Vec::leak is disallowed in the config.
// The diagnostic contains the message "no leaking memory".

let _now = Instant::now(); // Instant::now is disallowed in the config.

let _box = Box::new(3); // Box::new is disallowed in the config.
```

Use instead:

```rust
let mut xs = Vec::new(); // Vec::new is _not_ disallowed in the config.
xs.push(123); // Vec::push is _not_ disallowed in the config.
```

#### Past names

- disallowed_method

#### Configuration

- `disallowed-methods`:  The list of disallowed methods, written as fully qualified paths.

**Fields:**

- `path` (required): the fully qualified path to the method that should be disallowed
- `reason` (optional): explanation why this method is disallowed
- `replacement` (optional): suggested alternative method
- `allow-invalid` (optional, `false` by default): when set to `true`, it will ignore this entry
if the path doesn’t exist, instead of emitting an error

(default: `[]`)

---

### `disallowed_names`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of disallowed names for variables, such
as `foo`.

#### Why is this bad?

These names are usually placeholder names and should be
avoided.

#### Example

```rust
let foo = 3.14;
```

#### Past names

- blacklisted_name

#### Configuration

- `disallowed-names`:  The list of disallowed names to lint about. NB: `bar` is not here since it has legitimate uses. The value
`".."` can be used as part of the list to indicate that the configured values should be appended to the
default configuration of Clippy. By default, any configuration will replace the default value.

(default: `["foo", "baz", "quux"]`)

---

### `disallowed_types`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.55.0 |

#### What it does

Denies the configured types in clippy.toml.

Note: Even though this lint is warn-by-default, it will only trigger if
types are defined in the clippy.toml file.

#### Why is this bad?

Some types are undesirable in certain contexts.

#### Example:

An example clippy.toml configuration:

```toml
disallowed-types = [
    # Can use a string as the path of the disallowed type.
    "std::collections::BTreeMap",
    # Can also use an inline table with a `path` key.
    { path = "std::net::TcpListener" },
    # When using an inline table, can add a `reason` for why the type
    # is disallowed.
    { path = "std::net::Ipv4Addr", reason = "no IPv4 allowed" },
    # Can also add a `replacement` that will be offered as a suggestion.
    { path = "std::sync::Mutex", reason = "prefer faster & simpler non-poisonable mutex", replacement = "parking_lot::Mutex" },
    # This would normally error if the path is incorrect, but with `allow-invalid` = `true`,
    # it will be silently ignored
    { path = "std::invalid::Type", reason = "use alternative instead", allow-invalid = true }
]
```

```rust
use std::collections::BTreeMap;
// or its use
let x = std::collections::BTreeMap::new();
```

Use instead:

```rust
// A similar type that is allowed by the config
use std::collections::HashMap;
```

#### Past names

- disallowed_type

#### Configuration

- `disallowed-types`:  The list of disallowed types, written as fully qualified paths.

**Fields:**

- `path` (required): the fully qualified path to the type that should be disallowed
- `reason` (optional): explanation why this type is disallowed
- `replacement` (optional): suggested alternative type
- `allow-invalid` (optional, `false` by default): when set to `true`, it will ignore this entry
if the path doesn’t exist, instead of emitting an error

(default: `[]`)

---

### `doc_lazy_continuation`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.80.0 |

#### What it does

In CommonMark Markdown, the language used to write doc comments, a
paragraph nested within a list or block quote does not need any line
after the first one to be indented or marked. The specification calls
this a “lazy paragraph continuation.”

#### Why is this bad?

This is easy to write but hard to read. Lazy continuations makes
unintended markers hard to see, and make it harder to deduce the
document’s intended structure.

#### Example

This table is probably intended to have two rows,
but it does not. It has zero rows, and is followed by
a block quote.

```rust
/// Range | Description
/// ----- | -----------
/// >= 1  | fully opaque
/// < 1   | partially see-through
fn set_opacity(opacity: f32) {}
```

Fix it by escaping the marker:

```rust
/// Range | Description
/// ----- | -----------
/// \>= 1 | fully opaque
/// < 1   | partially see-through
fn set_opacity(opacity: f32) {}
```

This example is actually intended to be a list:

```rust
/// * Do nothing.
/// * Then do something. Whatever it is needs done,
/// it should be done right now.
```

Fix it by indenting the list contents:

```rust
/// * Do nothing.
/// * Then do something. Whatever it is needs done,
///   it should be done right now.
```

---

### `doc_overindented_list_items`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.86.0 |

#### What it does

Detects overindented list items in doc comments where the continuation
lines are indented more than necessary.

#### Why is this bad?

Overindented list items in doc comments can lead to inconsistent and
poorly formatted documentation when rendered. Excessive indentation may
cause the text to be misinterpreted as a nested list item or code block,
affecting readability and the overall structure of the documentation.

#### Example

```rust
/// - This is the first item in a list
///      and this line is overindented.
```

Fixes this into:

```rust
/// - This is the first item in a list
///   and this line is overindented.
```

---

### `double_must_use`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks for a `#[must_use]` attribute without
further information on functions and methods that return a type already
marked as `#[must_use]`.

#### Why is this bad?

The attribute isn’t needed. Not using the result
will already be reported. Alternatively, one can add some text to the
attribute to improve the lint message.

#### Examples

```rust
#[must_use]
fn double_must_use() -> Result<(), ()> {
    unimplemented!();
}
```

---

### `duplicate_underscore_argument`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for function arguments having the similar names
differing by an underscore.

#### Why is this bad?

It affects code readability.

#### Example

```rust
fn foo(a: i32, _a: i32) {}
```

Use instead:

```rust
fn bar(a: i32, _b: i32) {}
```

---

### `enum_variant_names`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Detects enumeration variants that are prefixed or suffixed
by the same characters.

#### Why is this bad?

Enumeration variant names should specify their variant,
not repeat the enumeration name.

#### Limitations

Characters with no casing will be considered when comparing prefixes/suffixes
This applies to numbers and non-ascii characters without casing
e.g. `Foo1` and `Foo2` is considered to have different prefixes
(the prefixes are `Foo1` and `Foo2` respectively), as also `Bar螃`, `Bar蟹`

#### Example

```rust
enum Cake {
    BlackForestCake,
    HummingbirdCake,
    BattenbergCake,
}
```

Use instead:

```rust
enum Cake {
    BlackForest,
    Hummingbird,
    Battenberg,
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)
- `enum-variant-name-threshold`:  The minimum number of enum variants for the lints about variant names to trigger

(default: `3`)

---

### `err_expect`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

Checks for `.err().expect()` calls on the `Result` type.

#### Why is this bad?

`.expect_err()` can be called directly to avoid the extra type conversion from `err()`.

#### Example

```rust
let x: Result<u32, &str> = Ok(10);
x.err().expect("Testing err().expect()");
```

Use instead:

```rust
let x: Result<u32, &str> = Ok(10);
x.expect_err("Testing expect_err");
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `excessive_precision`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for float literals with a precision greater
than that supported by the underlying type.

The lint is suppressed for literals with over `const_literal_digits_threshold` digits.

#### Why is this bad?

Rust will truncate the literal silently.

#### Example

```rust
let v: f32 = 0.123_456_789_9;
println!("{}", v); //  0.123_456_789
```

Use instead:

```rust
let v: f64 = 0.123_456_789_9;
println!("{}", v); //  0.123_456_789_9
```

#### Configuration

- `const-literal-digits-threshold`:  The minimum digits a const float literal must have to supress the `excessive_precicion` lint

(default: `30`)

---

### `field_reassign_with_default`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.49.0 |

#### What it does

Checks for immediate reassignment of fields initialized
with Default::default().

#### Why is this bad?

It’s more idiomatic to use the [functional update syntax](https://doc.rust-lang.org/reference/expressions/struct-expr.html#functional-update-syntax).

#### Known problems

Assignments to patterns that are of tuple type are not linted.

#### Example

```rust
let mut a: A = Default::default();
a.i = 42;
```

Use instead:

```rust
let a = A {
    i: 42,
    .. Default::default()
};
```

---

### `filter_map_bool_then`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.73.0 |

#### What it does

Checks for usage of `bool::then` in `Iterator::filter_map`.

#### Why is this bad?

This can be written with `filter` then `map` instead, which would reduce nesting and
separates the filtering from the transformation phase. This comes with no cost to
performance and is just cleaner.

#### Limitations

Does not lint `bool::then_some`, as it eagerly evaluates its arguments rather than lazily.
This can create differing behavior, so better safe than sorry.

#### Example

```rust
_ = v.into_iter().filter_map(|i| (i % 2 == 0).then(|| really_expensive_fn(i)));
```

Use instead:

```rust
_ = v.into_iter().filter(|i| i % 2 == 0).map(|i| really_expensive_fn(i));
```

---

### `fn_to_numeric_cast`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts of function pointers to something other than `usize`.

#### Why is this bad?

Casting a function pointer to anything other than `usize`/`isize` is
not portable across architectures. If the target type is too small the
address would be truncated, and target types larger than `usize` are
unnecessary.

Casting to `isize` also doesn’t make sense, since addresses are never
signed.

#### Example

```rust
fn fun() -> i32 { 1 }
let _ = fun as i64;
```

Use instead:

```rust
let _ = fun as usize;
```

---

### `fn_to_numeric_cast_with_truncation`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts of a function pointer to a numeric type not wide enough to
store an address.

#### Why is this bad?

Such a cast discards some bits of the function’s address. If this is intended, it would be more
clearly expressed by casting to `usize` first, then casting the `usize` to the intended type (with
a comment) to perform the truncation.

#### Example

```rust
fn fn1() -> i16 {
    1
};
let _ = fn1 as i32;
```

Use instead:

```rust
// Cast to usize first, then comment with the reason for the truncation
fn fn1() -> i16 {
    1
};
let fn_ptr = fn1 as usize;
let fn_ptr_truncated = fn_ptr as i32;
```

---

### `for_kv_map`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for iterating a map (`HashMap` or `BTreeMap`) and
ignoring either the keys or values.

#### Why is this bad?

Readability. There are `keys` and `values` methods that
can be used to express that don’t need the values or keys.

#### Example

```rust
for (k, _) in &map {
    ..
}
```

could be replaced by

```rust
for k in map.keys() {
    ..
}
```

---

### `from_over_into`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.51.0 |

#### What it does

Searches for implementations of the `Into<..>` trait and suggests to implement `From<..>` instead.

#### Why is this bad?

According the std docs implementing `From<..>` is preferred since it gives you `Into<..>` for free where the reverse isn’t true.

#### Example

```rust
struct StringWrapper(String);

impl Into<StringWrapper> for String {
    fn into(self) -> StringWrapper {
        StringWrapper(self)
    }
}
```

Use instead:

```rust
struct StringWrapper(String);

impl From<String> for StringWrapper {
    fn from(s: String) -> StringWrapper {
        StringWrapper(s)
    }
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `from_str_radix_10`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.52.0 |

#### What it does

Checks for function invocations of the form `primitive::from_str_radix(s, 10)`

#### Why is this bad?

This specific common use case can be rewritten as `s.parse::<primitive>()`
(and in most cases, the turbofish can be removed), which reduces code length
and complexity.

#### Known problems

This lint may suggest using `(&<expression>).parse()` instead of `<expression>.parse()`
directly in some cases, which is correct but adds unnecessary complexity to the code.

#### Example

```rust
let input: &str = get_input();
let num = u16::from_str_radix(input, 10)?;
```

Use instead:

```rust
let input: &str = get_input();
let num: u16 = input.parse()?;
```

---

### `get_first`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.63.0 |

#### What it does

Checks for usage of `x.get(0)` instead of
`x.first()` or `x.front()`.

#### Why is this bad?

Using `x.first()` for `Vec`s and slices or `x.front()`
for `VecDeque`s is easier to read and has the same result.

#### Example

```rust
let x = vec![2, 3, 5];
let first_element = x.get(0);
```

Use instead:

```rust
let x = vec![2, 3, 5];
let first_element = x.first();
```

---

### `if_same_then_else`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `if/else` with the same body as the *then* part
and the *else* part.

#### Why is this bad?

This is probably a copy & paste error.

#### Example

```rust
let foo = if … {
    42
} else {
    42
};
```

---

### `implicit_saturating_add`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.66.0 |

#### What it does

Checks for implicit saturating addition.

#### Why is this bad?

The built-in function is more readable and may be faster.

#### Example

```rust
let mut u:u32 = 7000;

if u != u32::MAX {
    u += 1;
}
```

Use instead:

```rust
let mut u:u32 = 7000;

u = u.saturating_add(1);
```

---

### `implicit_saturating_sub`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.44.0 |

#### What it does

Checks for implicit saturating subtraction.

#### Why is this bad?

Simplicity and readability. Instead we can easily use an builtin function.

#### Example

```rust
let mut i: u32 = end - start;

if i != 0 {
    i -= 1;
}
```

Use instead:

```rust
let mut i: u32 = end - start;

i = i.saturating_sub(1);
```

---

### `inconsistent_digit_grouping`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if an integral or floating-point constant is
grouped inconsistently with underscores.

#### Why is this bad?

Readers may incorrectly interpret inconsistently
grouped digits.

#### Example

```rust
618_64_9189_73_511
```

Use instead:

```rust
61_864_918_973_511
```

---

### `infallible_destructuring_match`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for matches being used to destructure a single-variant enum
or tuple struct where a `let` will suffice.

#### Why is this bad?

Just readability – `let` doesn’t nest, whereas a `match` does.

#### Example

```rust
enum Wrapper {
    Data(i32),
}

let wrapper = Wrapper::Data(42);

let data = match wrapper {
    Wrapper::Data(i) => i,
};
```

The correct use would be:

```rust
enum Wrapper {
    Data(i32),
}

let wrapper = Wrapper::Data(42);
let Wrapper::Data(data) = wrapper;
```

---

### `inherent_to_string`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.38.0 |

#### What it does

Checks for the definition of inherent methods with a signature of `to_string(&self) -> String`.

#### Why is this bad?

This method is also implicitly defined if a type implements the `Display` trait. As the functionality of `Display` is much more versatile, it should be preferred.

#### Example

```rust
pub struct A;

impl A {
    pub fn to_string(&self) -> String {
        "I am A".to_string()
    }
}
```

Use instead:

```rust
use std::fmt;

pub struct A;

impl fmt::Display for A {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "I am A")
    }
}
```

---

### `init_numbered_fields`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.59.0 |

#### What it does

Checks for tuple structs initialized with field syntax.
It will however not lint if a base initializer is present.
The lint will also ignore code in macros.

#### Why is this bad?

This may be confusing to the uninitiated and adds no
benefit as opposed to tuple initializers

#### Example

```rust
struct TupleStruct(u8, u16);

let _ = TupleStruct {
    0: 1,
    1: 23,
};

// should be written as
let base = TupleStruct(1, 23);

// This is OK however
let _ = TupleStruct { 0: 42, ..base };
```

---

### `into_iter_on_ref`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.32.0 |

#### What it does

Checks for `into_iter` calls on references which should be replaced by `iter`
or `iter_mut`.

#### Why is this bad?

Readability. Calling `into_iter` on a reference will not move out its
content into the resulting iterator, which is confusing. It is better just call `iter` or
`iter_mut` directly.

#### Example

```rust
(&vec).into_iter();
```

Use instead:

```rust
(&vec).iter();
```

---

### `io_other_error`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.87.0 |

This lint warns on calling `io::Error::new(..)` with a kind of
`io::ErrorKind::Other`.

#### Why is this bad?

Since Rust 1.74, there’s the `io::Error::other(_)` shortcut.

#### Example

```rust
use std::io;
let _ = io::Error::new(io::ErrorKind::Other, "bad".to_string());
```

Use instead:

```rust
let _ = std::io::Error::other("bad".to_string());
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `is_digit_ascii_radix`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

Finds usages of [char::is_digit](https://doc.rust-lang.org/stable/std/primitive.char.html#method.is_digit) that
can be replaced with [is_ascii_digit](https://doc.rust-lang.org/stable/std/primitive.char.html#method.is_ascii_digit) or
[is_ascii_hexdigit](https://doc.rust-lang.org/stable/std/primitive.char.html#method.is_ascii_hexdigit).

#### Why is this bad?

`is_digit(..)` is slower and requires specifying the radix.

#### Example

```rust
let c: char = '6';
c.is_digit(10);
c.is_digit(16);
```

Use instead:

```rust
let c: char = '6';
c.is_ascii_digit();
c.is_ascii_hexdigit();
```

---

### `items_after_test_module`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.71.0 |

#### What it does

Triggers if an item is declared after the testing module marked with `#[cfg(test)]`.

#### Why is this bad?

Having items declared after the testing module is confusing and may lead to bad test coverage.

#### Example

```rust
#[cfg(test)]
mod tests {
    // [...]
}

fn my_function() {
    // [...]
}
```

Use instead:

```rust
fn my_function() {
    // [...]
}

#[cfg(test)]
mod tests {
    // [...]
}
```

---

### `iter_cloned_collect`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the use of `.cloned().collect()` on slice to
create a `Vec`.

#### Why is this bad?

`.to_vec()` is clearer

#### Example

```rust
let s = [1, 2, 3, 4, 5];
let s2: Vec<isize> = s[..].iter().cloned().collect();
```

The better use would be:

```rust
let s = [1, 2, 3, 4, 5];
let s2: Vec<isize> = s.to_vec();
```

---

### `iter_next_slice`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.46.0 |

#### What it does

Checks for usage of `iter().next()` on a Slice or an Array

#### Why is this bad?

These can be shortened into `.get()`

#### Example

```rust
a[2..].iter().next();
b.iter().next();
```

should be written as:

```rust
a.get(2);
b.get(0);
```

---

### `iter_nth`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `.iter().nth()`/`.iter_mut().nth()` on standard library types that have
equivalent `.get()`/`.get_mut()` methods.

#### Why is this bad?

`.get()` and `.get_mut()` are equivalent but more concise.

#### Example

```rust
let some_vec = vec![0, 1, 2, 3];
let bad_vec = some_vec.iter().nth(3);
let bad_slice = &some_vec[..].iter().nth(3);
```

The correct use would be:

```rust
let some_vec = vec![0, 1, 2, 3];
let bad_vec = some_vec.get(3);
let bad_slice = &some_vec[..].get(3);
```

---

### `iter_nth_zero`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.42.0 |

#### What it does

Checks for the use of `iter.nth(0)`.

#### Why is this bad?

`iter.next()` is equivalent to
`iter.nth(0)`, as they both consume the next element,
but is more readable.

#### Example

```rust
let x = s.iter().nth(0);
```

Use instead:

```rust
let x = s.iter().next();
```

---

### `iter_skip_next`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `.skip(x).next()` on iterators.

#### Why is this bad?

`.nth(x)` is cleaner

#### Example

```rust
let some_vec = vec![0, 1, 2, 3];
let bad_vec = some_vec.iter().skip(3).next();
let bad_slice = &some_vec[..].iter().skip(3).next();
```

The correct use would be:

```rust
let some_vec = vec![0, 1, 2, 3];
let bad_vec = some_vec.iter().nth(3);
let bad_slice = &some_vec[..].iter().nth(3);
```

---

### `just_underscores_and_digits`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks if you have variables whose name consists of just
underscores and digits.

#### Why is this bad?

It’s hard to memorize what a variable means without a
descriptive name.

#### Example

```rust
let _1 = 1;
let ___1 = 1;
let __1___2 = 11;
```

---

### `legacy_numeric_constants`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.79.0 |

#### What it does

Checks for usage of `<integer>::max_value()`, `std::<integer>::MAX`,
`std::<float>::EPSILON`, etc.

#### Why is this bad?

All of these have been superseded by the associated constants on their respective types,
such as `i128::MAX`. These legacy items may be deprecated in a future version of rust.

#### Example

```rust
let eps = std::f32::EPSILON;
```

Use instead:

```rust
let eps = f32::EPSILON;
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `len_without_is_empty`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for items that implement `.len()` but not
`.is_empty()`.

#### Why is this bad?

It is good custom to have both methods, because for
some data structures, asking about the length will be a costly operation,
whereas `.is_empty()` can usually answer in constant time. Also it used to
lead to false positives on the [len_zero](#len_zero) lint – currently that
lint will ignore such entities.

#### Example

```rust
impl X {
    pub fn len(&self) -> usize {
        ..
    }
}
```

---

### `len_zero`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for getting the length of something via `.len()`
just to compare to zero, and suggests using `.is_empty()` where applicable.

#### Why is this bad?

Some structures can answer `.is_empty()` much faster
than calculating their length. So it is good to get into the habit of using
`.is_empty()`, and having it is cheap.
Besides, it makes the intent clearer than a manual comparison in some contexts.

#### Example

```rust
if x.len() == 0 {
    ..
}
if y.len() != 0 {
    ..
}
```

instead use

```rust
if x.is_empty() {
    ..
}
if !y.is_empty() {
    ..
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `let_and_return`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `let`-bindings, which are subsequently
returned.

#### Why is this bad?

It is just extraneous code. Remove it to make your code
more rusty.

#### Known problems

In the case of some temporaries, e.g. locks, eliding the variable binding could lead
to deadlocks. See [this issue](https://github.com/rust-lang/rust/issues/37612).
This could become relevant if the code is later changed to use the code that would have been
bound without first assigning it to a let-binding.

#### Example

```rust
fn foo() -> String {
    let x = String::new();
    x
}
```

instead, use

```rust
fn foo() -> String {
    String::new()
}
```

---

### `let_unit_value`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for binding a unit value.

#### Why is this bad?

A unit value cannot usefully be used anywhere. So
binding one is kind of pointless.

#### Example

```rust
let x = {
    1;
};
```

---

### `main_recursion`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.38.0 |

#### What it does

Checks for recursion using the entrypoint.

#### Why is this bad?

Apart from special setups (which we could detect following attributes like #![no_std]),
recursing into main() seems like an unintuitive anti-pattern we should be able to detect.

#### Example

```rust
fn main() {
    main();
}
```

---

### `manual_async_fn`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.45.0 |

#### What it does

It checks for manual implementations of `async` functions.

#### Why is this bad?

It’s more idiomatic to use the dedicated syntax.

#### Example

```rust
use std::future::Future;

fn foo() -> impl Future<Output = i32> { async { 42 } }
```

Use instead:

```rust
async fn foo() -> i32 { 42 }
```

---

### `manual_bits`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.60.0 |

#### What it does

Checks for usage of `size_of::<T>() * 8` when
`T::BITS` is available.

#### Why is this bad?

Can be written as the shorter `T::BITS`.

#### Example

```rust
size_of::<usize>() * 8;
```

Use instead:

```rust
usize::BITS as usize;
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_dangling_ptr`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.88.0 |

#### What it does

Checks for casts of small constant literals or `mem::align_of` results to raw pointers.

#### Why is this bad?

This creates a dangling pointer and is better expressed as
{`std`, `core`}`::ptr::`{`dangling`, `dangling_mut`}.

#### Example

```rust
let ptr = 4 as *const u32;
let aligned = std::mem::align_of::<u32>() as *const u32;
let mut_ptr: *mut i64 = 8 as *mut _;
```

Use instead:

```rust
let ptr = std::ptr::dangling::<u32>();
let aligned = std::ptr::dangling::<u32>();
let mut_ptr: *mut i64 = std::ptr::dangling_mut();
```

---

### `manual_is_ascii_check`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.67.0 |

#### What it does

Suggests to use dedicated built-in methods,
`is_ascii_(lowercase|uppercase|digit|hexdigit)` for checking on corresponding
ascii range

#### Why is this bad?

Using the built-in functions is more readable and makes it
clear that it’s not a specific subset of characters, but all
ASCII (lowercase|uppercase|digit|hexdigit) characters.

#### Example

```rust
fn main() {
    assert!(matches!('x', 'a'..='z'));
    assert!(matches!(b'X', b'A'..=b'Z'));
    assert!(matches!('2', '0'..='9'));
    assert!(matches!('x', 'A'..='Z' | 'a'..='z'));
    assert!(matches!('C', '0'..='9' | 'a'..='f' | 'A'..='F'));

    ('0'..='9').contains(&'0');
    ('a'..='z').contains(&'a');
    ('A'..='Z').contains(&'A');
}
```

Use instead:

```rust
fn main() {
    assert!('x'.is_ascii_lowercase());
    assert!(b'X'.is_ascii_uppercase());
    assert!('2'.is_ascii_digit());
    assert!('x'.is_ascii_alphabetic());
    assert!('C'.is_ascii_hexdigit());

    '0'.is_ascii_digit();
    'a'.is_ascii_lowercase();
    'A'.is_ascii_uppercase();
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_is_finite`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.73.0 |

#### What it does

Checks for manual `is_finite` reimplementations
(i.e., `x != <float>::INFINITY && x != <float>::NEG_INFINITY`).

#### Why is this bad?

The method `is_finite` is shorter and more readable.

#### Example

```rust
if x != f32::INFINITY && x != f32::NEG_INFINITY {}
if x.abs() < f32::INFINITY {}
```

Use instead:

```rust
if x.is_finite() {}
if x.is_finite() {}
```

---

### `manual_is_infinite`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.73.0 |

#### What it does

Checks for manual `is_infinite` reimplementations
(i.e., `x == <float>::INFINITY || x == <float>::NEG_INFINITY`).

#### Why is this bad?

The method `is_infinite` is shorter and more readable.

#### Example

```rust
if x == f32::INFINITY || x == f32::NEG_INFINITY {}
```

Use instead:

```rust
if x.is_infinite() {}
```

---

### `manual_map`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Checks for usage of `match` which could be implemented using `map`

#### Why is this bad?

Using the `map` method is clearer and more concise.

#### Example

```rust
match Some(0) {
    Some(x) => Some(x + 1),
    None => None,
};
```

Use instead:

```rust
Some(0).map(|x| x + 1);
```

---

### `manual_next_back`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.71.0 |

#### What it does

Checks for `.rev().next()` on a `DoubleEndedIterator`

#### Why is this bad?

`.next_back()` is cleaner.

#### Example

```rust
foo.iter().rev().next();
```

Use instead:

```rust
foo.iter().next_back();
```

---

### `manual_non_exhaustive`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.45.0 |

#### What it does

Checks for manual implementations of the non-exhaustive pattern.

#### Why is this bad?

Using the #[non_exhaustive] attribute expresses better the intent
and allows possible optimizations when applied to enums.

#### Example

```rust
struct S {
    pub a: i32,
    pub b: i32,
    _c: (),
}

enum E {
    A,
    B,
    #[doc(hidden)]
    _C,
}

struct T(pub i32, pub i32, ());
```

Use instead:

```rust
#[non_exhaustive]
struct S {
    pub a: i32,
    pub b: i32,
}

#[non_exhaustive]
enum E {
    A,
    B,
}

#[non_exhaustive]
struct T(pub i32, pub i32);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_ok_or`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Finds patterns that reimplement `Option::ok_or`.

#### Why is this bad?

Concise code helps focusing on behavior instead of boilerplate.

#### Examples

```rust
let foo: Option<i32> = None;
foo.map_or(Err("error"), |v| Ok(v));
```

Use instead:

```rust
let foo: Option<i32> = None;
foo.ok_or("error");
```

---

### `manual_pattern_char_comparison`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.81.0 |

#### What it does

Checks for manual `char` comparison in string patterns

#### Why is this bad?

This can be written more concisely using a `char` or an array of `char`.
This is more readable and more optimized when comparing to only one `char`.

#### Example

```rust
"Hello World!".trim_end_matches(|c| c == '.' || c == ',' || c == '!' || c == '?');
```

Use instead:

```rust
"Hello World!".trim_end_matches(['.', ',', '!', '?']);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_range_contains`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Checks for expressions like `x >= 3 && x < 8` that could
be more readably expressed as `(3..8).contains(x)`.

#### Why is this bad?

`contains` expresses the intent better and has less
failure modes (such as fencepost errors or using `||` instead of `&&`).

#### Example

```rust
// given
let x = 6;

assert!(x >= 3 && x < 8);
```

Use instead:

```rust
assert!((3..8).contains(&x));
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_repeat_n`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Checks for `repeat().take()` that can be replaced with `repeat_n()`.

#### Why is this bad?

Using `repeat_n()` is more concise and clearer. Also, `repeat_n()` is sometimes faster than `repeat().take()` when the type of the element is non-trivial to clone because the original value can be reused for the last `.next()` call rather than always cloning.

#### Example

```rust
let _ = std::iter::repeat(10).take(3);
```

Use instead:

```rust
let _ = std::iter::repeat_n(10, 3);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_rotate`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.81.0 |

#### What it does

It detects manual bit rotations that could be rewritten using standard
functions `rotate_left` or `rotate_right`.

#### Why is this bad?

Calling the function better conveys the intent.

#### Known issues

Currently, the lint only catches shifts by constant amount.

#### Example

```rust
let x = 12345678_u32;
let _ = (x >> 8) | (x << 24);
```

Use instead:

```rust
let x = 12345678_u32;
let _ = x.rotate_right(8);
```

---

### `manual_saturating_arithmetic`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.39.0 |

#### What it does

Checks for `.checked_add/sub(x).unwrap_or(MAX/MIN)`.

#### Why is this bad?

These can be written simply with `saturating_add/sub` methods.

#### Example

```rust
let add = x.checked_add(y).unwrap_or(u32::MAX);
let sub = x.checked_sub(y).unwrap_or(u32::MIN);
```

can be written using dedicated methods for saturating addition/subtraction as:

```rust
let add = x.saturating_add(y);
let sub = x.saturating_sub(y);
```

---

### `manual_slice_fill`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.86.0 |

#### What it does

Checks for manually filling a slice with a value.

#### Why is this bad?

Using the `fill` method is more idiomatic and concise.

#### Example

```rust
let mut some_slice = [1, 2, 3, 4, 5];
for i in 0..some_slice.len() {
    some_slice[i] = 0;
}
```

Use instead:

```rust
let mut some_slice = [1, 2, 3, 4, 5];
some_slice.fill(0);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_while_let_some`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.71.0 |

#### What it does

Looks for loops that check for emptiness of a `Vec` in the condition and pop an element
in the body as a separate operation.

#### Why is this bad?

Such loops can be written in a more idiomatic way by using a while-let loop and directly
pattern matching on the return value of `Vec::pop()`.

#### Example

```rust
let mut numbers = vec![1, 2, 3, 4, 5];
while !numbers.is_empty() {
    let number = numbers.pop().unwrap();
    // use `number`
}
```

Use instead:

```rust
let mut numbers = vec![1, 2, 3, 4, 5];
while let Some(number) = numbers.pop() {
    // use `number`
}
```

---

### `map_clone`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `map(|x| x.clone())` or
dereferencing closures for `Copy` types, on `Iterator` or `Option`,
and suggests `cloned()` or `copied()` instead

#### Why is this bad?

Readability, this can be written more concisely

#### Example

```rust
let x = vec![42, 43];
let y = x.iter();
let z = y.map(|i| *i);
```

The correct use would be:

```rust
let x = vec![42, 43];
let y = x.iter();
let z = y.cloned();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `map_collect_result_unit`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Checks for usage of `_.map(_).collect::<Result<(), _>()`.

#### Why is this bad?

Using `try_for_each` instead is more readable and idiomatic.

#### Example

```rust
(0..3).map(|t| Err(t)).collect::<Result<(), _>>();
```

Use instead:

```rust
(0..3).try_for_each(|t| Err(t));
```

---

### `match_like_matches_macro`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.47.0 |

#### What it does

Checks for `match`  or `if let` expressions producing a
`bool` that could be written using `matches!`

#### Why is this bad?

Readability and needless complexity.

#### Known problems

This lint falsely triggers, if there are arms with
`cfg` attributes that remove an arm evaluating to `false`.

#### Example

```rust
let x = Some(5);

let a = match x {
    Some(0) => true,
    _ => false,
};

let a = if let Some(0) = x {
    true
} else {
    false
};
```

Use instead:

```rust
let x = Some(5);
let a = matches!(x, Some(0));
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `match_overlapping_arm`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for overlapping match arms.

#### Why is this bad?

It is likely to be an error and if not, makes the code
less obvious.

#### Example

```rust
let x = 5;
match x {
    1..=10 => println!("1 ... 10"),
    5..=15 => println!("5 ... 15"),
    _ => (),
}
```

---

### `match_ref_pats`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for matches where all arms match a reference,
suggesting to remove the reference and deref the matched expression
instead. It also checks for `if let &foo = bar` blocks.

#### Why is this bad?

It just makes the code less readable. That reference
destructuring adds nothing to the code.

#### Example

```rust
match x {
    &A(ref y) => foo(y),
    &B => bar(),
    _ => frob(&x),
}
```

Use instead:

```rust
match *x {
    A(ref y) => foo(y),
    B => bar(),
    _ => frob(x),
}
```

---

### `match_result_ok`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.57.0 |

#### What it does

Checks for unnecessary `ok()` in `while let`.

#### Why is this bad?

Calling `ok()` in `while let` is unnecessary, instead match
on `Ok(pat)`

#### Example

```rust
while let Some(value) = iter.next().ok() {
    vec.push(value)
}

if let Some(value) = iter.next().ok() {
    vec.push(value)
}
```

Use instead:

```rust
while let Ok(value) = iter.next() {
    vec.push(value)
}

if let Ok(value) = iter.next() {
       vec.push(value)
}
```

#### Past names

- if_let_some_result

---

### `mem_replace_option_with_none`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.31.0 |

#### What it does

Checks for `mem::replace()` on an `Option` with
`None`.

#### Why is this bad?

`Option` already has the method `take()` for
taking its current value (Some(..) or None) and replacing it with
`None`.

#### Example

```rust
use std::mem;

let mut an_option = Some(0);
let replaced = mem::replace(&mut an_option, None);
```

Is better expressed with:

```rust
let mut an_option = Some(0);
let taken = an_option.take();
```

---

### `mem_replace_option_with_some`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.87.0 |

#### What it does

Checks for `mem::replace()` on an `Option` with `Some(…)`.

#### Why is this bad?

`Option` already has the method `replace()` for
taking its current value (Some(…) or None) and replacing it with
`Some(…)`.

#### Example

```rust
let mut an_option = Some(0);
let replaced = std::mem::replace(&mut an_option, Some(1));
```

Is better expressed with:

```rust
let mut an_option = Some(0);
let taken = an_option.replace(1);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `mem_replace_with_default`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.42.0 |

#### What it does

Checks for `std::mem::replace` on a value of type
`T` with `T::default()`.

#### Why is this bad?

`std::mem` module already has the method `take` to
take the current value and replace it with the default value of that type.

#### Example

```rust
let mut text = String::from("foo");
let replaced = std::mem::replace(&mut text, String::default());
```

Is better expressed with:

```rust
let mut text = String::from("foo");
let taken = std::mem::take(&mut text);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `missing_enforced_import_renames`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.55.0 |

#### What it does

Checks for imports that do not rename the item as specified
in the `enforced-import-renames` config option.

Note: Even though this lint is warn-by-default, it will only trigger if
import renames are defined in the `clippy.toml` file.

#### Why is this bad?

Consistency is important; if a project has defined import renames, then they should be
followed. More practically, some item names are too vague outside of their defining scope,
in which case this can enforce a more meaningful naming.

#### Example

An example clippy.toml configuration:

```toml
enforced-import-renames = [
    { path = "serde_json::Value", rename = "JsonValue" },
]
```

```rust
use serde_json::Value;
```

Use instead:

```rust
use serde_json::Value as JsonValue;
```

#### Configuration

- `enforced-import-renames`:  The list of imports to always rename, a fully qualified path followed by the rename.

(default: `[]`)

---

### `missing_safety_doc`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.39.0 |

#### What it does

Checks for the doc comments of publicly visible
unsafe functions and warns if there is no `# Safety` section.

#### Why is this bad?

Unsafe functions should document their safety
preconditions, so that users can be sure they are using them safely.

#### Examples

```rust
/// This function should really be documented
pub unsafe fn start_apocalypse(u: &mut Universe) {
    unimplemented!();
}
```

At least write a line about safety:

```rust
/// # Safety
///
/// This function should not be called before the horsemen are ready.
pub unsafe fn start_apocalypse(u: &mut Universe) {
    unimplemented!();
}
```

#### Configuration

- `check-private-items`:  Whether to also run the listed lints on private items.

(default: `false`)

---

### `mixed_attributes_style`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.78.0 |

#### What it does

Checks for items that have the same kind of attributes with mixed styles (inner/outer).

#### Why is this bad?

Having both style of said attributes makes it more complicated to read code.

#### Known problems

This lint currently has false-negatives when mixing same attributes
but they have different path symbols, for example:

```rust
#[custom_attribute]
pub fn foo() {
    #![my_crate::custom_attribute]
}
```

#### Example

```rust
#[cfg(linux)]
pub fn foo() {
    #![cfg(windows)]
}
```

Use instead:

```rust
#[cfg(linux)]
#[cfg(windows)]
pub fn foo() {
}
```

---

### `mixed_case_hex_literals`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns on hexadecimal literals with mixed-case letter
digits.

#### Why is this bad?

It looks confusing.

#### Example

```rust
0x1a9BAcD
```

Use instead:

```rust
0x1A9BACD
```

---

### `module_inception`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for modules that have the same name as their
parent module

#### Why is this bad?

A typical beginner mistake is to have `mod foo;` and
again `mod foo { .. }` in `foo.rs`.
The expectation is that items inside the inner `mod foo { .. }` are then
available
through `foo::x`, but they are only available through
`foo::foo::x`.
If this is done on purpose, it would be better to choose a more
representative module name.

#### Example

```rust
// lib.rs
mod foo;
// foo.rs
mod foo {
    ...
}
```

#### Configuration

- `allow-private-module-inception`:  Whether to allow module inception if it’s not public.

(default: `false`)

---

### `multiple_bound_locations`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.78.0 |

#### What it does

Check if a generic is defined both in the bound predicate and in the `where` clause.

#### Why is this bad?

It can be confusing for developers when seeing bounds for a generic in multiple places.

#### Example

```rust
fn ty<F: std::fmt::Debug>(a: F)
where
    F: Sized,
{}
```

Use instead:

```rust
fn ty<F>(a: F)
where
    F: Sized + std::fmt::Debug,
{}
```

---

### `must_use_unit`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.40.0 |

#### What it does

Checks for a `#[must_use]` attribute on
unit-returning functions and methods.

#### Why is this bad?

Unit values are useless. The attribute is likely
a remnant of a refactoring that removed the return type.

#### Examples

```rust
#[must_use]
fn useless() { }
```

---

### `mut_mutex_lock`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.49.0 |

#### What it does

Checks for `&mut Mutex::lock` calls

#### Why is this bad?

`Mutex::lock` is less efficient than
calling `Mutex::get_mut`. In addition you also have a statically
guarantee that the mutex isn’t locked, instead of just a runtime
guarantee.

#### Example

```rust
use std::sync::{Arc, Mutex};

let mut value_rc = Arc::new(Mutex::new(42_u8));
let value_mutex = Arc::get_mut(&mut value_rc).unwrap();

let mut value = value_mutex.lock().unwrap();
*value += 1;
```

Use instead:

```rust
use std::sync::{Arc, Mutex};

let mut value_rc = Arc::new(Mutex::new(42_u8));
let value_mutex = Arc::get_mut(&mut value_rc).unwrap();

let value = value_mutex.get_mut().unwrap();
*value += 1;
```

---

### `needless_borrow`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for address of operations (`&`) that are going to
be dereferenced immediately by the compiler.

#### Why is this bad?

Suggests that the receiver of the expression borrows
the expression.

#### Known problems

The lint cannot tell when the implementation of a trait
for `&T` and `T` do different things. Removing a borrow
in such a case can change the semantics of the code.

#### Example

```rust
fn fun(_a: &i32) {}

let x: &i32 = &&&&&&5;
fun(&x);
```

Use instead:

```rust
let x: &i32 = &5;
fun(x);
```

#### Past names

- ref_in_deref

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `needless_borrows_for_generic_args`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.74.0 |

#### What it does

Checks for borrow operations (`&`) that are used as a generic argument to a
function when the borrowed value could be used.

#### Why is this bad?

Suggests that the receiver of the expression borrows
the expression.

#### Known problems

The lint cannot tell when the implementation of a trait
for `&T` and `T` do different things. Removing a borrow
in such a case can change the semantics of the code.

#### Example

```rust
fn f(_: impl AsRef<str>) {}

let x = "foo";
f(&x);
```

Use instead:

```rust
fn f(_: impl AsRef<str>) {}

let x = "foo";
f(x);
```

---

### `needless_doctest_main`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks for `fn main() { .. }` in doctests

#### Why is this bad?

The test can be shorter (and likely more readable)
if the `fn main()` is left implicit.

#### Examples

```rust
/// An example of a doctest with a `main()` function
///
/// # Examples
///
/// ```
/// fn main() {
///     // this needs not be in an `fn`
/// }
/// ```
fn needless_main() {
    unimplemented!();
}
```

---

### `needless_else`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for empty `else` branches.

#### Why is this bad?

An empty else branch does nothing and can be removed.

#### Example

```rust
if check() {
    println!("Check successful!");
} else {
}
```

Use instead:

```rust
if check() {
    println!("Check successful!");
}
```

---

### `needless_late_init`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.59.0 |

#### What it does

Checks for late initializations that can be replaced by a `let` statement
with an initializer.

#### Why is this bad?

Assigning in the `let` statement is less repetitive.

#### Example

```rust
let a;
a = 1;

let b;
match 3 {
    0 => b = "zero",
    1 => b = "one",
    _ => b = "many",
}

let c;
if true {
    c = 1;
} else {
    c = -1;
}
```

Use instead:

```rust
let a = 1;

let b = match 3 {
    0 => "zero",
    1 => "one",
    _ => "many",
};

let c = if true {
    1
} else {
    -1
};
```

---

### `needless_parens_on_range_literals`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.63.0 |

#### What it does

The lint checks for parenthesis on literals in range statements that are
superfluous.

#### Why is this bad?

Having superfluous parenthesis makes the code less readable
overhead when reading.

#### Example

```rust
for i in (0)..10 {
  println!("{i}");
}
```

Use instead:

```rust
for i in 0..10 {
  println!("{i}");
}
```

---

### `needless_pub_self`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for usage of `pub(self)` and `pub(in self)`.

#### Why is this bad?

It’s unnecessary, omitting the `pub` entirely will give the same results.

#### Example

```rust
pub(self) type OptBox<T> = Option<Box<T>>;
```

Use instead:

```rust
type OptBox<T> = Option<Box<T>>;
```

---

### `needless_range_loop`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for looping over the range of `0..len` of some
collection just to get the values by index.

#### Why is this bad?

Just iterating the collection itself makes the intent
more clear and is probably faster because it eliminates
the bounds check that is done when indexing.

#### Example

```rust
let vec = vec!['a', 'b', 'c'];
for i in 0..vec.len() {
    println!("{}", vec[i]);
}
```

Use instead:

```rust
let vec = vec!['a', 'b', 'c'];
for i in vec {
    println!("{}", i);
}
```

---

### `needless_return`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for return statements at the end of a block.

#### Why is this bad?

Removing the `return` and semicolon will make the code
more rusty.

#### Example

```rust
fn foo(x: usize) -> usize {
    return x;
}
```

simplify to

```rust
fn foo(x: usize) -> usize {
    x
}
```

---

### `needless_return_with_question_mark`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.73.0 |

#### What it does

Checks for return statements on `Err` paired with the `?` operator.

#### Why is this bad?

The `return` is unnecessary.

Returns may be used to add attributes to the return expression. Return
statements with attributes are therefore be accepted by this lint.

#### Example

```rust
fn foo(x: usize) -> Result<(), Box<dyn Error>> {
    if x == 0 {
        return Err(...)?;
    }
    Ok(())
}
```

simplify to

```rust
fn foo(x: usize) -> Result<(), Box<dyn Error>> {
    if x == 0 {
        Err(...)?;
    }
    Ok(())
}
```

if paired with `try_err`, use instead:

```rust
fn foo(x: usize) -> Result<(), Box<dyn Error>> {
    if x == 0 {
        return Err(...);
    }
    Ok(())
}
```

---

### `neg_multiply`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for multiplication by -1 as a form of negation.

#### Why is this bad?

It’s more readable to just negate.

#### Example

```rust
let a = x * -1;
```

Use instead:

```rust
let a = -x;
```

---

### `new_ret_no_self`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `new` not returning a type that contains `Self`.

#### Why is this bad?

As a convention, `new` methods are used to make a new
instance of a type.

#### Example

In an impl block:

```rust
impl Foo {
    fn new() -> NotAFoo {
    }
}
```

```rust
struct Bar(Foo);
impl Foo {
    // Bad. The type name must contain `Self`
    fn new() -> Bar {
    }
}
```

```rust
impl Foo {
    // Good. Return type contains `Self`
    fn new() -> Result<Foo, FooError> {
    }
}
```

Or in a trait definition:

```rust
pub trait Trait {
    // Bad. The type name must contain `Self`
    fn new();
}
```

```rust
pub trait Trait {
    // Good. Return type contains `Self`
    fn new() -> Self;
}
```

---

### `new_without_default`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for public types with a `pub fn new() -> Self` method and no
implementation of
[Default](https://doc.rust-lang.org/std/default/trait.Default.html).

#### Why is this bad?

The user might expect to be able to use
[Default](https://doc.rust-lang.org/std/default/trait.Default.html) as the
type can be constructed without arguments.

#### Example

```rust
pub struct Foo(Bar);

impl Foo {
    pub fn new() -> Self {
        Foo(Bar::new())
    }
}
```

To fix the lint, add a `Default` implementation that delegates to `new`:

```rust
pub struct Foo(Bar);

impl Default for Foo {
    fn default() -> Self {
        Foo::new()
    }
}
```

#### Past names

- new_without_default_derive

---

### `non_minimal_cfg`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.71.0 |

#### What it does

Checks for `any` and `all` combinators in `cfg` with only one condition.

#### Why is this bad?

If there is only one condition, no need to wrap it into `any` or `all` combinators.

#### Example

```rust
#[cfg(any(unix))]
pub struct Bar;
```

Use instead:

```rust
#[cfg(unix)]
pub struct Bar;
```

---

### `obfuscated_if_else`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Checks for unnecessary method chains that can be simplified into `if .. else ..`.

#### Why is this bad?

This can be written more clearly with `if .. else ..`

#### Limitations

This lint currently only looks for usages of
`.{then, then_some}(..).{unwrap_or, unwrap_or_else, unwrap_or_default}(..)`, but will be expanded
to account for similar patterns.

#### Example

```rust
let x = true;
x.then_some("a").unwrap_or("b");
```

Use instead:

```rust
let x = true;
if x { "a" } else { "b" };
```

---

### `ok_expect`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `ok().expect(..)`.

Note: This lint only triggers for code marked compatible
with versions of the compiler older than Rust 1.82.0.

#### Why is this bad?

Because you usually call `expect()` on the `Result`
directly to get a better error message.

#### Example

```rust
x.ok().expect("why did I do this again?");
```

Use instead:

```rust
x.expect("why did I do this again?");
```

---

### `op_ref`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for arguments to `==` which have their address
taken to satisfy a bound
and suggests to dereference the other argument instead

#### Why is this bad?

It is more idiomatic to dereference the other argument.

#### Example

```rust
&x == y
```

Use instead:

```rust
x == *y
```

---

### `option_map_or_none`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `_.map_or(None, _)`.

#### Why is this bad?

Readability, this can be written more concisely as
`_.and_then(_)`.

#### Known problems

The order of the arguments is not in execution order.

#### Example

```rust
opt.map_or(None, |a| Some(a + 1));
```

Use instead:

```rust
opt.and_then(|a| Some(a + 1));
```

---

### `owned_cow`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.87.0 |

#### What it does

Detects needlessly owned `Cow` types.

#### Why is this bad?

The borrowed types are usually more flexible, in that e.g. a
`Cow<'_, str>` can accept both `&str` and `String` while
`Cow<'_, String>` can only accept `&String` and `String`. In
particular, `&str` is more general, because it allows for string
literals while `&String` can only be borrowed from a heap-owned
`String`).

#### Known Problems

The lint does not check for usage of the type. There may be external
interfaces that require the use of an owned type.

At least the `CString` type also has a different API than `CStr`: The
former has an `as_bytes` method which the latter calls `to_bytes`.
There is no guarantee that other types won’t gain additional methods
leading to a similar mismatch.

In addition, the lint only checks for the known problematic types
`String`, `Vec<_>`, `CString`, `OsString` and `PathBuf`. Custom types
that implement `ToOwned` will not be detected.

#### Example

```rust
let wrogn: std::borrow::Cow<'_, Vec<u8>>;
```

Use instead:

```rust
let right: std::borrow::Cow<'_, [u8]>;
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `partialeq_to_none`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.65.0 |

#### What it does

Checks for binary comparisons to a literal `Option::None`.

#### Why is this bad?

A programmer checking if some `foo` is `None` via a comparison `foo == None`
is usually inspired from other programming languages (e.g. `foo is None`
in Python).
Checking if a value of type `Option<T>` is (not) equal to `None` in that
way relies on `T: PartialEq` to do the comparison, which is unneeded.

#### Example

```rust
fn foo(f: Option<u32>) -> &'static str {
    if f != None { "yay" } else { "nay" }
}
```

Use instead:

```rust
fn foo(f: Option<u32>) -> &'static str {
    if f.is_some() { "yay" } else { "nay" }
}
```

---

### `print_literal`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint warns about the use of literals as `print!`/`println!` args.

#### Why is this bad?

Using literals as `println!` args is inefficient
(c.f., https://github.com/matthiaskrgr/rust-str-bench) and unnecessary
(i.e., just put the literal in the format string)

#### Example

```rust
println!("{}", "foo");
```

use the literal without formatting:

```rust
println!("foo");
```

---

### `print_with_newline`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint warns when you use `print!()` with a format
string that ends in a newline.

#### Why is this bad?

You should use `println!()` instead, which appends the
newline.

#### Example

```rust
print!("Hello {}!\n", name);
```

use println!() instead

```rust
println!("Hello {}!", name);
```

---

### `println_empty_string`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint warns when you use `println!("")` to
print a newline.

#### Why is this bad?

You should use `println!()`, which is simpler.

#### Example

```rust
println!("");
```

Use instead:

```rust
println!();
```

---

### `ptr_arg`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint checks for function arguments of type `&String`, `&Vec`,
`&PathBuf`, and `Cow<_>`. It will also suggest you replace `.clone()` calls
with the appropriate `.to_owned()`/`to_string()` calls.

#### Why is this bad?

Requiring the argument to be of the specific type
makes the function less useful for no benefit; slices in the form of `&[T]`
or `&str` usually suffice and can be obtained from other types, too.

#### Known problems

There may be `fn(&Vec)`-typed references pointing to your function.
If you have them, you will get a compiler error after applying this lint’s
suggestions. You then have the choice to undo your changes or change the
type of the reference.

Note that if the function is part of your public interface, there may be
other crates referencing it, of which you may not be aware. Carefully
deprecate the function before applying the lint suggestions in this case.

#### Example

```rust
fn foo(&Vec<u32>) { .. }
```

Use instead:

```rust
fn foo(&[u32]) { .. }
```

---

### `ptr_eq`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Use `std::ptr::eq` when applicable

#### Why is this bad?

`ptr::eq` can be used to compare `&T` references
(which coerce to `*const T` implicitly) by their address rather than
comparing the values they point to.

#### Example

```rust
let a = &[1, 2, 3];
let b = &[1, 2, 3];

assert!(a as *const _ as usize == b as *const _ as usize);
```

Use instead:

```rust
let a = &[1, 2, 3];
let b = &[1, 2, 3];

assert!(std::ptr::eq(a, b));
```

---

### `question_mark`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for expressions that could be replaced by the `?` operator.

#### Why is this bad?

Using the `?` operator is shorter and more idiomatic.

#### Example

```rust
if option.is_none() {
    return None;
}
```

Could be written:

```rust
option?;
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `redundant_closure`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for closures which just call another function where
the function can be called directly. `unsafe` functions, calls where types
get adjusted or where the callee is marked `#[track_caller]` are ignored.

#### Why is this bad?

Needlessly creating a closure adds code for no benefit
and gives the optimizer more work.

#### Example

```rust
xs.map(|x| foo(x))
```

Use instead:

```rust
// where `foo(_)` is a plain function that takes the exact argument type of `x`.
xs.map(foo)
```

---

### `redundant_field_names`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for fields in struct literals where shorthands
could be used.

#### Why is this bad?

If the field and variable names are the same,
the field name is redundant.

#### Example

```rust
let bar: u8 = 123;

struct Foo {
    bar: u8,
}

let foo = Foo { bar: bar };
```

the last line can be simplified to

```rust
let foo = Foo { bar };
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `redundant_pattern`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for patterns in the form `name @ _`.

#### Why is this bad?

It’s almost always more readable to just use direct
bindings.

#### Example

```rust
match v {
    Some(x) => (),
    y @ _ => (),
}
```

Use instead:

```rust
match v {
    Some(x) => (),
    y => (),
}
```

---

### `redundant_pattern_matching`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.31.0 |

#### What it does

Lint for redundant pattern matching over `Result`, `Option`,
`std::task::Poll`, `std::net::IpAddr` or `bool`s

#### Why is this bad?

It’s more concise and clear to just use the proper
utility function or using the condition directly

#### Known problems

For suggestions involving bindings in patterns, this will change the drop order for the matched type.
Both `if let` and `while let` will drop the value at the end of the block, both `if` and `while` will drop the
value before entering the block. For most types this change will not matter, but for a few
types this will not be an acceptable change (e.g. locks). See the
[reference](https://doc.rust-lang.org/reference/destructors.html#drop-scopes) for more about
drop order.

#### Example

```rust
if let Ok(_) = Ok::<i32, i32>(42) {}
if let Err(_) = Err::<i32, i32>(42) {}
if let None = None::<()> {}
if let Some(_) = Some(42) {}
if let Poll::Pending = Poll::Pending::<()> {}
if let Poll::Ready(_) = Poll::Ready(42) {}
if let IpAddr::V4(_) = IpAddr::V4(Ipv4Addr::LOCALHOST) {}
if let IpAddr::V6(_) = IpAddr::V6(Ipv6Addr::LOCALHOST) {}
match Ok::<i32, i32>(42) {
    Ok(_) => true,
    Err(_) => false,
};

let cond = true;
if let true = cond {}
matches!(cond, true);
```

The more idiomatic use would be:

```rust
if Ok::<i32, i32>(42).is_ok() {}
if Err::<i32, i32>(42).is_err() {}
if None::<()>.is_none() {}
if Some(42).is_some() {}
if Poll::Pending::<()>.is_pending() {}
if Poll::Ready(42).is_ready() {}
if IpAddr::V4(Ipv4Addr::LOCALHOST).is_ipv4() {}
if IpAddr::V6(Ipv6Addr::LOCALHOST).is_ipv6() {}
Ok::<i32, i32>(42).is_ok();

let cond = true;
if cond {}
cond;
```

#### Past names

- if_let_redundant_pattern_matching

---

### `redundant_static_lifetimes`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.37.0 |

#### What it does

Checks for constants and statics with an explicit `'static` lifetime.

#### Why is this bad?

Adding `'static` to every reference can create very
complicated types.

#### Example

```rust
const FOO: &'static [(&'static str, &'static str, fn(&Bar) -> bool)] =
&[...]
static FOO: &'static [(&'static str, &'static str, fn(&Bar) -> bool)] =
&[...]
```

This code can be rewritten as

```rust
const FOO: &[(&str, &str, fn(&Bar) -> bool)] = &[...]
 static FOO: &[(&str, &str, fn(&Bar) -> bool)] = &[...]
```

#### Past names

- const_static_lifetime

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `result_map_or_into_option`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.44.0 |

#### What it does

Checks for usage of `_.map_or(None, Some)`.

#### Why is this bad?

Readability, this can be written more concisely as
`_.ok()`.

#### Example

```rust
assert_eq!(Some(1), r.map_or(None, Some));
```

Use instead:

```rust
assert_eq!(Some(1), r.ok());
```

---

### `result_unit_err`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.49.0 |

#### What it does

Checks for public functions that return a `Result`
with an `Err` type of `()`. It suggests using a custom type that
implements `std::error::Error`.

#### Why is this bad?

Unit does not implement `Error` and carries no
further information about what went wrong.

#### Known problems

Of course, this lint assumes that `Result` is used
for a fallible operation (which is after all the intended use). However
code may opt to (mis)use it as a basic two-variant-enum. In that case,
the suggestion is misguided, and the code should use a custom enum
instead.

#### Examples

```rust
pub fn read_u8() -> Result<u8, ()> { Err(()) }
```

should become

```rust
use std::fmt;

#[derive(Debug)]
pub struct EndOfStream;

impl fmt::Display for EndOfStream {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "End of Stream")
    }
}

impl std::error::Error for EndOfStream { }

pub fn read_u8() -> Result<u8, EndOfStream> { Err(EndOfStream) }
```

Note that there are crates that simplify creating the error type, e.g.
[thiserror](https://docs.rs/thiserror).

---

### `same_item_push`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.47.0 |

#### What it does

Checks whether a for loop is being used to push a constant
value into a Vec.

#### Why is this bad?

This kind of operation can be expressed more succinctly with
`vec![item; SIZE]` or `vec.resize(NEW_SIZE, item)` and using these alternatives may also
have better performance.

#### Example

```rust
let item1 = 2;
let item2 = 3;
let mut vec: Vec<u8> = Vec::new();
for _ in 0..20 {
    vec.push(item1);
}
for _ in 0..30 {
    vec.push(item2);
}
```

Use instead:

```rust
let item1 = 2;
let item2 = 3;
let mut vec: Vec<u8> = vec![item1; 20];
vec.resize(20 + 30, item2);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `self_named_constructors`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.55.0 |

#### What it does

Warns when constructors have the same name as their types.

#### Why is this bad?

Repeating the name of the type is redundant.

#### Example

```rust
struct Foo {}

impl Foo {
    pub fn foo() -> Foo {
        Foo {}
    }
}
```

Use instead:

```rust
struct Foo {}

impl Foo {
    pub fn new() -> Foo {
        Foo {}
    }
}
```

---

### `should_implement_trait`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for methods that should live in a trait
implementation of a `std` trait (see [llogiq’s blog
post](http://llogiq.github.io/2015/07/30/traits.html) for further
information) instead of an inherent implementation.

#### Why is this bad?

Implementing the traits improve ergonomics for users of
the code, often with very little cost. Also people seeing a `mul(...)`
method
may expect `*` to work equally, so you should have good reason to disappoint
them.

#### Example

```rust
struct X;
impl X {
    fn add(&self, other: &X) -> X {
        // ..
    }
}
```

---

### `single_char_add_str`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Warns when using `push_str`/`insert_str` with a single-character string literal
where `push`/`insert` with a `char` would work fine.

#### Why is this bad?

It’s less clear that we are pushing a single character.

#### Example

```rust
string.insert_str(0, "R");
string.push_str("R");
```

Use instead:

```rust
string.insert(0, 'R');
string.push('R');
```

#### Past names

- single_char_push_str

---

### `single_component_path_imports`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.43.0 |

#### What it does

Checking for imports with single component use path.

#### Why is this bad?

Import with single component use path such as `use cratename;`
is not necessary, and thus should be removed.

#### Example

```rust
use regex;

fn main() {
    regex::Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
}
```

Better as

```rust
fn main() {
    regex::Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
}
```

---

### `single_match`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for matches with a single arm where an `if let`
will usually suffice.

This intentionally does not lint if there are comments
inside of the other arm, so as to allow the user to document
why having another explicit pattern with an empty body is necessary,
or because the comments need to be preserved for other reasons.

#### Why is this bad?

Just readability – `if let` nests less than a `match`.

#### Example

```rust
match x {
    Some(ref foo) => bar(foo),
    _ => (),
}
```

Use instead:

```rust
if let Some(ref foo) = x {
    bar(foo);
}
```

---

### `string_extend_chars`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the use of `.extend(s.chars())` where s is a
`&str` or `String`.

#### Why is this bad?

`.push_str(s)` is clearer

#### Example

```rust
let abc = "abc";
let def = String::from("def");
let mut s = String::new();
s.extend(abc.chars());
s.extend(def.chars());
```

The correct use would be:

```rust
let abc = "abc";
let def = String::from("def");
let mut s = String::new();
s.push_str(abc);
s.push_str(&def);
```

---

### `tabs_in_doc_comments`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.41.0 |

#### What it does

Checks doc comments for usage of tab characters.

#### Why is this bad?

The rust style-guide promotes spaces instead of tabs for indentation.
To keep a consistent view on the source, also doc comments should not have tabs.
Also, explaining ascii-diagrams containing tabs can get displayed incorrectly when the
display settings of the author and reader differ.

#### Example

```rust
///
/// Struct to hold two strings:
/// 	- first		one
/// 	- second	one
pub struct DoubleString {
   ///
   /// 	- First String:
   /// 		- needs to be inside here
   first_string: String,
   ///
   /// 	- Second String:
   /// 		- needs to be inside here
   second_string: String,
}
```

Will be converted to:

```rust
///
/// Struct to hold two strings:
///     - first        one
///     - second    one
pub struct DoubleString {
   ///
   ///     - First String:
   ///         - needs to be inside here
   first_string: String,
   ///
   ///     - Second String:
   ///         - needs to be inside here
   second_string: String,
}
```

---

### `to_digit_is_some`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.41.0 |

#### What it does

Checks for `.to_digit(..).is_some()` on `char`s.

#### Why is this bad?

This is a convoluted way of checking if a `char` is a digit. It’s
more straight forward to use the dedicated `is_digit` method.

#### Example

```rust
let is_digit = c.to_digit(radix).is_some();
```

can be written as:

```rust
let is_digit = c.is_digit(radix);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `to_string_trait_impl`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.78.0 |

#### What it does

Checks for direct implementations of `ToString`.

#### Why is this bad?

This trait is automatically implemented for any type which implements the `Display` trait.
As such, `ToString` shouldn’t be implemented directly: `Display` should be implemented instead,
and you get the `ToString` implementation for free.

#### Example

```rust
struct Point {
  x: usize,
  y: usize,
}

impl ToString for Point {
  fn to_string(&self) -> String {
    format!("({}, {})", self.x, self.y)
  }
}
```

Use instead:

```rust
struct Point {
  x: usize,
  y: usize,
}

impl std::fmt::Display for Point {
  fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
    write!(f, "({}, {})", self.x, self.y)
  }
}
```

---

### `toplevel_ref_arg`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for function arguments and let bindings denoted as
`ref`.

#### Why is this bad?

The `ref` declaration makes the function take an owned
value, but turns the argument into a reference (which means that the value
is destroyed when exiting the function). This adds not much value: either
take a reference type, or take an owned value and create references in the
body.

For let bindings, `let x = &foo;` is preferred over `let ref x = foo`. The
type of `x` is more obvious with the former.

#### Known problems

If the argument is dereferenced within the function,
removing the `ref` will lead to errors. This can be fixed by removing the
dereferences, e.g., changing `*x` to `x` within the function.

#### Example

```rust
fn foo(ref _x: u8) {}
```

Use instead:

```rust
fn foo(_x: &u8) {}
```

---

### `trim_split_whitespace`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

Warns about calling `str::trim` (or variants) before `str::split_whitespace`.

#### Why is this bad?

`split_whitespace` already ignores leading and trailing whitespace.

#### Example

```rust
" A B C ".trim().split_whitespace();
```

Use instead:

```rust
" A B C ".split_whitespace();
```

---

### `unnecessary_fallible_conversions`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.75.0 |

#### What it does

Checks for calls to `TryInto::try_into` and `TryFrom::try_from` when their infallible counterparts
could be used.

#### Why is this bad?

In those cases, the `TryInto` and `TryFrom` trait implementation is a blanket impl that forwards
to `Into` or `From`, which always succeeds.
The returned `Result<_, Infallible>` requires error handling to get the contained value
even though the conversion can never fail.

#### Example

```rust
let _: Result<i64, _> = 1i32.try_into();
let _: Result<i64, _> = <_>::try_from(1i32);
```

Use `from`/`into` instead:

```rust
let _: i64 = 1i32.into();
let _: i64 = <_>::from(1i32);
```

---

### `unnecessary_fold`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `fold` when a more succinct alternative exists.
Specifically, this checks for `fold`s which could be replaced by `any`, `all`,
`sum` or `product`.

#### Why is this bad?

Readability.

#### Example

```rust
(0..3).fold(false, |acc, x| acc || x > 2);
```

Use instead:

```rust
(0..3).any(|x| x > 2);
```

---

### `unnecessary_lazy_evaluations`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.48.0 |

#### What it does

As the counterpart to `or_fun_call`, this lint looks for unnecessary
lazily evaluated closures on `Option` and `Result`.

This lint suggests changing the following functions, when eager evaluation results in
simpler code:

- `unwrap_or_else` to `unwrap_or`
- `and_then` to `and`
- `or_else` to `or`
- `get_or_insert_with` to `get_or_insert`
- `ok_or_else` to `ok_or`
- `then` to `then_some` (for msrv >= 1.62.0)

#### Why is this bad?

Using eager evaluation is shorter and simpler in some cases.

#### Known problems

It is possible, but not recommended for `Deref` and `Index` to have
side effects. Eagerly evaluating them can change the semantics of the program.

#### Example

```rust
let opt: Option<u32> = None;

opt.unwrap_or_else(|| 42);
```

Use instead:

```rust
let opt: Option<u32> = None;

opt.unwrap_or(42);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `unnecessary_map_or`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.84.0 |

#### What it does

Converts some constructs mapping an Enum value for equality comparison.

#### Why is this bad?

Calls such as `opt.map_or(false, |val| val == 5)` are needlessly long and cumbersome,
and can be reduced to, for example, `opt == Some(5)` assuming `opt` implements `PartialEq`.
Also, calls such as `opt.map_or(true, |val| val == 5)` can be reduced to
`opt.is_none_or(|val| val == 5)`.
This lint offers readability and conciseness improvements.

#### Example

```rust
pub fn a(x: Option<i32>) -> (bool, bool) {
    (
        x.map_or(false, |n| n == 5),
        x.map_or(true, |n| n > 5),
    )
}
```

Use instead:

```rust
pub fn a(x: Option<i32>) -> (bool, bool) {
    (
        x == Some(5),
        x.is_none_or(|n| n > 5),
    )
}
```

---

### `unnecessary_mut_passed`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Detects passing a mutable reference to a function that only
requires an immutable reference.

#### Why is this bad?

The mutable reference rules out all other references to
the value. Also the code misleads about the intent of the call site.

#### Example

```rust
vec.push(&mut value);
```

Use instead:

```rust
vec.push(&value);
```

---

### `unnecessary_owned_empty_strings`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

Detects cases of owned empty strings being passed as an argument to a function expecting `&str`

#### Why is this bad?

This results in longer and less readable code

#### Example

```rust
vec!["1", "2", "3"].join(&String::new());
```

Use instead:

```rust
vec!["1", "2", "3"].join("");
```

---

### `unneeded_struct_pattern`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Checks for struct patterns that match against unit variant.

#### Why is this bad?

Struct pattern `{ }` or `{ .. }` is not needed for unit variant.

#### Example

```rust
match Some(42) {
    Some(v) => v,
    None { .. } => 0,
};
// Or
match Some(42) {
    Some(v) => v,
    None { } => 0,
};
```

Use instead:

```rust
match Some(42) {
    Some(v) => v,
    None => 0,
};
```

---

### `unsafe_removed_from_name`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for imports that remove “unsafe” from an item’s
name.

#### Why is this bad?

Renaming makes it less clear which traits and
structures are unsafe.

#### Example

```rust
use std::cell::{UnsafeCell as TotallySafeCell};

extern crate crossbeam;
use crossbeam::{spawn_unsafe as spawn};
```

---

### `unused_enumerate_index`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.75.0 |

#### What it does

Checks for uses of the `enumerate` method where the index is unused (`_`)

#### Why is this bad?

The index from `.enumerate()` is immediately dropped.

#### Example

```rust
let v = vec![1, 2, 3, 4];
for (_, x) in v.iter().enumerate() {
    println!("{x}");
}
```

Use instead:

```rust
let v = vec![1, 2, 3, 4];
for x in v.iter() {
    println!("{x}");
}
```

---

### `unused_unit`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.31.0 |

#### What it does

Checks for unit (`()`) expressions that can be removed.

#### Why is this bad?

Such expressions add no value, but can make the code
less readable. Depending on formatting they can make a `break` or `return`
statement look like a function call.

#### Example

```rust
fn return_unit() -> () {
    ()
}
```

is equivalent to

```rust
fn return_unit() {}
```

---

### `unusual_byte_groupings`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.49.0 |

#### What it does

Warns if hexadecimal or binary literals are not grouped
by nibble or byte.

#### Why is this bad?

Negatively impacts readability.

#### Example

```rust
let x: u32 = 0xFFF_FFF;
let y: u8 = 0b01_011_101;
```

---

### `unwrap_or_default`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.56.0 |

#### What it does

Checks for usages of the following functions with an argument that constructs a default value
(e.g., `Default::default` or `String::new`):

- `unwrap_or`
- `unwrap_or_else`
- `or_insert`
- `or_insert_with`

#### Why is this bad?

Readability. Using `unwrap_or_default` in place of `unwrap_or`/`unwrap_or_else`, or `or_default`
in place of `or_insert`/`or_insert_with`, is simpler and more concise.

#### Known problems

In some cases, the argument of `unwrap_or`, etc. is needed for type inference. The lint uses a
heuristic to try to identify such cases. However, the heuristic can produce false negatives.

#### Examples

```rust
x.unwrap_or(Default::default());
map.entry(42).or_insert_with(String::new);
```

Use instead:

```rust
x.unwrap_or_default();
map.entry(42).or_default();
```

#### Past names

- unwrap_or_else_default

---

### `upper_case_acronyms`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.51.0 |

#### What it does

Checks for fully capitalized names and optionally names containing a capitalized acronym.

#### Why is this bad?

In CamelCase, acronyms count as one word.
See [naming conventions](https://rust-lang.github.io/api-guidelines/naming.html#casing-conforms-to-rfc-430-c-case)
for more.

By default, the lint only triggers on fully-capitalized names.
You can use the `upper-case-acronyms-aggressive: true` config option to enable linting
on all camel case names

#### Known problems

When two acronyms are contiguous, the lint can’t tell where
the first acronym ends and the second starts, so it suggests to lowercase all of
the letters in the second acronym.

#### Example

```rust
struct HTTPResponse;
```

Use instead:

```rust
struct HttpResponse;
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)
- `upper-case-acronyms-aggressive`:  Enables verbose mode. Triggers if there is more than one uppercase char next to each other

(default: `false`)

---

### `while_let_on_iterator`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `while let` expressions on iterators.

#### Why is this bad?

Readability. A simple `for` loop is shorter and conveys
the intent better.

#### Example

```rust
while let Some(val) = iter.next() {
    ..
}
```

Use instead:

```rust
for val in &mut iter {
    ..
}
```

---

### `write_literal`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint warns about the use of literals as `write!`/`writeln!` args.

#### Why is this bad?

Using literals as `writeln!` args is inefficient
(c.f., https://github.com/matthiaskrgr/rust-str-bench) and unnecessary
(i.e., just put the literal in the format string)

#### Example

```rust
writeln!(buf, "{}", "foo");
```

Use instead:

```rust
writeln!(buf, "foo");
```

---

### `write_with_newline`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint warns when you use `write!()` with a format
string that
ends in a newline.

#### Why is this bad?

You should use `writeln!()` instead, which appends the
newline.

#### Example

```rust
write!(buf, "Hello {}!\n", name);
```

Use instead:

```rust
writeln!(buf, "Hello {}!", name);
```

---

### `writeln_empty_string`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint warns when you use `writeln!(buf, "")` to
print a newline.

#### Why is this bad?

You should use `writeln!(buf)`, which is simpler.

#### Example

```rust
writeln!(buf, "");
```

Use instead:

```rust
writeln!(buf);
```

---

### `wrong_self_convention`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for methods with certain name prefixes or suffixes, and which
do not adhere to standard conventions regarding how `self` is taken.
The actual rules are:

| Prefix | Postfix | self taken | self type |
| --- | --- | --- | --- |
| as_ | none | &self or &mut self | any |
| from_ | none | none | any |
| into_ | none | self | any |
| is_ | none | &mut self or &self or none | any |
| to_ | _mut | &mut self | any |
| to_ | not _mut | self | Copy |
| to_ | not _mut | &self | not Copy |

Note: Clippy doesn’t trigger methods with `to_` prefix in:

- Traits definition.
Clippy can not tell if a type that implements a trait is `Copy` or not.
- Traits implementation, when `&self` is taken.
The method signature is controlled by the trait and often `&self` is required for all types that implement the trait
(see e.g. the `std::string::ToString` trait).

Clippy allows `Pin<&Self>` and `Pin<&mut Self>` if `&self` and `&mut self` is required.

Please find more info here:
https://rust-lang.github.io/api-guidelines/naming.html#ad-hoc-conversions-follow-as_-to_-into_-conventions-c-conv

#### Why is this bad?

Consistency breeds readability. If you follow the
conventions, your users won’t be surprised that they, e.g., need to supply a
mutable reference to a `as_..` function.

#### Example

```rust
impl X {
    fn as_str(self) -> &'static str {
        // ..
    }
}
```

Use instead:

```rust
impl X {
    fn as_str(&self) -> &'static str {
        // ..
    }
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `zero_ptr`

| 属性 | 值 |
|------|----|
| 分组 | style |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Catch casts from `0` to some pointer type

#### Why is this bad?

This generally means `null` and is better expressed as
{`std`, `core`}`::ptr::`{`null`, `null_mut`}.

#### Example

```rust
let a = 0 as *const u32;
```

Use instead:

```rust
let a = std::ptr::null::<u32>();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

## Complexity (137)

*复杂度 - 做简单事情的复杂代码，默认 warn*

### `bind_instead_of_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.45.0 |

#### What it does

Checks for usage of `_.and_then(|x| Some(y))`, `_.and_then(|x| Ok(y))`
or `_.or_else(|x| Err(y))`.

#### Why is this bad?

This can be written more concisely as `_.map(|x| y)` or `_.map_err(|x| y)`.

#### Example

```rust
let _ = opt().and_then(|s| Some(s.len()));
let _ = res().and_then(|s| if s.len() == 42 { Ok(10) } else { Ok(20) });
let _ = res().or_else(|s| if s.len() == 42 { Err(10) } else { Err(20) });
```

The correct use would be:

```rust
let _ = opt().map(|s| s.len());
let _ = res().map(|s| if s.len() == 42 { 10 } else { 20 });
let _ = res().map_err(|s| if s.len() == 42 { 10 } else { 20 });
```

#### Past names

- option_and_then_some

---

### `bool_comparison`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for expressions of the form `x == true`,
`x != true` and order comparisons such as `x < true` (or vice versa) and
suggest using the variable directly.

#### Why is this bad?

Unnecessary code.

#### Example

```rust
if x == true {}
if y == false {}
```

use `x` directly:

```rust
if x {}
if !y {}
```

---

### `borrow_deref_ref`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.63.0 |

#### What it does

Checks for `&*(&T)`.

#### Why is this bad?

Dereferencing and then borrowing a reference value has no effect in most cases.

#### Known problems

False negative on such code:

```rust
let x = &12;
let addr_x = &x as *const _ as usize;
let addr_y = &&*x as *const _ as usize; // assert ok now, and lint triggered.
                                        // But if we fix it, assert will fail.
assert_ne!(addr_x, addr_y);
```

#### Example

```rust
let s = &String::new();

let a: &String = &* s;
```

Use instead:

```rust
let a: &String = s;
```

---

### `borrowed_box`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `&Box<T>` anywhere in the code.
Check the [Box documentation](https://doc.rust-lang.org/std/boxed/index.html) for more information.

#### Why is this bad?

A `&Box<T>` parameter requires the function caller to box `T` first before passing it to a function.
Using `&T` defines a concrete type for the parameter and generalizes the function, this would also
auto-deref to `&T` at the function call site if passed a `&Box<T>`.

#### Example

```rust
fn foo(bar: &Box<T>) { ... }
```

Better:

```rust
fn foo(bar: &T) { ... }
```

---

### `bytes_count_to_len`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

It checks for `str::bytes().count()` and suggests replacing it with
`str::len()`.

#### Why is this bad?

`str::bytes().count()` is longer and may not be as performant as using
`str::len()`.

#### Example

```rust
"hello".bytes().count();
String::from("hello").bytes().count();
```

Use instead:

```rust
"hello".len();
String::from("hello").len();
```

---

### `char_lit_as_u8`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for expressions where a character literal is cast
to `u8` and suggests using a byte literal instead.

#### Why is this bad?

In general, casting values to smaller types is
error-prone and should be avoided where possible. In the particular case of
converting a character literal to `u8`, it is easy to avoid by just using a
byte literal instead. As an added bonus, `b'a'` is also slightly shorter
than `'a' as u8`.

#### Example

```rust
'x' as u8
```

A better version, using the byte literal:

```rust
b'x'
```

---

### `clone_on_copy`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `.clone()` on a `Copy` type.

#### Why is this bad?

The only reason `Copy` types implement `Clone` is for
generics, not for using the `clone` method on a concrete type.

#### Example

```rust
42u64.clone();
```

---

### `default_constructed_unit_structs`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.71.0 |

#### What it does

Checks for construction on unit struct using `default`.

#### Why is this bad?

This adds code complexity and an unnecessary function call.

#### Example

```rust
#[derive(Default)]
struct S<T> {
    _marker: PhantomData<T>
}

let _: S<i32> = S {
    _marker: PhantomData::default()
};
```

Use instead:

```rust
struct S<T> {
    _marker: PhantomData<T>
}

let _: S<i32> = S {
    _marker: PhantomData
};
```

---

### `deprecated_cfg_attr`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.32.0 |

#### What it does

Checks for `#[cfg_attr(rustfmt, rustfmt_skip)]` and suggests to replace it
with `#[rustfmt::skip]`.

#### Why is this bad?

Since tool_attributes ([rust-lang/rust#44690](https://github.com/rust-lang/rust/issues/44690))
are stable now, they should be used instead of the old `cfg_attr(rustfmt)` attributes.

#### Known problems

This lint doesn’t detect crate level inner attributes, because they get
processed before the PreExpansionPass lints get executed. See
[#3123](https://github.com/rust-lang/rust-clippy/pull/3123#issuecomment-422321765)

#### Example

```rust
#[cfg_attr(rustfmt, rustfmt_skip)]
fn main() { }
```

Use instead:

```rust
#[rustfmt::skip]
fn main() { }
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `deref_addrof`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `*&` and `*&mut` in expressions.

#### Why is this bad?

Immediately dereferencing a reference is no-op and
makes the code less clear.

#### Known problems

Multiple dereference/addrof pairs are not handled so
the suggested fix for `x = **&&y` is `x = *&y`, which is still incorrect.

#### Example

```rust
let a = f(*&mut b);
let c = *&d;
```

Use instead:

```rust
let a = f(b);
let c = d;
```

---

### `derivable_impls`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.57.0 |

#### What it does

Detects manual `std::default::Default` implementations that are identical to a derived implementation.

#### Why is this bad?

It is less concise.

#### Example

```rust
struct Foo {
    bar: bool
}

impl Default for Foo {
    fn default() -> Self {
        Self {
            bar: false
        }
    }
}
```

Use instead:

```rust
#[derive(Default)]
struct Foo {
    bar: bool
}
```

#### Known problems

Derive macros [sometimes use incorrect bounds](https://github.com/rust-lang/rust/issues/26925)
in generic types and the user defined `impl` may be more generalized or
specialized than what derive will produce. This lint can’t detect the manual `impl`
has exactly equal bounds, and therefore this lint is disabled for types with
generic parameters.

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `diverging_sub_expression`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for diverging calls that are not match arms or
statements.

#### Why is this bad?

It is often confusing to read. In addition, the
sub-expression evaluation order for Rust is not well documented.

#### Known problems

Someone might want to use `some_bool || panic!()` as a
shorthand.

#### Example

```rust
let a = b() || panic!() || c();
// `c()` is dead, `panic!()` is only called if `b()` returns `false`
let x = (a, b, c, panic!());
// can simply be replaced by `panic!()`
```

---

### `double_comparisons`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for double comparisons that could be simplified to a single expression.

#### Why is this bad?

Readability.

#### Example

```rust
if x == y || x < y {}
```

Use instead:

```rust
if x <= y {}
```

---

### `double_parens`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for unnecessary double parentheses.

#### Why is this bad?

This makes code harder to read and might indicate a
mistake.

#### Example

```rust
fn simple_double_parens() -> i32 {
    ((0))
}

foo((0));
```

Use instead:

```rust
fn simple_no_parens() -> i32 {
    (0)
}

foo(0);
```

---

### `duration_subsec`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for calculation of subsecond microseconds or milliseconds
from other `Duration` methods.

#### Why is this bad?

It’s more concise to call `Duration::subsec_micros()` or
`Duration::subsec_millis()` than to calculate them.

#### Example

```rust
let micros = duration.subsec_nanos() / 1_000;
let millis = duration.subsec_nanos() / 1_000_000;
```

Use instead:

```rust
let micros = duration.subsec_micros();
let millis = duration.subsec_millis();
```

---

### `excessive_nesting`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for blocks which are nested beyond a certain threshold.

Note: Even though this lint is warn-by-default, it will only trigger if a maximum nesting level is defined in the clippy.toml file.

#### Why is this bad?

It can severely hinder readability.

#### Example

An example clippy.toml configuration:

```toml
excessive-nesting-threshold = 3
```

```rust
// lib.rs
pub mod a {
    pub struct X;
    impl X {
        pub fn run(&self) {
            if true {
                // etc...
            }
        }
    }
}
```

Use instead:

```rust
// a.rs
fn private_run(x: &X) {
    if true {
        // etc...
    }
}

pub struct X;
impl X {
    pub fn run(&self) {
        private_run(self);
    }
}
```

```rust
// lib.rs
pub mod a;
```

#### Configuration

- `excessive-nesting-threshold`:  The maximum amount of nesting a block can reside in

(default: `0`)

---

### `explicit_auto_deref`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Checks for dereferencing expressions which would be covered by auto-deref.

#### Why is this bad?

This unnecessarily complicates the code.

#### Example

```rust
let x = String::new();
let y: &str = &*x;
```

Use instead:

```rust
let x = String::new();
let y: &str = &x;
```

---

### `explicit_counter_loop`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks `for` loops over slices with an explicit counter
and suggests the use of `.enumerate()`.

#### Why is this bad?

Using `.enumerate()` makes the intent more clear,
declutters the code and may be faster in some instances.

#### Example

```rust
let mut i = 0;
for item in &v {
    bar(i, *item);
    i += 1;
}
```

Use instead:

```rust
for (i, item) in v.iter().enumerate() { bar(i, *item); }
```

---

### `explicit_write`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `write!()` / `writeln()!` which can be
replaced with `(e)print!()` / `(e)println!()`

#### Why is this bad?

Using `(e)println!` is clearer and more concise

#### Example

```rust
writeln!(&mut std::io::stderr(), "foo: {:?}", bar).unwrap();
writeln!(&mut std::io::stdout(), "foo: {:?}", bar).unwrap();
```

Use instead:

```rust
eprintln!("foo: {:?}", bar);
println!("foo: {:?}", bar);
```

---

### `extra_unused_lifetimes`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for lifetimes in generics that are never used
anywhere else.

#### Why is this bad?

The additional lifetimes make the code look more
complicated, while there is nothing out of the ordinary going on. Removing
them leads to more readable code.

#### Example

```rust
// unnecessary lifetimes
fn unused_lifetime<'a>(x: u8) {
    // ..
}
```

Use instead:

```rust
fn no_lifetime(x: u8) {
    // ...
}
```

---

### `extra_unused_type_parameters`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.69.0 |

#### What it does

Checks for type parameters in generics that are never used anywhere else.

#### Why is this bad?

Functions cannot infer the value of unused type parameters; therefore, calling them
requires using a turbofish, which serves no purpose but to satisfy the compiler.

#### Example

```rust
fn unused_ty<T>(x: u8) {
    // ..
}
```

Use instead:

```rust
fn no_unused_ty(x: u8) {
    // ..
}
```

---

### `filter_map_identity`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Checks for usage of `filter_map(|x| x)`.

#### Why is this bad?

Readability, this can be written more concisely by using `flatten`.

#### Example

```rust
iter.filter_map(|x| x);
```

Use instead:

```rust
iter.flatten();
```

---

### `filter_next`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `_.filter(_).next()`.

#### Why is this bad?

Readability, this can be written more concisely as
`_.find(_)`.

#### Example

```rust
vec.iter().filter(|x| **x == 0).next();
```

Use instead:

```rust
vec.iter().find(|x| **x == 0);
```

---

### `flat_map_identity`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.39.0 |

#### What it does

Checks for usage of `flat_map(|x| x)`.

#### Why is this bad?

Readability, this can be written more concisely by using `flatten`.

#### Example

```rust
iter.flat_map(|x| x);
```

Can be written as

```rust
iter.flatten();
```

---

### `get_last_with_len`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.37.0 |

#### What it does

Checks for usage of `x.get(x.len() - 1)` instead of
`x.last()`.

#### Why is this bad?

Using `x.last()` is easier to read and has the same
result.

Note that using `x[x.len() - 1]` is semantically different from
`x.last()`.  Indexing into the array will panic on out-of-bounds
accesses, while `x.get()` and `x.last()` will return `None`.

There is another lint (get_unwrap) that covers the case of using
`x.get(index).unwrap()` instead of `x[index]`.

#### Example

```rust
let x = vec![2, 3, 5];
let last_element = x.get(x.len() - 1);
```

Use instead:

```rust
let x = vec![2, 3, 5];
let last_element = x.last();
```

---

### `identity_op`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for identity operations, e.g., `x + 0`.

#### Why is this bad?

This code can be removed without changing the
meaning. So it just obscures what’s going on. Delete it mercilessly.

#### Example

```rust
x / 1 + 0 * 1 - 0 | 0;
```

---

### `implied_bounds_in_impls`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.74.0 |

#### What it does

Looks for bounds in `impl Trait` in return position that are implied by other bounds.
This can happen when a trait is specified that another trait already has as a supertrait
(e.g. `fn() -> impl Deref + DerefMut<Target = i32>` has an unnecessary `Deref` bound,
because `Deref` is a supertrait of `DerefMut`)

#### Why is this bad?

Specifying more bounds than necessary adds needless complexity for the reader.

#### Limitations

This lint does not check for implied bounds transitively. Meaning that
it doesn’t check for implied bounds from supertraits of supertraits
(e.g. `trait A {} trait B: A {} trait C: B {}`, then having an `fn() -> impl A + C`)

#### Example

```rust
fn f() -> impl Deref<Target = i32> + DerefMut<Target = i32> {
//             ^^^^^^^^^^^^^^^^^^^ unnecessary bound, already implied by the `DerefMut` trait bound
    Box::new(123)
}
```

Use instead:

```rust
fn f() -> impl DerefMut<Target = i32> {
    Box::new(123)
}
```

---

### `inspect_for_each`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.51.0 |

#### What it does

Checks for usage of `inspect().for_each()`.

#### Why is this bad?

It is the same as performing the computation
inside `inspect` at the beginning of the closure in `for_each`.

#### Example

```rust
[1,2,3,4,5].iter()
.inspect(|&x| println!("inspect the number: {}", x))
.for_each(|&x| {
    assert!(x >= 0);
});
```

Can be written as

```rust
[1,2,3,4,5].iter()
.for_each(|&x| {
    println!("inspect the number: {}", x);
    assert!(x >= 0);
});
```

---

### `int_plus_one`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `x >= y + 1` or `x - 1 >= y` (and `<=`) in a block

#### Why is this bad?

Readability – better to use `> y` instead of `>= y + 1`.

#### Example

```rust
if x >= y + 1 {}
```

Use instead:

```rust
if x > y {}
```

---

### `iter_count`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Checks for the use of `.iter().count()`.

#### Why is this bad?

`.len()` is more efficient and more
readable.

#### Example

```rust
let some_vec = vec![0, 1, 2, 3];

some_vec.iter().count();
&some_vec[..].iter().count();
```

Use instead:

```rust
let some_vec = vec![0, 1, 2, 3];

some_vec.len();
&some_vec[..].len();
```

---

### `iter_kv_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.66.0 |

#### What it does

Checks for iterating a map (`HashMap` or `BTreeMap`) and
ignoring either the keys or values.

#### Why is this bad?

Readability. There are `keys` and `values` methods that
can be used to express that we only need the keys or the values.

#### Example

```rust
let map: HashMap<u32, u32> = HashMap::new();
let values = map.iter().map(|(_, value)| value).collect::<Vec<_>>();
```

Use instead:

```rust
let map: HashMap<u32, u32> = HashMap::new();
let values = map.values().collect::<Vec<_>>();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `let_with_type_underscore`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.70.0 |

#### What it does

Detects when a variable is declared with an explicit type of `_`.

#### Why is this bad?

It adds noise, `: _` provides zero clarity or utility.

#### Example

```rust
let my_number: _ = 1;
```

Use instead:

```rust
let my_number = 1;
```

---

### `manual_abs_diff`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.88.0 |

#### What it does

Detects patterns like `if a > b { a - b } else { b - a }` and suggests using `a.abs_diff(b)`.

#### Why is this bad?

Using `abs_diff` is shorter, more readable, and avoids control flow.

#### Examples

```rust
if a > b {
    a - b
} else {
    b - a
}
```

Use instead:

```rust
a.abs_diff(b)
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_c_str_literals`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.78.0 |

#### What it does

Checks for the manual creation of C strings (a string with a `NUL` byte at the end), either
through one of the `CStr` constructor functions, or more plainly by calling `.as_ptr()`
on a (byte) string literal with a hardcoded `\0` byte at the end.

#### Why is this bad?

This can be written more concisely using `c"str"` literals and is also less error-prone,
because the compiler checks for interior `NUL` bytes and the terminating `NUL` byte is inserted automatically.

#### Example

```rust
fn needs_cstr(_: &CStr) {}

needs_cstr(CStr::from_bytes_with_nul(b"Hello\0").unwrap());
unsafe { libc::puts("World\0".as_ptr().cast()) }
```

Use instead:

```rust
fn needs_cstr(_: &CStr) {}

needs_cstr(c"Hello");
unsafe { libc::puts(c"World".as_ptr()) }
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_checked_ops`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.95.0 |

#### What it does

Detects manual zero checks before dividing integers, such as `if x != 0 { y / x }`.

#### Why is this bad?

`checked_div` already handles the zero case and makes the intent clearer while avoiding a
panic from a manual division.

#### Example

```rust
if b != 0 {
    let result = a / b;
    println!("{result}");
}
```

Use instead:

```rust
if let Some(result) = a.checked_div(b) {
    println!("{result}");
}
```

---

### `manual_clamp`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.66.0 |

#### What it does

Identifies good opportunities for a clamp function from std or core, and suggests using it.

#### Why is this bad?

clamp is much shorter, easier to read, and doesn’t use any control flow.

#### Limitations

This lint will only trigger if max and min are known at compile time, and max is
greater than min.

#### Known issue(s)

If the clamped variable is NaN this suggestion will cause the code to propagate NaN
rather than returning either `max` or `min`.

`clamp` functions will panic if `max < min`, `max.is_nan()`, or `min.is_nan()`.
Some may consider panicking in these situations to be desirable, but it also may
introduce panicking where there wasn’t any before.

See also [the discussion in the
PR](https://github.com/rust-lang/rust-clippy/pull/9484#issuecomment-1278922613).

#### Examples

```rust
if input > max {
    max
} else if input < min {
    min
} else {
    input
}
```

```rust
input.max(min).min(max)
```

```rust
match input {
    x if x > max => max,
    x if x < min => min,
    x => x,
}
```

```rust
let mut x = input;
if x < min { x = min; }
if x > max { x = max; }
```

Use instead:

```rust
input.clamp(min, max)
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_div_ceil`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.83.0 |

#### What it does

Checks for an expression like `(x + (y - 1)) / y` which is a common manual reimplementation
of `x.div_ceil(y)`.

#### Why is this bad?

It’s simpler, clearer and more readable.

#### Example

```rust
let x: i32 = 7;
let y: i32 = 4;
let div = (x + (y - 1)) / y;
```

Use instead:

```rust
#![feature(int_roundings)]
let x: i32 = 7;
let y: i32 = 4;
let div = x.div_ceil(y);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_filter`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.66.0 |

#### What it does

Checks for usage of `match` which could be implemented using `filter`

#### Why is this bad?

Using the `filter` method is clearer and more concise.

#### Example

```rust
match Some(0) {
    Some(x) => if x % 2 == 0 {
                    Some(x)
               } else {
                    None
                },
    None => None,
};
```

Use instead:

```rust
Some(0).filter(|&x| x % 2 == 0);
```

---

### `manual_filter_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.51.0 |

#### What it does

Checks for usage of `_.filter(_).map(_)` that can be written more simply
as `filter_map(_)`.

#### Why is this bad?

Redundant code in the `filter` and `map` operations is poor style and
less performant.

#### Example

```rust
(0_i32..10)
    .filter(|n| n.checked_add(1).is_some())
    .map(|n| n.checked_add(1).unwrap());
```

Use instead:

```rust
(0_i32..10).filter_map(|n| n.checked_add(1));
```

#### Past names

- filter_map

---

### `manual_find`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Checks for manual implementations of Iterator::find

#### Why is this bad?

It doesn’t affect performance, but using `find` is shorter and easier to read.

#### Example

```rust
fn example(arr: Vec<i32>) -> Option<i32> {
    for el in arr {
        if el == 1 {
            return Some(el);
        }
    }
    None
}
```

Use instead:

```rust
fn example(arr: Vec<i32>) -> Option<i32> {
    arr.into_iter().find(|&el| el == 1)
}
```

---

### `manual_find_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.51.0 |

#### What it does

Checks for usage of `_.find(_).map(_)` that can be written more simply
as `find_map(_)`.

#### Why is this bad?

Redundant code in the `find` and `map` operations is poor style and
less performant.

#### Example

```rust
(0_i32..10)
    .find(|n| n.checked_add(1).is_some())
    .map(|n| n.checked_add(1).unwrap());
```

Use instead:

```rust
(0_i32..10).find_map(|n| n.checked_add(1));
```

#### Past names

- find_map

---

### `manual_flatten`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Checks for unnecessary `if let` usage in a for loop
where only the `Some` or `Ok` variant of the iterator element is used.

#### Why is this bad?

It is verbose and can be simplified
by first calling the `flatten` method on the `Iterator`.

#### Example

```rust
let x = vec![Some(1), Some(2), Some(3)];
for n in x {
    if let Some(n) = n {
        println!("{}", n);
    }
}
```

Use instead:

```rust
let x = vec![Some(1), Some(2), Some(3)];
for n in x.into_iter().flatten() {
    println!("{}", n);
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_hash_one`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.75.0 |

#### What it does

Checks for cases where [BuildHasher::hash_one](https://doc.rust-lang.org/std/hash/trait.BuildHasher.html#method.hash_one) can be used.

#### Why is this bad?

It is more concise to use the `hash_one` method.

#### Example

```rust
use std::hash::{BuildHasher, Hash, Hasher};
use std::collections::hash_map::RandomState;

let s = RandomState::new();
let value = vec![1, 2, 3];

let mut hasher = s.build_hasher();
value.hash(&mut hasher);
let hash = hasher.finish();
```

Use instead:

```rust
use std::hash::BuildHasher;
use std::collections::hash_map::RandomState;

let s = RandomState::new();
let value = vec![1, 2, 3];

let hash = s.hash_one(&value);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_inspect`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.81.0 |

#### What it does

Checks for uses of `map` which return the original item.

#### Why is this bad?

`inspect` is both clearer in intent and shorter.

#### Example

```rust
let x = Some(0).map(|x| { println!("{x}"); x });
```

Use instead:

```rust
let x = Some(0).inspect(|x| println!("{x}"));
```

---

### `manual_is_multiple_of`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.90.0 |

#### What it does

Checks for manual implementation of `.is_multiple_of()` on
unsigned integer types.

#### Why is this bad?

`a.is_multiple_of(b)` is a clearer way to check for divisibility
of `a` by `b`. This expression can never panic.

#### Example

```rust
if a % b == 0 {
    println!("{a} is divisible by {b}");
}
```

Use instead:

```rust
if a.is_multiple_of(b) {
    println!("{a} is divisible by {b}");
}
```

---

### `manual_main_separator_str`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.70.0 |

#### What it does

Checks for references on `std::path::MAIN_SEPARATOR.to_string()` used
to build a `&str`.

#### Why is this bad?

There exists a `std::path::MAIN_SEPARATOR_STR` which does not require
an extra memory allocation.

#### Example

```rust
let s: &str = &std::path::MAIN_SEPARATOR.to_string();
```

Use instead:

```rust
let s: &str = std::path::MAIN_SEPARATOR_STR;
```

---

### `manual_ok_err`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Checks for manual implementation of `.ok()` or `.err()`
on `Result` values.

#### Why is this bad?

Using `.ok()` or `.err()` rather than a `match` or
`if let` is less complex and more readable.

#### Example

```rust
let a = match func() {
    Ok(v) => Some(v),
    Err(_) => None,
};
let b = if let Err(v) = func() {
    Some(v)
} else {
    None
};
```

Use instead:

```rust
let a = func().ok();
let b = func().err();
```

---

### `manual_option_as_slice`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

This detects various manual reimplementations of `Option::as_slice`.

#### Why is this bad?

Those implementations are both more complex than calling `as_slice`
and unlike that incur a branch, pessimizing performance and leading
to more generated code.

#### Example

```rust
_ = opt.as_ref().map_or(&[][..], std::slice::from_ref);
_ = match opt.as_ref() {
    Some(f) => std::slice::from_ref(f),
    None => &[],
};
```

Use instead:

```rust
_ = opt.as_slice();
_ = opt.as_slice();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_pop_if`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.95.0 |

#### What it does

Checks for code to be replaced by `pop_if` methods.

#### Why is this bad?

Using `pop_if` is more concise and idiomatic.

#### Known issues

Currently, the lint does not handle the case where the
`if` condition is part of an `else if` branch.

The lint also does not handle the case where
the popped value is assigned and used.

#### Examples

```rust
if vec.last().is_some_and(|x| *x > 5) {
    vec.pop().unwrap();
}
if deque.back().is_some_and(|x| *x > 5) {
    deque.pop_back().unwrap();
}
if deque.front().is_some_and(|x| *x > 5) {
    deque.pop_front().unwrap();
}
```

Use instead:

```rust
vec.pop_if(|x| *x > 5);
deque.pop_back_if(|x| *x > 5);
deque.pop_front_if(|x| *x > 5);
```

---

### `manual_range_patterns`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Looks for combined OR patterns that are all contained in a specific range,
e.g. `6 | 4 | 5 | 9 | 7 | 8` can be rewritten as `4..=9`.

#### Why is this bad?

Using an explicit range is more concise and easier to read.

#### Known issues

This lint intentionally does not handle numbers greater than `i128::MAX` for `u128` literals
in order to support negative numbers.

#### Example

```rust
let x = 6;
let foo = matches!(x, 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10);
```

Use instead:

```rust
let x = 6;
let foo = matches!(x, 1..=10);
```

---

### `manual_rem_euclid`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Checks for an expression like `((x % 4) + 4) % 4` which is a common manual reimplementation
of `x.rem_euclid(4)`.

#### Why is this bad?

It’s simpler and more readable.

#### Example

```rust
let x: i32 = 24;
let rem = ((x % 4) + 4) % 4;
```

Use instead:

```rust
let x: i32 = 24;
let rem = x.rem_euclid(4);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_slice_size_calculation`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.70.0 |

#### What it does

When `a` is `&[T]`, detect `a.len() * size_of::<T>()` and suggest `size_of_val(a)`
instead.

#### Why is this better?

- Shorter to write
- Removes the need for the human and the compiler to worry about overflow in the
multiplication
- Potentially faster at runtime as rust emits special no-wrapping flags when it
calculates the byte length
- Less turbofishing

#### Example

```rust
let newlen = data.len() * size_of::<i32>();
```

Use instead:

```rust
let newlen = size_of_val(data);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_split_once`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.57.0 |

#### What it does

Checks for usage of `str::splitn(2, _)`

#### Why is this bad?

`split_once` is both clearer in intent and slightly more efficient.

#### Example

```rust
let s = "key=value=add";
let (key, value) = s.splitn(2, '=').next_tuple()?;
let value = s.splitn(2, '=').nth(1)?;

let mut parts = s.splitn(2, '=');
let key = parts.next()?;
let value = parts.next()?;
```

Use instead:

```rust
let s = "key=value=add";
let (key, value) = s.split_once('=')?;
let value = s.split_once('=')?.1;

let (key, value) = s.split_once('=')?;
```

#### Limitations

The multiple statement variant currently only detects `iter.next()?`/`iter.next().unwrap()`
in two separate `let` statements that immediately follow the `splitn()`

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_strip`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.48.0 |

#### What it does

Suggests using `strip_{prefix,suffix}` over `str::{starts,ends}_with` and slicing using
the pattern’s length.

#### Why is this bad?

Using `str:strip_{prefix,suffix}` is safer and may have better performance as there is no
slicing which may panic and the compiler does not need to insert this panic code. It is
also sometimes more readable as it removes the need for duplicating or storing the pattern
used by `str::{starts,ends}_with` and in the slicing.

#### Example

```rust
let s = "hello, world!";
if s.starts_with("hello, ") {
    assert_eq!(s["hello, ".len()..].to_uppercase(), "WORLD!");
}
```

Use instead:

```rust
let s = "hello, world!";
if let Some(end) = s.strip_prefix("hello, ") {
    assert_eq!(end.to_uppercase(), "WORLD!");
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_swap`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for manual swapping.

Note that the lint will not be emitted in const blocks, as the suggestion would not be applicable.

#### Why is this bad?

The `std::mem::swap` function exposes the intent better
without deinitializing or copying either variable.

#### Example

```rust
let mut a = 42;
let mut b = 1337;

let t = b;
b = a;
a = t;
```

Use std::mem::swap():

```rust
let mut a = 1;
let mut b = 2;
std::mem::swap(&mut a, &mut b);
```

---

### `manual_take`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.94.0 |

#### What it does

Detects manual re-implementations of `std::mem::take`.

#### Why is this bad?

Because the function call is shorter and easier to read.

#### Known issues

Currently the lint only detects cases involving `bool`s.

#### Example

```rust
let mut x = true;
let _ = if x {
    x = false;
    true
} else {
    false
};
```

Use instead:

```rust
let mut x = true;
let _ = std::mem::take(&mut x);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_unwrap_or`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Finds patterns that reimplement `Option::unwrap_or` or `Result::unwrap_or`.

#### Why is this bad?

Concise code helps focusing on behavior instead of boilerplate.

#### Example

```rust
let foo: Option<i32> = None;
match foo {
    Some(v) => v,
    None => 1,
};
```

Use instead:

```rust
let foo: Option<i32> = None;
foo.unwrap_or(1);
```

---

### `map_all_any_identity`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.84.0 |

#### What it does

Checks for usage of `.map(…)`, followed by `.all(identity)` or `.any(identity)`.

#### Why is this bad?

The `.all(…)` or `.any(…)` methods can be called directly in place of `.map(…)`.

#### Example

```rust
let e1 = v.iter().map(|s| s.is_empty()).all(|a| a);
let e2 = v.iter().map(|s| s.is_empty()).any(std::convert::identity);
```

Use instead:

```rust
let e1 = v.iter().all(|s| s.is_empty());
let e2 = v.iter().any(|s| s.is_empty());
```

---

### `map_flatten`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.31.0 |

#### What it does

Checks for usage of `_.map(_).flatten(_)` on `Iterator` and `Option`

#### Why is this bad?

Readability, this can be written more concisely as
`_.flat_map(_)` for `Iterator` or `_.and_then(_)` for `Option`

#### Example

```rust
let vec = vec![vec![1]];
let opt = Some(5);

vec.iter().map(|x| x.iter()).flatten();
opt.map(|x| Some(x * 2)).flatten();
```

Use instead:

```rust
vec.iter().flat_map(|x| x.iter());
opt.and_then(|x| Some(x * 2));
```

---

### `map_identity`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.47.0 |

#### What it does

Checks for instances of `map(f)` where `f` is the identity function.

#### Why is this bad?

It can be written more concisely without the call to `map`.

#### Example

```rust
let x = [1, 2, 3];
let y: Vec<_> = x.iter().map(|x| x).map(|x| 2*x).collect();
```

Use instead:

```rust
let x = [1, 2, 3];
let y: Vec<_> = x.iter().map(|x| 2*x).collect();
```

---

### `match_as_ref`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for match which is used to add a reference to an
`Option` value.

#### Why is this bad?

Using `as_ref()` or `as_mut()` instead is shorter.

#### Example

```rust
let x: Option<()> = None;

let r: Option<&()> = match x {
    None => None,
    Some(ref v) => Some(v),
};
```

Use instead:

```rust
let x: Option<()> = None;

let r: Option<&()> = x.as_ref();
```

---

### `match_single_binding`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.43.0 |

#### What it does

Checks for useless match that binds to only one value.

#### Why is this bad?

Readability and needless complexity.

#### Known problems

Suggested replacements may be incorrect when `match`
is actually binding temporary value, bringing a ‘dropped while borrowed’ error.

#### Example

```rust
match (a, b) {
    (c, d) => {
        // useless match
    }
}
```

Use instead:

```rust
let (c, d) = (a, b);
```

---

### `needless_arbitrary_self_type`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.47.0 |

#### What it does

The lint checks for `self` in fn parameters that
specify the `Self`-type explicitly

#### Why is this bad?

Increases the amount and decreases the readability of code

#### Example

```rust
enum ValType {
    I32,
    I64,
    F32,
    F64,
}

impl ValType {
    pub fn bytes(self: Self) -> usize {
        match self {
            Self::I32 | Self::F32 => 4,
            Self::I64 | Self::F64 => 8,
        }
    }
}
```

Could be rewritten as

```rust
enum ValType {
    I32,
    I64,
    F32,
    F64,
}

impl ValType {
    pub fn bytes(self) -> usize {
        match self {
            Self::I32 | Self::F32 => 4,
            Self::I64 | Self::F64 => 8,
        }
    }
}
```

---

### `needless_as_bytes`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.84.0 |

#### What it does

It detects useless calls to `str::as_bytes()` before calling `len()` or `is_empty()`.

#### Why is this bad?

The `len()` and `is_empty()` methods are also directly available on strings, and they
return identical results. In particular, `len()` on a string returns the number of
bytes.

#### Example

```rust
let len = "some string".as_bytes().len();
let b = "some string".as_bytes().is_empty();
```

Use instead:

```rust
let len = "some string".len();
let b = "some string".is_empty();
```

---

### `needless_bool`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for expressions of the form `if c { true } else { false }` (or vice versa) and suggests using the condition directly.

#### Why is this bad?

Redundant code.

#### Known problems

Maybe false positives: Sometimes, the two branches are
painstakingly documented (which we, of course, do not detect), so they *may*
have some value. Even then, the documentation can be rewritten to match the
shorter code.

#### Example

```rust
if x {
    false
} else {
    true
}
```

Use instead:

```rust
!x
```

---

### `needless_bool_assign`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.71.0 |

#### What it does

Checks for expressions of the form `if c { x = true } else { x = false }`
(or vice versa) and suggest assigning the variable directly from the
condition.

#### Why is this bad?

Redundant code.

#### Example

```rust
if must_keep(x, y) {
    skip = false;
} else {
    skip = true;
}
```

Use instead:

```rust
skip = !must_keep(x, y);
```

---

### `needless_borrowed_reference`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for bindings that needlessly destructure a reference and borrow the inner
value with `&ref`.

#### Why is this bad?

This pattern has no effect in almost all cases.

#### Example

```rust
let mut v = Vec::<String>::new();
v.iter_mut().filter(|&ref a| a.is_empty());

if let &[ref first, ref second] = v.as_slice() {}
```

Use instead:

```rust
let mut v = Vec::<String>::new();
v.iter_mut().filter(|a| a.is_empty());

if let [first, second] = v.as_slice() {}
```

---

### `needless_ifs`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for empty `if` branches with no else branch.

#### Why is this bad?

It can be entirely omitted, and often the condition too.

#### Known issues

This will usually only suggest to remove the `if` statement, not the condition. Other lints
such as `no_effect` will take care of removing the condition if it’s unnecessary.

#### Example

```rust
if really_expensive_condition(&i) {}
if really_expensive_condition_with_side_effects(&mut i) {}
```

Use instead:

```rust
// <omitted>
really_expensive_condition_with_side_effects(&mut i);
```

#### Past names

- needless_if

---

### `needless_lifetimes`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for lifetime annotations which can be removed by
relying on lifetime elision.

#### Why is this bad?

The additional lifetimes make the code look more
complicated, while there is nothing out of the ordinary going on. Removing
them leads to more readable code.

#### Known problems

This lint ignores functions with `where` clauses that reference
lifetimes to prevent false positives.

#### Example

```rust
// Unnecessary lifetime annotations
fn in_and_out<'a>(x: &'a u8, y: u8) -> &'a u8 {
    x
}
```

Use instead:

```rust
fn elided(x: &u8, y: u8) -> &u8 {
    x
}
```

---

### `needless_match`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.61.0 |

#### What it does

Checks for unnecessary `match` or match-like `if let` returns for `Option` and `Result`
when function signatures are the same.

#### Why is this bad?

This `match` block does nothing and might not be what the coder intended.

#### Example

```rust
fn foo() -> Result<(), i32> {
    match result {
        Ok(val) => Ok(val),
        Err(err) => Err(err),
    }
}

fn bar() -> Option<i32> {
    if let Some(val) = option {
        Some(val)
    } else {
        None
    }
}
```

Could be replaced as

```rust
fn foo() -> Result<(), i32> {
    result
}

fn bar() -> Option<i32> {
    option
}
```

---

### `needless_option_as_deref`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.57.0 |

#### What it does

Checks for no-op uses of `Option::{as_deref, as_deref_mut}`,
for example, `Option<&T>::as_deref()` returns the same type.

#### Why is this bad?

Redundant code and improving readability.

#### Example

```rust
let a = Some(&1);
let b = a.as_deref(); // goes from Option<&i32> to Option<&i32>
```

Use instead:

```rust
let a = Some(&1);
let b = a;
```

---

### `needless_option_take`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.62.0 |

#### What it does

Checks for calling `take` function after `as_ref`.

#### Why is this bad?

Redundant code. `take` writes `None` to its argument.
In this case the modification is useless as it’s a temporary that cannot be read from afterwards.

#### Example

```rust
let x = Some(3);
x.as_ref().take();
```

Use instead:

```rust
let x = Some(3);
x.as_ref();
```

---

### `needless_question_mark`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.51.0 |

#### What it does

Suggests replacing `Ok(x?)` or `Some(x?)` with `x` in return positions where the `?` operator
is not needed to convert the type of `x`.

#### Why is this bad?

There’s no reason to use `?` to short-circuit when execution of the body will end there anyway.

#### Example

```rust
fn f(s: &str) -> Option<usize> {
    Some(s.find('x')?)
}

fn g(s: &str) -> Result<usize, ParseIntError> {
    Ok(s.parse()?)
}
```

Use instead:

```rust
fn f(s: &str) -> Option<usize> {
    s.find('x')
}

fn g(s: &str) -> Result<usize, ParseIntError> {
    s.parse()
}
```

---

### `needless_splitn`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.59.0 |

#### What it does

Checks for usage of `str::splitn` (or `str::rsplitn`) where using `str::split` would be the same.

#### Why is this bad?

The function `split` is simpler and there is no performance difference in these cases, considering
that both functions return a lazy iterator.

#### Example

```rust
let str = "key=value=add";
let _ = str.splitn(3, '=').next().unwrap();
```

Use instead:

```rust
let str = "key=value=add";
let _ = str.split('=').next().unwrap();
```

---

### `needless_update`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for needlessly including a base struct on update
when all fields are changed anyway.

This lint is not applied to structs marked with
[non_exhaustive](https://doc.rust-lang.org/reference/attributes/type_system.html).

#### Why is this bad?

This will cost resources (because the base has to be
somewhere), and make the code less readable.

#### Example

```rust
Point {
    x: 1,
    y: 1,
    z: 1,
    ..zero_point
};
```

Use instead:

```rust
// Missing field `z`
Point {
    x: 1,
    y: 1,
    ..zero_point
};
```

---

### `neg_cmp_op_on_partial_ord`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the usage of negated comparison operators on types which only implement
`PartialOrd` (e.g., `f64`).

#### Why is this bad?

These operators make it easy to forget that the underlying types actually allow not only three
potential Orderings (Less, Equal, Greater) but also a fourth one (Uncomparable). This is
especially easy to miss if the operator based comparison result is negated.

#### Example

```rust
let a = 1.0;
let b = f64::NAN;

let not_less_or_equal = !(a <= b);
```

Use instead:

```rust
use std::cmp::Ordering;

let _not_less_or_equal = match a.partial_cmp(&b) {
    None | Some(Ordering::Greater) => true,
    _ => false,
};
```

---

### `no_effect`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for statements which have no effect.

#### Why is this bad?

Unlike dead code, these statements are actually
executed. However, as they have no effect, all they do is make the code less
readable.

#### Example

```rust
0;
```

---

### `nonminimal_bool`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for boolean expressions that can be written more
concisely.

#### Why is this bad?

Readability of boolean expressions suffers from
unnecessary duplication.

#### Known problems

Ignores short circuiting behavior of `||` and
`&&`. Ignores `|`, `&` and `^`.

#### Example

```rust
if a && true {}
if !(a == b) {}
```

Use instead:

```rust
if a {}
if a != b {}
```

---

### `only_used_in_recursion`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.61.0 |

#### What it does

Checks for arguments that are only used in recursion with no side-effects.

#### Why is this bad?

It could contain a useless calculation and can make function simpler.

The arguments can be involved in calculations and assignments but as long as
the calculations have no side-effects (function calls or mutating dereference)
and the assigned variables are also only in recursion, it is useless.

#### Example

```rust
fn f(a: usize, b: usize) -> usize {
    if a == 0 {
        1
    } else {
        f(a - 1, b + 1)
    }
}
```

Use instead:

```rust
fn f(a: usize) -> usize {
    if a == 0 {
        1
    } else {
        f(a - 1)
    }
}
```

#### Known problems

Too many code paths in the linting code are currently untested and prone to produce false
positives or are prone to have performance implications.

In some cases, this would not catch all useless arguments.

```rust
fn foo(a: usize, b: usize) -> usize {
    let f = |x| x + 1;

    if a == 0 {
        1
    } else {
        foo(a - 1, f(b))
    }
}
```

For example, the argument `b` is only used in recursion, but the lint would not catch it.

List of some examples that can not be caught:

- binary operation of non-primitive types
- closure usage
- some `break` relative operations
- struct pattern binding

Also, when you recurse the function name with path segments, it is not possible to detect.

---

### `option_as_ref_deref`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.42.0 |

#### What it does

Checks for usage of `_.as_ref().map(Deref::deref)` or its aliases (such as String::as_str).

#### Why is this bad?

Readability, this can be written more concisely as
`_.as_deref()`.

#### Example

```rust
opt.as_ref().map(String::as_str)
```

Can be written as

```rust
opt.as_deref()
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `option_filter_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.53.0 |

#### What it does

Checks for iterators of `Option`s using `.filter(Option::is_some).map(Option::unwrap)` that may
be replaced with a `.flatten()` call.

#### Why is this bad?

`Option` is like a collection of 0-1 things, so `flatten`
automatically does this without suspicious-looking `unwrap` calls.

#### Example

```rust
let _ = std::iter::empty::<Option<i32>>().filter(Option::is_some).map(Option::unwrap);
```

Use instead:

```rust
let _ = std::iter::empty::<Option<i32>>().flatten();
```

---

### `option_map_unit_fn`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `option.map(f)` where f is a function
or closure that returns the unit type `()`.

#### Why is this bad?

Readability, this can be written more clearly with
an if let statement

#### Example

```rust
let x: Option<String> = do_stuff();
x.map(log_err_msg);
x.map(|msg| log_err_msg(format_msg(msg)));
```

The correct use would be:

```rust
let x: Option<String> = do_stuff();
if let Some(msg) = x {
    log_err_msg(msg);
}

if let Some(msg) = x {
    log_err_msg(format_msg(msg));
}
```

---

### `or_then_unwrap`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.61.0 |

#### What it does

Checks for `.or(…).unwrap()` calls to Options and Results.

#### Why is this bad?

You should use `.unwrap_or(…)` instead for clarity.

#### Example

```rust
// Result
let value = result.or::<Error>(Ok(fallback)).unwrap();

// Option
let value = option.or(Some(fallback)).unwrap();
```

Use instead:

```rust
// Result
let value = result.unwrap_or(fallback);

// Option
let value = option.unwrap_or(fallback);
```

---

### `partialeq_ne_impl`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for manual re-implementations of `PartialEq::ne`.

#### Why is this bad?

`PartialEq::ne` is required to always return the
negated result of `PartialEq::eq`, which is exactly what the default
implementation does. Therefore, there should never be any need to
re-implement it.

#### Example

```rust
struct Foo;

impl PartialEq for Foo {
    fn eq(&self, other: &Foo) -> bool { true }
    fn ne(&self, other: &Foo) -> bool { !(self == other) }
}
```

---

### `precedence`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for operations where precedence may be unclear and suggests to add parentheses.
It catches a mixed usage of arithmetic and bit shifting/combining operators,
as well as method calls applied to closures.

#### Why is this bad?

Not everyone knows the precedence of those operators by
heart, so expressions like these may trip others trying to reason about the
code.

#### Example

`1 << 2 + 3` equals 32, while `(1 << 2) + 3` equals 7

---

### `ptr_offset_with_cast`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.30.0 |

#### What it does

Checks for usage of the `offset` pointer method with a `usize` casted to an
`isize`.

#### Why is this bad?

If we’re always increasing the pointer address, we can avoid the numeric
cast by using the `add` method instead.

#### Example

```rust
let vec = vec![b'a', b'b', b'c'];
let ptr = vec.as_ptr();
let offset = 1_usize;

unsafe {
    ptr.offset(offset as isize);
}
```

Could be written:

```rust
let vec = vec![b'a', b'b', b'c'];
let ptr = vec.as_ptr();
let offset = 1_usize;

unsafe {
    ptr.add(offset);
}
```

---

### `range_zip_with_len`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for zipping a collection with the range of
`0.._.len()`.

#### Why is this bad?

The code is better expressed with `.enumerate()`.

#### Example

```rust
let _ = x.iter().zip(0..x.len());
```

Use instead:

```rust
let _ = x.iter().enumerate();
```

---

### `redundant_as_str`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.74.0 |

#### What it does

Checks for usage of `as_str()` on a `String` chained with a method available on the `String` itself.

#### Why is this bad?

The `as_str()` conversion is pointless and can be removed for simplicity and cleanliness.

#### Example

```rust
let owned_string = "This is a string".to_owned();
owned_string.as_str().as_bytes()
```

Use instead:

```rust
let owned_string = "This is a string".to_owned();
owned_string.as_bytes()
```

---

### `redundant_async_block`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.70.0 |

#### What it does

Checks for `async` block that only returns `await` on a future.

#### Why is this bad?

It is simpler and more efficient to use the future directly.

#### Example

```rust
let f = async {
    1 + 2
};
let fut = async {
    f.await
};
```

Use instead:

```rust
let f = async {
    1 + 2
};
let fut = f;
```

---

### `redundant_at_rest_pattern`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for `[all @ ..]` patterns.

#### Why is this bad?

In all cases, `all` works fine and can often make code simpler, as you possibly won’t need
to convert from say a `Vec` to a slice by dereferencing.

#### Example

```rust
if let [all @ ..] = &*v {
    // NOTE: Type is a slice here
    println!("all elements: {all:#?}");
}
```

Use instead:

```rust
if let all = v {
    // NOTE: Type is a `Vec` here
    println!("all elements: {all:#?}");
}
// or
println!("all elements: {v:#?}");
```

---

### `redundant_closure_call`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Detects closures called in the same expression where they
are defined.

#### Why is this bad?

It is unnecessarily adding to the expression’s
complexity.

#### Example

```rust
let a = (|| 42)();
```

Use instead:

```rust
let a = 42;
```

---

### `redundant_guards`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.73.0 |

#### What it does

Checks for unnecessary guards in match expressions.

#### Why is this bad?

It’s more complex and much less readable. Making it part of the pattern can improve
exhaustiveness checking as well.

#### Example

```rust
match x {
    Some(x) if matches!(x, Some(1)) => ..,
    Some(x) if x == Some(2) => ..,
    _ => todo!(),
}
```

Use instead:

```rust
match x {
    Some(Some(1)) => ..,
    Some(Some(2)) => ..,
    _ => todo!(),
}
```

---

### `redundant_slicing`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.51.0 |

#### What it does

Checks for redundant slicing expressions which use the full range, and
do not change the type.

#### Why is this bad?

It unnecessarily adds complexity to the expression.

#### Known problems

If the type being sliced has an implementation of `Index<RangeFull>`
that actually changes anything then it can’t be removed. However, this would be surprising
to people reading the code and should have a note with it.

#### Example

```rust
fn get_slice(x: &[u32]) -> &[u32] {
    &x[..]
}
```

Use instead:

```rust
fn get_slice(x: &[u32]) -> &[u32] {
    x
}
```

---

### `repeat_once`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.47.0 |

#### What it does

Checks for usage of `.repeat(1)` and suggest the following method for each types.

- `.to_string()` for `str`
- `.clone()` for `String`
- `.to_vec()` for `slice`

The lint will evaluate constant expressions and values as arguments of `.repeat(..)` and emit a message if
they are equivalent to `1`. (Related discussion in [rust-clippy#7306](https://github.com/rust-lang/rust-clippy/issues/7306))

#### Why is this bad?

For example, `String.repeat(1)` is equivalent to `.clone()`. If cloning
the string is the intention behind this, `clone()` should be used.

#### Example

```rust
fn main() {
    let x = String::from("hello world").repeat(1);
}
```

Use instead:

```rust
fn main() {
    let x = String::from("hello world").clone();
}
```

---

### `reserve_after_initialization`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | 1.74.0 |

#### What it does

Informs the user about a more concise way to create a vector with a known capacity.

#### Why is this bad?

The `Vec::with_capacity` constructor is less complex.

#### Example

```rust
let mut v: Vec<usize> = vec![];
v.reserve(10);
```

Use instead:

```rust
let mut v: Vec<usize> = Vec::with_capacity(10);
```

---

### `result_filter_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.77.0 |

#### What it does

Checks for iterators of `Result`s using `.filter(Result::is_ok).map(Result::unwrap)` that may
be replaced with a `.flatten()` call.

#### Why is this bad?

`Result` implements `IntoIterator<Item = T>`. This means that `Result` can be flattened
automatically without suspicious-looking `unwrap` calls.

#### Example

```rust
let _ = std::iter::empty::<Result<i32, ()>>().filter(Result::is_ok).map(Result::unwrap);
```

Use instead:

```rust
let _ = std::iter::empty::<Result<i32, ()>>().flatten();
```

---

### `result_map_unit_fn`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `result.map(f)` where f is a function
or closure that returns the unit type `()`.

#### Why is this bad?

Readability, this can be written more clearly with
an if let statement

#### Example

```rust
let x: Result<String, String> = do_stuff();
x.map(log_err_msg);
x.map(|msg| log_err_msg(format_msg(msg)));
```

The correct use would be:

```rust
let x: Result<String, String> = do_stuff();
if let Ok(msg) = x {
    log_err_msg(msg);
};
if let Ok(msg) = x {
    log_err_msg(format_msg(msg));
};
```

---

### `seek_from_current`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.67.0 |

#### What it does

Checks if the `seek` method of the `Seek` trait is called with `SeekFrom::Current(0)`,
and if it is, suggests using `stream_position` instead.

#### Why is this bad?

Readability. Use dedicated method.

#### Example

```rust
use std::fs::File;
use std::io::{self, Write, Seek, SeekFrom};

fn main() -> io::Result<()> {
    let mut f = File::create("foo.txt")?;
    f.write_all(b"Hello")?;
    eprintln!("Written {} bytes", f.seek(SeekFrom::Current(0))?);

    Ok(())
}
```

Use instead:

```rust
use std::fs::File;
use std::io::{self, Write, Seek, SeekFrom};

fn main() -> io::Result<()> {
    let mut f = File::create("foo.txt")?;
    f.write_all(b"Hello")?;
    eprintln!("Written {} bytes", f.stream_position()?);

    Ok(())
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `seek_to_start_instead_of_rewind`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.67.0 |

#### What it does

Checks for jumps to the start of a stream that implements `Seek`
and uses the `seek` method providing `Start` as parameter.

#### Why is this bad?

Readability. There is a specific method that was implemented for
this exact scenario.

#### Example

```rust
fn foo<T: io::Seek>(t: &mut T) {
    t.seek(io::SeekFrom::Start(0));
}
```

Use instead:

```rust
fn foo<T: io::Seek>(t: &mut T) {
    t.rewind();
}
```

---

### `short_circuit_statement`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the use of short circuit boolean conditions as
a
statement.

#### Why is this bad?

Using a short circuit boolean condition as a statement
may hide the fact that the second part is executed or not depending on the
outcome of the first part.

#### Example

```rust
f() && g(); // We should write `if f() { g(); }`.
```

---

### `single_element_loop`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.49.0 |

#### What it does

Checks whether a for loop has a single element.

#### Why is this bad?

There is no reason to have a loop of a
single element.

#### Example

```rust
let item1 = 2;
for item in &[item1] {
    println!("{}", item);
}
```

Use instead:

```rust
let item1 = 2;
let item = &item1;
println!("{}", item);
```

---

### `skip_while_next`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.42.0 |

#### What it does

Checks for usage of `_.skip_while(condition).next()`.

#### Why is this bad?

Readability, this can be written more concisely as
`_.find(!condition)`.

#### Example

```rust
vec.iter().skip_while(|x| **x == 0).next();
```

Use instead:

```rust
vec.iter().find(|x| **x != 0);
```

---

### `string_from_utf8_as_bytes`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.50.0 |

#### What it does

Check if the string is transformed to byte array and casted back to string.

#### Why is this bad?

It’s unnecessary, the string can be used directly.

#### Example

```rust
std::str::from_utf8(&"Hello World!".as_bytes()[6..11]).unwrap();
```

Use instead:

```rust
&"Hello World!"[6..11];
```

---

### `strlen_on_c_strings`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.55.0 |

#### What it does

Checks for usage of `libc::strlen` on a `CString` or `CStr` value,
and suggest calling `count_bytes()` instead.

#### Why is this bad?

libc::strlen is an unsafe function, which we don’t need to call
if all we want to know is the length of the c-string.

#### Example

```rust
use std::ffi::CString;
let cstring = CString::new("foo").expect("CString::new failed");
let len = unsafe { libc::strlen(cstring.as_ptr()) };
```

Use instead:

```rust
use std::ffi::CString;
let cstring = CString::new("foo").expect("CString::new failed");
let len = cstring.count_bytes();
```

---

### `swap_with_temporary`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.88.0 |

#### What it does

Checks for usage of `std::mem::swap` with temporary values.

#### Why is this bad?

Storing a new value in place of a temporary value which will
be dropped right after the `swap` is an inefficient way of performing
an assignment. The same result can be achieved by using a regular
assignment.

#### Examples

```rust
fn replace_string(s: &mut String) {
    std::mem::swap(s, &mut String::from("replaced"));
}
```

Use instead:

```rust
fn replace_string(s: &mut String) {
    *s = String::from("replaced");
}
```

Also, swapping two temporary values has no effect, as they will
both be dropped right after swapping them. This is likely an indication
of a bug. For example, the following code swaps the references to
the last element of the vectors, instead of swapping the elements
themselves:

```rust
fn bug(v1: &mut [i32], v2: &mut [i32]) {
    // Incorrect: swapping temporary references (`&mut &mut` passed to swap)
    std::mem::swap(&mut v1.last_mut().unwrap(), &mut v2.last_mut().unwrap());
}
```

Use instead:

```rust
fn correct(v1: &mut [i32], v2: &mut [i32]) {
    std::mem::swap(v1.last_mut().unwrap(), v2.last_mut().unwrap());
}
```

---

### `temporary_assignment`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for construction of a structure or tuple just to
assign a value in it.

#### Why is this bad?

Readability. If the structure is only created to be
updated, why not write the structure you want in the first place?

#### Example

```rust
(0, 0).0 = 1
```

---

### `too_many_arguments`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for functions with too many parameters.

#### Why is this bad?

Functions with lots of parameters are considered bad
style and reduce readability (“what does the 5th parameter mean?”). Consider
grouping some parameters into a new type.

#### Example

```rust
fn foo(x: u32, y: u32, name: &str, c: Color, w: f32, h: f32, a: f32, b: f32) {
    // ..
}
```

#### Configuration

- `too-many-arguments-threshold`:  The maximum number of argument a function or method can have

(default: `7`)

---

### `transmute_bytes_to_str`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for transmutes from a `&[u8]` to a `&str`.

#### Why is this bad?

Not every byte slice is a valid UTF-8 string.

#### Known problems

- [from_utf8](https://doc.rust-lang.org/std/str/fn.from_utf8.html) which this lint suggests using is slower than `transmute`
as it needs to validate the input.
If you are certain that the input is always a valid UTF-8,
use [from_utf8_unchecked](https://doc.rust-lang.org/std/str/fn.from_utf8_unchecked.html) which is as fast as `transmute`
but has a semantically meaningful name.
- You might want to handle errors returned from [from_utf8](https://doc.rust-lang.org/std/str/fn.from_utf8.html) instead of calling `unwrap`.

#### Example

```rust
let b: &[u8] = &[1_u8, 2_u8];
unsafe {
    let _: &str = std::mem::transmute(b); // where b: &[u8]
}

// should be:
let _ = std::str::from_utf8(b).unwrap();
```

---

### `transmute_int_to_bool`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for transmutes from an integer to a `bool`.

#### Why is this bad?

This might result in an invalid in-memory representation of a `bool`.

#### Example

```rust
let x = 1_u8;
unsafe {
    let _: bool = std::mem::transmute(x); // where x: u8
}

// should be:
let _: bool = x != 0;
```

---

### `transmute_int_to_non_zero`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.69.0 |

#### What it does

Checks for transmutes from `T` to `NonZero<T>`, and suggests the `new_unchecked`
method instead.

#### Why is this bad?

Transmutes work on any types and thus might cause unsoundness when those types change
elsewhere. `new_unchecked` only works for the appropriate types instead.

#### Example

```rust
let _: NonZero<u32> = unsafe { std::mem::transmute(123) };
```

Use instead:

```rust
let _: NonZero<u32> = unsafe { NonZero::new_unchecked(123) };
```

---

### `transmute_ptr_to_ref`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for transmutes from a pointer to a reference.

#### Why is this bad?

This can always be rewritten with `&` and `*`.

#### Known problems

- `mem::transmute` in statics and constants is stable from Rust 1.46.0,
while dereferencing raw pointer is not stable yet.
If you need to do this in those places,
you would have to use `transmute` instead.

#### Example

```rust
unsafe {
    let _: &T = std::mem::transmute(p); // where p: *const T
}

// can be written:
let _: &T = &*p;
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `transmutes_expressible_as_ptr_casts`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.47.0 |

#### What it does

Checks for transmutes that could be a pointer cast.

#### Why is this bad?

Readability. The code tricks people into thinking that
something complex is going on.

#### Example

```rust
unsafe { std::mem::transmute::<*const [i32], *const [u16]>(p) };
```

Use instead:

```rust
p as *const [u16];
```

---

### `type_complexity`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for types used in structs, parameters and `let`
declarations above a certain complexity threshold.

#### Why is this bad?

Too complex types make the code less readable. Consider
using a `type` definition to simplify them.

#### Example

```rust
struct PointMatrixContainer {
    matrix: Rc<Vec<Vec<Box<(u32, u32, u32, u32)>>>>,
}

fn main() {
    let point_matrix: Vec<Vec<Box<(u32, u32, u32, u32)>>> = vec![
        vec![
            Box::new((1, 2, 3, 4)),
            Box::new((5, 6, 7, 8)),
        ],
        vec![
            Box::new((9, 10, 11, 12)),
        ],
    ];

    let shared_point_matrix: Rc<Vec<Vec<Box<(u32, u32, u32, u32)>>>> = Rc::new(point_matrix);

    let container = PointMatrixContainer {
        matrix: shared_point_matrix,
    };

    // ...
}
```

Use instead:

#### Example

```rust
type PointMatrix = Vec<Vec<Box<(u32, u32, u32, u32)>>>;
type SharedPointMatrix = Rc<PointMatrix>;

struct PointMatrixContainer {
    matrix: SharedPointMatrix,
}

fn main() {
    let point_matrix: PointMatrix = vec![
        vec![
            Box::new((1, 2, 3, 4)),
            Box::new((5, 6, 7, 8)),
        ],
        vec![
            Box::new((9, 10, 11, 12)),
        ],
    ];

    let shared_point_matrix: SharedPointMatrix = Rc::new(point_matrix);

    let container = PointMatrixContainer {
        matrix: shared_point_matrix,
    };

    // ...
}
```

#### Configuration

- `type-complexity-threshold`:  The maximum complexity a type can have

(default: `250`)

---

### `unit_arg`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for passing a unit value as an argument to a function without using a
unit literal (`()`).

#### Why is this bad?

This is likely the result of an accidental semicolon.

#### Example

```rust
foo({
    let a = bar();
    baz(a);
})
```

---

### `unnecessary_cast`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts to the same type, casts of int literals to integer
types, casts of float literals to float types, and casts between raw
pointers that don’t change type or constness.

#### Why is this bad?

It’s just unnecessary.

#### Known problems

When the expression on the left is a function call, the lint considers
the return type to be a type alias if it’s aliased through a `use`
statement (like `use std::io::Result as IoResult`). It will not lint
such cases.

This check will only work on primitive types without any intermediate
references: raw pointers and trait objects may or may not work.

#### Example

```rust
let _ = 2i32 as i32;
let _ = 0.5 as f32;
```

Better:

```rust
let _ = 2_i32;
let _ = 0.5_f32;
```

---

### `unnecessary_filter_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.31.0 |

#### What it does

Checks for `filter_map` calls that could be replaced by `filter` or `map`.
More specifically it checks if the closure provided is only performing one of the
filter or map operations and suggests the appropriate option.

#### Why is this bad?

Complexity. The intent is also clearer if only a single
operation is being performed.

#### Example

```rust
let _ = (0..3).filter_map(|x| if x > 2 { Some(x) } else { None });

// As there is no transformation of the argument this could be written as:
let _ = (0..3).filter(|&x| x > 2);
```

```rust
let _ = (0..4).filter_map(|x| Some(x + 1));

// As there is no conditional check on the argument this could be written as:
let _ = (0..4).map(|x| x + 1);
```

---

### `unnecessary_find_map`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.61.0 |

#### What it does

Checks for `find_map` calls that could be replaced by `find` or `map`. More
specifically it checks if the closure provided is only performing one of the
find or map operations and suggests the appropriate option.

#### Why is this bad?

Complexity. The intent is also clearer if only a single
operation is being performed.

#### Example

```rust
let _ = (0..3).find_map(|x| if x > 2 { Some(x) } else { None });

// As there is no transformation of the argument this could be written as:
let _ = (0..3).find(|&x| x > 2);
```

```rust
let _ = (0..4).find_map(|x| Some(x + 1));

// As there is no conditional check on the argument this could be written as:
let _ = (0..4).map(|x| x + 1).next();
```

---

### `unnecessary_first_then_check`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.83.0 |

#### What it does

Checks the usage of `.first().is_some()` or `.first().is_none()` to check if a slice is
empty.

#### Why is this bad?

Using `.is_empty()` is shorter and better communicates the intention.

#### Example

```rust
let v = vec![1, 2, 3];
if v.first().is_none() {
    // The vector is empty...
}
```

Use instead:

```rust
let v = vec![1, 2, 3];
if v.is_empty() {
    // The vector is empty...
}
```

---

### `unnecessary_literal_unwrap`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for `.unwrap()` related calls on `Result`s and `Option`s that are constructed.

#### Why is this bad?

It is better to write the value directly without the indirection.

#### Examples

```rust
let val1 = Some(1).unwrap();
let val2 = Ok::<_, ()>(1).unwrap();
let val3 = Err::<(), _>(1).unwrap_err();
```

Use instead:

```rust
let val1 = 1;
let val2 = 1;
let val3 = 1;
```

---

### `unnecessary_map_on_constructor`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.74.0 |

#### What it does

Suggests removing the use of a `map()` (or `map_err()`) method when an `Option` or `Result`
is being constructed.

#### Why is this bad?

It introduces unnecessary complexity. Instead, the function can be called before
constructing the `Option` or `Result` from its return value.

#### Example

```rust
Some(4).map(i32::swap_bytes)
```

Use instead:

```rust
Some(i32::swap_bytes(4))
```

---

### `unnecessary_min_or_max`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.81.0 |

#### What it does

Checks for unnecessary calls to `min()` or `max()` in the following cases

- Either both side is constant
- One side is clearly larger than the other, like i32::MIN and an i32 variable

#### Why is this bad?

In the aforementioned cases it is not necessary to call `min()` or `max()`
to compare values, it may even cause confusion.

#### Example

```rust
let _ = 0.min(7_u32);
```

Use instead:

```rust
let _ = 0;
```

---

### `unnecessary_operation`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for expression statements that can be reduced to a
sub-expression.

#### Why is this bad?

Expressions by themselves often have no side-effects.
Having such expressions reduces readability.

#### Example

```rust
compute_array()[0];
```

---

### `unnecessary_sort_by`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.46.0 |

#### What it does

Checks for usage of `Vec::sort_by` passing in a closure
which compares the two arguments, either directly or indirectly.

#### Why is this bad?

It is more clear to use `Vec::sort_by_key` (or `Vec::sort` if
possible) than to use `Vec::sort_by` and a more complicated
closure.

#### Known problems

If the suggested `Vec::sort_by_key` uses Reverse and it isn’t already
imported by a use statement, then it will need to be added manually.

#### Example

```rust
vec.sort_by(|a, b| a.foo().cmp(&b.foo()));
```

Use instead:

```rust
vec.sort_by_key(|a| a.foo());
```

---

### `unnecessary_unwrap`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for calls of `unwrap[_err]()` that cannot fail.

#### Why is this bad?

Using `if let` or `match` is more idiomatic.

#### Example

```rust
if option.is_some() {
    do_something_with(option.unwrap())
}
```

Could be written:

```rust
if let Some(value) = option {
    do_something_with(value)
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `unneeded_wildcard_pattern`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.40.0 |

#### What it does

Checks for tuple patterns with a wildcard
pattern (`_`) is next to a rest pattern (`..`).

*NOTE*: While `_, ..` means there is at least one element left, `..`
means there are 0 or more elements left. This can make a difference
when refactoring, but shouldn’t result in errors in the refactored code,
since the wildcard pattern isn’t used anyway.

#### Why is this bad?

The wildcard pattern is unneeded as the rest pattern
can match that element as well.

#### Example

```rust
match t {
    TupleStruct(0, .., _) => (),
    _ => (),
}
```

Use instead:

```rust
match t {
    TupleStruct(0, ..) => (),
    _ => (),
}
```

---

### `unused_format_specs`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.66.0 |

#### What it does

Detects [formatting parameters](https://doc.rust-lang.org/std/fmt/index.html#formatting-parameters) that have no effect on the output of
`format!()`, `println!()` or similar macros.

#### Why is this bad?

Shorter format specifiers are easier to read, it may also indicate that
an expected formatting operation such as adding padding isn’t happening.

#### Example

```rust
println!("{:.}", 1.0);

println!("not padded: {:5}", format_args!("..."));
```

Use instead:

```rust
println!("{}", 1.0);

println!("not padded: {}", format_args!("..."));
// OR
println!("padded: {:5}", format!("..."));
```

---

### `useless_asref`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `.as_ref()` or `.as_mut()` where the
types before and after the call are the same.

#### Why is this bad?

The call is unnecessary.

#### Example

```rust
let x: &[i32] = &[1, 2, 3, 4, 5];
do_stuff(x.as_ref());
```

The correct use would be:

```rust
let x: &[i32] = &[1, 2, 3, 4, 5];
do_stuff(x);
```

---

### `useless_concat`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.89.0 |

#### What it does

Checks that the `concat!` macro has at least two arguments.

#### Why is this bad?

If there are less than 2 arguments, then calling the macro is doing nothing.

#### Example

```rust
let x = concat!("a");
```

Use instead:

```rust
let x = "a";
```

---

### `useless_conversion`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.45.0 |

#### What it does

Checks for `Into`, `TryInto`, `From`, `TryFrom`, or `IntoIter` calls
which uselessly convert to the same type.

#### Why is this bad?

Redundant code.

#### Example

```rust
// format!() returns a `String`
let s: String = format!("hello").into();
```

Use instead:

```rust
let s: String = format!("hello");
```

#### Past names

- identity_conversion

---

### `useless_format`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the use of `format!("string literal with no argument")` and `format!("{}", foo)` where `foo` is a string.

#### Why is this bad?

There is no point of doing that. `format!("foo")` can
be replaced by `"foo".to_owned()` if you really need a `String`. The even
worse `&format!("foo")` is often encountered in the wild. `format!("{}", foo)` can be replaced by `foo.clone()` if `foo: String` or `foo.to_owned()`
if `foo: &str`.

#### Examples

```rust
let foo = "foo";
format!("{}", foo);
```

Use instead:

```rust
let foo = "foo";
foo.to_owned();
```

---

### `useless_nonzero_new_unchecked`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Checks for `NonZero*::new_unchecked()` being used in a `const` context.

#### Why is this bad?

Using `NonZero*::new_unchecked()` is an `unsafe` function and requires an `unsafe` context. When used in a
context evaluated at compilation time, `NonZero*::new().unwrap()` will provide the same result with identical
runtime performances while not requiring `unsafe`.

#### Example

```rust
use std::num::NonZeroUsize;
const PLAYERS: NonZeroUsize = unsafe { NonZeroUsize::new_unchecked(3) };
```

Use instead:

```rust
use std::num::NonZeroUsize;
const PLAYERS: NonZeroUsize = NonZeroUsize::new(3).unwrap();
```

---

### `useless_transmute`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for transmutes to the original type of the object
and transmutes that could be a cast.

#### Why is this bad?

Readability. The code tricks people into thinking that
something complex is going on.

#### Example

```rust
core::intrinsics::transmute(t); // where the result type is the same as `t`'s
```

---

### `vec_box`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.33.0 |

#### What it does

Checks for usage of `Vec<Box<T>>` where T: Sized anywhere in the code.
Check the [Box documentation](https://doc.rust-lang.org/std/boxed/index.html) for more information.

#### Why is this bad?

`Vec` already keeps its contents in a separate area on
the heap. So if you `Box` its contents, you just add another level of indirection.

#### Example

```rust
struct X {
    values: Vec<Box<i32>>,
}
```

Better:

```rust
struct X {
    values: Vec<i32>,
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)
- `vec-box-size-threshold`:  The size of the boxed type in bytes, where boxing in a `Vec` is allowed

(default: `4096`)

---

### `while_let_loop`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | pre 1.29.0 |

#### What it does

Detects `loop + match` combinations that are easier
written as a `while let` loop.

#### Why is this bad?

The `while let` loop is usually shorter and more
readable.

#### Example

```rust
let y = Some(1);
loop {
    let x = match y {
        Some(x) => x,
        None => break,
    };
    // ..
}
```

Use instead:

```rust
let y = Some(1);
while let Some(x) = y {
    // ..
};
```

---

### `wildcard_in_or_patterns`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.42.0 |

#### What it does

Checks for wildcard pattern used with others patterns in same match arm.

#### Why is this bad?

Wildcard pattern already covers any other pattern as it will match anyway.
It makes the code less readable, especially to spot wildcard pattern use in match arm.

#### Example

```rust
match s {
    "a" => {},
    "bar" | _ => {},
}
```

Use instead:

```rust
match s {
    "a" => {},
    _ => {},
}
```

---

### `zero_divided_by_zero`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `0.0 / 0.0`.

#### Why is this bad?

It’s less readable than `f32::NAN` or `f64::NAN`.

#### Example

```rust
let nan = 0.0f32 / 0.0;
```

Use instead:

```rust
let nan = f32::NAN;
```

---

### `zero_prefixed_literal`

| 属性 | 值 |
|------|----|
| 分组 | complexity |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if an integral constant literal starts with `0`.

#### Why is this bad?

In some languages (including the infamous C language
and most of its
family), this marks an octal constant. In Rust however, this is a decimal
constant. This could
be confusing for both the writer and a reader of the constant.

#### Example

In Rust:

```rust
fn main() {
    let a = 0123;
    println!("{}", a);
}
```

prints `123`, while in C:

```c
#include <stdio.h>

int main() {
    int a = 0123;
    printf("%d\n", a);
}
```

prints `83` (as `83 == 0o123` while `123 == 0o173`).

---

## Perf (36)

*性能 - 可以用更快方式编写的代码，默认 warn*

### `box_collection`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

Checks for usage of `Box<T>` where T is a collection such as Vec anywhere in the code.
Check the [Box documentation](https://doc.rust-lang.org/std/boxed/index.html) for more information.

#### Why is this bad?

Collections already keeps their contents in a separate area on
the heap. So if you `Box` them, you just add another level of indirection
without any benefit whatsoever.

#### Example

```rust
struct X {
    values: Box<Vec<Foo>>,
}
```

Better:

```rust
struct X {
    values: Vec<Foo>,
}
```

#### Past names

- box_vec

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `boxed_local`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `Box<T>` where an unboxed `T` would
work fine.

#### Why is this bad?

This is an unnecessary allocation, and bad for
performance. It is only necessary to allocate if you wish to move the box
into something.

#### Example

```rust
fn foo(x: Box<u32>) {}
```

Use instead:

```rust
fn foo(x: u32) {}
```

#### Configuration

- `too-large-for-stack`:  The maximum size of objects (in bytes) that will be linted. Larger objects are ok on the heap

(default: `200`)

---

### `cloned_ref_to_slice_refs`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.89.0 |

#### What it does

Checks for slice references with cloned references such as `&[f.clone()]`.

#### Why is this bad

A reference does not need to be owned in order to be used as a slice.

#### Known problems

This lint does not know whether or not a clone implementation has side effects.

#### Example

```rust
let data = 10;
let data_ref = &data;
take_slice(&[data_ref.clone()]);
```

Use instead:

```rust
use std::slice;
let data = 10;
let data_ref = &data;
take_slice(slice::from_ref(data_ref));
```

---

### `cmp_owned`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for conversions to owned values just for the sake
of a comparison.

#### Why is this bad?

The comparison can operate on a reference, so creating
an owned value effectively throws it away directly afterwards, which is
needlessly consuming code and heap space.

#### Example

```rust
if x.to_owned() == y {}
```

Use instead:

```rust
if x == y {}
```

---

### `collapsible_str_replace`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.65.0 |

#### What it does

Checks for consecutive calls to `str::replace` (2 or more)
that can be collapsed into a single call.

#### Why is this bad?

Consecutive `str::replace` calls scan the string multiple times
with repetitive code.

#### Example

```rust
let hello = "hesuo worpd"
    .replace('s', "l")
    .replace("u", "l")
    .replace('p', "l");
```

Use instead:

```rust
let hello = "hesuo worpd".replace(['s', 'u', 'p'], "l");
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `double_ended_iterator_last`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Checks for `Iterator::last` being called on a  `DoubleEndedIterator`, which can be replaced
with `DoubleEndedIterator::next_back`.

#### Why is this bad?

`Iterator::last` is implemented by consuming the iterator, which is unnecessary if
the iterator is a `DoubleEndedIterator`. Since Rust traits do not allow specialization,
`Iterator::last` cannot be optimized for `DoubleEndedIterator`.

#### Example

```rust
let last_arg = "echo hello world".split(' ').last();
```

Use instead:

```rust
let last_arg = "echo hello world".split(' ').next_back();
```

---

### `drain_collect`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for calls to `.drain()` that clear the collection, immediately followed by a call to `.collect()`.

> “Collection” in this context refers to any type with a `drain` method:
> `Vec`, `VecDeque`, `BinaryHeap`, `HashSet`,`HashMap`, `String`

#### Why is this bad?

Using `mem::take` is faster as it avoids the allocation.
When using `mem::take`, the old collection is replaced with an empty one and ownership of
the old collection is returned.

#### Known issues

`mem::take(&mut vec)` is almost equivalent to `vec.drain(..).collect()`, except that
it also moves the **capacity**. The user might have explicitly written it this way
to keep the capacity on the original `Vec`.

#### Example

```rust
fn remove_all(v: &mut Vec<i32>) -> Vec<i32> {
    v.drain(..).collect()
}
```

Use instead:

```rust
use std::mem;
fn remove_all(v: &mut Vec<i32>) -> Vec<i32> {
    mem::take(v)
}
```

---

### `expect_fun_call`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for calls to `.expect(&format!(...))`, `.expect(foo(..))`,
etc., and suggests to use `unwrap_or_else` instead

#### Why is this bad?

The function will always be called.

#### Known problems

If the function has side-effects, not calling it will
change the semantics of the program, but you shouldn’t rely on that anyway.

#### Example

```rust
foo.expect(&format!("Err {}: {}", err_code, err_msg));

// or

foo.expect(format!("Err {}: {}", err_code, err_msg).as_str());
```

Use instead:

```rust
foo.unwrap_or_else(|| panic!("Err {}: {}", err_code, err_msg));
```

---

### `extend_with_drain`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.55.0 |

#### What it does

Checks for occurrences where one vector gets extended instead of append

#### Why is this bad?

Using `append` instead of `extend` is more concise and faster

#### Example

```rust
let mut a = vec![1, 2, 3];
let mut b = vec![4, 5, 6];

a.extend(b.drain(..));
```

Use instead:

```rust
let mut a = vec![1, 2, 3];
let mut b = vec![4, 5, 6];

a.append(&mut b);
```

---

### `format_in_format_args`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.58.0 |

#### What it does

Detects `format!` within the arguments of another macro that does
formatting such as `format!` itself, `write!` or `println!`. Suggests
inlining the `format!` call.

#### Why is this bad?

The recommended code is both shorter and avoids a temporary allocation.

#### Example

```rust
println!("error: {}", format!("something failed at {}", Location::caller()));
```

Use instead:

```rust
println!("error: something failed at {}", Location::caller());
```

---

### `iter_overeager_cloned`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.60.0 |

#### What it does

Checks for usage of `_.cloned().<func>()` where call to `.cloned()` can be postponed.

#### Why is this bad?

It’s often inefficient to clone all elements of an iterator, when eventually, only some
of them will be consumed.

#### Known Problems

This `lint` removes the side of effect of cloning items in the iterator.
A code that relies on that side-effect could fail.

#### Examples

```rust
vec.iter().cloned().take(10);
vec.iter().cloned().last();
```

Use instead:

```rust
vec.iter().take(10).cloned();
vec.iter().last().cloned();
```

---

### `large_const_arrays`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.44.0 |

#### What it does

Checks for large `const` arrays that should
be defined as `static` instead.

#### Why is this bad?

Performance: const variables are inlined upon use.
Static items result in only one instance and has a fixed location in memory.

#### Example

```rust
pub const a = [0u32; 1_000_000];
```

Use instead:

```rust
pub static a = [0u32; 1_000_000];
```

#### Configuration

- `array-size-threshold`:  The maximum allowed size for arrays on the stack

(default: `16384`)

---

### `large_enum_variant`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for large size differences between variants on
`enum`s.

#### Why is this bad?

Enum size is bounded by the largest variant. Having one
large variant can penalize the memory layout of that enum.

#### Known problems

This lint obviously cannot take the distribution of
variants in your running program into account. It is possible that the
smaller variants make up less than 1% of all instances, in which case
the overhead is negligible and the boxing is counter-productive. Always
measure the change this lint suggests.

For types that implement `Copy`, the suggestion to `Box` a variant’s
data would require removing the trait impl. The types can of course
still be `Clone`, but that is worse ergonomically. Depending on the
use case it may be possible to store the large data in an auxiliary
structure (e.g. Arena or ECS).

The lint will ignore the impact of generic types to the type layout by
assuming every type parameter is zero-sized. Depending on your use case,
this may lead to a false positive.

#### Example

```rust
enum Test {
    A(i32),
    B([i32; 8000]),
}
```

Use instead:

```rust
// Possibly better
enum Test2 {
    A(i32),
    B(Box<[i32; 8000]>),
}
```

#### Configuration

- `enum-variant-size-threshold`:  The maximum size of an enum’s variant to avoid box suggestion

(default: `200`)

---

### `manual_contains`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.87.0 |

#### What it does

Checks for usage of `iter().any()` on slices when it can be replaced with `contains()` and suggests doing so.

#### Why is this bad?

`contains()` is more concise and idiomatic, while also being faster in some cases.

#### Example

```rust
fn foo(values: &[u8]) -> bool {
    values.iter().any(|&v| v == 10)
}
```

Use instead:

```rust
fn foo(values: &[u8]) -> bool {
    values.contains(&10)
}
```

---

### `manual_ignore_case_cmp`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.84.0 |

#### What it does

Checks for manual case-insensitive ASCII comparison.

#### Why is this bad?

The `eq_ignore_ascii_case` method is faster because it does not allocate
memory for the new strings, and it is more readable.

#### Example

```rust
fn compare(a: &str, b: &str) -> bool {
    a.to_ascii_lowercase() == b.to_ascii_lowercase() || a.to_ascii_lowercase() == "abc"
}
```

Use instead:

```rust
fn compare(a: &str, b: &str) -> bool {
    a.eq_ignore_ascii_case(b) || a.eq_ignore_ascii_case("abc")
}
```

---

### `manual_memcpy`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for for-loops that manually copy items between
slices that could be optimized by having a memcpy.

#### Why is this bad?

It is not as fast as a memcpy.

#### Example

```rust
for i in 0..src.len() {
    dst[i + 64] = src[i];
}
```

Use instead:

```rust
dst[64..(src.len() + 64)].clone_from_slice(&src[..]);
```

---

### `manual_retain`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Checks for code to be replaced by `.retain()`.

#### Why is this bad?

`.retain()` is simpler and avoids needless allocation.

#### Example

```rust
let mut vec = vec![0, 1, 2];
vec = vec.iter().filter(|&x| x % 2 == 0).copied().collect();
vec = vec.into_iter().filter(|x| x % 2 == 0).collect();
```

Use instead:

```rust
let mut vec = vec![0, 1, 2];
vec.retain(|x| x % 2 == 0);
vec.retain(|x| x % 2 == 0);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_str_repeat`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.54.0 |

#### What it does

Checks for manual implementations of `str::repeat`

#### Why is this bad?

These are both harder to read, as well as less performant.

#### Example

```rust
let x: String = std::iter::repeat('x').take(10).collect();
```

Use instead:

```rust
let x: String = "x".repeat(10);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_try_fold`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | 1.72.0 |

#### What it does

Checks for usage of `Iterator::fold` with a type that implements `Try`.

#### Why is this bad?

The code should use `try_fold` instead, which short-circuits on failure, thus opening the
door for additional optimizations not possible with `fold` as rustc can guarantee the
function is never called on `None`, `Err`, etc., alleviating otherwise necessary checks. It’s
also slightly more idiomatic.

#### Known issues

This lint doesn’t take into account whether a function does something on the failure case,
i.e., whether short-circuiting will affect behavior. Refactoring to `try_fold` is not
desirable in those cases.

#### Example

```rust
vec![1, 2, 3].iter().fold(Some(0i32), |sum, i| sum?.checked_add(*i));
```

Use instead:

```rust
vec![1, 2, 3].iter().try_fold(0i32, |sum, i| sum.checked_add(*i));
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `map_entry`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `contains_key` + `insert` on `HashMap`
or `BTreeMap`.

#### Why is this bad?

Using `entry` is more efficient.

#### Known problems

The suggestion may have type inference errors in some cases. e.g.

```rust
let mut map = std::collections::HashMap::new();
let _ = if !map.contains_key(&0) {
    map.insert(0, 0)
} else {
    None
};
```

#### Example

```rust
if !map.contains_key(&k) {
    map.insert(k, v);
}
```

Use instead:

```rust
map.entry(k).or_insert(v);
```

---

### `missing_const_for_thread_local`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.77.0 |

#### What it does

Suggests to use `const` in `thread_local!` macro if possible.

#### Why is this bad?

The `thread_local!` macro wraps static declarations and makes them thread-local.
It supports using a `const` keyword that may be used for declarations that can
be evaluated as a constant expression. This can enable a more efficient thread
local implementation that can avoid lazy initialization. For types that do not
need to be dropped, this can enable an even more efficient implementation that
does not need to track any additional state.

https://doc.rust-lang.org/std/macro.thread_local.html

#### Example

```rust
thread_local! {
    static BUF: String = String::new();
}
```

Use instead:

```rust
thread_local! {
    static BUF: String = const { String::new() };
}
```

#### Past names

- thread_local_initializer_can_be_made_const

---

### `missing_spin_loop`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.61.0 |

#### What it does

Checks for empty spin loops

#### Why is this bad?

The loop body should have something like `thread::park()` or at least
`std::hint::spin_loop()` to avoid needlessly burning cycles and conserve
energy. Perhaps even better use an actual lock, if possible.

#### Known problems

This lint doesn’t currently trigger on `while let` or
`loop { match .. { .. } }` loops, which would be considered idiomatic in
combination with e.g. `AtomicBool::compare_exchange_weak`.

#### Example

```rust
use core::sync::atomic::{AtomicBool, Ordering};
let b = AtomicBool::new(true);
// give a ref to `b` to another thread,wait for it to become false
while b.load(Ordering::Acquire) {};
```

Use instead:

```rust
while b.load(Ordering::Acquire) {
    std::hint::spin_loop()
}
```

---

### `readonly_write_lock`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.73.0 |

#### What it does

Looks for calls to `RwLock::write` where the lock is only used for reading.

#### Why is this bad?

The write portion of `RwLock` is exclusive, meaning that no other thread
can access the lock while this writer is active.

#### Example

```rust
use std::sync::RwLock;
fn assert_is_zero(lock: &RwLock<i32>) {
    let num = lock.write().unwrap();
    assert_eq!(*num, 0);
}
```

Use instead:

```rust
use std::sync::RwLock;
fn assert_is_zero(lock: &RwLock<i32>) {
    let num = lock.read().unwrap();
    assert_eq!(*num, 0);
}
```

---

### `redundant_allocation`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.44.0 |

#### What it does

Checks for usage of redundant allocations anywhere in the code.

#### Why is this bad?

Expressions such as `Rc<&T>`, `Rc<Rc<T>>`, `Rc<Arc<T>>`, `Rc<Box<T>>`, `Arc<&T>`, `Arc<Rc<T>>`,
`Arc<Arc<T>>`, `Arc<Box<T>>`, `Box<&T>`, `Box<Rc<T>>`, `Box<Arc<T>>`, `Box<Box<T>>`, add an unnecessary level of indirection.

#### Example

```rust
fn foo(bar: Rc<&usize>) {}
```

Better:

```rust
fn foo(bar: &usize) {}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `redundant_iter_cloned`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.92.0 |

#### What it does

Checks for calls to `Iterator::cloned` where the original value could be used
instead.

#### Why is this bad?

It is not always possible for the compiler to eliminate useless allocations and
deallocations generated by redundant `clone()`s.

#### Example

```rust
let x = vec![String::new()];
let _ = x.iter().cloned().map(|x| x.len());
```

Use instead:

```rust
let x = vec![String::new()];
let _ = x.iter().map(|x| x.len());
```

---

### `regex_creation_in_loops`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.84.0 |

#### What it does

Checks for [regex](https://crates.io/crates/regex) compilation inside a loop with a literal.

#### Why is this bad?

Compiling a regex is a much more expensive operation than using one, and a compiled regex can be used multiple times.
This is documented as an antipattern [on the regex documentation](https://docs.rs/regex/latest/regex/#avoid-re-compiling-regexes-especially-in-a-loop)

#### Example

```rust
for haystack in haystacks {
    let regex = regex::Regex::new(MY_REGEX).unwrap();
    if regex.is_match(haystack) {
        // Perform operation
    }
}
```

can be replaced with

```rust
let regex = regex::Regex::new(MY_REGEX).unwrap();
for haystack in haystacks {
    if regex.is_match(haystack) {
        // Perform operation
    }
}
```

---

### `replace_box`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.92.0 |

#### What it does

Detects assignments of `Default::default()` or `Box::new(value)`
to a place of type `Box<T>`.

#### Why is this bad?

This incurs an extra heap allocation compared to assigning the boxed
storage.

#### Example

```rust
let mut b = Box::new(1u32);
b = Default::default();
```

Use instead:

```rust
let mut b = Box::new(1u32);
*b = Default::default();
```

---

### `result_large_err`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.65.0 |

#### What it does

Checks for functions that return `Result` with an unusually large
`Err`-variant.

#### Why is this bad?

A `Result` is at least as large as the `Err`-variant. While we
expect that variant to be seldom used, the compiler needs to reserve
and move that much memory every single time.
Furthermore, errors are often simply passed up the call-stack, making
use of the `?`-operator and its type-conversion mechanics. If the
`Err`-variant further up the call-stack stores the `Err`-variant in
question (as library code often does), it itself needs to be at least
as large, propagating the problem.

#### Known problems

The size determined by Clippy is platform-dependent.

#### Examples

```rust
pub enum ParseError {
    UnparsedBytes([u8; 512]),
    UnexpectedEof,
}

// The `Result` has at least 512 bytes, even in the `Ok`-case
pub fn parse() -> Result<(), ParseError> {
    Ok(())
}
```

should be

```rust
pub enum ParseError {
    UnparsedBytes(Box<[u8; 512]>),
    UnexpectedEof,
}

// The `Result` is slightly larger than a pointer
pub fn parse() -> Result<(), ParseError> {
    Ok(())
}
```

#### Configuration

- `large-error-ignored`:  A list of paths to types that should be ignored as overly large `Err`-variants in a
`Result` returned from a function

(default: `[]`)
- `large-error-threshold`:  The maximum size of the `Err`-variant in a `Result` returned from a function

(default: `128`)

---

### `sliced_string_as_bytes`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.86.0 |

#### What it does

Checks for string slices immediately followed by `as_bytes`.

#### Why is this bad?

It involves doing an unnecessary UTF-8 alignment check which is less efficient, and can cause a panic.

#### Known problems

In some cases, the UTF-8 validation and potential panic from string slicing may be required for
the code’s correctness. If you need to ensure the slice boundaries fall on valid UTF-8 character
boundaries, the original form (`s[1..5].as_bytes()`) should be preferred.

#### Example

```rust
let s = "Lorem ipsum";
s[1..5].as_bytes();
```

Use instead:

```rust
let s = "Lorem ipsum";
&s.as_bytes()[1..5];
```

---

### `slow_vector_initialization`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.32.0 |

#### What it does

Checks slow zero-filled vector initialization

#### Why is this bad?

These structures are non-idiomatic and less efficient than simply using
`vec![0; len]`.

Specifically, for `vec![0; len]`, the compiler can use a specialized type of allocation
that also zero-initializes the allocated memory in the same call
(see: [alloc_zeroed](https://doc.rust-lang.org/stable/std/alloc/trait.GlobalAlloc.html#method.alloc_zeroed)).

Writing `Vec::new()` followed by `vec.resize(len, 0)` is suboptimal because,
while it does do the same number of allocations,
it involves two operations for allocating and initializing.
The `resize` call first allocates memory (since `Vec::new()` did not), and only *then* zero-initializes it.

#### Example

```rust
let mut vec1 = Vec::new();
vec1.resize(len, 0);

let mut vec2 = Vec::with_capacity(len);
vec2.resize(len, 0);

let mut vec3 = Vec::with_capacity(len);
vec3.extend(repeat(0).take(len));
```

Use instead:

```rust
let mut vec1 = vec![0; len];
let mut vec2 = vec![0; len];
let mut vec3 = vec![0; len];
```

---

### `to_string_in_format_args`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.58.0 |

#### What it does

Checks for [ToString::to_string](https://doc.rust-lang.org/std/string/trait.ToString.html#tymethod.to_string)
applied to a type that implements [Display](https://doc.rust-lang.org/std/fmt/trait.Display.html)
in a macro that does formatting.

#### Why is this bad?

Since the type implements `Display`, the use of `to_string` is
unnecessary.

#### Example

```rust
println!("error: something failed at {}", Location::caller().to_string());
```

Use instead:

```rust
println!("error: something failed at {}", Location::caller());
```

---

### `unbuffered_bytes`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | Unspecified |
| 引入版本 | 1.87.0 |

#### What it does

Checks for calls to `Read::bytes` on types which don’t implement `BufRead`.

#### Why is this bad?

The default implementation calls `read` for each byte, which can be very inefficient for data that’s not in memory, such as `File`.

#### Example

```rust
use std::io::Read;
use std::fs::File;
let file = File::open("./bytes.txt").unwrap();
file.bytes();
```

Use instead:

```rust
use std::io::{BufReader, Read};
use std::fs::File;
let file = BufReader::new(File::open("./bytes.txt").unwrap());
file.bytes();
```

---

### `unnecessary_to_owned`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.59.0 |

#### What it does

Checks for unnecessary calls to [ToOwned::to_owned](https://doc.rust-lang.org/std/borrow/trait.ToOwned.html#tymethod.to_owned)
and other `to_owned`-like functions.

#### Why is this bad?

The unnecessary calls result in useless allocations.

#### Known problems

`unnecessary_to_owned` can falsely trigger if `IntoIterator::into_iter` is applied to an
owned copy of a resource and the resource is later used mutably. See
[#8148](https://github.com/rust-lang/rust-clippy/issues/8148).

#### Example

```rust
let path = std::path::Path::new("x");
foo(&path.to_string_lossy().to_string());
fn foo(s: &str) {}
```

Use instead:

```rust
let path = std::path::Path::new("x");
foo(&path.to_string_lossy());
fn foo(s: &str) {}
```

---

### `useless_vec`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `vec![..]` when using `[..]` would
be possible.

#### Why is this bad?

This is less efficient.

#### Example

```rust
fn foo(_x: &[u8]) {}

foo(&vec![1, 2]);
```

Use instead:

```rust
foo(&[1, 2]);
```

#### Configuration

- `allow-useless-vec-in-tests`:  Whether `useless_vec` should ignore test functions or `#[cfg(test)]`

(default: `false`)
- `too-large-for-stack`:  The maximum size of objects (in bytes) that will be linted. Larger objects are ok on the heap

(default: `200`)

---

### `vec_init_then_push`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | HasPlaceholders |
| 引入版本 | 1.51.0 |

#### What it does

Checks for calls to `push` immediately after creating a new `Vec`.

If the `Vec` is created using `with_capacity` this will only lint if the capacity is a
constant and the number of pushes is greater than or equal to the initial capacity.

If the `Vec` is extended after the initial sequence of pushes and it was default initialized
then this will only lint after there were at least four pushes. This number may change in
the future.

#### Why is this bad?

The `vec![]` macro is both more performant and easier to read than
multiple `push` calls.

#### Example

```rust
let mut v = Vec::new();
v.push(0);
v.push(1);
v.push(2);
```

Use instead:

```rust
let v = vec![0, 1, 2];
```

---

### `waker_clone_wake`

| 属性 | 值 |
|------|----|
| 分组 | perf |
| 默认级别 | warn |
| Applicability | MachineApplicable |
| 引入版本 | 1.75.0 |

#### What it does

Checks for usage of `waker.clone().wake()`

#### Why is this bad?

Cloning the waker is not necessary, `wake_by_ref()` enables the same operation
without extra cloning/dropping.

#### Example

```rust
waker.clone().wake();
```

Should be written

```rust
waker.wake_by_ref();
```

---

## Pedantic (140)

*严格 - 非常严格的 lint，默认 allow*

### `assigning_clones`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.78.0 |

#### What it does

Checks for code like `foo = bar.clone();`

#### Why is this bad?

Custom `Clone::clone_from()` or `ToOwned::clone_into` implementations allow the objects
to share resources and therefore avoid allocations.

#### Example

```rust
struct Thing;

impl Clone for Thing {
    fn clone(&self) -> Self { todo!() }
    fn clone_from(&mut self, other: &Self) { todo!() }
}

pub fn assign_to_ref(a: &mut Thing, b: Thing) {
    *a = b.clone();
}
```

Use instead:

```rust
struct Thing;

impl Clone for Thing {
    fn clone(&self) -> Self { todo!() }
    fn clone_from(&mut self, other: &Self) { todo!() }
}

pub fn assign_to_ref(a: &mut Thing, b: Thing) {
    a.clone_from(&b);
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `bool_to_int_with_if`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.65.0 |

#### What it does

Instead of using an if statement to convert a bool to an int,
this lint suggests using a `from()` function or an `as` coercion.

#### Why is this bad?

Coercion or `from()` is another way to convert bool to a number.
Both methods are guaranteed to return 1 for true, and 0 for false.

See https://doc.rust-lang.org/std/primitive.bool.html#impl-From%3Cbool%3E

#### Example

```rust
if condition {
    1_i64
} else {
    0
};
```

Use instead:

```rust
i64::from(condition);
```

or

```rust
condition as i64;
```

---

### `borrow_as_ptr`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.60.0 |

#### What it does

Checks for the usage of `&expr as *const T` or
`&mut expr as *mut T`, and suggest using `&raw const` or
`&raw mut` instead.

#### Why is this bad?

This would improve readability and avoid creating a reference
that points to an uninitialized value or unaligned place.
Read the `&raw` explanation in the Reference for more information.

#### Example

```rust
let val = 1;
let p = &val as *const i32;

let mut val_mut = 1;
let p_mut = &mut val_mut as *mut i32;
```

Use instead:

```rust
let val = 1;
let p = &raw const val;

let mut val_mut = 1;
let p_mut = &raw mut val_mut;
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `case_sensitive_file_extension_comparisons`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.51.0 |

#### What it does

Checks for calls to `ends_with` with possible file extensions
and suggests to use a case-insensitive approach instead.

#### Why is this bad?

`ends_with` is case-sensitive and may not detect files with a valid extension.

#### Example

```rust
fn is_rust_file(filename: &str) -> bool {
    filename.ends_with(".rs")
}
```

Use instead:

```rust
fn is_rust_file(filename: &str) -> bool {
    let filename = std::path::Path::new(filename);
    filename.extension()
        .map_or(false, |ext| ext.eq_ignore_ascii_case("rs"))
}
```

---

### `cast_lossless`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts between numeric types that can be replaced by safe
conversion functions.

#### Why is this bad?

Rust’s `as` keyword will perform many kinds of conversions, including
silently lossy conversions. Conversion functions such as `i32::from`
will only perform lossless conversions. Using the conversion functions
prevents conversions from becoming silently lossy if the input types
ever change, and makes it clear for people reading the code that the
conversion is lossless.

#### Example

```rust
fn as_u64(x: u8) -> u64 {
    x as u64
}
```

Using `::from` would look like this:

```rust
fn as_u64(x: u8) -> u64 {
    u64::from(x)
}
```

---

### `cast_possible_truncation`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts between numeric types that may
truncate large values. This is expected behavior, so the cast is `Allow` by
default. It suggests user either explicitly ignore the lint,
or use `try_from()` and handle the truncation, default, or panic explicitly.

#### Why is this bad?

In some problem domains, it is good practice to avoid
truncation. This lint can be activated to help assess where additional
checks could be beneficial.

#### Example

```rust
fn as_u8(x: u64) -> u8 {
    x as u8
}
```

Use instead:

```rust
fn as_u8(x: u64) -> u8 {
    if let Ok(x) = u8::try_from(x) {
        x
    } else {
        todo!();
    }
}
// Or
#[allow(clippy::cast_possible_truncation)]
fn as_u16(x: u64) -> u16 {
    x as u16
}
```

---

### `cast_possible_wrap`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts from an unsigned type to a signed type of
the same size, or possibly smaller due to target-dependent integers.
Performing such a cast is a no-op for the compiler (that is, nothing is
changed at the bit level), and the binary representation of the value is
reinterpreted. This can cause wrapping if the value is too big
for the target signed type. However, the cast works as defined, so this lint
is `Allow` by default.

#### Why is this bad?

While such a cast is not bad in itself, the results can
be surprising when this is not the intended behavior:

#### Example

```rust
let _ = u32::MAX as i32; // will yield a value of `-1`
```

Use instead:

```rust
let _ = i32::try_from(u32::MAX).ok();
```

If the wrapping is intended, you can use:

```rust
let _ = u32::MAX.cast_signed();
let _ = (-1i32).cast_unsigned();
```

---

### `cast_precision_loss`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts from any numeric type to a float type where
the receiving type cannot store all values from the original type without
rounding errors. This possible rounding is to be expected, so this lint is
`Allow` by default.

Basically, this warns on casting any integer with 32 or more bits to `f32`
or any 64-bit integer to `f64`.

#### Why is this bad?

It’s not bad at all. But in some applications it can be
helpful to know where precision loss can take place. This lint can help find
those places in the code.

#### Example

```rust
let x = u64::MAX;
x as f64;
```

---

### `cast_ptr_alignment`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts, using `as` or `pointer::cast`, from a
less strictly aligned pointer to a more strictly aligned pointer.

#### Why is this bad?

Dereferencing the resulting pointer may be undefined behavior.

#### Known problems

Using [std::ptr::read_unaligned](https://doc.rust-lang.org/std/ptr/fn.read_unaligned.html) and [std::ptr::write_unaligned](https://doc.rust-lang.org/std/ptr/fn.write_unaligned.html) or
similar on the resulting pointer is fine. Is over-zealous: casts with
manual alignment checks or casts like `u64` -> `u8` -> `u16` can be
fine. Miri is able to do a more in-depth analysis.

#### Example

```rust
let _ = (&1u8 as *const u8) as *const u16;
let _ = (&mut 1u8 as *mut u8) as *mut u16;

(&1u8 as *const u8).cast::<u16>();
(&mut 1u8 as *mut u8).cast::<u16>();
```

---

### `cast_sign_loss`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for casts from a signed to an unsigned numeric
type. In this case, negative values wrap around to large positive values,
which can be quite surprising in practice. However, since the cast works as
defined, this lint is `Allow` by default.

#### Why is this bad?

Possibly surprising results. You can activate this lint
as a one-time check to see where numeric wrapping can arise.

#### Example

```rust
let y: i8 = -1;
y as u64; // will return 18446744073709551615
```

---

### `checked_conversions`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.37.0 |

#### What it does

Checks for explicit bounds checking when casting.

#### Why is this bad?

Reduces the readability of statements & is error prone.

#### Example

```rust
foo <= i32::MAX as u32;
```

Use instead:

```rust
i32::try_from(foo).is_ok();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `cloned_instead_of_copied`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.53.0 |

#### What it does

Checks for usage of `cloned()` on an `Iterator` or `Option` where
`copied()` could be used instead.

#### Why is this bad?

`copied()` is better because it guarantees that the type being cloned
implements `Copy`.

#### Example

```rust
[1, 2, 3].iter().cloned();
```

Use instead:

```rust
[1, 2, 3].iter().copied();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `collapsible_else_if`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.51.0 |

#### What it does

Checks for collapsible `else { if ... }` expressions
that can be collapsed to `else if ...`.

#### Why is this bad?

Each `if`-statement adds one level of nesting, which
makes code look more complex than it really is.

#### Example

```rust
if x {
    …
} else {
    if y {
        …
    }
}
```

Should be written:

```rust
if x {
    …
} else if y {
    …
}
```

#### Configuration

- `lint-commented-code`:  Whether collapsible `if` and `else if` chains are linted if they contain comments inside the parts
that would be collapsed.

(default: `false`)

---

### `comparison_chain`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | 1.40.0 |

#### What it does

Checks comparison chains written with `if` that can be
rewritten with `match` and `cmp`.

#### Why is this bad?

`if` is not guaranteed to be exhaustive and conditionals can get
repetitive

#### Example

```rust
fn f(x: u8, y: u8) {
    if x > y {
        a()
    } else if x < y {
        b()
    } else {
        c()
    }
}
```

Use instead:

```rust
use std::cmp::Ordering;
fn f(x: u8, y: u8) {
     match x.cmp(&y) {
         Ordering::Greater => a(),
         Ordering::Less => b(),
         Ordering::Equal => c()
     }
}
```

---

### `copy_iterator`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.30.0 |

#### What it does

Checks for types that implement `Copy` as well as
`Iterator`.

#### Why is this bad?

Implicit copies can be confusing when working with
iterator combinators.

#### Example

```rust
#[derive(Copy, Clone)]
struct Countdown(u8);

impl Iterator for Countdown {
    // ...
}

let a: Vec<_> = my_iterator.take(1).collect();
let b: Vec<_> = my_iterator.collect();
```

---

### `decimal_bitwise_operands`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.94.0 |

#### What it does

Checks for decimal literals used as bit masks in bitwise operations.

#### Why is this bad?

Using decimal literals for bit masks can make the code less readable and obscure the intended bit pattern.
Binary, hexadecimal, or octal literals make the bit pattern more explicit and easier to understand at a glance.

#### Example

```rust
let a = 14 & 6; // Bit pattern is not immediately clear
```

Use instead:

```rust
let a = 0b1110 & 0b0110;
```

---

### `default_trait_access`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for literal calls to `Default::default()`.

#### Why is this bad?

It’s easier for the reader if the name of the type is used, rather than the
generic `Default`.

#### Example

```rust
let s: String = Default::default();
```

Use instead:

```rust
let s = String::default();
```

---

### `doc_broken_link`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.90.0 |

#### What it does

Checks the doc comments have unbroken links, mostly caused
by bad formatted links such as broken across multiple lines.

#### Why is this bad?

Because documentation generated by rustdoc will be broken
since expected links won’t be links and just text.

#### Examples

This link is broken:

```rust
/// [example of a bad link](https://
/// github.com/rust-lang/rust-clippy/)
pub fn do_something() {}
```

It shouldn’t be broken across multiple lines to work:

```rust
/// [example of a good link](https://github.com/rust-lang/rust-clippy/)
pub fn do_something() {}
```

---

### `doc_comment_double_space_linebreaks`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.87.0 |

#### What it does

Detects doc comment linebreaks that use double spaces to separate lines, instead of back-slash (`\`).

#### Why is this bad?

Double spaces, when used as doc comment linebreaks, can be difficult to see, and may
accidentally be removed during automatic formatting or manual refactoring. The use of a back-slash (`\`)
is clearer in this regard.

#### Example

The two replacement dots in this example represent a double space.

```rust
/// This command takes two numbers as inputs and··
/// adds them together, and then returns the result.
fn add(l: i32, r: i32) -> i32 {
    l + r
}
```

Use instead:

```rust
/// This command takes two numbers as inputs and\
/// adds them together, and then returns the result.
fn add(l: i32, r: i32) -> i32 {
    l + r
}
```

---

### `doc_link_with_quotes`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.63.0 |

#### What it does

Detects the syntax `['foo']` in documentation comments (notice quotes instead of backticks)
outside of code blocks

#### Why is this bad?

It is likely a typo when defining an intra-doc link

#### Example

```rust
/// See also: ['foo']
fn bar() {}
```

Use instead:

```rust
/// See also: [`foo`]
fn bar() {}
```

---

### `doc_markdown`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the presence of `_`, `::` or camel-case words
outside ticks in documentation.

#### Why is this bad?

*Rustdoc* supports markdown formatting, `_`, `::` and
camel-case probably indicates some code which should be included between
ticks. `_` can also be used for emphasis in markdown, this lint tries to
consider that.

#### Known problems

Lots of bad docs won’t be fixed, what the lint checks
for is limited, and there are still false positives. HTML elements and their
content are not linted.

In addition, when writing documentation comments, including `[]` brackets
inside a link text would trip the parser. Therefore, documenting link with
`[`SmallVec<[T; INLINE_CAPACITY]>`]` and then [`SmallVec<[T; INLINE_CAPACITY]>`]: SmallVec
would fail.

#### Examples

```rust
/// Do something with the foo_bar parameter. See also
/// that::other::module::foo.
// ^ `foo_bar` and `that::other::module::foo` should be ticked.
fn doit(foo_bar: usize) {}
```

```rust
// Link text with `[]` brackets should be written as following:
/// Consume the array and return the inner
/// [`SmallVec<[T; INLINE_CAPACITY]>`][SmallVec].
/// [SmallVec]: SmallVec
fn main() {}
```

#### Configuration

- `doc-valid-idents`:  The list of words this lint should not consider as identifiers needing ticks. The value
`".."` can be used as part of the list to indicate that the configured values should be appended to the
default configuration of Clippy. By default, any configuration will replace the default value. For example:

- `doc-valid-idents = ["ClipPy"]` would replace the default list with `["ClipPy"]`.
- `doc-valid-idents = ["ClipPy", ".."]` would append `ClipPy` to the default list.

(default: `["KiB", "MiB", "GiB", "TiB", "PiB", "EiB", "MHz", "GHz", "THz", "AccessKit", "CoAP", "CoreFoundation", "CoreGraphics", "CoreText", "DevOps", "Direct2D", "Direct3D", "DirectWrite", "DirectX", "ECMAScript", "GPLv2", "GPLv3", "GitHub", "GitLab", "IPv4", "IPv6", "InfiniBand", "RoCE", "ClojureScript", "CoffeeScript", "JavaScript", "PostScript", "PureScript", "TypeScript", "PowerPC", "PowerShell", "WebAssembly", "NaN", "NaNs", "OAuth", "GraphQL", "OCaml", "OpenAL", "OpenDNS", "OpenGL", "OpenMP", "OpenSSH", "OpenSSL", "OpenStreetMap", "OpenTelemetry", "OpenType", "WebGL", "WebGL2", "WebGPU", "WebRTC", "WebSocket", "WebTransport", "WebP", "OpenExr", "YCbCr", "sRGB", "TensorFlow", "TrueType", "iOS", "macOS", "FreeBSD", "NetBSD", "OpenBSD", "NixOS", "TeX", "LaTeX", "BibTeX", "BibLaTeX", "MinGW", "CamelCase"]`)

---

### `duration_suboptimal_units`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.95.0 |

#### What it does

Checks for instances where a `std::time::Duration` is constructed using a smaller time unit
when the value could be expressed more clearly using a larger unit.

#### Why is this bad?

Using a smaller unit for a duration that is evenly divisible by a larger unit reduces
readability. Readers have to mentally convert values, which can be error-prone and makes
the code less clear.

#### Example

```rust
use std::time::Duration;

let dur = Duration::from_millis(5_000);
let dur = Duration::from_secs(180);
let dur = Duration::from_mins(10 * 60);
```

Use instead:

```rust
use std::time::Duration;

let dur = Duration::from_secs(5);
let dur = Duration::from_mins(3);
let dur = Duration::from_hours(10);
```

---

### `elidable_lifetime_names`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.87.0 |

#### What it does

Checks for lifetime annotations which can be replaced with anonymous lifetimes (`'_`).

#### Why is this bad?

The additional lifetimes can make the code look more complicated.

#### Known problems

This lint ignores functions with `where` clauses that reference
lifetimes to prevent false positives.

#### Example

```rust
fn f<'a>(x: &'a str) -> Chars<'a> {
    x.chars()
}
```

Use instead:

```rust
fn f(x: &str) -> Chars<'_> {
    x.chars()
}
```

---

### `empty_enums`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `enum`s with no variants, which therefore are uninhabited types
(cannot be instantiated).

As of this writing, the `never_type` is still a nightly-only experimental API.
Therefore, this lint is only triggered if `#![feature(never_type)]` is enabled.

#### Why is this bad?

- If you only want a type which can’t be instantiated, you should use [!](https://doc.rust-lang.org/std/primitive.never.html)
(the primitive type “never”), because [!](https://doc.rust-lang.org/std/primitive.never.html) has more extensive compiler support
(type inference, etc.) and implementations of common traits.
- If you need to introduce a distinct type, consider using a [newtype](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#using-the-newtype-pattern-for-type-safety-and-abstraction) `struct`
containing [!](https://doc.rust-lang.org/std/primitive.never.html) instead (`struct MyType(pub !)`), because it is more idiomatic
to use a `struct` rather than an `enum` when an `enum` is unnecessary.

If you do this, note that the [visibility](https://doc.rust-lang.org/reference/visibility-and-privacy.html) of the [!](https://doc.rust-lang.org/std/primitive.never.html) field determines whether
the uninhabitedness is visible in documentation, and whether it can be pattern
matched to mark code unreachable. If the field is not visible, then the struct
acts like any other struct with private fields.

For further information, visit
[the never type’s documentation](https://doc.rust-lang.org/std/primitive.never.html).

#### Example

```rust
enum CannotExist {}
```

Use instead:

```rust
#![feature(never_type)]

/// Use the `!` type directly...
type CannotExist = !;

/// ...or define a newtype which is distinct.
struct CannotExist2(pub !);
```

#### Past names

- empty_enum

---

### `enum_glob_use`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `use Enum::*`.

#### Why is this bad?

It is usually better style to use the prefixed name of
an enumeration variant, rather than importing variants.

#### Known problems

Old-style enumerations that prefix the variants are
still around.

#### Example

```rust
use std::cmp::Ordering::*;

foo(Less);
```

Use instead:

```rust
use std::cmp::Ordering;

foo(Ordering::Less)
```

---

### `expl_impl_clone_on_copy`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for explicit `Clone` implementations for `Copy`
types.

#### Why is this bad?

To avoid surprising behavior, these traits should
agree and the behavior of `Copy` cannot be overridden. In almost all
situations a `Copy` type should have a `Clone` implementation that does
nothing more than copy the object, which is what `#[derive(Copy, Clone)]`
gets you.

#### Example

```rust
#[derive(Copy)]
struct Foo;

impl Clone for Foo {
    // ..
}
```

---

### `explicit_deref_methods`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.44.0 |

#### What it does

Checks for explicit `deref()` or `deref_mut()` method calls.

Doesn’t lint inside the implementation of the `Deref` or `DerefMut` traits.

#### Why is this bad?

Dereferencing by `&*x` or `&mut *x` is clearer and more concise,
when not part of a method chain.

#### Example

```rust
use std::ops::Deref;
let a: &mut String = &mut String::from("foo");
let b: &str = a.deref();
```

Use instead:

```rust
let a: &mut String = &mut String::from("foo");
let b = &*a;
```

This lint excludes all of:

```rust
let _ = d.unwrap().deref();
let _ = Foo::deref(&foo);
let _ = <Foo as Deref>::deref(&foo);
```

---

### `explicit_into_iter_loop`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for loops on `y.into_iter()` where `y` will do, and
suggests the latter.

#### Why is this bad?

Readability.

#### Example

```rust
// with `y` a `Vec` or slice:
for x in y.into_iter() {
    // ..
}
```

can be rewritten to

```rust
for x in y {
    // ..
}
```

---

### `explicit_iter_loop`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for loops on `x.iter()` where `&x` will do, and
suggests the latter.

#### Why is this bad?

Readability.

#### Known problems

False negatives. We currently only warn on some known
types.

#### Example

```rust
// with `y` a `Vec` or slice:
for x in y.iter() {
    // ..
}
```

Use instead:

```rust
for x in &y {
    // ..
}
```

#### Configuration

- `enforce-iter-loop-reborrow`:  Whether to recommend using implicit into iter for reborrowed values.

#### Example

```rust
let mut vec = vec![1, 2, 3];
let rmvec = &mut vec;
for _ in rmvec.iter() {}
for _ in rmvec.iter_mut() {}
```

Use instead:

```rust
let mut vec = vec![1, 2, 3];
let rmvec = &mut vec;
for _ in &*rmvec {}
for _ in &mut *rmvec {}
```

(default: `false`)

---

### `filter_map_next`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.36.0 |

#### What it does

Checks for usage of `_.filter_map(_).next()`.

#### Why is this bad?

Readability, this can be written more concisely as
`_.find_map(_)`.

#### Example

```rust
(0..3).filter_map(|x| if x == 2 { Some(x) } else { None }).next();
```

Can be written as

```rust
(0..3).find_map(|x| if x == 2 { Some(x) } else { None });
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `flat_map_option`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.53.0 |

#### What it does

Checks for usage of `Iterator::flat_map()` where `filter_map()` could be
used instead.

#### Why is this bad?

`filter_map()` is known to always produce 0 or 1 output items per input item,
rather than however many the inner iterator type produces.
Therefore, it maintains the upper bound in `Iterator::size_hint()`,
and communicates to the reader that the input items are not being expanded into
multiple output items without their having to notice that the mapping function
returns an `Option`.

#### Example

```rust
let nums: Vec<i32> = ["1", "2", "whee!"].iter().flat_map(|x| x.parse().ok()).collect();
```

Use instead:

```rust
let nums: Vec<i32> = ["1", "2", "whee!"].iter().filter_map(|x| x.parse().ok()).collect();
```

---

### `float_cmp`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for (in-)equality comparisons on floating-point
values (apart from zero), except in functions called `*eq*` (which probably
implement equality for a type involving floats).

#### Why is this bad?

Floating point calculations are usually imprecise, so asking if two values are *exactly*
equal is asking for trouble because arriving at the same logical result via different
routes (e.g. calculation versus constant) may yield different values.

#### Example

```rust
let a: f64 = 1000.1;
let b: f64 = 0.2;
let x = a + b;
let y = 1000.3; // Expected value.

// Actual value: 1000.3000000000001
println!("{x}");

let are_equal = x == y;
println!("{are_equal}"); // false
```

The correct way to compare floating point numbers is to define an allowed error margin. This
may be challenging if there is no “natural” error margin to permit. Broadly speaking, there
are two cases:

1. If your values are in a known range and you can define a threshold for “close enough to
be equal”, it may be appropriate to define an absolute error margin. For example, if your
data is “length of vehicle in centimeters”, you may consider 0.1 cm to be “close enough”.
2. If your code is more general and you do not know the range of values, you should use a
relative error margin, accepting e.g. 0.1% of error regardless of specific values.

For the scenario where you can define a meaningful absolute error margin, consider using:

```rust
let a: f64 = 1000.1;
let b: f64 = 0.2;
let x = a + b;
let y = 1000.3; // Expected value.

const ALLOWED_ERROR_VEHICLE_LENGTH_CM: f64 = 0.1;
let within_tolerance = (x - y).abs() < ALLOWED_ERROR_VEHICLE_LENGTH_CM;
println!("{within_tolerance}"); // true
```

NOTE: Do not use `f64::EPSILON` - while the error margin is often called “epsilon”, this is
a different use of the term that is not suitable for floating point equality comparison.
Indeed, for the example above using `f64::EPSILON` as the allowed error would return `false`.

For the scenario where no meaningful absolute error can be defined, refer to
[the floating point guide](https://www.floating-point-gui.de/errors/comparison)
for a reference implementation of relative error based comparison of floating point values.
`MIN_NORMAL` in the reference implementation is equivalent to `MIN_POSITIVE` in Rust.

---

### `fn_params_excessive_bools`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.43.0 |

#### What it does

Checks for excessive use of
bools in function definitions.

#### Why is this bad?

Calls to such functions
are confusing and error prone, because it’s
hard to remember argument order and you have
no type system support to back you up. Using
two-variant enums instead of bools often makes
API easier to use.

#### Example

```rust
fn f(is_round: bool, is_hot: bool) { ... }
```

Use instead:

```rust
enum Shape {
    Round,
    Spiky,
}

enum Temperature {
    Hot,
    IceCold,
}

fn f(shape: Shape, temperature: Temperature) { ... }
```

#### Configuration

- `max-fn-params-bools`:  The maximum number of bool parameters a function can have

(default: `3`)

---

### `format_collect`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Checks for usage of `.map(|_| format!(..)).collect::<String>()`.

#### Why is this bad?

This allocates a new string for every element in the iterator.
This can be done more efficiently by creating the `String` once and appending to it in `Iterator::fold`,
using either the `write!` macro which supports exactly the same syntax as the `format!` macro,
or concatenating with `+` in case the iterator yields `&str`/`String`.

Note also that `write!`-ing into a `String` can never fail, despite the return type of `write!` being `std::fmt::Result`,
so it can be safely ignored or unwrapped.

#### Example

```rust
fn hex_encode(bytes: &[u8]) -> String {
    bytes.iter().map(|b| format!("{b:02X}")).collect()
}
```

Use instead:

```rust
use std::fmt::Write;
fn hex_encode(bytes: &[u8]) -> String {
    bytes.iter().fold(String::new(), |mut output, b| {
        let _ = write!(output, "{b:02X}");
        output
    })
}
```

---

### `format_push_string`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.62.0 |

#### What it does

Detects cases where the result of a `format!` call is
appended to an existing `String`.

#### Why is this bad?

Introduces an extra, avoidable heap allocation.

#### Known problems

`format!` returns a `String` but `write!` returns a `Result`.
Thus you are forced to ignore the `Err` variant to achieve the same API.

While using `write!` in the suggested way should never fail, this isn’t necessarily clear to the programmer.

#### Example

```rust
let mut s = String::new();
s += &format!("0x{:X}", 1024);
s.push_str(&format!("0x{:X}", 1024));
```

Use instead:

```rust
use std::fmt::Write as _; // import without risk of name clashing

let mut s = String::new();
let _ = write!(s, "0x{:X}", 1024);
```

---

### `from_iter_instead_of_collect`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.49.0 |

#### What it does

Checks for `from_iter()` function calls on types that implement the `FromIterator`
trait.

#### Why is this bad?

If it’s needed to create a collection from the contents of an iterator, the `Iterator::collect(_)`
method is preferred. However, when it’s needed to specify the container type,
`Vec::from_iter(_)` can be more readable than using a turbofish (e.g. `_.collect::<Vec<_>>()`). See
[FromIterator documentation](https://doc.rust-lang.org/std/iter/trait.FromIterator.html)

#### Example

```rust
let five_fives = std::iter::repeat(5).take(5);

let v = Vec::from_iter(five_fives);

assert_eq!(v, vec![5, 5, 5, 5, 5]);
```

Use instead:

```rust
let five_fives = std::iter::repeat(5).take(5);

let v: Vec<i32> = five_fives.collect();

assert_eq!(v, vec![5, 5, 5, 5, 5]);
```

but prefer to use

```rust
let numbers: Vec<i32> = FromIterator::from_iter(1..=5);
```

instead of

```rust
let numbers = (1..=5).collect::<Vec<_>>();
```

---

### `if_not_else`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `!` or `!=` in an if condition with an
else branch.

#### Why is this bad?

Negations reduce the readability of statements.

#### Example

```rust
if !v.is_empty() {
    a()
} else {
    b()
}
```

Could be written:

```rust
if v.is_empty() {
    b()
} else {
    a()
}
```

---

### `ignore_without_reason`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.88.0 |

#### What it does

Checks for ignored tests without messages.

#### Why is this bad?

The reason for ignoring the test may not be obvious.

#### Example

```rust
#[test]
#[ignore]
fn test() {}
```

Use instead:

```rust
#[test]
#[ignore = "Some good reason"]
fn test() {}
```

#### Note

Clippy can only lint compiled code. For this lint to trigger, you must configure `cargo clippy`
to include test compilation, for instance, by using flags such as `--tests` or `--all-targets`.

---

### `ignored_unit_patterns`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.73.0 |

#### What it does

Checks for usage of `_` in patterns of type `()`.

#### Why is this bad?

Matching with `()` explicitly instead of `_` outlines
the fact that the pattern contains no data. Also it
would detect a type change that `_` would ignore.

#### Example

```rust
match std::fs::create_dir("tmp-work-dir") {
    Ok(_) => println!("Working directory created"),
    Err(s) => eprintln!("Could not create directory: {s}"),
}
```

Use instead:

```rust
match std::fs::create_dir("tmp-work-dir") {
    Ok(()) => println!("Working directory created"),
    Err(s) => eprintln!("Could not create directory: {s}"),
}
```

---

### `implicit_clone`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Checks for the usage of `_.to_owned()`, `vec.to_vec()`, or similar when calling `_.clone()` would be clearer.

#### Why is this bad?

These methods do the same thing as `_.clone()` but may be confusing as
to why we are calling `to_vec` on something that is already a `Vec` or calling `to_owned` on something that is already owned.

#### Example

```rust
let a = vec![1, 2, 3];
let b = a.to_vec();
let c = a.to_owned();
```

Use instead:

```rust
let a = vec![1, 2, 3];
let b = a.clone();
let c = a.clone();
```

---

### `implicit_hasher`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for public `impl` or `fn` missing generalization
over different hashers and implicitly defaulting to the default hashing
algorithm (`SipHash`).

#### Why is this bad?

`HashMap` or `HashSet` with custom hashers cannot be
used with them.

#### Known problems

Suggestions for replacing constructors can contain
false-positives. Also applying suggestions can require modification of other
pieces of code, possibly including external crates.

#### Example

```rust
impl<K: Hash + Eq, V> Serialize for HashMap<K, V> { }

pub fn foo(map: &mut HashMap<i32, i32>) { }
```

could be rewritten as

```rust
impl<K: Hash + Eq, V, S: BuildHasher> Serialize for HashMap<K, V, S> { }

pub fn foo<S: BuildHasher>(map: &mut HashMap<i32, i32, S>) { }
```

---

### `inconsistent_struct_constructor`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Checks for struct constructors where the order of the field
init in the constructor is inconsistent with the order in the
struct definition.

#### Why is this bad?

Since the order of fields in a constructor doesn’t affect the
resulted instance as the below example indicates,

```rust
#[derive(Debug, PartialEq, Eq)]
struct Foo {
    x: i32,
    y: i32,
}
let x = 1;
let y = 2;

// This assertion never fails:
assert_eq!(Foo { x, y }, Foo { y, x });
```

inconsistent order can be confusing and decreases readability and consistency.

#### Example

```rust
struct Foo {
    x: i32,
    y: i32,
}
let x = 1;
let y = 2;

Foo { y, x };
```

Use instead:

```rust
Foo { x, y };
```

#### Configuration

- `check-inconsistent-struct-field-initializers`:  Whether to suggest reordering constructor fields when initializers are present.

Warnings produced by this configuration aren’t necessarily fixed by just reordering the fields. Even if the
suggested code would compile, it can change semantics if the initializer expressions have side effects. The
following example [from rust-clippy#11846](https://github.com/rust-lang/rust-clippy/issues/11846#issuecomment-1820747924) shows how the suggestion can run into borrow check errors:

```rust
struct MyStruct {
    vector: Vec<u32>,
    length: usize
}
fn main() {
    let vector = vec![1,2,3];
    MyStruct { length: vector.len(), vector};
}
```

(default: `false`)

---

### `index_refutable_slice`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.59.0 |

#### What it does

The lint checks for slice bindings in patterns that are only used to
access individual slice values.

#### Why is this bad?

Accessing slice values using indices can lead to panics. Using refutable
patterns can avoid these. Binding to individual values also improves the
readability as they can be named.

#### Limitations

This lint currently only checks for immutable access inside `if let`
patterns.

#### Example

```rust
let slice: Option<&[u32]> = Some(&[1, 2, 3]);

if let Some(slice) = slice {
    println!("{}", slice[0]);
}
```

Use instead:

```rust
let slice: Option<&[u32]> = Some(&[1, 2, 3]);

if let Some(&[first, ..]) = slice {
    println!("{}", first);
}
```

#### Configuration

- `max-suggested-slice-pattern-length`:  When Clippy suggests using a slice pattern, this is the maximum number of elements allowed in
the slice pattern that is suggested. If more elements are necessary, the lint is suppressed.
For example, `[_, _, _, e, ..]` is a slice pattern with 4 elements.

(default: `3`)
- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `inefficient_to_string`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.40.0 |

#### What it does

Checks for usage of `.to_string()` on an `&&T` where
`T` implements `ToString` directly (like `&&str` or `&&String`).

#### Why is this bad?

In versions of the compiler before Rust 1.82.0, this bypasses the specialized
implementation of `ToString` and instead goes through the more expensive string
formatting facilities.

#### Example

```rust
// Generic implementation for `T: Display` is used (slow)
["foo", "bar"].iter().map(|s| s.to_string());

// OK, the specialized impl is used
["foo", "bar"].iter().map(|&s| s.to_string());
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `inline_always`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for items annotated with `#[inline(always)]`,
unless the annotated function is empty or simply panics.

#### Why is this bad?

While there are valid uses of this annotation (and once
you know when to use it, by all means `allow` this lint), it’s a common
newbie-mistake to pepper one’s code with it.

As a rule of thumb, before slapping `#[inline(always)]` on a function,
measure if that additional function call really affects your runtime profile
sufficiently to make up for the increase in compile time.

#### Known problems

False positives, big time. This lint is meant to be
deactivated by everyone doing serious performance work. This means having
done the measurement.

#### Example

```rust
#[inline(always)]
fn not_quite_hot_code(..) { ... }
```

---

### `into_iter_without_iter`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.75.0 |

#### What it does

This is the opposite of the `iter_without_into_iter` lint.
It looks for `IntoIterator for (&|&mut) Type` implementations without an inherent `iter` or `iter_mut` method
on the type or on any of the types in its `Deref` chain.

#### Why is this bad?

It’s not bad, but having them is idiomatic and allows the type to be used in iterator chains
by just calling `.iter()`, instead of the more awkward `<&Type>::into_iter` or `(&val).into_iter()` syntax
in case of ambiguity with another `IntoIterator` impl.

#### Limitations

This lint focuses on providing an idiomatic API. Therefore, it will only
lint on types which are accessible outside of the crate. For internal types,
these methods can be added on demand if they are actually needed. Otherwise,
it would trigger the [dead_code](https://doc.rust-lang.org/rustc/lints/listing/warn-by-default.html#dead-code) lint for the unused method.

#### Example

```rust
struct MySlice<'a>(&'a [u8]);
impl<'a> IntoIterator for &MySlice<'a> {
    type Item = &'a u8;
    type IntoIter = std::slice::Iter<'a, u8>;
    fn into_iter(self) -> Self::IntoIter {
        self.0.iter()
    }
}
```

Use instead:

```rust
struct MySlice<'a>(&'a [u8]);
impl<'a> MySlice<'a> {
    pub fn iter(&self) -> std::slice::Iter<'a, u8> {
        self.into_iter()
    }
}
impl<'a> IntoIterator for &MySlice<'a> {
    type Item = &'a u8;
    type IntoIter = std::slice::Iter<'a, u8>;
    fn into_iter(self) -> Self::IntoIter {
        self.0.iter()
    }
}
```

---

### `invalid_upcast_comparisons`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for comparisons where the relation is always either
true or false, but where one side has been upcast so that the comparison is
necessary. Only integer types are checked.

#### Why is this bad?

An expression like `let x : u8 = ...; (x as u32) > 300`
will mistakenly imply that it is possible for `x` to be outside the range of
`u8`.

#### Example

```rust
let x: u8 = 1;
(x as u32) > 300;
```

---

### `ip_constant`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.89.0 |

#### What it does

Checks for IP addresses that could be replaced with predefined constants such as
`Ipv4Addr::new(127, 0, 0, 1)` instead of using the appropriate constants.

#### Why is this bad?

Using specific IP addresses like `127.0.0.1` or `::1` is less clear and less maintainable than using the
predefined constants `Ipv4Addr::LOCALHOST` or `Ipv6Addr::LOCALHOST`. These constants improve code
readability, make the intent explicit, and are less error-prone.

#### Example

```rust
use std::net::{Ipv4Addr, Ipv6Addr};

// IPv4 loopback
let addr_v4 = Ipv4Addr::new(127, 0, 0, 1);

// IPv6 loopback
let addr_v6 = Ipv6Addr::new(0, 0, 0, 0, 0, 0, 0, 1);
```

Use instead:

```rust
use std::net::{Ipv4Addr, Ipv6Addr};

// IPv4 loopback
let addr_v4 = Ipv4Addr::LOCALHOST;

// IPv6 loopback
let addr_v6 = Ipv6Addr::LOCALHOST;
```

---

### `items_after_statements`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for items declared after some statement in a block.

#### Why is this bad?

Items live for the entire scope they are declared
in. But statements are processed in order. This might cause confusion as
it’s hard to figure out which item is meant in a statement.

#### Example

```rust
fn foo() {
    println!("cake");
}

fn main() {
    foo(); // prints "foo"
    fn foo() {
        println!("foo");
    }
    foo(); // prints "foo"
}
```

Use instead:

```rust
fn foo() {
    println!("cake");
}

fn main() {
    fn foo() {
        println!("foo");
    }
    foo(); // prints "foo"
    foo(); // prints "foo"
}
```

---

### `iter_filter_is_ok`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | 1.77.0 |

#### What it does

Checks for usage of `.filter(Result::is_ok)` that may be replaced with a `.flatten()` call.
This lint will require additional changes to the follow-up calls as it affects the type.

#### Why is this bad?

This pattern is often followed by manual unwrapping of `Result`. The simplification
results in more readable and succinct code without the need for manual unwrapping.

#### Example

```rust
vec![Ok::<i32, String>(1)].into_iter().filter(Result::is_ok);
```

Use instead:

```rust
vec![Ok::<i32, String>(1)].into_iter().flatten();
```

---

### `iter_filter_is_some`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | 1.77.0 |

#### What it does

Checks for usage of `.filter(Option::is_some)` that may be replaced with a `.flatten()` call.
This lint will require additional changes to the follow-up calls as it affects the type.

#### Why is this bad?

This pattern is often followed by manual unwrapping of the `Option`. The simplification
results in more readable and succinct code without the need for manual unwrapping.

#### Example

```rust
vec![Some(1)].into_iter().filter(Option::is_some);
```

Use instead:

```rust
vec![Some(1)].into_iter().flatten();
```

---

### `iter_not_returning_iterator`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

Detects methods named `iter` or `iter_mut` that do not have a return type that implements `Iterator`.

#### Why is this bad?

Methods named `iter` or `iter_mut` conventionally return an `Iterator`.

#### Example

```rust
// `String` does not implement `Iterator`
struct Data {}
impl Data {
    fn iter(&self) -> String {
        todo!()
    }
}
```

Use instead:

```rust
use std::str::Chars;
struct Data {}
impl Data {
    fn iter(&self) -> Chars<'static> {
        todo!()
    }
}
```

---

### `iter_without_into_iter`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.75.0 |

#### What it does

Looks for `iter` and `iter_mut` methods without an associated `IntoIterator for (&|&mut) Type` implementation.

#### Why is this bad?

It’s not bad, but having them is idiomatic and allows the type to be used in for loops directly
(`for val in &iter {}`), without having to first call `iter()` or `iter_mut()`.

#### Limitations

This lint focuses on providing an idiomatic API. Therefore, it will only
lint on types which are accessible outside of the crate. For internal types,
the `IntoIterator` trait can be implemented on demand if it is actually needed.

#### Example

```rust
struct MySlice<'a>(&'a [u8]);
impl<'a> MySlice<'a> {
    pub fn iter(&self) -> std::slice::Iter<'a, u8> {
        self.0.iter()
    }
}
```

Use instead:

```rust
struct MySlice<'a>(&'a [u8]);
impl<'a> MySlice<'a> {
    pub fn iter(&self) -> std::slice::Iter<'a, u8> {
        self.0.iter()
    }
}
impl<'a> IntoIterator for &MySlice<'a> {
    type Item = &'a u8;
    type IntoIter = std::slice::Iter<'a, u8>;
    fn into_iter(self) -> Self::IntoIter {
        self.iter()
    }
}
```

---

### `large_digit_groups`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if the digits of an integral or floating-point
constant are grouped into groups that
are too large.

#### Why is this bad?

Negatively impacts readability.

#### Example

```rust
let x: u64 = 6186491_8973511;
```

---

### `large_futures`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.70.0 |

#### What it does

It checks for the size of a `Future` created by `async fn` or `async {}`.

#### Why is this bad?

Due to the current [unideal implementation](https://github.com/rust-lang/rust/issues/69826) of `Coroutine`,
large size of a `Future` may cause stack overflows.

#### Example

```rust
async fn large_future(_x: [u8; 16 * 1024]) {}

pub async fn trigger() {
    large_future([0u8; 16 * 1024]).await;
}
```

`Box::pin` the big future instead.

```rust
async fn large_future(_x: [u8; 16 * 1024]) {}

pub async fn trigger() {
    Box::pin(large_future([0u8; 16 * 1024])).await;
}
```

#### Configuration

- `future-size-threshold`:  The maximum byte size a `Future` can have, before it triggers the `clippy::large_futures` lint

(default: `16384`)

---

### `large_stack_arrays`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.41.0 |

#### What it does

Checks for local arrays that may be too large.

#### Why is this bad?

Large local arrays may cause stack overflow.

#### Example

```rust
let a = [0u32; 1_000_000];
```

#### Configuration

- `array-size-threshold`:  The maximum allowed size for arrays on the stack

(default: `16384`)

---

### `large_types_passed_by_value`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.49.0 |

#### What it does

Checks for functions taking arguments by value, where
the argument type is `Copy` and large enough to be worth considering
passing by reference. Does not trigger if the function is being exported,
because that might induce API breakage, if the parameter is declared as mutable,
or if the argument is a `self`.

#### Why is this bad?

Arguments passed by value might result in an unnecessary
shallow copy, taking up more space in the stack and requiring a call to
`memcpy`, which can be expensive.

#### Example

```rust
#[derive(Clone, Copy)]
struct TooLarge([u8; 2048]);

fn foo(v: TooLarge) {}
```

Use instead:

```rust
fn foo(v: &TooLarge) {}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)
- `pass-by-value-size-limit`:  The minimum size (in bytes) to consider a type for passing by reference instead of by value.

(default: `256`)

---

### `linkedlist`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of any `LinkedList`, suggesting to use a
`Vec` or a `VecDeque` (formerly called `RingBuf`).

#### Why is this bad?

Gankra says:

> The TL;DR of `LinkedList` is that it’s built on a massive amount of
> pointers and indirection.
> It wastes memory, it has terrible cache locality, and is all-around slow.
> `RingBuf`, while
> “only” amortized for push/pop, should be faster in the general case for
> almost every possible
> workload, and isn’t even amortized at all if you can predict the capacity
> you need.
> 
> 
> `LinkedList`s are only really good if you’re doing a lot of merging or
> splitting of lists.
> This is because they can just mangle some pointers instead of actually
> copying the data. Even
> if you’re doing a lot of insertion in the middle of the list, `RingBuf`
> can still be better
> because of how expensive it is to seek to the middle of a `LinkedList`.

#### Known problems

False positives – the instances where using a
`LinkedList` makes sense are few and far between, but they can still happen.

#### Example

```rust
let x: LinkedList<usize> = LinkedList::new();
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `macro_use_imports`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.44.0 |

#### What it does

Checks for `#[macro_use] use...`.

#### Why is this bad?

Since the Rust 2018 edition you can import
macro’s directly, this is considered idiomatic.

#### Example

```rust
#[macro_use]
extern crate some_crate;

fn main() {
    some_macro!();
}
```

Use instead:

```rust
use some_crate::some_macro;

fn main() {
    some_macro!();
}
```

---

### `manual_assert`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.57.0 |

#### What it does

Detects `if`-then-`panic!` that can be replaced with `assert!`.

#### Why is this bad?

`assert!` is simpler than `if`-then-`panic!`.

#### Example

```rust
let sad_people: Vec<&str> = vec![];
if !sad_people.is_empty() {
    panic!("there are sad people: {:?}", sad_people);
}
```

Use instead:

```rust
let sad_people: Vec<&str> = vec![];
assert!(sad_people.is_empty(), "there are sad people: {:?}", sad_people);
```

---

### `manual_ilog2`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.94.0 |

#### What it does

Checks for expressions like `N - x.leading_zeros()` (where `N` is one less than bit width
of `x`) or `x.ilog(2)`, which are manual reimplementations of `x.ilog2()`

#### Why is this bad?

Manual reimplementations of `ilog2` increase code complexity for little benefit.

#### Example

```rust
let x: u32 = 5;
let log = 31 - x.leading_zeros();
let log = x.ilog(2);
```

Use instead:

```rust
let x: u32 = 5;
let log = x.ilog2();
let log = x.ilog2();
```

---

### `manual_instant_elapsed`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.65.0 |

#### What it does

Lints subtraction between `Instant::now()` and another `Instant`.

#### Why is this bad?

It is easy to accidentally write `prev_instant - Instant::now()`, which will always be 0ns
as `Instant` subtraction saturates.

`prev_instant.elapsed()` also more clearly signals intention.

#### Example

```rust
use std::time::Instant;
let prev_instant = Instant::now();
let duration = Instant::now() - prev_instant;
```

Use instead:

```rust
use std::time::Instant;
let prev_instant = Instant::now();
let duration = prev_instant.elapsed();
```

---

### `manual_is_power_of_two`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.83.0 |

#### What it does

Checks for expressions like `x.count_ones() == 1` or `x & (x - 1) == 0`, with x and unsigned integer, which may be manual
reimplementations of `x.is_power_of_two()`.

#### Why is this bad?

Manual reimplementations of `is_power_of_two` increase code complexity for little benefit.

#### Example

```rust
let a: u32 = 4;
let result = a.count_ones() == 1;
```

Use instead:

```rust
let a: u32 = 4;
let result = a.is_power_of_two();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_is_variant_and`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.77.0 |

#### What it does

Checks for usage of `option.map(f).unwrap_or_default()` and `result.map(f).unwrap_or_default()` where `f` is a function or closure that returns the `bool` type.

Also checks for equality comparisons like `option.map(f) == Some(true)` and `result.map(f) == Ok(true)`.

#### Why is this bad?

Readability. These can be written more concisely as `option.is_some_and(f)` and `result.is_ok_and(f)`.

#### Example

```rust
option.map(|a| a > 10).unwrap_or_default();
result.map(|a| a > 10).unwrap_or_default();

option.map(|a| a > 10) == Some(true);
result.map(|a| a > 10) == Ok(true);
option.map(|a| a > 10) != Some(true);
result.map(|a| a > 10) != Ok(true);
```

Use instead:

```rust
option.is_some_and(|a| a > 10);
result.is_ok_and(|a| a > 10);

option.is_some_and(|a| a > 10);
result.is_ok_and(|a| a > 10);
option.is_none_or(|a| a > 10);
!result.is_ok_and(|a| a > 10);
```

---

### `manual_let_else`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | 1.67.0 |

#### What it does

Warn of cases where `let...else` could be used

#### Why is this bad?

`let...else` provides a standard construct for this pattern
that people can easily recognize. It’s also more compact.

#### Example

```rust
let v = if let Some(v) = w { v } else { return };
```

Could be written:

```rust
let Some(v) = w else { return };
```

#### Configuration

- `matches-for-let-else`:  Whether the matches should be considered by the lint, and whether there should
be filtering for common types.

(default: `"WellKnownTypes"`)
- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_midpoint`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.87.0 |

#### What it does

Checks for manual implementation of `midpoint`.

#### Why is this bad?

Using `(x + y) / 2` might cause an overflow on the intermediate
addition result.

#### Example

```rust
let c = (a + 10) / 2;
```

Use instead:

```rust
let c = u32::midpoint(a, 10);
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `manual_string_new`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.65.0 |

#### What it does

Checks for usage of `""` to create a `String`, such as `"".to_string()`, `"".to_owned()`,
`String::from("")` and others.

#### Why is this bad?

Different ways of creating an empty string makes your code less standardized, which can
be confusing.

#### Example

```rust
let a = "".to_string();
let b: String = "".into();
```

Use instead:

```rust
let a = String::new();
let b = String::new();
```

---

### `many_single_char_names`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for too many variables whose name consists of a
single character.

#### Why is this bad?

It’s hard to memorize what a variable means without a
descriptive name.

#### Example

```rust
let (a, b, c, d, e, f, g) = (...);
```

#### Configuration

- `single-char-binding-names-threshold`:  The maximum number of single char bindings a scope may have

(default: `4`)

---

### `map_unwrap_or`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.45.0 |

#### What it does

Checks for usage of `option.map(_).unwrap_or(_)` or `option.map(_).unwrap_or_else(_)` or
`result.map(_).unwrap_or_else(_)`.

#### Why is this bad?

Readability, these can be written more concisely (resp.) as
`option.map_or(_, _)`, `option.map_or_else(_, _)` and `result.map_or_else(_, _)`.

#### Known problems

The order of the arguments is not in execution order

#### Examples

```rust
option.map(|a| a + 1).unwrap_or(0);
option.map(|a| a > 10).unwrap_or(false);
result.map(|a| a + 1).unwrap_or_else(some_function);
```

Use instead:

```rust
option.map_or(0, |a| a + 1);
option.is_some_and(|a| a > 10);
result.map_or_else(some_function, |a| a + 1);
```

#### Past names

- option_map_unwrap_or
- option_map_unwrap_or_else
- result_map_unwrap_or_else

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `match_bool`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for matches where match expression is a `bool`. It
suggests to replace the expression with an `if...else` block.

#### Why is this bad?

It makes the code less readable.

#### Example

```rust
let condition: bool = true;
match condition {
    true => foo(),
    false => bar(),
}
```

Use if/else instead:

```rust
let condition: bool = true;
if condition {
    foo();
} else {
    bar();
}
```

---

### `match_same_arms`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for `match` with identical arm bodies.

Note: Does not lint on wildcards if the `non_exhaustive_omitted_patterns_lint` feature is
enabled and disallowed.

#### Why is this bad?

This is probably a copy & paste error. If arm bodies
are the same on purpose, you can factor them
[using |](https://doc.rust-lang.org/book/patterns.html#multiple-patterns).

#### Example

```rust
match foo {
    Bar => bar(),
    Quz => quz(),
    Baz => bar(), // <= oops
}
```

This should probably be

```rust
match foo {
    Bar => bar(),
    Quz => quz(),
    Baz => baz(), // <= fixed
}
```

or if the original code was not a typo:

```rust
match foo {
    Bar | Baz => bar(), // <= shows the intent better
    Quz => quz(),
}
```

---

### `match_wild_err_arm`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for arm which matches all errors with `Err(_)`
and take drastic actions like `panic!`.

#### Why is this bad?

It is generally a bad practice, similar to
catching all exceptions in java with `catch(Exception)`

#### Example

```rust
let x: Result<i32, &str> = Ok(3);
match x {
    Ok(_) => println!("ok"),
    Err(_) => panic!("err"),
}
```

---

### `match_wildcard_for_single_variants`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.45.0 |

#### What it does

Checks for wildcard enum matches for a single variant.

#### Why is this bad?

New enum variants added by library updates can be missed.

#### Known problems

Suggested replacements may not use correct path to enum
if it’s not present in the current scope.

#### Example

```rust
match x {
    Foo::A => {},
    Foo::B => {},
    _ => {},
}
```

Use instead:

```rust
match x {
    Foo::A => {},
    Foo::B => {},
    Foo::C => {},
}
```

---

### `maybe_infinite_iter`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for iteration that may be infinite.

#### Why is this bad?

While there may be places where this is acceptable
(e.g., in event streams), in most cases this is simply an error.

#### Known problems

The code may have a condition to stop iteration, but
this lint is not clever enough to analyze it.

#### Example

```rust
let infinite_iter = 0..;
[0..].iter().zip(infinite_iter.take_while(|x| *x > 5));
```

---

### `mismatching_type_param_order`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.63.0 |

#### What it does

Checks for type parameters which are positioned inconsistently between
a type definition and impl block. Specifically, a parameter in an impl
block which has the same name as a parameter in the type def, but is in
a different place.

#### Why is this bad?

Type parameters are determined by their position rather than name.
Naming type parameters inconsistently may cause you to refer to the
wrong type parameter.

#### Limitations

This lint only applies to impl blocks with simple generic params, e.g.
`A`. If there is anything more complicated, such as a tuple, it will be
ignored.

#### Example

```rust
struct Foo<A, B> {
    x: A,
    y: B,
}
// inside the impl, B refers to Foo::A
impl<B, A> Foo<B, A> {}
```

Use instead:

```rust
struct Foo<A, B> {
    x: A,
    y: B,
}
impl<A, B> Foo<A, B> {}
```

---

### `missing_errors_doc`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.41.0 |

#### What it does

Checks the doc comments of publicly visible functions that
return a `Result` type and warns if there is no `# Errors` section.

#### Why is this bad?

Documenting the type of errors that can be returned from a
function can help callers write code to handle the errors appropriately.

#### Examples

Since the following function returns a `Result` it has an `# Errors` section in
its doc comment:

```rust
/// # Errors
///
/// Will return `Err` if `filename` does not exist or the user does not have
/// permission to read it.
pub fn read(filename: String) -> io::Result<String> {
    unimplemented!();
}
```

#### Configuration

- `check-private-items`:  Whether to also run the listed lints on private items.

(default: `false`)

---

### `missing_fields_in_debug`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.70.0 |

#### What it does

Checks for manual [core::fmt::Debug](https://doc.rust-lang.org/core/fmt/trait.Debug.html) implementations that do not use all fields.

#### Why is this bad?

A common mistake is to forget to update manual `Debug` implementations when adding a new field
to a struct or a new variant to an enum.

At the same time, it also acts as a style lint to suggest using [core::fmt::DebugStruct::finish_non_exhaustive](https://doc.rust-lang.org/core/fmt/struct.DebugStruct.html#method.finish_non_exhaustive)
for the times when the user intentionally wants to leave out certain fields (e.g. to hide implementation details).

#### Known problems

This lint works based on the `DebugStruct` helper types provided by the `Formatter`,
so this won’t detect `Debug` impls that use the `write!` macro.
Oftentimes there is more logic to a `Debug` impl if it uses `write!` macro, so it tries
to be on the conservative side and not lint in those cases in an attempt to prevent false positives.

This lint also does not look through function calls, so calling a function does not consider fields
used inside of that function as used by the `Debug` impl.

Lastly, it also ignores tuple structs as their `DebugTuple` formatter does not have a `finish_non_exhaustive`
method, as well as enums because their exhaustiveness is already checked by the compiler when matching on the enum,
making it much less likely to accidentally forget to update the `Debug` impl when adding a new variant.

#### Example

```rust
use std::fmt;
struct Foo {
    data: String,
    // implementation detail
    hidden_data: i32
}
impl fmt::Debug for Foo {
    fn fmt(&self, formatter: &mut fmt::Formatter<'_>) -> fmt::Result {
        formatter
            .debug_struct("Foo")
            .field("data", &self.data)
            .finish()
    }
}
```

Use instead:

```rust
use std::fmt;
struct Foo {
    data: String,
    // implementation detail
    hidden_data: i32
}
impl fmt::Debug for Foo {
    fn fmt(&self, formatter: &mut fmt::Formatter<'_>) -> fmt::Result {
        formatter
            .debug_struct("Foo")
            .field("data", &self.data)
            .finish_non_exhaustive()
    }
}
```

---

### `missing_panics_doc`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.51.0 |

#### What it does

Checks the doc comments of publicly visible functions that
may panic and warns if there is no `# Panics` section.

#### Why is this bad?

Documenting the scenarios in which panicking occurs
can help callers who do not want to panic to avoid those situations.

#### Examples

Since the following function may panic it has a `# Panics` section in
its doc comment:

```rust
/// # Panics
///
/// Will panic if y is 0
pub fn divide_by(x: i32, y: i32) -> i32 {
    if y == 0 {
        panic!("Cannot divide by 0")
    } else {
        x / y
    }
}
```

Individual panics within a function can be ignored with `#[expect]` or
`#[allow]`:

```rust
pub fn will_not_panic(x: usize) {
    #[expect(clippy::missing_panics_doc, reason = "infallible")]
    let y = NonZeroUsize::new(1).unwrap();

    // If any panics are added in the future the lint will still catch them
}
```

#### Configuration

- `check-private-items`:  Whether to also run the listed lints on private items.

(default: `false`)

---

### `must_use_candidate`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.40.0 |

#### What it does

Checks for public functions that have no
`#[must_use]` attribute, but return something not already marked
must-use, have no mutable arg and mutate no statics.

#### Why is this bad?

Not bad at all, this lint just shows places where
you could add the attribute.

#### Known problems

The lint only checks the arguments for mutable
types without looking if they are actually changed. On the other hand,
it also ignores a broad range of potentially interesting side effects,
because we cannot decide whether the programmer intends the function to
be called for the side effect or the result. Expect many false
positives. At least we don’t lint if the result type is unit or already
`#[must_use]`.

#### Examples

```rust
// this could be annotated with `#[must_use]`.
pub fn id<T>(t: T) -> T { t }
```

---

### `mut_mut`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for instances of `mut mut` references.

#### Why is this bad?

This is usually just a typo or a misunderstanding of how references work.

#### Example

```rust
let x = &mut &mut 1;

let mut x = &mut 1;
let y = &mut x;

fn foo(x: &mut &mut u32) {}
```

Use instead

```rust
let x = &mut 1;

let mut x = &mut 1;
let y = &mut *x; // reborrow

fn foo(x: &mut u32) {}
```

---

### `naive_bytecount`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for naive byte counts

#### Why is this bad?

The [bytecount](https://crates.io/crates/bytecount)
crate has methods to count your bytes faster, especially for large slices.

#### Known problems

If you have predominantly small slices, the
`bytecount::count(..)` method may actually be slower. However, if you can
ensure that less than 2³²-1 matches arise, the `naive_count_32(..)` can be
faster in those cases.

#### Example

```rust
let count = vec.iter().filter(|x| **x == 0u8).count();
```

Use instead:

```rust
let count = bytecount::count(&vec, 0u8);
```

---

### `needless_bitwise_bool`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.54.0 |

#### What it does

Checks for usage of bitwise and/or operators between booleans, where performance may be improved by using
a lazy and.

#### Why is this bad?

The bitwise operators do not support short-circuiting, so it may hinder code performance.
Additionally, boolean logic “masked” as bitwise logic is not caught by lints like `unnecessary_fold`

#### Known problems

This lint evaluates only when the right side is determined to have no side effects. At this time, that
determination is quite conservative.

#### Example

```rust
let (x,y) = (true, false);
if x & !y {} // where both x and y are booleans
```

Use instead:

```rust
let (x,y) = (true, false);
if x && !y {}
```

---

### `needless_continue`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

The lint checks for `if`-statements appearing in loops
that contain a `continue` statement in either their main blocks or their
`else`-blocks, when omitting the `else`-block possibly with some
rearrangement of code can make the code easier to understand.
The lint also checks if the last statement in the loop is a `continue`

#### Why is this bad?

Having explicit `else` blocks for `if` statements
containing `continue` in their THEN branch adds unnecessary branching and
nesting to the code. Having an else block containing just `continue` can
also be better written by grouping the statements following the whole `if`
statement within the THEN block and omitting the else block completely.

#### Example

```rust
while condition() {
    update_condition();
    if x {
        // ...
    } else {
        continue;
    }
    println!("Hello, world");
}
```

Could be rewritten as

```rust
while condition() {
    update_condition();
    if x {
        // ...
        println!("Hello, world");
    }
}
```

As another example, the following code

```rust
loop {
    if waiting() {
        continue;
    } else {
        // Do something useful
    }
    # break;
}
```

Could be rewritten as

```rust
loop {
    if waiting() {
        continue;
    }
    // Do something useful
    # break;
}
```

```rust
fn foo() -> ErrorKind { ErrorKind::NotFound }
for _ in 0..10 {
    match foo() {
        ErrorKind::NotFound => {
            eprintln!("not found");
            continue
        }
        ErrorKind::TimedOut => {
            eprintln!("timeout");
            continue
        }
        _ => {
            eprintln!("other error");
            continue
        }
    }
}
```

Could be rewritten as

```rust
fn foo() -> ErrorKind { ErrorKind::NotFound }
for _ in 0..10 {
    match foo() {
        ErrorKind::NotFound => {
            eprintln!("not found");
        }
        ErrorKind::TimedOut => {
            eprintln!("timeout");
        }
        _ => {
            eprintln!("other error");
        }
    }
}
```

---

### `needless_for_each`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.53.0 |

#### What it does

Checks for usage of `for_each` that would be more simply written as a
`for` loop.

#### Why is this bad?

`for_each` may be used after applying iterator transformers like
`filter` for better readability and performance. It may also be used to fit a simple
operation on one line.
But when none of these apply, a simple `for` loop is more idiomatic.

#### Example

```rust
let v = vec![0, 1, 2];
v.iter().for_each(|elem| {
    println!("{elem}");
})
```

Use instead:

```rust
let v = vec![0, 1, 2];
for elem in &v {
    println!("{elem}");
}
```

#### Known Problems

When doing things such as:

```rust
let v = vec![0, 1, 2];
v.iter().for_each(|elem| unsafe {
    libc::printf(c"%d\n".as_ptr(), elem);
});
```

This lint will not trigger.

---

### `needless_pass_by_value`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for functions taking arguments by value, but not
consuming them in its
body.

#### Why is this bad?

Taking arguments by reference is more flexible and can
sometimes avoid
unnecessary allocations.

#### Known problems

- This lint suggests taking an argument by reference,
however sometimes it is better to let users decide the argument type
(by using `Borrow` trait, for example), depending on how the function is used.

#### Example

```rust
fn foo(v: Vec<i32>) {
    assert_eq!(v.len(), 42);
}
```

should be

```rust
fn foo(v: &[i32]) {
    assert_eq!(v.len(), 42);
}
```

---

### `needless_raw_string_hashes`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for raw string literals with an unnecessary amount of hashes around them.

#### Why is this bad?

It’s just unnecessary, and makes it look like there’s more escaping needed than is actually
necessary.

#### Example

```rust
let r = r###"Hello, "world"!"###;
```

Use instead:

```rust
let r = r#"Hello, "world"!"#;
```

#### Configuration

- `allow-one-hash-in-raw-strings`:  Whether to allow `r#""#` when `r""` can be used

(default: `false`)

---

### `no_effect_underscore_binding`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.58.0 |

#### What it does

Checks for binding to underscore prefixed variable without side-effects.

#### Why is this bad?

Unlike dead code, these bindings are actually
executed. However, as they have no effect and shouldn’t be used further on, all they
do is make the code less readable.

#### Example

```rust
let _i_serve_no_purpose = 1;
```

---

### `no_mangle_with_rust_abi`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.69.0 |

#### What it does

Checks for Rust ABI functions with the `#[no_mangle]` attribute.

#### Why is this bad?

The Rust ABI is not stable, but in many simple cases matches
enough with the C ABI that it is possible to forget to add
`extern "C"` to a function called from C. Changes to the
Rust ABI can break this at any point.

#### Example

```rust
#[no_mangle]
 fn example(arg_one: u32, arg_two: usize) {}
```

Use instead:

```rust
#[no_mangle]
 extern "C" fn example(arg_one: u32, arg_two: usize) {}
```

---

### `non_std_lazy_statics`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Lints when `once_cell::sync::Lazy` or `lazy_static!` are used to define a static variable,
and suggests replacing such cases with `std::sync::LazyLock` instead.

Note: This lint will not trigger in crate with `no_std` context, or with MSRV < 1.80.0. It
also will not trigger on `once_cell::sync::Lazy` usage in crates which use other types
from `once_cell`, such as `once_cell::race::OnceBox`.

#### Why restrict this?

- Reduces the need for an extra dependency
- Enforce convention of using standard library types when possible

#### Example

```rust
lazy_static! {
    static ref FOO: String = "foo".to_uppercase();
}
static BAR: once_cell::sync::Lazy<String> = once_cell::sync::Lazy::new(|| "BAR".to_lowercase());
```

Use instead:

```rust
static FOO: std::sync::LazyLock<String> = std::sync::LazyLock::new(|| "FOO".to_lowercase());
static BAR: std::sync::LazyLock<String> = std::sync::LazyLock::new(|| "BAR".to_lowercase());
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `option_as_ref_cloned`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.77.0 |

#### What it does

Checks for usage of `.as_ref().cloned()` and `.as_mut().cloned()` on `Option`s

#### Why is this bad?

This can be written more concisely by cloning the `Option` directly.

#### Example

```rust
fn foo(bar: &Option<Vec<u8>>) -> Option<Vec<u8>> {
    bar.as_ref().cloned()
}
```

Use instead:

```rust
fn foo(bar: &Option<Vec<u8>>) -> Option<Vec<u8>> {
    bar.clone()
}
```

---

### `option_option`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `Option<Option<_>>` in function signatures and type
definitions

#### Why is this bad?

`Option<_>` represents an optional value. `Option<Option<_>>`
represents an optional value which itself wraps an optional. This is logically the
same thing as an optional value but has an unneeded extra level of wrapping.

If you have a case where `Some(Some(_))`, `Some(None)` and `None` are distinct cases,
consider a custom `enum` instead, with clear names for each case.

#### Example

```rust
fn get_data() -> Option<Option<u32>> {
    None
}
```

Better:

```rust
pub enum Contents {
    Data(Vec<u8>), // Was Some(Some(Vec<u8>))
    NotYetFetched, // Was Some(None)
    None,          // Was None
}

fn get_data() -> Contents {
    Contents::None
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `ptr_as_ptr`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.51.0 |

#### What it does

Checks for `as` casts between raw pointers that don’t change their
constness, namely `*const T` to `*const U` and `*mut T` to `*mut U`.

#### Why is this bad?

Though `as` casts between raw pointers are not terrible,
`pointer::cast` is safer because it cannot accidentally change the
pointer’s mutability, nor cast the pointer to other types like `usize`.

#### Example

```rust
let ptr: *const u32 = &42_u32;
let mut_ptr: *mut u32 = &mut 42_u32;
let _ = ptr as *const i32;
let _ = mut_ptr as *mut i32;
```

Use instead:

```rust
let ptr: *const u32 = &42_u32;
let mut_ptr: *mut u32 = &mut 42_u32;
let _ = ptr.cast::<i32>();
let _ = mut_ptr.cast::<i32>();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `ptr_cast_constness`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for `as` casts between raw pointers that change their constness, namely `*const T` to
`*mut T` and `*mut T` to `*const T`.

#### Why is this bad?

Though `as` casts between raw pointers are not terrible, `pointer::cast_mut` and
`pointer::cast_const` are safer because they cannot accidentally cast the pointer to another
type. Or, when null pointers are involved, `null()` and `null_mut()` can be used directly.

#### Example

```rust
let ptr: *const u32 = &42_u32;
let mut_ptr = ptr as *mut u32;
let ptr = mut_ptr as *const u32;
let ptr1 = std::ptr::null::<u32>() as *mut u32;
let ptr2 = std::ptr::null_mut::<u32>() as *const u32;
let ptr3 = std::ptr::null::<u32>().cast_mut();
let ptr4 = std::ptr::null_mut::<u32>().cast_const();
```

Use instead:

```rust
let ptr: *const u32 = &42_u32;
let mut_ptr = ptr.cast_mut();
let ptr = mut_ptr.cast_const();
let ptr1 = std::ptr::null_mut::<u32>();
let ptr2 = std::ptr::null::<u32>();
let ptr3 = std::ptr::null_mut::<u32>();
let ptr4 = std::ptr::null::<u32>();
```

---

### `ptr_offset_by_literal`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.94.0 |

#### What it does

Checks for usage of the `offset` pointer method with an integer
literal.

#### Why is this bad?

The `add` and `sub` methods more accurately express the intent.

#### Example

```rust
let vec = vec![b'a', b'b', b'c'];
let ptr = vec.as_ptr();

unsafe {
    ptr.offset(-8);
}
```

Could be written:

```rust
let vec = vec![b'a', b'b', b'c'];
let ptr = vec.as_ptr();

unsafe {
    ptr.sub(8);
}
```

---

### `pub_underscore_fields`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.77.0 |

#### What it does

Checks whether any field of the struct is prefixed with an `_` (underscore) and also marked
`pub` (public)

#### Why is this bad?

Fields prefixed with an `_` are inferred as unused, which suggests it should not be marked
as `pub`, because marking it as `pub` infers it will be used.

#### Example

```rust
struct FileHandle {
    pub _descriptor: usize,
}
```

Use instead:

```rust
struct FileHandle {
    _descriptor: usize,
}
```

OR

```rust
struct FileHandle {
    pub descriptor: usize,
}
```

#### Configuration

- `pub-underscore-fields-behavior`:  Lint “public” fields in a struct that are prefixed with an underscore based on their
exported visibility, or whether they are marked as “pub”.

(default: `"PubliclyExported"`)

---

### `range_minus_one`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for inclusive ranges where 1 is subtracted from
the upper bound, e.g., `x..=(y-1)`.

#### Why is this bad?

The code is more readable with an exclusive range
like `x..y`.

#### Limitations

The lint is conservative and will trigger only when switching
from an inclusive to an exclusive range is provably safe from
a typing point of view. This corresponds to situations where
the range is used as an iterator, or for indexing.

#### Example

```rust
for i in x..=(y-1) {
    // ..
}
```

Use instead:

```rust
for i in x..y {
    // ..
}
```

---

### `range_plus_one`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for exclusive ranges where 1 is added to the
upper bound, e.g., `x..(y+1)`.

#### Why is this bad?

The code is more readable with an inclusive range
like `x..=y`.

#### Limitations

The lint is conservative and will trigger only when switching
from an exclusive to an inclusive range is provably safe from
a typing point of view. This corresponds to situations where
the range is used as an iterator, or for indexing.

#### Known problems

Will add unnecessary pair of parentheses when the
expression is not wrapped in a pair but starts with an opening parenthesis
and ends with a closing one.
I.e., `let _ = (f()+1)..(f()+1)` results in `let _ = ((f()+1)..=f())`.

Also in many cases, inclusive ranges are still slower to run than
exclusive ranges, because they essentially add an extra branch that
LLVM may fail to hoist out of the loop.

#### Example

```rust
for i in x..(y+1) {
    // ..
}
```

Use instead:

```rust
for i in x..=y {
    // ..
}
```

---

### `redundant_closure_for_method_calls`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.35.0 |

#### What it does

Checks for closures which only invoke a method on the closure
argument and can be replaced by referencing the method directly.

#### Why is this bad?

It’s unnecessary to create the closure.

#### Example

```rust
Some('a').map(|s| s.to_uppercase());
```

may be rewritten as

```rust
Some('a').map(char::to_uppercase);
```

---

### `redundant_else`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.50.0 |

#### What it does

Checks for `else` blocks that can be removed without changing semantics.

#### Why is this bad?

The `else` block adds unnecessary indentation and verbosity.

#### Known problems

Some may prefer to keep the `else` block for clarity.

#### Example

```rust
fn my_func(count: u32) {
    if count == 0 {
        print!("Nothing to do");
        return;
    } else {
        print!("Moving on...");
    }
}
```

Use instead:

```rust
fn my_func(count: u32) {
    if count == 0 {
        print!("Nothing to do");
        return;
    }
    print!("Moving on...");
}
```

---

### `ref_as_ptr`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.78.0 |

#### What it does

Checks for casts of references to pointer using `as`
and suggests `std::ptr::from_ref` and `std::ptr::from_mut` instead.

#### Why is this bad?

Using `as` casts may result in silently changing mutability or type.

#### Example

```rust
let a_ref = &1;
let a_ptr = a_ref as *const _;
```

Use instead:

```rust
let a_ref = &1;
let a_ptr = std::ptr::from_ref(a_ref);
```

---

### `ref_binding_to_reference`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.54.0 |

#### What it does

Checks for `ref` bindings which create a reference to a reference.

#### Why is this bad?

The address-of operator at the use site is clearer about the need for a reference.

#### Example

```rust
let x = Some("");
if let Some(ref x) = x {
    // use `x` here
}
```

Use instead:

```rust
let x = Some("");
if let Some(x) = x {
    // use `&x` here
}
```

---

### `ref_option`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.83.0 |

#### What it does

Warns when a function signature uses `&Option<T>` instead of `Option<&T>`.

#### Why is this bad?

More flexibility, better memory optimization, and more idiomatic Rust code.

`&Option<T>` in a function signature breaks encapsulation because the caller must own T
and move it into an Option to call with it. When returned, the owner must internally store
it as `Option<T>` in order to return it.
At a lower level, `&Option<T>` points to memory with the `presence` bit flag plus the `T` value,
whereas `Option<&T>` is usually [optimized](https://doc.rust-lang.org/1.81.0/std/option/index.html#representation)
to a single pointer, so it may be more optimal.

See this [YouTube video](https://www.youtube.com/watch?v=6c7pZYP_iIE) by
Logan Smith for an in-depth explanation of why this is important.

#### Known problems

This lint recommends changing the function signatures, but it cannot
automatically change the function calls or the function implementations.

#### Example

```rust
// caller uses  foo(&opt)
fn foo(a: &Option<String>) {}
fn bar(&self) -> &Option<String> { &None }
```

Use instead:

```rust
// caller should use  `foo1(opt.as_ref())`
fn foo1(a: Option<&String>) {}
// better yet, use string slice  `foo2(opt.as_deref())`
fn foo2(a: Option<&str>) {}
fn bar(&self) -> Option<&String> { None }
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `ref_option_ref`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.49.0 |

#### What it does

Checks for usage of `&Option<&T>`.

#### Why is this bad?

Since `&` is Copy, it’s useless to have a
reference on `Option<&T>`.

#### Known problems

It may be irrelevant to use this lint on
public API code as it will make a breaking change to apply it.

#### Example

```rust
let x: &Option<&u32> = &Some(&0u32);
```

Use instead:

```rust
let x: Option<&u32> = Some(&0u32);
```

---

### `return_self_not_must_use`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.59.0 |

#### What it does

This lint warns when a method returning `Self` doesn’t have the `#[must_use]` attribute.

#### Why is this bad?

Methods returning `Self` often create new values, having the `#[must_use]` attribute
prevents users from “forgetting” to use the newly created value.

The `#[must_use]` attribute can be added to the type itself to ensure that instances
are never forgotten. Functions returning a type marked with `#[must_use]` will not be
linted, as the usage is already enforced by the type attribute.

#### Limitations

This lint is only applied on methods taking a `self` argument. It would be mostly noise
if it was added on constructors for example.

#### Example

```rust
pub struct Bar;
impl Bar {
    // Missing attribute
    pub fn bar(&self) -> Self {
        Self
    }
}
```

Use instead:

```rust
// It's better to have the `#[must_use]` attribute on the method like this:
pub struct Bar;
impl Bar {
    #[must_use]
    pub fn bar(&self) -> Self {
        Self
    }
}

// Or on the type definition like this:
#[must_use]
pub struct Bar;
impl Bar {
    pub fn bar(&self) -> Self {
        Self
    }
}
```

---

### `same_functions_in_if_condition`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.41.0 |

#### What it does

Checks for consecutive `if`s with the same function call.

#### Why is this bad?

This is probably a copy & paste error.
Despite the fact that function can have side effects and `if` works as
intended, such an approach is implicit and can be considered a “code smell”.

#### Example

```rust
if foo() == bar {
    …
} else if foo() == bar {
    …
}
```

This probably should be:

```rust
if foo() == bar {
    …
} else if foo() == baz {
    …
}
```

or if the original code was not a typo and called function mutates a state,
consider move the mutation out of the `if` condition to avoid similarity to
a copy & paste error:

```rust
let first = foo();
if first == bar {
    …
} else {
    let second = foo();
    if second == bar {
    …
    }
}
```

---

### `same_length_and_capacity`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.94.0 |

#### What it does

Checks for usages of `Vec::from_raw_parts` and `String::from_raw_parts`
where the same expression is used for the length and the capacity.

#### Why is this bad?

If the same expression is being passed for the length and
capacity, it is most likely a semantic error. In the case of a
Vec, for example, the only way to end up with one that has
the same length and capacity is by going through a boxed slice,
e.g. `Box::from(some_vec)`, which shrinks the capacity to match
the length.

#### Example

```rust
#![feature(vec_into_raw_parts)]
let mut original: Vec::<i32> = Vec::with_capacity(20);
original.extend([1, 2, 3, 4, 5]);

let (ptr, mut len, cap) = original.into_raw_parts();

// I will add three more integers:
unsafe {
   let ptr = ptr as *mut i32;

   for i in 6..9 {
       *ptr.add(i - 1) = i as i32;
       len += 1;
   }
}

// But I forgot the capacity was separate from the length:
let reconstructed = unsafe { Vec::from_raw_parts(ptr, len, len) };
```

Use instead:

```rust
#![feature(vec_into_raw_parts)]
let mut original: Vec::<i32> = Vec::with_capacity(20);
original.extend([1, 2, 3, 4, 5]);

let (ptr, mut len, cap) = original.into_raw_parts();

// I will add three more integers:
unsafe {
   let ptr = ptr as *mut i32;

   for i in 6..9 {
       *ptr.add(i - 1) = i as i32;
       len += 1;
   }
}

// This time, leverage the previously saved capacity:
let reconstructed = unsafe { Vec::from_raw_parts(ptr, len, cap) };
```

---

### `self_only_used_in_recursion`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.92.0 |

#### What it does

Checks for `self` receiver that is only used in recursion with no side-effects.

#### Why is this bad?

It may be possible to remove the `self` argument, allowing the function to be
used without an object of type `Self`.

#### Example

```rust
struct Foo;
impl Foo {
    fn f(&self, n: u32) -> u32 {
        if n == 0 {
            1
        } else {
            n * self.f(n - 1)
        }
    }
}
```

Use instead:

```rust
struct Foo;
impl Foo {
    fn f(n: u32) -> u32 {
        if n == 0 {
            1
        } else {
            n * Self::f(n - 1)
        }
    }
}
```

#### Known problems

Too many code paths in the linting code are currently untested and prone to produce false
positives or are prone to have performance implications.

In some cases, this would not catch all useless arguments.

```rust
struct Foo;
impl Foo {
    fn foo(&self, a: usize) -> usize {
        let f = |x| x;

        if a == 0 {
            1
        } else {
            f(self).foo(a)
        }
    }
}
```

For example, here `self` is only used in recursion, but the lint would not catch it.

List of some examples that can not be caught:

- binary operation of non-primitive types
- closure usage
- some `break` relative operations
- struct pattern binding

Also, when you recurse the function name with path segments, it is not possible to detect.

---

### `semicolon_if_nothing_returned`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.52.0 |

#### What it does

Looks for blocks of expressions and fires if the last expression returns
`()` but is not followed by a semicolon.

#### Why is this bad?

The semicolon might be optional but when extending the block with new
code, it doesn’t require a change in previous last line.

#### Example

```rust
fn main() {
    println!("Hello world")
}
```

Use instead:

```rust
fn main() {
    println!("Hello world");
}
```

---

### `should_panic_without_expect`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | 1.74.0 |

#### What it does

Checks for `#[should_panic]` attributes without specifying the expected panic message.

#### Why is this bad?

The expected panic message should be specified to ensure that the test is actually
panicking with the expected message, and not another unrelated panic.

#### Example

```rust
fn random() -> i32 { 0 }

#[should_panic]
#[test]
fn my_test() {
    let _ = 1 / random();
}
```

Use instead:

```rust
fn random() -> i32 { 0 }

#[should_panic = "attempt to divide by zero"]
#[test]
fn my_test() {
    let _ = 1 / random();
}
```

---

### `similar_names`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for names that are very similar and thus confusing.

Note: this lint looks for similar names throughout each
scope. To allow it, you need to allow it on the scope
level, not on the name that is reported.

#### Why is this bad?

It’s hard to distinguish between names that differ only
by a single character.

#### Example

```rust
let checked_exp = something;
let checked_expr = something_else;
```

---

### `single_char_pattern`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for string methods that receive a single-character
`str` as an argument, e.g., `_.split("x")`.

#### Why is this bad?

While this can make a perf difference on some systems,
benchmarks have proven inconclusive. But at least using a
char literal makes it clear that we are looking at a single
character.

#### Known problems

Does not catch multi-byte unicode characters. This is by
design, on many machines, splitting by a non-ascii char is
actually slower. Please do your own measurements instead of
relying solely on the results of this lint.

#### Example

```rust
_.split("x");
```

Use instead:

```rust
_.split('x');
```

---

### `single_match_else`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for matches with two arms where an `if let else` will
usually suffice.

#### Why is this bad?

Just readability – `if let` nests less than a `match`.

#### Known problems

Personal style preferences may differ.

#### Example

Using `match`:

```rust
match x {
    Some(ref foo) => bar(foo),
    _ => bar(&other_ref),
}
```

Using `if let` with `else`:

```rust
if let Some(ref foo) = x {
    bar(foo);
} else {
    bar(&other_ref);
}
```

---

### `stable_sort_primitive`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.47.0 |

#### What it does

When sorting primitive values (integers, bools, chars, as well
as arrays, slices, and tuples of such items), it is typically better to
use an unstable sort than a stable sort.

#### Why is this bad?

Typically, using a stable sort consumes more memory and cpu cycles.
Because values which compare equal are identical, preserving their
relative order (the guarantee that a stable sort provides) means
nothing, while the extra costs still apply.

#### Known problems

As pointed out in
[issue #8241](https://github.com/rust-lang/rust-clippy/issues/8241),
a stable sort can instead be significantly faster for certain scenarios
(eg. when a sorted vector is extended with new data and resorted).

For more information and benchmarking results, please refer to the
issue linked above.

#### Example

```rust
let mut vec = vec![2, 1, 3];
vec.sort();
```

Use instead:

```rust
let mut vec = vec![2, 1, 3];
vec.sort_unstable();
```

---

### `str_split_at_newline`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.77.0 |

#### What it does

Checks for usages of `str.trim().split("\n")` and `str.trim().split("\r\n")`.

#### Why is this bad?

Hard-coding the line endings makes the code less compatible. `str.lines` should be used instead.

#### Example

```rust
"some\ntext\nwith\nnewlines\n".trim().split('\n');
```

Use instead:

```rust
"some\ntext\nwith\nnewlines\n".lines();
```

#### Known Problems

This lint cannot detect if the split is intentionally restricted to a single type of newline (`"\n"` or
`"\r\n"`), for example during the parsing of a specific file format in which precisely one newline type is
valid.

---

### `string_add_assign`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for string appends of the form `x = x + y` (without
`let`!).

#### Why is this bad?

It’s not really bad, but some people think that the
`.push_str(_)` method is more readable.

#### Example

```rust
let mut x = "Hello".to_owned();
x = x + ", World";

// More readable
x += ", World";
x.push_str(", World");
```

---

### `struct_excessive_bools`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.43.0 |

#### What it does

Checks for excessive
use of bools in structs.

#### Why is this bad?

Excessive bools in a struct is often a sign that
the type is being used to represent a state
machine, which is much better implemented as an
enum.

The reason an enum is better for state machines
over structs is that enums more easily forbid
invalid states.

Structs with too many booleans may benefit from refactoring
into multi variant enums for better readability and API.

#### Example

```rust
struct S {
    is_pending: bool,
    is_processing: bool,
    is_finished: bool,
}
```

Use instead:

```rust
enum S {
    Pending,
    Processing,
    Finished,
}
```

#### Configuration

- `max-struct-bools`:  The maximum number of bool fields a struct can have

(default: `3`)

---

### `struct_field_names`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.75.0 |

#### What it does

Detects struct fields that are prefixed or suffixed
by the same characters or the name of the struct itself.

#### Why is this bad?

Information common to all struct fields is better represented in the struct name.

#### Limitations

Characters with no casing will be considered when comparing prefixes/suffixes
This applies to numbers and non-ascii characters without casing
e.g. `foo1` and `foo2` is considered to have different prefixes
(the prefixes are `foo1` and `foo2` respectively), as also `bar螃`, `bar蟹`

#### Example

```rust
struct Cake {
    cake_sugar: u8,
    cake_flour: u8,
    cake_eggs: u8
}
```

Use instead:

```rust
struct Cake {
    sugar: u8,
    flour: u8,
    eggs: u8
}
```

#### Configuration

- `struct-field-name-threshold`:  The minimum number of struct fields for the lints about field names to trigger

(default: `3`)

---

### `too_many_lines`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.34.0 |

#### What it does

Checks for functions with a large amount of lines.

#### Why is this bad?

Functions with a lot of lines are harder to understand
due to having to look at a larger amount of code to understand what the
function is doing. Consider splitting the body of the function into
multiple functions.

#### Example

```rust
fn im_too_long() {
    println!("");
    // ... 100 more LoC
    println!("");
}
```

#### Configuration

- `too-many-lines-threshold`:  The maximum number of lines a function or method can have

(default: `100`)

---

### `transmute_ptr_to_ptr`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for transmutes from a pointer to a pointer, or
from a reference to a reference.

#### Why is this bad?

Transmutes are dangerous, and these can instead be
written as casts.

#### Example

```rust
let ptr = &1u32 as *const u32;
unsafe {
    // pointer-to-pointer transmute
    let _: *const f32 = std::mem::transmute(ptr);
    // ref-ref transmute
    let _: &f32 = std::mem::transmute(&1u32);
}
// These can be respectively written:
let _ = ptr as *const f32;
let _ = unsafe{ &*(&1u32 as *const u32 as *const f32) };
```

---

### `trivially_copy_pass_by_ref`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for functions taking arguments by reference, where
the argument type is `Copy` and small enough to be more efficient to always
pass by value.

#### Why is this bad?

In many calling conventions instances of structs will
be passed through registers if they fit into two or less general purpose
registers.

#### Known problems

This lint is target dependent, some cases will lint on 64-bit targets but
not 32-bit or lower targets.

The configuration option `trivial_copy_size_limit` can be set to override
this limit for a project.

This lint attempts to allow passing arguments by reference if a reference
to that argument is returned. This is implemented by comparing the lifetime
of the argument and return value for equality. However, this can cause
false positives in cases involving multiple lifetimes that are bounded by
each other.

Also, it does not take account of other similar cases where getting memory addresses
matters; namely, returning the pointer to the argument in question,
and passing the argument, as both references and pointers,
to a function that needs the memory address. For further details, refer to
[this issue](https://github.com/rust-lang/rust-clippy/issues/5953)
that explains a real case in which this false positive
led to an **undefined behavior** introduced with unsafe code.

#### Example

```rust
fn foo(v: &u32) {}
```

Use instead:

```rust
fn foo(v: u32) {}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)
- `trivial-copy-size-limit`:  The maximum size (in bytes) to consider a `Copy` type for passing by value instead of by
reference.

(default: `target_pointer_width`)

---

### `unchecked_time_subtraction`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.67.0 |

#### What it does

Lints subtraction between an `Instant` and a `Duration`, or between two `Duration` values.

#### Why is this bad?

Unchecked subtraction could cause underflow on certain platforms, leading to
unintentional panics.

#### Example

```rust
let time_passed = Instant::now() - Duration::from_secs(5);
let dur1 = Duration::from_secs(3);
let dur2 = Duration::from_secs(5);
let diff = dur1 - dur2;
```

Use instead:

```rust
let time_passed = Instant::now().checked_sub(Duration::from_secs(5));
let dur1 = Duration::from_secs(3);
let dur2 = Duration::from_secs(5);
let diff = dur1.checked_sub(dur2);
```

#### Past names

- unchecked_duration_subtraction

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `unicode_not_nfc`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for string literals that contain Unicode in a form
that is not equal to its
[NFC-recomposition](http://www.unicode.org/reports/tr15/#Norm_Forms).

#### Why is this bad?

If such a string is compared to another, the results
may be surprising.

#### Example

You may not see it, but “à”“ and “à”“ aren’t the same string. The
former when escaped is actually `"a\u{300}"` while the latter is `"\u{e0}"`.

---

### `uninlined_format_args`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.66.0 |

#### What it does

Detect when a variable is not inlined in a format string,
and suggests to inline it.

#### Why is this bad?

Non-inlined code is slightly more difficult to read and understand,
as it requires arguments to be matched against the format string.
The inlined syntax, where allowed, is simpler.

#### Example

```rust
format!("{}", var);
format!("{:?}", var);
format!("{v:?}", v = var);
format!("{0} {0}", var);
format!("{0:1$}", var, width);
format!("{:.*}", prec, var);
```

Use instead:

```rust
format!("{var}");
format!("{var:?}");
format!("{var:?}");
format!("{var} {var}");
format!("{var:width$}");
format!("{var:.prec$}");
```

If `allow-mixed-uninlined-format-args` is set to `false` in clippy.toml,
the following code will also trigger the lint:

```rust
format!("{} {}", var, 1+2);
```

Use instead:

```rust
format!("{var} {}", 1+2);
```

#### Known Problems

If a format string contains a numbered argument that cannot be inlined
nothing will be suggested, e.g. `println!("{0}={1}", var, 1+2)`.

#### Configuration

- `allow-mixed-uninlined-format-args`:  Whether to allow mixed uninlined format args, e.g. `format!("{} {}", a, foo.bar)`

(default: `true`)
- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `unnecessary_box_returns`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.70.0 |

#### What it does

Checks for a return type containing a `Box<T>` where `T` implements `Sized`

The lint ignores `Box<T>` where `T` is larger than `unnecessary_box_size`,
as returning a large `T` directly may be detrimental to performance.

#### Why is this bad?

It’s better to just return `T` in these cases. The caller may not need
the value to be boxed, and it’s expensive to free the memory once the
`Box<T>` been dropped.

#### Example

```rust
fn foo() -> Box<String> {
    Box::new(String::from("Hello, world!"))
}
```

Use instead:

```rust
fn foo() -> String {
    String::from("Hello, world!")
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)
- `unnecessary-box-size`:  The byte size a `T` in `Box<T>` can have, below which it triggers the `clippy::unnecessary_box` lint

(default: `128`)

---

### `unnecessary_debug_formatting`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.87.0 |

#### What it does

Checks for `Debug` formatting (`{:?}`) applied to an `OsStr` or `Path`.

#### Why is this bad?

Rust doesn’t guarantee what `Debug` formatting looks like, and it could
change in the future. `OsStr`s and `Path`s can be `Display` formatted
using their `display` methods.

Furthermore, with `Debug` formatting, certain characters are escaped.
Thus, a `Debug` formatted `Path` is less likely to be clickable.

#### Example

```rust
let path = Path::new("...");
println!("The path is {:?}", path);
```

Use instead:

```rust
let path = Path::new("…");
println!("The path is {}", path.display());
```

---

### `unnecessary_join`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.61.0 |

#### What it does

Checks for usage of `.collect::<Vec<String>>().join("")` on iterators.

#### Why is this bad?

`.collect::<String>()` is more concise and might be more performant

#### Example

```rust
let vector = vec!["hello",  "world"];
let output = vector.iter().map(|item| item.to_uppercase()).collect::<Vec<String>>().join("");
println!("{}", output);
```

The correct use would be:

```rust
let vector = vec!["hello",  "world"];
let output = vector.iter().map(|item| item.to_uppercase()).collect::<String>();
println!("{}", output);
```

#### Known problems

While `.collect::<String>()` is sometimes more performant, there are cases where
using `.collect::<String>()` over `.collect::<Vec<String>>().join("")`
will prevent loop unrolling and will result in a negative performance impact.

Additionally, differences have been observed between aarch64 and x86_64 assembly output,
with aarch64 tending to producing faster assembly in more cases when using `.collect::<String>()`

---

### `unnecessary_literal_bound`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.84.0 |

#### What it does

Detects functions that are written to return `&str` that could return `&'static str` but instead return a `&'a str`.

#### Why is this bad?

This leaves the caller unable to use the `&str` as `&'static str`, causing unnecessary allocations or confusion.
This is also most likely what you meant to write.

#### Example

```rust
impl MyType {
    fn returns_literal(&self) -> &str {
        "Literal"
    }
}
```

Use instead:

```rust
impl MyType {
    fn returns_literal(&self) -> &'static str {
        "Literal"
    }
}
```

Or, in case you may return a non-literal `str` in future:

```rust
impl MyType {
    fn returns_literal<'a>(&'a self) -> &'a str {
        "Literal"
    }
}
```

---

### `unnecessary_semicolon`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Checks for the presence of a semicolon at the end of
a `match` or `if` statement evaluating to `()`.

#### Why is this bad?

The semicolon is not needed, and may be removed to
avoid confusion and visual clutter.

#### Example

```rust
if a > 10 {
    println!("a is greater than 10");
};
```

Use instead:

```rust
if a > 10 {
    println!("a is greater than 10");
}
```

---

### `unnecessary_trailing_comma`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.95.0 |

#### What it does

Suggests removing an unnecessary trailing comma before the closing parenthesis in
single-line macro invocations.

#### Why is this bad?

The trailing comma is redundant and removing it is more consistent with how
`rustfmt` formats regular function calls.

#### Known limitations

This lint currently only runs on format-like macros (e.g. `format!`, `println!`,
`write!`) because it relies on format-argument parsing; applying it to arbitrary
user macros could cause incorrect suggestions. It may be extended to other
macros in the future. Only single-line macro invocations are linted.

#### Example

```rust
println!("Foo={}", 1,);
```

Use instead:

```rust
println!("Foo={}", 1);
```

---

### `unnecessary_wraps`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.50.0 |

#### What it does

Checks for private functions that only return `Ok` or `Some`.

#### Why is this bad?

It is not meaningful to wrap values when no `None` or `Err` is returned.

#### Known problems

There can be false positives if the function signature is designed to
fit some external requirement.

#### Example

```rust
fn get_cool_number(a: bool, b: bool) -> Option<i32> {
    if a && b {
        return Some(50);
    }
    if a {
        Some(0)
    } else {
        Some(10)
    }
}
```

Use instead:

```rust
fn get_cool_number(a: bool, b: bool) -> i32 {
    if a && b {
        return 50;
    }
    if a {
        0
    } else {
        10
    }
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `unnested_or_patterns`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.46.0 |

#### What it does

Checks for unnested or-patterns, e.g., `Some(0) | Some(2)` and
suggests replacing the pattern with a nested one, `Some(0 | 2)`.

Another way to think of this is that it rewrites patterns in
*disjunctive normal form (DNF)* into *conjunctive normal form (CNF)*.

#### Why is this bad?

In the example above, `Some` is repeated, which unnecessarily complicates the pattern.

#### Example

```rust
fn main() {
    if let Some(0) | Some(2) = Some(0) {}
}
```

Use instead:

```rust
fn main() {
    if let Some(0 | 2) = Some(0) {}
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `unreadable_literal`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if a long integral or floating-point constant does
not contain underscores.

#### Why is this bad?

Reading long numbers is difficult without separators.

#### Example

```rust
61864918973511
```

Use instead:

```rust
61_864_918_973_511
```

#### Configuration

- `unreadable-literal-lint-fractions`:  Should the fraction of a decimal be linted to include separators.

(default: `true`)

---

### `unsafe_derive_deserialize`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.45.0 |

#### What it does

Checks for deriving `serde::Deserialize` on a type that
has methods using `unsafe`.

#### Why is this bad?

Deriving `serde::Deserialize` will create a constructor
that may violate invariants held by another constructor.

#### Example

```rust
use serde::Deserialize;

#[derive(Deserialize)]
pub struct Foo {
    // ..
}

impl Foo {
    pub fn new() -> Self {
        // setup here ..
    }

    pub unsafe fn parts() -> (&str, &str) {
        // assumes invariants hold
    }
}
```

---

### `unused_async`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.54.0 |

#### What it does

Checks for functions that are declared `async` but have no `.await`s inside of them.

#### Why is this bad?

Async functions with no async code create overhead, both mentally and computationally.
Callers of async methods either need to be calling from an async function themselves or run it on an executor, both of which
causes runtime overhead and hassle for the caller.

#### Example

```rust
async fn get_random_number() -> i64 {
    4 // Chosen by fair dice roll. Guaranteed to be random.
}
let number_future = get_random_number();
```

Use instead:

```rust
fn get_random_number_improved() -> i64 {
    4 // Chosen by fair dice roll. Guaranteed to be random.
}
let number_future = async { get_random_number_improved() };
```

---

### `unused_self`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks methods that contain a `self` argument but don’t use it

#### Why is this bad?

It may be clearer to define the method as an associated function instead
of an instance method if it doesn’t require `self`.

#### Example

```rust
struct A;
impl A {
    fn method(&self) {}
}
```

Could be written:

```rust
struct A;
impl A {
    fn method() {}
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `used_underscore_binding`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the use of bindings with a single leading
underscore.

#### Why is this bad?

A single leading underscore is usually used to indicate
that a binding will not be used. Using such a binding breaks this
expectation.

#### Known problems

The lint does not work properly with desugaring and
macro, it has been allowed in the meantime.

#### Example

```rust
let _x = 0;
let y = _x + 1; // Here we are using `_x`, even though it has a leading
                // underscore. We should rename `_x` to `x`
```

---

### `used_underscore_items`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.83.0 |

#### What it does

Checks for the use of item with a single leading
underscore.

#### Why is this bad?

A single leading underscore is usually used to indicate
that a item will not be used. Using such a item breaks this
expectation.

#### Example

```rust
fn _foo() {}

struct _FooStruct {}

fn main() {
    _foo();
    let _ = _FooStruct{};
}
```

Use instead:

```rust
fn foo() {}

struct FooStruct {}

fn main() {
    foo();
    let _ = FooStruct{};
}
```

---

### `verbose_bit_mask`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for bit masks that can be replaced by a call
to `trailing_zeros`

#### Why is this bad?

`x.trailing_zeros() >= 4` is much clearer than `x & 15 == 0`

#### Example

```rust
if x & 0b1111 == 0 { }
```

Use instead:

```rust
if x.trailing_zeros() >= 4 { }
```

#### Configuration

- `verbose-bit-mask-threshold`:  The maximum allowed size of a bit mask before suggesting to use ‘trailing_zeros’

(default: `1`)

---

### `wildcard_imports`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.43.0 |

#### What it does

Checks for wildcard imports `use _::*`.

#### Why is this bad?

wildcard imports can pollute the namespace. This is especially bad if
you try to import something through a wildcard, that already has been imported by name from
a different source:

```rust
use crate1::foo; // Imports a function named foo
use crate2::*; // Has a function named foo

foo(); // Calls crate1::foo
```

This can lead to confusing error messages at best and to unexpected behavior at worst.

#### Exceptions

Wildcard imports are allowed from modules that their name contains `prelude`. Many crates
(including the standard library) provide modules named “prelude” specifically designed
for wildcard import.

Wildcard imports reexported through `pub use` are also allowed.

`use super::*` is allowed in test modules. This is defined as any module with “test” in the name.

These exceptions can be disabled using the `warn-on-all-wildcard-imports` configuration flag.

#### Known problems

If macros are imported through the wildcard, this macro is not included
by the suggestion and has to be added by hand.

Applying the suggestion when explicit imports of the things imported with a glob import
exist, may result in `unused_imports` warnings.

#### Example

```rust
use crate1::*;

foo();
```

Use instead:

```rust
use crate1::foo;

foo();
```

#### Configuration

- `allowed-wildcard-imports`:  List of path segments allowed to have wildcard imports.

#### Example

```toml
allowed-wildcard-imports = [ "utils", "common" ]
```

#### Noteworthy

1. This configuration has no effects if used with `warn_on_all_wildcard_imports = true`.
2. Paths with any segment that containing the word ‘prelude’
are already allowed by default.

(default: `[]`)

- `warn-on-all-wildcard-imports`:  Whether to emit warnings on all wildcard imports, including those from `prelude`, from `super` in tests,
or for `pub use` reexports.

(default: `false`)

---

### `zero_sized_map_values`

| 属性 | 值 |
|------|----|
| 分组 | pedantic |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.50.0 |

#### What it does

Checks for maps with zero-sized value types anywhere in the code.

#### Why is this bad?

Since there is only a single value for a zero-sized type, a map
containing zero sized values is effectively a set. Using a set in that case improves
readability and communicates intent more clearly.

#### Known problems

- A zero-sized type cannot be recovered later if it contains private fields.
- This lints the signature of public items

#### Example

```rust
fn unique_words(text: &str) -> HashMap<&str, ()> {
    todo!();
}
```

Use instead:

```rust
fn unique_words(text: &str) -> HashSet<&str> {
    todo!();
}
```

---

## Restriction (130)

*限制 - 不一定是坏的但可能需要限制的代码，默认 allow*

### `absolute_paths`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Checks for usage of items through absolute paths, like `std::env::current_dir`.

#### Why restrict this?

Many codebases have their own style when it comes to importing, but one that is seldom used
is using absolute paths *everywhere*. This is generally considered unidiomatic, and you
should add a `use` statement.

The default maximum segments (2) is pretty strict, you may want to increase this in
`clippy.toml`.

Note: One exception to this is code from macro expansion - this does not lint such cases, as
using absolute paths is the proper way of referencing items in one.

#### Known issues

There are currently a few cases which are not caught by this lint:

- Macro calls. e.g. `path::to::macro!()`
- Derive macros. e.g. `#[derive(path::to::macro)]`
- Attribute macros. e.g. `#[path::to::macro]`

#### Example

```rust
let x = std::f64::consts::PI;
```

Use any of the below instead, or anything else:

```rust
use std::f64;
use std::f64::consts;
use std::f64::consts::PI;
let x = f64::consts::PI;
let x = consts::PI;
let x = PI;
use std::f64::consts as f64_consts;
let x = f64_consts::PI;
```

#### Configuration

- `absolute-paths-allowed-crates`:  Which crates to allow absolute paths from

(default: `[]`)
- `absolute-paths-max-segments`:  The maximum number of segments a path can have before being linted, anything above this will
be linted.

(default: `2`)

---

### `alloc_instead_of_core`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Finds items imported through `alloc` when available through `core`.

#### Why restrict this?

Crates which have `no_std` compatibility and may optionally require alloc may wish to ensure types are
imported from core to ensure disabling `alloc` does not cause the crate to fail to compile. This lint
is also useful for crates migrating to become `no_std` compatible.

#### Known problems

The lint is only partially aware of the required MSRV for items that were originally in `std` but moved
to `core`.

#### Example

```rust
use alloc::slice::from_ref;
```

Use instead:

```rust
use core::slice::from_ref;
```

---

### `allow_attributes`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.70.0 |

#### What it does

Checks for usage of the `#[allow]` attribute and suggests replacing it with
the `#[expect]` attribute (See [RFC 2383](https://rust-lang.github.io/rfcs/2383-lint-reasons.html))

This lint only warns outer attributes (`#[allow]`), as inner attributes
(`#![allow]`) are usually used to enable or disable lints on a global scale.

#### Why is this bad?

`#[expect]` attributes suppress the lint emission, but emit a warning, if
the expectation is unfulfilled. This can be useful to be notified when the
lint is no longer triggered.

#### Example

```rust
#[allow(unused_mut)]
fn foo() -> usize {
    let mut a = Vec::new();
    a.len()
}
```

Use instead:

```rust
#[expect(unused_mut)]
fn foo() -> usize {
    let mut a = Vec::new();
    a.len()
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `allow_attributes_without_reason`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.61.0 |

#### What it does

Checks for attributes that allow lints without a reason.

#### Why restrict this?

Justifying each `allow` helps readers understand the reasoning,
and may allow removing `allow` attributes if their purpose is obsolete.

#### Example

```rust
#![allow(clippy::some_lint)]
```

Use instead:

```rust
#![allow(clippy::some_lint, reason = "False positive rust-lang/rust-clippy#1002020")]
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `arbitrary_source_item_ordering`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.84.0 |

#### What it does

Confirms that items are sorted in source files as per configuration.

#### Why restrict this?

Keeping a consistent ordering throughout the codebase helps with working
as a team, and possibly improves maintainability of the codebase. The
idea is that by defining a consistent and enforceable rule for how
source files are structured, less time will be wasted during reviews on
a topic that is (under most circumstances) not relevant to the logic
implemented in the code. Sometimes this will be referred to as
“bikeshedding”.

The content of items with a representation clause attribute, such as
`#[repr(C)]` will not be checked, as the order of their fields or
variants might be dictated by an external API (application binary
interface).

#### Default Ordering and Configuration

As there is no generally applicable rule, and each project may have
different requirements, the lint can be configured with high
granularity. The configuration is split into two stages:

1. Which item kinds that should have an internal order enforced.
2. Individual ordering rules per item kind.

The item kinds that can be linted are:

- Module (with customized groupings, alphabetical within - configurable)
- Trait (with customized order of associated items, alphabetical within)
- Enum, Impl, Struct (purely alphabetical)

#### Module Item Order

Due to the large variation of items within modules, the ordering can be
configured on a very granular level. Item kinds can be grouped together
arbitrarily, items within groups will be ordered alphabetically. The
following table shows the default groupings:

| Group | Item Kinds |
| --- | --- |
| modules | “mod”, “foreign_mod” |
| use | “use” |
| macros | “macro” |
| global_asm | “global_asm” |
| UPPER_SNAKE_CASE | “static”, “const” |
| PascalCase | “ty_alias”, “opaque_ty”, “enum”, “struct”, “union”, “trait”, “trait_alias”, “impl” |
| lower_snake_case | “fn” |

The groups’ names are arbitrary and can be changed to suit the
conventions that should be enforced for a specific project.

All item kinds must be accounted for to create an enforceable linting
rule set. Following are some example configurations that may be useful.

Example: *module inclusions and use statements to be at the top*

```toml
module-item-order-groupings = [
    [ "modules", [ "extern_crate", "mod", "foreign_mod" ], ],
    [ "use", [ "use", ], ],
    [ "everything_else", [ "macro", "global_asm", "static", "const", "ty_alias", "enum", "struct", "union", "trait", "trait_alias", "impl", "fn", ], ],
]
```

Example: *only consts and statics should be alphabetically ordered*

It is also possible to configure a selection of module item groups that
should be ordered alphabetically. This may be useful if for example
statics and consts should be ordered, but the rest should be left open.

```toml
module-items-ordered-within-groupings = ["UPPER_SNAKE_CASE"]
```

#### Known Problems

#### Performance Impact

Keep in mind, that ordering source code alphabetically can lead to
reduced performance in cases where the most commonly used enum variant
isn’t the first entry anymore, and similar optimizations that can reduce
branch misses, cache locality and such. Either don’t use this lint if
that’s relevant, or disable the lint in modules or items specifically
where it matters. Other solutions can be to use profile guided
optimization (PGO), post-link optimization (e.g. using BOLT for LLVM),
or other advanced optimization methods. A good starting point to dig
into optimization is [cargo-pgo](https://github.com/Kobzol/cargo-pgo/blob/main/README.md).

#### Lints on a Contains basis

The lint can be disabled only on a “contains” basis, but not per element
within a “container”, e.g. the lint works per-module, per-struct,
per-enum, etc. but not for “don’t order this particular enum variant”.

#### Module documentation

Module level rustdoc comments are not part of the resulting syntax tree
and as such cannot be linted from within `check_mod`. Instead, the
`rustdoc::missing_documentation` lint may be used.

#### Module Tests

This lint does not implement detection of module tests (or other feature
dependent elements for that matter). To lint the location of mod tests,
the lint `items_after_test_module` can be used instead.

#### Example

```rust
trait TraitUnordered {
    const A: bool;
    const C: bool;
    const B: bool;

    type SomeType;

    fn a();
    fn c();
    fn b();
}
```

Use instead:

```rust
trait TraitOrdered {
    const A: bool;
    const B: bool;
    const C: bool;

    type SomeType;

    fn a();
    fn b();
    fn c();
}
```

#### Configuration

- `module-item-order-groupings`:  The named groupings of different source item kinds within modules.

(default: `[["modules", ["extern_crate", "mod", "foreign_mod"]], ["use", ["use"]], ["macros", ["macro"]], ["global_asm", ["global_asm"]], ["UPPER_SNAKE_CASE", ["static", "const"]], ["PascalCase", ["ty_alias", "enum", "struct", "union", "trait", "trait_alias", "impl"]], ["lower_snake_case", ["fn"]]]`)
- `module-items-ordered-within-groupings`:  Whether the items within module groups should be ordered alphabetically or not.

This option can be configured to “all”, “none”, or a list of specific grouping names that should be checked
(e.g. only “enums”).

(default: `"none"`)

- `source-item-ordering`:  Which kind of elements should be ordered internally, possible values being `enum`, `impl`, `module`, `struct`, `trait`.

(default: `["enum", "impl", "module", "struct", "trait"]`)
- `trait-assoc-item-kinds-order`:  The order of associated items in traits.

(default: `["const", "type", "fn"]`)

---

### `arithmetic_side_effects`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.64.0 |

#### What it does

Checks any kind of arithmetic operation of any type.

Operators like `+`, `-`, `*` or `<<` are usually capable of overflowing according to the [Rust
Reference](https://doc.rust-lang.org/reference/expressions/operator-expr.html#overflow),
or can panic (`/`, `%`).

Known safe built-in types like `Wrapping` or `Saturating`, floats, operations in constant
environments, allowed types and non-constant operations that won’t overflow are ignored.

#### Why restrict this?

For integers, overflow will trigger a panic in debug builds or wrap the result in
release mode; division by zero will cause a panic in either mode. As a result, it is
desirable to explicitly call checked, wrapping or saturating arithmetic methods.

#### Example

```rust
// `n` can be any number, including `i32::MAX`.
fn foo(n: i32) -> i32 {
    n + 1
}
```

Third-party types can also overflow or present unwanted side-effects.

#### Example

```rust
use rust_decimal::Decimal;
let _n = Decimal::MAX + Decimal::MAX;
```

#### Past names

- integer_arithmetic

#### Configuration

- `arithmetic-side-effects-allowed`:  Suppress checking of the passed type names in all types of operations.

If a specific operation is desired, consider using `arithmetic_side_effects_allowed_binary` or `arithmetic_side_effects_allowed_unary` instead.

#### Example

```toml
arithmetic-side-effects-allowed = ["SomeType", "AnotherType"]
```

#### Noteworthy

A type, say `SomeType`, listed in this configuration has the same behavior of
`["SomeType" , "*"], ["*", "SomeType"]` in `arithmetic_side_effects_allowed_binary`.

(default: `[]`)

- `arithmetic-side-effects-allowed-binary`:  Suppress checking of the passed type pair names in binary operations like addition or
multiplication.

Supports the “*” wildcard to indicate that a certain type won’t trigger the lint regardless
of the involved counterpart. For example, `["SomeType", "*"]` or `["*", "AnotherType"]`.

Pairs are asymmetric, which means that `["SomeType", "AnotherType"]` is not the same as
`["AnotherType", "SomeType"]`.

#### Example

```toml
arithmetic-side-effects-allowed-binary = [["SomeType" , "f32"], ["AnotherType", "*"]]
```

(default: `[]`)

- `arithmetic-side-effects-allowed-unary`:  Suppress checking of the passed type names in unary operations like “negation” (`-`).

#### Example

```toml
arithmetic-side-effects-allowed-unary = ["SomeType", "AnotherType"]
```

(default: `[]`)

---

### `as_conversions`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.41.0 |

#### What it does

Checks for usage of `as` conversions.

Note that this lint is specialized in linting *every single* use of `as`
regardless of whether good alternatives exist or not. If you want more
precise lints for `as`, please consider using these separate lints:

- `clippy::cast_lossless`
- `clippy::cast_possible_truncation`
- `clippy::cast_possible_wrap`
- `clippy::cast_precision_loss`
- `clippy::cast_sign_loss`
- `clippy::char_lit_as_u8`
- `clippy::fn_to_numeric_cast`
- `clippy::fn_to_numeric_cast_with_truncation`
- `clippy::ptr_as_ptr`
- `clippy::unnecessary_cast`
- `invalid_reference_casting`

There is a good explanation the reason why this lint should work in this
way and how it is useful [in this
issue](https://github.com/rust-lang/rust-clippy/issues/5122).

#### Why restrict this?

`as` conversions will perform many kinds of
conversions, including silently lossy conversions and dangerous coercions.
There are cases when it makes sense to use `as`, so the lint is
Allow by default.

#### Example

```rust
let a: u32;
...
f(a as u16);
```

Use instead:

```rust
f(a.try_into()?);

// or

f(a.try_into().expect("Unexpected u16 overflow in f"));
```

---

### `as_pointer_underscore`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.85.0 |

#### What it does

Checks for the usage of `as *const _` or `as *mut _` conversion using inferred type.

#### Why restrict this?

The conversion might include a dangerous cast that might go undetected due to the type being inferred.

#### Example

```rust
fn as_usize<T>(t: &T) -> usize {
    // BUG: `t` is already a reference, so we will here
    // return a dangling pointer to a temporary value instead
    &t as *const _ as usize
}
```

Use instead:

```rust
fn as_usize<T>(t: &T) -> usize {
    t as *const T as usize
}
```

---

### `as_underscore`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.63.0 |

#### What it does

Checks for the usage of `as _` conversion using inferred type.

#### Why restrict this?

The conversion might include lossy conversion or a dangerous cast that might go
undetected due to the type being inferred.

The lint is allowed by default as using `_` is less wordy than always specifying the type.

#### Example

```rust
fn foo(n: usize) {}
let n: u16 = 256;
foo(n as _);
```

Use instead:

```rust
fn foo(n: usize) {}
let n: u16 = 256;
foo(n as usize);
```

---

### `assertions_on_result_states`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Checks for `assert!(r.is_ok())` or `assert!(r.is_err())` calls.

#### Why restrict this?

This form of assertion does not show any of the information present in the `Result`
other than which variant it isn’t.

#### Known problems

The suggested replacement decreases the readability of code and log output.

#### Example

```rust
assert!(r.is_ok());
assert!(r.is_err());
```

Use instead:

```rust
r.unwrap();
r.unwrap_err();
```

---

### `big_endian_bytes`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for the usage of the `to_be_bytes` method and/or the function `from_be_bytes`.

#### Why restrict this?

To ensure use of little-endian or the target’s endianness rather than big-endian.

#### Example

```rust
let _x = 2i32.to_be_bytes();
let _y = 2i64.to_be_bytes();
```

---

### `cfg_not_test`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.81.0 |

#### What it does

Checks for usage of `cfg` that excludes code from `test` builds. (i.e., `#[cfg(not(test))]`)

#### Why is this bad?

This may give the false impression that a codebase has 100% coverage, yet actually has untested code.
Enabling this also guards against excessive mockery as well, which is an anti-pattern.

#### Example

```rust
#[cfg(not(test))]
important_check(); // I'm not actually tested, but not including me will falsely increase coverage!
```

Use instead:

```rust
important_check();
```

---

### `clone_on_ref_ptr`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `.clone()` on a ref-counted pointer,
(`Rc`, `Arc`, `rc::Weak`, or `sync::Weak`), and suggests calling Clone via unified
function syntax instead (e.g., `Rc::clone(foo)`).

#### Why restrict this?

Calling `.clone()` on an `Rc`, `Arc`, or `Weak`
can obscure the fact that only the pointer is being cloned, not the underlying
data.

#### Example

```rust
let x = Rc::new(1);

x.clone();
```

Use instead:

```rust
Rc::clone(&x);
```

---

### `cognitive_complexity`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.35.0 |

#### What it does

We used to think it measured how hard a method is to understand.

#### Why is this bad?

Ideally, we would like to be able to measure how hard a function is
to understand given its context (what we call its Cognitive Complexity).
But that’s not what this lint does. See “Known problems”

#### Known problems

The true Cognitive Complexity of a method is not something we can
calculate using modern technology. This lint has been left in
`restriction` so as to not mislead users into using this lint as a
measurement tool.

For more detailed information, see [rust-clippy#3793](https://github.com/rust-lang/rust-clippy/issues/3793)

#### Lints to consider instead of this

- [excessive_nesting](https://rust-lang.github.io/rust-clippy/master/index.html#excessive_nesting)
- [too_many_lines](https://rust-lang.github.io/rust-clippy/master/index.html#too_many_lines)

#### Past names

- cyclomatic_complexity

#### Configuration

- `cognitive-complexity-threshold`:  The maximum cognitive complexity a function can have

(default: `25`)

---

### `create_dir`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.48.0 |

#### What it does

Checks usage of `std::fs::create_dir` and suggest using `std::fs::create_dir_all` instead.

#### Why restrict this?

Sometimes `std::fs::create_dir` is mistakenly chosen over `std::fs::create_dir_all`,
resulting in failure when more than one directory needs to be created or when the directory already exists.
Crates which never need to specifically create a single directory may wish to prevent this mistake.

#### Example

```rust
std::fs::create_dir("foo");
```

Use instead:

```rust
std::fs::create_dir_all("foo");
```

---

### `dbg_macro`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.34.0 |

#### What it does

Checks for usage of the [dbg!](https://doc.rust-lang.org/std/macro.dbg.html) macro.

#### Why restrict this?

The `dbg!` macro is intended as a debugging tool. It should not be present in released
software or committed to a version control system.

#### Example

```rust
dbg!(true)
```

Use instead:

```rust
true
```

#### Configuration

- `allow-dbg-in-tests`:  Whether `dbg!` should be allowed in test functions or `#[cfg(test)]`

(default: `false`)

---

### `decimal_literal_representation`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if there is a better representation for a numeric literal.

#### Why restrict this?

Especially for big powers of 2, a hexadecimal representation is usually more
readable than a decimal representation.

#### Example

```text
`255` => `0xFF`
`65_535` => `0xFFFF`
`4_042_322_160` => `0xF0F0_F0F0`
```

#### Configuration

- `literal-representation-threshold`:  The lower bound for linting decimal literals

(default: `16384`)

---

### `default_numeric_fallback`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.52.0 |

#### What it does

Checks for usage of unconstrained numeric literals which may cause default numeric fallback in type
inference.

Default numeric fallback means that if numeric types have not yet been bound to concrete
types at the end of type inference, then integer type is bound to `i32`, and similarly
floating type is bound to `f64`.

See [RFC0212](https://github.com/rust-lang/rfcs/blob/master/text/0212-restore-int-fallback.md) for more information about the fallback.

#### Why restrict this?

To ensure that every numeric type is chosen explicitly rather than implicitly.

#### Known problems

This lint is implemented using a custom algorithm independent of rustc’s inference,
which results in many false positives and false negatives.

#### Example

```rust
let i = 10;
let f = 1.23;
```

Use instead:

```rust
let i = 10_i32;
let f = 1.23_f64;
```

---

### `default_union_representation`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.60.0 |

#### What it does

Displays a warning when a union is declared with the default representation (without a `#[repr(C)]` attribute).

#### Why restrict this?

Unions in Rust have unspecified layout by default, despite many people thinking that they
lay out each field at the start of the union (like C does). That is, there are no guarantees
about the offset of the fields for unions with multiple non-ZST fields without an explicitly
specified layout. These cases may lead to undefined behavior in unsafe blocks.

#### Example

```rust
union Foo {
    a: i32,
    b: u32,
}

fn main() {
    let _x: u32 = unsafe {
        Foo { a: 0_i32 }.b // Undefined behavior: `b` is allowed to be padding
    };
}
```

Use instead:

```rust
#[repr(C)]
union Foo {
    a: i32,
    b: u32,
}

fn main() {
    let _x: u32 = unsafe {
        Foo { a: 0_i32 }.b // Now defined behavior, this is just an i32 -> u32 transmute
    };
}
```

---

### `deref_by_slicing`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.61.0 |

#### What it does

Checks for slicing expressions which are equivalent to dereferencing the
value.

#### Why restrict this?

Some people may prefer to dereference rather than slice.

#### Example

```rust
let vec = vec![1, 2, 3];
let slice = &vec[..];
```

Use instead:

```rust
let vec = vec![1, 2, 3];
let slice = &*vec;
```

---

### `disallowed_script_idents`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.55.0 |

#### What it does

Checks for usage of unicode scripts other than those explicitly allowed
by the lint config.

This lint doesn’t take into account non-text scripts such as `Unknown` and `Linear_A`.
It also ignores the `Common` script type.
While configuring, be sure to use official script name [aliases](http://www.unicode.org/reports/tr24/tr24-31.html#Script_Value_Aliases) from
[the list of supported scripts](https://www.unicode.org/iso15924/iso15924-codes.html).

See also: [non_ascii_idents](https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#non-ascii-idents).

#### Why restrict this?

It may be not desired to have many different scripts for
identifiers in the codebase.

Note that if you only want to allow typical English, you might want to use
built-in [non_ascii_idents](https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#non-ascii-idents) lint instead.

#### Example

```rust
// Assuming that `clippy.toml` contains the following line:
// allowed-scripts = ["Latin", "Cyrillic"]
let counter = 10; // OK, latin is allowed.
let счётчик = 10; // OK, cyrillic is allowed.
let zähler = 10; // OK, it's still latin.
let カウンタ = 10; // Will spawn the lint.
```

#### Configuration

- `allowed-scripts`:  The list of unicode scripts allowed to be used in the scope.

(default: `["Latin"]`)

---

### `doc_include_without_cfg`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.85.0 |

#### What it does

Checks if included files in doc comments are included only for `cfg(doc)`.

#### Why restrict this?

These files are not useful for compilation but will still be included.
Also, if any of these non-source code file is updated, it will trigger a
recompilation.

#### Known problems

Excluding this will currently result in the file being left out if
the item’s docs are inlined from another crate. This may be fixed in a
future version of rustdoc.

#### Example

```rust
#![doc = include_str!("some_file.md")]
```

Use instead:

```rust
#![cfg_attr(doc, doc = include_str!("some_file.md"))]
```

---

### `doc_paragraphs_missing_punctuation`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.93.0 |

#### What it does

Checks for doc comments whose paragraphs do not end with a period or another punctuation mark.
Various Markdowns constructs are taken into account to avoid false positives.

#### Why is this bad?

A project may wish to enforce consistent doc comments by making sure paragraphs end with a
punctuation mark.

#### Example

```rust
/// Returns a random number
///
/// It was chosen by a fair dice roll
```

Use instead:

```rust
/// Returns a random number.
///
/// It was chosen by a fair dice roll.
```

#### Terminal punctuation marks

This lint treats these characters as end markers: ‘.’, ‘?’, ‘!’, ‘…’ and ‘:’.

The colon is not exactly a terminal punctuation mark, but this is required for paragraphs that
introduce a table or a list for example.

---

### `else_if_without_else`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of if expressions with an `else if` branch,
but without a final `else` branch.

#### Why restrict this?

Some coding guidelines require this (e.g., MISRA-C:2004 Rule 14.10).

#### Example

```rust
if x.is_positive() {
    a();
} else if x.is_negative() {
    b();
}
```

Use instead:

```rust
if x.is_positive() {
    a();
} else if x.is_negative() {
    b();
} else {
    // We don't care about zero.
}
```

---

### `empty_drop`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.62.0 |

#### What it does

Checks for empty `Drop` implementations.

#### Why restrict this?

Empty `Drop` implementations have no effect when dropping an instance of the type. They are
most likely useless. However, an empty `Drop` implementation prevents a type from being
destructured, which might be the intention behind adding the implementation as a marker.

#### Example

```rust
struct S;

impl Drop for S {
    fn drop(&mut self) {}
}
```

Use instead:

```rust
struct S;
```

---

### `empty_enum_variants_with_brackets`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.77.0 |

#### What it does

Finds enum variants without fields that are declared with empty brackets.

#### Why restrict this?

Empty brackets after a enum variant declaration are redundant and can be omitted,
and it may be desirable to do so consistently for style.

However, removing the brackets also introduces a public constant named after the variant,
so this is not just a syntactic simplification but an API change, and adding them back
is a *breaking* API change.

#### Example

```rust
enum MyEnum {
    HasData(u8),
    HasNoData(),       // redundant parentheses
    NoneHereEither {}, // redundant braces
}
```

Use instead:

```rust
enum MyEnum {
    HasData(u8),
    HasNoData,
    NoneHereEither,
}
```

---

### `empty_structs_with_brackets`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.62.0 |

#### What it does

Finds structs without fields (a so-called “empty struct”) that are declared with brackets.

#### Why restrict this?

Empty brackets after a struct declaration can be omitted,
and it may be desirable to do so consistently for style.

However, removing the brackets also introduces a public constant named after the struct,
so this is not just a syntactic simplification but an API change, and adding them back
is a *breaking* API change.

#### Example

```rust
struct Cookie {}
struct Biscuit();
```

Use instead:

```rust
struct Cookie;
struct Biscuit;
```

---

### `error_impl_error`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Checks for types named `Error` that implement `Error`.

#### Why restrict this?

It can become confusing when a codebase has 20 types all named `Error`, requiring either
aliasing them in the `use` statement or qualifying them like `my_module::Error`. This
hinders comprehension, as it requires you to memorize every variation of importing `Error`
used across a codebase.

#### Example

```rust
#[derive(Debug)]
pub enum Error { ... }

impl std::fmt::Display for Error { ... }

impl std::error::Error for Error { ... }
```

---

### `exhaustive_enums`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.51.0 |

#### What it does

Warns on any exported `enum`s that are not tagged `#[non_exhaustive]`

#### Why restrict this?

Making an `enum` exhaustive is a stability commitment: adding a variant is a breaking change.
A project may wish to ensure that there are no exhaustive enums or that every exhaustive
`enum` is explicitly `#[allow]`ed.

#### Example

```rust
enum Foo {
    Bar,
    Baz
}
```

Use instead:

```rust
#[non_exhaustive]
enum Foo {
    Bar,
    Baz
}
```

---

### `exhaustive_structs`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.51.0 |

#### What it does

Warns on any exported `struct`s that are not tagged `#[non_exhaustive]`

#### Why restrict this?

Making a `struct` exhaustive is a stability commitment: adding a field is a breaking change.
A project may wish to ensure that there are no exhaustive structs or that every exhaustive
`struct` is explicitly `#[allow]`ed.

#### Example

```rust
struct Foo {
    bar: u8,
    baz: String,
}
```

Use instead:

```rust
#[non_exhaustive]
struct Foo {
    bar: u8,
    baz: String,
}
```

---

### `exit`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.41.0 |

#### What it does

Detects calls to the `exit()` function that are not in the `main` function. Calls to `exit()`
immediately terminate the program.

#### Why restrict this?

`exit()` immediately terminates the program with no information other than an exit code.
This provides no means to troubleshoot a problem, and may be an unexpected side effect.

Codebases may use this lint to require that all exits are performed either by panicking
(which produces a message, a code location, and optionally a backtrace)
or by calling `exit()` from `main()` (which is a single place to look).

#### Good example

```rust
fn main() {
    std::process::exit(0);
}
```

#### Bad example

```rust
fn main() {
    other_function();
}

fn other_function() {
    std::process::exit(0);
}
```

Use instead:

```rust
// To provide a stacktrace and additional information
panic!("message");

// or a main method with a return
fn main() -> Result<(), i32> {
    Ok(())
}
```

---

### `expect_used`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.45.0 |

#### What it does

Checks for `.expect()` or `.expect_err()` calls on `Result`s and `.expect()` call on `Option`s.

#### Why restrict this?

Usually it is better to handle the `None` or `Err` case.
Still, for a lot of quick-and-dirty code, `expect` is a good choice, which is why
this lint is `Allow` by default.

`result.expect()` will let the thread panic on `Err`
values. Normally, you want to implement more sophisticated error handling,
and propagate errors upwards with `?` operator.

#### Examples

```rust
option.expect("one");
result.expect("one");
```

Use instead:

```rust
option?;

// or

result?;
```

#### Past names

- option_expect_used
- result_expect_used

#### Configuration

- `allow-expect-in-consts`:  Whether `expect` should be allowed in code always evaluated at compile time

(default: `true`)
- `allow-expect-in-tests`:  Whether `expect` should be allowed in test functions or `#[cfg(test)]`

(default: `false`)
- `allow-unwrap-types`:  List of types to allow `unwrap()` and `expect()` on.

#### Example

```toml
allow-unwrap-types = [ "std::sync::LockResult" ]
```

(default: `[]`)

---

### `field_scoped_visibility_modifiers`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.81.0 |

#### What it does

Checks for usage of scoped visibility modifiers, like `pub(crate)`, on fields. These
make a field visible within a scope between public and private.

#### Why restrict this?

Scoped visibility modifiers cause a field to be accessible within some scope between
public and private, potentially within an entire crate. This allows for fields to be
non-private while upholding internal invariants, but can be a code smell. Scoped visibility
requires checking a greater area, potentially an entire crate, to verify that an invariant
is upheld, and global analysis requires a lot of effort.

#### Example

```rust
pub mod public_module {
    struct MyStruct {
        pub(crate) first_field: bool,
        pub(super) second_field: bool
    }
}
```

Use instead:

```rust
pub mod public_module {
    struct MyStruct {
        first_field: bool,
        second_field: bool
    }
    impl MyStruct {
        pub(crate) fn get_first_field(&self) -> bool {
            self.first_field
        }
        pub(super) fn get_second_field(&self) -> bool {
            self.second_field
        }
    }
}
```

---

### `filetype_is_file`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.42.0 |

#### What it does

Checks for `FileType::is_file()`.

#### Why restrict this?

When people testing a file type with `FileType::is_file`
they are testing whether a path is something they can get bytes from. But
`is_file` doesn’t cover special file types in unix-like systems, and doesn’t cover
symlink in windows. Using `!FileType::is_dir()` is a better way to that intention.

#### Example

```rust
let metadata = std::fs::metadata("foo.txt")?;
let filetype = metadata.file_type();

if filetype.is_file() {
    // read file
}
```

should be written as:

```rust
let metadata = std::fs::metadata("foo.txt")?;
let filetype = metadata.file_type();

if !filetype.is_dir() {
    // read file
}
```

---

### `float_arithmetic`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for float arithmetic.

#### Why restrict this?

For some embedded systems or kernel development, it
can be useful to rule out floating-point numbers.

#### Example

```rust
a + 1.0;
```

---

### `float_cmp_const`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for (in-)equality comparisons on constant floating-point
values (apart from zero), except in functions called `*eq*` (which probably
implement equality for a type involving floats).

#### Why restrict this?

Floating point calculations are usually imprecise, so asking if two values are *exactly*
equal is asking for trouble because arriving at the same logical result via different
routes (e.g. calculation versus constant) may yield different values.

#### Example

```rust
let a: f64 = 1000.1;
let b: f64 = 0.2;
let x = a + b;
const Y: f64 = 1000.3; // Expected value.

// Actual value: 1000.3000000000001
println!("{x}");

let are_equal = x == Y;
println!("{are_equal}"); // false
```

The correct way to compare floating point numbers is to define an allowed error margin. This
may be challenging if there is no “natural” error margin to permit. Broadly speaking, there
are two cases:

1. If your values are in a known range and you can define a threshold for “close enough to
be equal”, it may be appropriate to define an absolute error margin. For example, if your
data is “length of vehicle in centimeters”, you may consider 0.1 cm to be “close enough”.
2. If your code is more general and you do not know the range of values, you should use a
relative error margin, accepting e.g. 0.1% of error regardless of specific values.

For the scenario where you can define a meaningful absolute error margin, consider using:

```rust
let a: f64 = 1000.1;
let b: f64 = 0.2;
let x = a + b;
const Y: f64 = 1000.3; // Expected value.

const ALLOWED_ERROR_VEHICLE_LENGTH_CM: f64 = 0.1;
let within_tolerance = (x - Y).abs() < ALLOWED_ERROR_VEHICLE_LENGTH_CM;
println!("{within_tolerance}"); // true
```

NOTE: Do not use `f64::EPSILON` - while the error margin is often called “epsilon”, this is
a different use of the term that is not suitable for floating point equality comparison.
Indeed, for the example above using `f64::EPSILON` as the allowed error would return `false`.

For the scenario where no meaningful absolute error can be defined, refer to
[the floating point guide](https://www.floating-point-gui.de/errors/comparison)
for a reference implementation of relative error based comparison of floating point values.
`MIN_NORMAL` in the reference implementation is equivalent to `MIN_POSITIVE` in Rust.

---

### `fn_to_numeric_cast_any`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.58.0 |

#### What it does

Checks for casts of a function pointer to any integer type.

#### Why restrict this?

Casting a function pointer to an integer can have surprising results and can occur
accidentally if parentheses are omitted from a function call. If you aren’t doing anything
low-level with function pointers then you can opt out of casting functions to integers in
order to avoid mistakes. Alternatively, you can use this lint to audit all uses of function
pointer casts in your code.

#### Example

```rust
// fn1 is cast as `usize`
fn fn1() -> u16 {
    1
};
let _ = fn1 as usize;
```

Use instead:

```rust
// maybe you intended to call the function?
fn fn2() -> u16 {
    1
};
let _ = fn2() as usize;

// or

// maybe you intended to cast it to a function type?
fn fn3() -> u16 {
    1
}
let _ = fn3 as fn() -> u16;
```

---

### `get_unwrap`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `.get().unwrap()` (or
`.get_mut().unwrap`) on a standard library type which implements `Index`

#### Why restrict this?

Using the Index trait (`[]`) is more clear and more
concise.

#### Known problems

Not a replacement for error handling: Using either
`.unwrap()` or the Index trait (`[]`) carries the risk of causing a `panic`
if the value being accessed is `None`. If the use of `.get().unwrap()` is a
temporary placeholder for dealing with the `Option` type, then this does
not mitigate the need for error handling. If there is a chance that `.get()`
will be `None` in your program, then it is advisable that the `None` case
is handled in a future refactor instead of using `.unwrap()` or the Index
trait.

#### Example

```rust
let mut some_vec = vec![0, 1, 2, 3];
let last = some_vec.get(3).unwrap();
*some_vec.get_mut(0).unwrap() = 1;
```

The correct use would be:

```rust
let mut some_vec = vec![0, 1, 2, 3];
let last = some_vec[3];
some_vec[0] = 1;
```

---

### `host_endian_bytes`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for the usage of the `to_ne_bytes` method and/or the function `from_ne_bytes`.

#### Why restrict this?

To ensure use of explicitly chosen endianness rather than the target’s endianness,
such as when implementing network protocols or file formats rather than FFI.

#### Example

```rust
let _x = 2i32.to_ne_bytes();
let _y = 2i64.to_ne_bytes();
```

---

### `if_then_some_else_none`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.53.0 |

#### What it does

Checks for if-else that could be written using either `bool::then` or `bool::then_some`.

#### Why restrict this?

Looks a little redundant. Using `bool::then` is more concise and incurs no loss of clarity.
For simple calculations and known values, use `bool::then_some`, which is eagerly evaluated
in comparison to `bool::then`.

#### Example

```rust
let a = if v.is_empty() {
    println!("true!");
    Some(42)
} else {
    None
};
```

Could be written:

```rust
let a = v.is_empty().then(|| {
    println!("true!");
    42
});
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `impl_trait_in_params`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | 1.69.0 |

#### What it does

Lints when `impl Trait` is being used in a function’s parameters.

#### Why restrict this?

Turbofish syntax (`::<>`) cannot be used to specify the type of an `impl Trait` parameter,
making `impl Trait` less powerful. Readability may also be a factor.

#### Example

```rust
trait MyTrait {}
fn foo(a: impl MyTrait) {
	// [...]
}
```

Use instead:

```rust
trait MyTrait {}
fn foo<T: MyTrait>(a: T) {
	// [...]
}
```

---

### `implicit_return`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.33.0 |

#### What it does

Checks for missing return statements at the end of a block.

#### Why restrict this?

Omitting the return keyword whenever possible is idiomatic Rust code, but:

- Programmers coming from other languages might prefer the expressiveness of `return`.
- It’s possible to miss the last returning statement because the only difference is a missing `;`.
- Especially in bigger code with multiple return paths, having a `return` keyword makes it easier to find the
corresponding statements.

#### Example

```rust
fn foo(x: usize) -> usize {
    x
}
```

add return

```rust
fn foo(x: usize) -> usize {
    return x;
}
```

---

### `indexing_slicing`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of indexing or slicing that may panic at runtime.

This lint does not report on indexing or slicing operations
that always panic, [out_of_bounds_indexing](#out_of_bounds_indexing) already
handles those cases.

#### Why restrict this?

To avoid implicit panics from indexing and slicing.

There are “checked” alternatives which do not panic, and can be used with `unwrap()` to make
an explicit panic when it is desired.

#### Limitations

This lint does not check for the usage of indexing or slicing on strings. These are covered
by the more specific `string_slice` lint.

#### Example

```rust
// Vector
let x = vec![0, 1, 2, 3];

x[2];
x[100];
&x[2..100];

// Array
let y = [0, 1, 2, 3];

let i = 10; // Could be a runtime value
let j = 20;
&y[i..j];
```

Use instead:

```rust
x.get(2);
x.get(100);
x.get(2..100);

let i = 10;
let j = 20;
y.get(i..j);
```

#### Configuration

- `allow-indexing-slicing-in-tests`:  Whether `indexing_slicing` should be allowed in test functions or `#[cfg(test)]`

(default: `false`)
- `suppress-restriction-lint-in-const`:  Whether to suppress a restriction lint in constant code. In same
cases the restructured operation might not be unavoidable, as the
suggested counterparts are unavailable in constant code. This
configuration will cause restriction lints to trigger even
if no suggestion can be made.

(default: `false`)

---

### `infinite_loop`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.76.0 |

#### What it does

Checks for infinite loops in a function where the return type is not `!`
and lint accordingly.

#### Why restrict this?

Making the return type `!` serves as documentation that the function does not return.
If the function is not intended to loop infinitely, then this lint may detect a bug.

#### Example

```rust
fn run_forever() {
    loop {
        // do something
    }
}
```

If infinite loops are as intended:

```rust
fn run_forever() -> ! {
    loop {
        // do something
    }
}
```

Otherwise add a `break` or `return` condition:

```rust
fn run_forever() {
    loop {
        // do something
        if condition {
            break;
        }
    }
}
```

---

### `inline_asm_x86_att_syntax`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.49.0 |

#### What it does

Checks for usage of AT&T x86 assembly syntax.

#### Why restrict this?

To enforce consistent use of Intel x86 assembly syntax.

#### Example

```rust
asm!("lea ({}), {}", in(reg) ptr, lateout(reg) _, options(att_syntax));
```

Use instead:

```rust
asm!("lea {}, [{}]", lateout(reg) _, in(reg) ptr);
```

---

### `inline_asm_x86_intel_syntax`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.49.0 |

#### What it does

Checks for usage of Intel x86 assembly syntax.

#### Why restrict this?

To enforce consistent use of AT&T x86 assembly syntax.

#### Example

```rust
asm!("lea {}, [{}]", lateout(reg) _, in(reg) ptr);
```

Use instead:

```rust
asm!("lea ({}), {}", in(reg) ptr, lateout(reg) _, options(att_syntax));
```

---

### `integer_division`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.37.0 |

#### What it does

Checks for division of integers

#### Why restrict this?

When outside of some very specific algorithms,
integer division is very often a mistake because it discards the
remainder.

#### Example

```rust
let x = 3 / 2;
println!("{}", x);
```

Use instead:

```rust
let x = 3f32 / 2f32;
println!("{}", x);
```

---

### `integer_division_remainder_used`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.79.0 |

#### What it does

Checks for the usage of division (`/`) and remainder (`%`) operations
when performed on any integer types using the default `Div` and `Rem` trait implementations.

#### Why restrict this?

In cryptographic contexts, division can result in timing sidechannel vulnerabilities,
and needs to be replaced with constant-time code instead (e.g. Barrett reduction).

#### Example

```rust
let my_div = 10 / 2;
```

Use instead:

```rust
let my_div = 10 >> 1;
```

---

### `iter_over_hash_type`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.76.0 |

#### What it does

This is a restriction lint which prevents the use of hash types (i.e., `HashSet` and `HashMap`) in for loops.

#### Why restrict this?

Because hash types are unordered, when iterated through such as in a `for` loop, the values are returned in
an undefined order. As a result, on redundant systems this may cause inconsistencies and anomalies.
In addition, the unknown order of the elements may reduce readability or introduce other undesired
side effects.

#### Example

```rust
let my_map = std::collections::HashMap::<i32, String>::new();
    for (key, value) in my_map { /* ... */ }
```

Use instead:

```rust
let my_map = std::collections::HashMap::<i32, String>::new();
    let mut keys = my_map.keys().clone().collect::<Vec<_>>();
    keys.sort();
    for key in keys {
        let value = &my_map[key];
    }
```

---

### `large_include_file`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.62.0 |

#### What it does

Checks for the inclusion of large files via `include_bytes!()`
or `include_str!()`.

#### Why restrict this?

Including large files can undesirably increase the size of the binary produced by the compiler.
This lint may be used to catch mistakes where an unexpectedly large file is included, or
temporarily to obtain a list of all large files.

#### Example

```rust
let included_str = include_str!("very_large_file.txt");
let included_bytes = include_bytes!("very_large_file.txt");
```

Use instead:

```rust
use std::fs;

// You can load the file at runtime
let string = fs::read_to_string("very_large_file.txt")?;
let bytes = fs::read("very_large_file.txt")?;
```

#### Configuration

- `max-include-file-size`:  The maximum size of a file included via `include_bytes!()` or `include_str!()`, in bytes

(default: `1000000`)

---

### `let_underscore_must_use`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.42.0 |

#### What it does

Checks for `let _ = <expr>` where expr is `#[must_use]`

#### Why restrict this?

To ensure that all `#[must_use]` types are used rather than ignored.

#### Example

```rust
fn f() -> Result<u32, u32> {
    Ok(0)
}

let _ = f();
// is_ok() is marked #[must_use]
let _ = f().is_ok();
```

---

### `let_underscore_untyped`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.69.0 |

#### What it does

Checks for `let _ = <expr>` without a type annotation, and suggests to either provide one,
or remove the `let` keyword altogether.

#### Why restrict this?

The `let _ = <expr>` expression ignores the value of `<expr>`, but will continue to do so even
if the type were to change, thus potentially introducing subtle bugs. By supplying a type
annotation, one will be forced to re-visit the decision to ignore the value in such cases.

#### Known problems

The `_ = <expr>` is not properly supported by some tools (e.g. IntelliJ) and may seem odd
to many developers. This lint also partially overlaps with the other `let_underscore_*`
lints.

#### Example

```rust
fn foo() -> Result<u32, ()> {
    Ok(123)
}
let _ = foo();
```

Use instead:

```rust
fn foo() -> Result<u32, ()> {
    Ok(123)
}
// Either provide a type annotation:
let _: Result<u32, ()> = foo();
// …or drop the let keyword:
_ = foo();
```

---

### `little_endian_bytes`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for the usage of the `to_le_bytes` method and/or the function `from_le_bytes`.

#### Why restrict this?

To ensure use of big-endian or the target’s endianness rather than little-endian.

#### Example

```rust
let _x = 2i32.to_le_bytes();
let _y = 2i64.to_le_bytes();
```

---

### `lossy_float_literal`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.43.0 |

#### What it does

Checks for whole number float literals that
cannot be represented as the underlying type without loss.

#### Why restrict this?

If the value was intended to be exact, it will not be.
This may be especially surprising when the lost precision is to the left of the decimal point.

#### Example

```rust
let _: f32 = 16_777_217.0; // 16_777_216.0
```

Use instead:

```rust
let _: f32 = 16_777_216.0;
let _: f64 = 16_777_217.0;
```

---

### `map_err_ignore`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.48.0 |

#### What it does

Checks for instances of `map_err(|_| Some::Enum)`

#### Why restrict this?

This `map_err` throws away the original error rather than allowing the enum to
contain and report the cause of the error.

#### Example

Before:

```rust
use std::fmt;

#[derive(Debug)]
enum Error {
    Indivisible,
    Remainder(u8),
}

impl fmt::Display for Error {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            Error::Indivisible => write!(f, "could not divide input by three"),
            Error::Remainder(remainder) => write!(
                f,
                "input is not divisible by three, remainder = {}",
                remainder
            ),
        }
    }
}

impl std::error::Error for Error {}

fn divisible_by_3(input: &str) -> Result<(), Error> {
    input
        .parse::<i32>()
        .map_err(|_| Error::Indivisible)
        .map(|v| v % 3)
        .and_then(|remainder| {
            if remainder == 0 {
                Ok(())
            } else {
                Err(Error::Remainder(remainder as u8))
            }
        })
}
```

After:

```rust
use std::{fmt, num::ParseIntError};

#[derive(Debug)]
enum Error {
    Indivisible(ParseIntError),
    Remainder(u8),
}

impl fmt::Display for Error {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            Error::Indivisible(_) => write!(f, "could not divide input by three"),
            Error::Remainder(remainder) => write!(
                f,
                "input is not divisible by three, remainder = {}",
                remainder
            ),
        }
    }
}

impl std::error::Error for Error {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            Error::Indivisible(source) => Some(source),
            _ => None,
        }
    }
}

fn divisible_by_3(input: &str) -> Result<(), Error> {
    input
        .parse::<i32>()
        .map_err(Error::Indivisible)
        .map(|v| v % 3)
        .and_then(|remainder| {
            if remainder == 0 {
                Ok(())
            } else {
                Err(Error::Remainder(remainder as u8))
            }
        })
}
```

---

### `map_with_unused_argument_over_ranges`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.84.0 |

#### What it does

Checks for `Iterator::map` over ranges without using the parameter which
could be more clearly expressed using `std::iter::repeat(...).take(...)`
or `std::iter::repeat_n`.

#### Why is this bad?

It expresses the intent more clearly to `take` the correct number of times
from a generating function than to apply a closure to each number in a
range only to discard them.

#### Example

```rust
let random_numbers : Vec<_> = (0..10).map(|_| { 3 + 1 }).collect();
```

Use instead:

```rust
let f : Vec<_> = std::iter::repeat( 3 + 1 ).take(10).collect();
```

#### Known Issues

This lint may suggest replacing a `Map<Range>` with a `Take<RepeatWith>`.
The former implements some traits that the latter does not, such as
`DoubleEndedIterator`.

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `mem_forget`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `std::mem::forget(t)` where `t` is
`Drop` or has a field that implements `Drop`.

#### Why restrict this?

`std::mem::forget(t)` prevents `t` from running its destructor, possibly causing leaks.
It is not possible to detect all means of creating leaks, but it may be desirable to
prohibit the simple ones.

#### Example

```rust
mem::forget(Rc::new(55))
```

---

### `min_ident_chars`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for identifiers which consist of a single character (or fewer than the configured threshold).

Note: This lint can be very noisy when enabled; it may be desirable to only enable it
temporarily.

#### Why restrict this?

To improve readability by requiring that every variable has a name more specific than a single letter can be.

#### Example

```rust
for m in movies {
    let title = m.t;
}
```

Use instead:

```rust
for movie in movies {
    let title = movie.title;
}
```

#### Limitations

Trait implementations which use the same function or parameter name as the trait declaration will
not be warned about, even if the name is below the configured limit.

#### Configuration

- `allowed-idents-below-min-chars`:  Allowed names below the minimum allowed characters. The value `".."` can be used as part of
the list to indicate that the configured values should be appended to the default
configuration of Clippy. By default, any configuration will replace the default value.

(default: `["i", "j", "x", "y", "z", "w", "n"]`)
- `min-ident-chars-threshold`:  Minimum chars an ident can have, anything below or equal to this will be linted.

(default: `1`)

---

### `missing_assert_message`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.70.0 |

#### What it does

Checks assertions without a custom panic message.

#### Why restrict this?

Without a good custom message, it’d be hard to understand what went wrong when the assertion fails.
A good custom message should be more about why the failure of the assertion is problematic
and not what is failed because the assertion already conveys that.

Although the same reasoning applies to testing functions, this lint ignores them as they would be too noisy.
Also, in most cases understanding the test failure would be easier
compared to understanding a complex invariant distributed around the codebase.

#### Known problems

This lint cannot check the quality of the custom panic messages.
Hence, you can suppress this lint simply by adding placeholder messages
like “assertion failed”. However, we recommend coming up with good messages
that provide useful information instead of placeholder messages that
don’t provide any extra information.

#### Example

```rust
fn call(service: Service) {
    assert!(service.ready);
}
```

Use instead:

```rust
fn call(service: Service) {
    assert!(service.ready, "`service.poll_ready()` must be called first to ensure that service is ready to receive requests");
}
```

---

### `missing_asserts_for_indexing`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.74.0 |

#### What it does

Checks for repeated slice indexing without asserting beforehand that the length
is greater than the largest index used to index into the slice.

#### Why restrict this?

In the general case where the compiler does not have a lot of information
about the length of a slice, indexing it repeatedly will generate a bounds check
for every single index.

Asserting that the length of the slice is at least as large as the largest value
to index beforehand gives the compiler enough information to elide the bounds checks,
effectively reducing the number of bounds checks from however many times
the slice was indexed to just one (the assert).

#### Drawbacks

False positives. It is, in general, very difficult to predict how well
the optimizer will be able to elide bounds checks and it very much depends on
the surrounding code. For example, indexing into the slice yielded by the
[slice::chunks_exact](https://doc.rust-lang.org/stable/std/primitive.slice.html#method.chunks_exact)
iterator will likely have all of the bounds checks elided even without an assert
if the `chunk_size` is a constant.

Asserts are not tracked across function calls. Asserting the length of a slice
in a different function likely gives the optimizer enough information
about the length of a slice, but this lint will not detect that.

#### Example

```rust
fn sum(v: &[u8]) -> u8 {
    // 4 bounds checks
    v[0] + v[1] + v[2] + v[3]
}
```

Use instead:

```rust
fn sum(v: &[u8]) -> u8 {
    assert!(v.len() > 3);
    // no bounds checks
    v[0] + v[1] + v[2] + v[3]
}
```

---

### `missing_docs_in_private_items`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if there is missing documentation for any private documentable item.

#### Why restrict this?

Doc is good. *rustc* has a `MISSING_DOCS`
allowed-by-default lint for
public members, but has no way to enforce documentation of private items.
This lint fixes that.

#### Configuration

- `missing-docs-allow-unused`:  Whether to allow fields starting with an underscore to skip documentation requirements

(default: `false`)
- `missing-docs-in-crate-items`:  Whether to **only** check for missing documentation in items visible within the current
crate. For example, `pub(crate)` items.

(default: `false`)

---

### `missing_inline_in_public_items`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

It lints if an exported function, method, trait method with default impl,
or trait method impl is not `#[inline]`.

#### Why restrict this?

When a function is not marked `#[inline]`, it is not
[a “small” candidate for automatic inlining](https://github.com/rust-lang/rust/pull/116505), and LTO is not in use, then it is not
possible for the function to be inlined into the code of any crate other than the one in
which it is defined.  Depending on the role of the function and the relationship of the crates,
this could significantly reduce performance.

Certain types of crates might intend for most of the methods in their public API to be able
to be inlined across crates even when LTO is disabled.
This lint allows those crates to require all exported methods to be `#[inline]` by default, and
then opt out for specific methods where this might not make sense.

#### Example

```rust
pub fn foo() {} // missing #[inline]
fn ok() {} // ok
#[inline] pub fn bar() {} // ok
#[inline(always)] pub fn baz() {} // ok

pub trait Bar {
  fn bar(); // ok
  fn def_bar() {} // missing #[inline]
}

struct Baz;
impl Baz {
    fn private() {} // ok
}

impl Bar for Baz {
  fn bar() {} // ok - Baz is not exported
}

pub struct PubBaz;
impl PubBaz {
    fn private() {} // ok
    pub fn not_private() {} // missing #[inline]
}

impl Bar for PubBaz {
    fn bar() {} // missing #[inline]
    fn def_bar() {} // missing #[inline]
}
```

---

### `missing_trait_methods`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.66.0 |

#### What it does

Checks if a provided method is used implicitly by a trait
implementation.

#### Why restrict this?

To ensure that a certain implementation implements every method; for example,
a wrapper type where every method should delegate to the corresponding method of
the inner type’s implementation.

This lint should typically be enabled on a specific trait `impl` item
rather than globally.

#### Example

```rust
trait Trait {
    fn required();

    fn provided() {}
}

#[warn(clippy::missing_trait_methods)]
impl Trait for Type {
    fn required() { /* ... */ }
}
```

Use instead:

```rust
trait Trait {
    fn required();

    fn provided() {}
}

#[warn(clippy::missing_trait_methods)]
impl Trait for Type {
    fn required() { /* ... */ }

    fn provided() { /* ... */ }
}
```

---

### `mixed_read_write_in_expression`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for a read and a write to the same variable where
whether the read occurs before or after the write depends on the evaluation
order of sub-expressions.

#### Why restrict this?

While [the evaluation order of sub-expressions] is fully specified in Rust,
it still may be confusing to read an expression where the evaluation order
affects its behavior.

#### Known problems

Code which intentionally depends on the evaluation
order, or which is correct for any evaluation order.

#### Example

```rust
let mut x = 0;

let a = {
    x = 1;
    1
} + x;
// Unclear whether a is 1 or 2.
```

Use instead:

```rust
let tmp = {
    x = 1;
    1
};
let a = tmp + x;
```

#### Past names

- eval_order_dependence

---

### `mod_module_files`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

Checks that module layout uses only self named module files; bans `mod.rs` files.

#### Why restrict this?

Having multiple module layout styles in a project can be confusing.

#### Example

```text
src/
  stuff/
    stuff_files.rs
    mod.rs
  lib.rs
```

Use instead:

```text
src/
  stuff/
    stuff_files.rs
  stuff.rs
  lib.rs
```

---

### `module_name_repetitions`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.33.0 |

#### What it does

Detects public item names that are prefixed or suffixed by the
containing public module’s name.

#### Why is this bad?

It requires the user to type the module name twice in each usage,
especially if they choose to import the module rather than its contents.

Lack of such repetition is also the style used in the Rust standard library;
e.g. `io::Error` and `fmt::Error` rather than `io::IoError` and `fmt::FmtError`;
and `array::from_ref` rather than `array::array_from_ref`.

#### Known issues

Glob re-exports are ignored; e.g. this will not warn even though it should:

```rust
pub mod foo {
    mod iteration {
        pub struct FooIter {}
    }
    pub use iteration::*; // creates the path `foo::FooIter`
}
```

#### Example

```rust
mod cake {
    struct BlackForestCake;
}
```

Use instead:

```rust
mod cake {
    struct BlackForest;
}
```

#### Past names

- stutter

#### Configuration

- `allow-exact-repetitions`:  Whether an item should be allowed to have the same name as its containing module

(default: `true`)
- `allowed-prefixes`:  List of prefixes to allow when determining whether an item’s name ends with the module’s name.
If the rest of an item’s name is an allowed prefix (e.g. item `ToFoo` or `to_foo` in module `foo`),
then don’t emit a warning.

#### Example

```toml
allowed-prefixes = [ "to", "from" ]
```

#### Noteworthy

- By default, the following prefixes are allowed: `to`, `as`, `into`, `from`, `try_into` and `try_from`
- PascalCase variant is included automatically for each snake_case variant (e.g. if `try_into` is included,
`TryInto` will also be included)
- Use `".."` as part of the list to indicate that the configured values should be appended to the
default configuration of Clippy. By default, any configuration will replace the default value

(default: `["to", "as", "into", "from", "try_into", "try_from"]`)

---

### `modulo_arithmetic`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.42.0 |

#### What it does

Checks for modulo arithmetic.

#### Why restrict this?

The results of modulo (`%`) operation might differ
depending on the language, when negative numbers are involved.
If you interop with different languages it might be beneficial
to double check all places that use modulo arithmetic.

For example, in Rust `17 % -3 = 2`, but in Python `17 % -3 = -1`.

#### Example

```rust
let x = -17 % 3;
```

#### Configuration

- `allow-comparison-to-zero`:  Don’t lint when comparing the result of a modulo operation to zero.

(default: `true`)

---

### `multiple_inherent_impl`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for multiple inherent implementations of a struct

The config option controls the scope in which multiple inherent `impl` blocks for the same
struct are linted, allowing values of `module` (only within the same module), `file`
(within the same file), or `crate` (anywhere in the crate, default).

#### Why restrict this?

Splitting the implementation of a type makes the code harder to navigate.

#### Example

```rust
struct X;
impl X {
    fn one() {}
}
impl X {
    fn other() {}
}
```

Could be written:

```rust
struct X;
impl X {
    fn one() {}
    fn other() {}
}
```

#### Configuration

- `inherent-impl-lint-scope`:  Sets the scope (“crate”, “file”, or “module”) in which duplicate inherent `impl` blocks for the same type are linted.

(default: `"crate"`)

---

### `multiple_unsafe_ops_per_block`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.69.0 |

#### What it does

Checks for `unsafe` blocks that contain more than one unsafe operation.

#### Why restrict this?

Combined with `undocumented_unsafe_blocks`,
this lint ensures that each unsafe operation must be independently justified.
Combined with `unused_unsafe`, this lint also ensures
elimination of unnecessary unsafe blocks through refactoring.

#### Example

```rust
/// Reads a `char` from the given pointer.
///
/// # Safety
///
/// `ptr` must point to four consecutive, initialized bytes which
/// form a valid `char` when interpreted in the native byte order.
fn read_char(ptr: *const u8) -> char {
    // SAFETY: The caller has guaranteed that the value pointed
    // to by `bytes` is a valid `char`.
    unsafe { char::from_u32_unchecked(*ptr.cast::<u32>()) }
}
```

Use instead:

```rust
/// Reads a `char` from the given pointer.
///
/// # Safety
///
/// - `ptr` must be 4-byte aligned, point to four consecutive
///   initialized bytes, and be valid for reads of 4 bytes.
/// - The bytes pointed to by `ptr` must represent a valid
///   `char` when interpreted in the native byte order.
fn read_char(ptr: *const u8) -> char {
    // SAFETY: `ptr` is 4-byte aligned, points to four consecutive
    // initialized bytes, and is valid for reads of 4 bytes.
    let int_value = unsafe { *ptr.cast::<u32>() };

    // SAFETY: The caller has guaranteed that the four bytes
    // pointed to by `bytes` represent a valid `char`.
    unsafe { char::from_u32_unchecked(int_value) }
}
```

#### Notes

- Unsafe operations only count towards the total for the innermost
enclosing `unsafe` block.
- Each call to a macro expanding to unsafe operations count for one
unsafe operation.
- Taking a raw pointer to a union field is always safe and will
not be considered unsafe by this lint, even when linting code written
with a specified Rust version of 1.91 or earlier (which required
using an `unsafe` block).

---

### `mutex_atomic`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `Mutex<X>` where an atomic will do.

#### Why restrict this?

Using a mutex just to make access to a plain bool or
reference sequential is shooting flies with cannons.
`std::sync::atomic::AtomicBool` and `std::sync::atomic::AtomicPtr` are leaner and
faster.

On the other hand, `Mutex`es are, in general, easier to
verify correctness. An atomic does not behave the same as
an equivalent mutex. See [this issue](https://github.com/rust-lang/rust-clippy/issues/4295)’s
commentary for more details.

#### Known problems

- This lint cannot detect if the mutex is actually used
for waiting before a critical section.
- This lint has a false positive that warns without considering the case
where `Mutex` is used together with `Condvar`.

#### Example

```rust
let x = Mutex::new(&y);
```

Use instead:

```rust
let x = AtomicBool::new(y);
```

---

### `mutex_integer`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `Mutex<X>` where `X` is an integral
type.

#### Why restrict this?

Using a mutex just to make access to a plain integer
sequential is
shooting flies with cannons. `std::sync::atomic::AtomicUsize` is leaner and faster.

On the other hand, `Mutex`es are, in general, easier to
verify correctness. An atomic does not behave the same as
an equivalent mutex. See [this issue](https://github.com/rust-lang/rust-clippy/issues/4295)’s
commentary for more details.

#### Known problems

- This lint cannot detect if the mutex is actually used
for waiting before a critical section.
- This lint has a false positive that warns without considering the case
where `Mutex` is used together with `Condvar`.
- This lint suggest using `AtomicU64` instead of `Mutex<u64>`, but
`AtomicU64` is not available on some 32-bit platforms.

#### Example

```rust
let x = Mutex::new(0usize);
```

Use instead:

```rust
let x = AtomicUsize::new(0usize);
```

---

### `needless_raw_strings`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for raw string literals where a string literal can be used instead.

#### Why restrict this?

For consistent style by using simpler string literals whenever possible.

However, there are many cases where using a raw string literal is more
idiomatic than a string literal, so it’s opt-in.

#### Example

```rust
let r = r"Hello, world!";
```

Use instead:

```rust
let r = "Hello, world!";
```

---

### `non_ascii_literal`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for non-ASCII characters in string and char literals.

#### Why restrict this?

Yeah, we know, the 90’s called and wanted their charset
back. Even so, there still are editors and other programs out there that
don’t work well with Unicode. So if the code is meant to be used
internationally, on multiple operating systems, or has other portability
requirements, activating this lint could be useful.

#### Example

```rust
let x = String::from("€");
```

Use instead:

```rust
let x = String::from("\u{20ac}");
```

---

### `non_zero_suggestions`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.83.0 |

#### What it does

Checks for conversions from `NonZero` types to regular integer types,
and suggests using `NonZero` types for the target as well.

#### Why is this bad?

Converting from `NonZero` types to regular integer types and then back to `NonZero`
types is less efficient and loses the type-safety guarantees provided by `NonZero` types.
Using `NonZero` types consistently can lead to more optimized code and prevent
certain classes of errors related to zero values.

#### Example

```rust
use std::num::{NonZeroU32, NonZeroU64};

fn example(x: u64, y: NonZeroU32) {
    // Bad: Converting NonZeroU32 to u64 unnecessarily
    let r1 = x / u64::from(y.get());
    let r2 = x % u64::from(y.get());
}
```

Use instead:

```rust
use std::num::{NonZeroU32, NonZeroU64};

fn example(x: u64, y: NonZeroU32) {
    // Good: Preserving the NonZero property
    let r1 = x / NonZeroU64::from(y);
    let r2 = x % NonZeroU64::from(y);
}
```

---

### `panic`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks for usage of `panic!`.

#### Why restrict this?

This macro, or panics in general, may be unwanted in production code.

#### Example

```rust
panic!("even with a good reason");
```

#### Configuration

- `allow-panic-in-tests`:  Whether `panic` should be allowed in test functions or `#[cfg(test)]`

(default: `false`)

---

### `panic_in_result_fn`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.48.0 |

#### What it does

Checks for usage of `panic!` or assertions in a function whose return type is `Result`.

#### Why restrict this?

For some codebases, it is desirable for functions of type result to return an error instead of crashing. Hence panicking macros should be avoided.

#### Known problems

Functions called from a function returning a `Result` may invoke a panicking macro. This is not checked.

#### Example

```rust
fn result_with_panic() -> Result<bool, String>
{
    panic!("error");
}
```

Use instead:

```rust
fn result_without_panic() -> Result<bool, String> {
    Err(String::from("error"))
}
```

---

### `partial_pub_fields`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.66.0 |

#### What it does

Checks whether some but not all fields of a `struct` are public.

Either make all fields of a type public, or make none of them public

#### Why restrict this?

Most types should either be:

- Abstract data types: complex objects with opaque implementation which guard
interior invariants and expose intentionally limited API to the outside world.
- Data: relatively simple objects which group a bunch of related attributes together,
but have no invariants.

#### Example

```rust
pub struct Color {
    pub r: u8,
    pub g: u8,
    b: u8,
}
```

Use instead:

```rust
pub struct Color {
    pub r: u8,
    pub g: u8,
    pub b: u8,
}
```

---

### `pathbuf_init_then_push`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | 1.82.0 |

#### What it does

Checks for calls to `push` immediately after creating a new `PathBuf`.

#### Why is this bad?

Multiple `.join()` calls are usually easier to read than multiple `.push`
calls across multiple statements. It might also be possible to use
`PathBuf::from` instead.

#### Known problems

`.join()` introduces an implicit `clone()`. `PathBuf::from` can alternatively be
used when the `PathBuf` is newly constructed. This will avoid the implicit clone.

#### Example

```rust
let mut path_buf = PathBuf::new();
path_buf.push("foo");
```

Use instead:

```rust
let path_buf = PathBuf::from("foo");
// or
let path_buf = PathBuf::new().join("foo");
```

---

### `pattern_type_mismatch`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.47.0 |

#### What it does

Checks for patterns that aren’t exact representations of the types
they are applied to.

To satisfy this lint, you will have to adjust either the expression that is matched
against or the pattern itself, as well as the bindings that are introduced by the
adjusted patterns. For matching you will have to either dereference the expression
with the `*` operator, or amend the patterns to explicitly match against `&<pattern>`
or `&mut <pattern>` depending on the reference mutability. For the bindings you need
to use the inverse. You can leave them as plain bindings if you wish for the value
to be copied, but you must use `ref mut <variable>` or `ref <variable>` to construct
a reference into the matched structure.

If you are looking for a way to learn about ownership semantics in more detail, it
is recommended to look at IDE options available to you to highlight types, lifetimes
and reference semantics in your code. The available tooling would expose these things
in a general way even outside of the various pattern matching mechanics. Of course
this lint can still be used to highlight areas of interest and ensure a good understanding
of ownership semantics.

#### Why restrict this?

It increases ownership hints in the code, and will guard against some changes
in ownership.

#### Example

This example shows the basic adjustments necessary to satisfy the lint. Note how
the matched expression is explicitly dereferenced with `*` and the `inner` variable
is bound to a shared borrow via `ref inner`.

```rust
// Bad
let value = &Some(Box::new(23));
match value {
    Some(inner) => println!("{}", inner),
    None => println!("none"),
}

// Good
let value = &Some(Box::new(23));
match *value {
    Some(ref inner) => println!("{}", inner),
    None => println!("none"),
}
```

The following example demonstrates one of the advantages of the more verbose style.
Note how the second version uses `ref mut a` to explicitly declare `a` a shared mutable
borrow, while `b` is simply taken by value. This ensures that the loop body cannot
accidentally modify the wrong part of the structure.

```rust
// Bad
let mut values = vec![(2, 3), (3, 4)];
for (a, b) in &mut values {
    *a += *b;
}

// Good
let mut values = vec![(2, 3), (3, 4)];
for &mut (ref mut a, b) in &mut values {
    *a += b;
}
```

---

### `pointer_format`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.89.0 |

#### What it does

Detects [pointer format](https://doc.rust-lang.org/std/fmt/index.html#formatting-traits) as well as `Debug` formatting of raw pointers or function pointers
or any types that have a derived `Debug` impl that recursively contains them.

#### Why restrict this?

The addresses are only useful in very specific contexts, and certain projects may want to keep addresses of
certain data structures or functions from prying hacker eyes as an additional line of security.

#### Known problems

The lint currently only looks through derived `Debug` implementations. Checking whether a manual
implementation prints an address is left as an exercise to the next lint implementer.

#### Example

```rust
let foo = &0_u32;
fn bar() {}
println!("{:p}", foo);
let _ = format!("{:?}", &(bar as fn()));
```

---

### `precedence_bits`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Checks for bit shifting operations combined with bit masking/combining operators
and suggest using parentheses.

#### Why restrict this?

Not everyone knows the precedence of those operators by
heart, so expressions like these may trip others trying to reason about the
code.

#### Example

`0x2345 & 0xF000 >> 12` equals 5, while `(0x2345 & 0xF000) >> 12` equals 2

---

### `print_stderr`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.50.0 |

#### What it does

Checks for printing on *stderr*. The purpose of this lint
is to catch debugging remnants.

#### Why restrict this?

People often print on *stderr* while debugging an
application and might forget to remove those prints afterward.

#### Known problems

Only catches `eprint!` and `eprintln!` calls.

#### Example

```rust
eprintln!("Hello world!");
```

#### Configuration

- `allow-print-in-tests`:  Whether print macros (ex. `println!`) should be allowed in test functions or `#[cfg(test)]`

(default: `false`)

---

### `print_stdout`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for printing on *stdout*. The purpose of this lint
is to catch debugging remnants.

#### Why restrict this?

People often print on *stdout* while debugging an
application and might forget to remove those prints afterward.

#### Known problems

Only catches `print!` and `println!` calls.

#### Example

```rust
println!("Hello world!");
```

#### Configuration

- `allow-print-in-tests`:  Whether print macros (ex. `println!`) should be allowed in test functions or `#[cfg(test)]`

(default: `false`)

---

### `pub_use`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.62.0 |

#### What it does

Restricts the usage of `pub use ...`

#### Why restrict this?

A project may wish to limit `pub use` instances to prevent
unintentional exports, or to encourage placing exported items directly in public modules.

#### Example

```rust
pub mod outer {
    mod inner {
        pub struct Test {}
    }
    pub use inner::Test;
}

use outer::Test;
```

Use instead:

```rust
pub mod outer {
    pub struct Test {}
}

use outer::Test;
```

---

### `pub_with_shorthand`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for usage of `pub(<loc>)` with `in`.

#### Why restrict this?

Consistency. Use it or don’t, just be consistent about it.

Also see the `pub_without_shorthand` lint for an alternative.

#### Example

```rust
pub(super) type OptBox<T> = Option<Box<T>>;
```

Use instead:

```rust
pub(in super) type OptBox<T> = Option<Box<T>>;
```

---

### `pub_without_shorthand`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.72.0 |

#### What it does

Checks for usage of `pub(<loc>)` without `in`.

Note: As you cannot write a module’s path in `pub(<loc>)`, this will only trigger on
`pub(super)` and the like.

#### Why restrict this?

Consistency. Use it or don’t, just be consistent about it.

Also see the `pub_with_shorthand` lint for an alternative.

#### Example

```rust
pub(in super) type OptBox<T> = Option<Box<T>>;
```

Use instead:

```rust
pub(super) type OptBox<T> = Option<Box<T>>;
```

---

### `question_mark_used`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.69.0 |

#### What it does

Checks for expressions that use the `?` operator and rejects them.

#### Why restrict this?

Sometimes code wants to avoid the `?` operator because for instance a local
block requires a macro to re-throw errors to attach additional information to the
error.

#### Example

```rust
let result = expr?;
```

Could be written:

```rust
utility_macro!(expr);
```

---

### `rc_buffer`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.48.0 |

#### What it does

Checks for `Rc<T>` and `Arc<T>` when `T` is a mutable buffer type such as `String` or `Vec`.

#### Why restrict this?

Expressions such as `Rc<String>` usually have no advantage over `Rc<str>`, since
it is larger and involves an extra level of indirection, and doesn’t implement `Borrow<str>`.

While mutating a buffer type would still be possible with `Rc::get_mut()`, it only
works if there are no additional references yet, which usually defeats the purpose of
enclosing it in a shared ownership type. Instead, additionally wrapping the inner
type with an interior mutable container (such as `RefCell` or `Mutex`) would normally
be used.

#### Known problems

This pattern can be desirable to avoid the overhead of a `RefCell` or `Mutex` for
cases where mutation only happens before there are any additional references.

#### Example

```rust
fn foo(interned: Rc<String>) { ... }
```

Better:

```rust
fn foo(interned: Rc<str>) { ... }
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `rc_mutex`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.55.0 |

#### What it does

Checks for `Rc<Mutex<T>>`.

#### Why restrict this?

`Rc` is used in single thread and `Mutex` is used in multi thread.
Consider using `Rc<RefCell<T>>` in single thread or `Arc<Mutex<T>>` in multi thread.

#### Known problems

Sometimes combining generic types can lead to the requirement that a
type use Rc in conjunction with Mutex. We must consider those cases false positives, but
alas they are quite hard to rule out. Luckily they are also rare.

#### Example

```rust
use std::rc::Rc;
use std::sync::Mutex;
fn foo(interned: Rc<Mutex<i32>>) { ... }
```

Better:

```rust
use std::rc::Rc;
use std::cell::RefCell
fn foo(interned: Rc<RefCell<i32>>) { ... }
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `redundant_test_prefix`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.88.0 |

#### What it does

Checks for test functions (functions annotated with `#[test]`) that are prefixed
with `test_` which is redundant.

#### Why is this bad?

This is redundant because test functions are already annotated with `#[test]`.
Moreover, it clutters the output of `cargo test` since test functions are expanded as
`module::tests::test_use_case` in the output. Without the redundant prefix, the output
becomes `module::tests::use_case`, which is more readable.

#### Example

```rust
#[cfg(test)]
mod tests {
  use super::*;

  #[test]
  fn test_use_case() {
      // test code
  }
}
```

Use instead:

```rust
#[cfg(test)]
mod tests {
  use super::*;

  #[test]
  fn use_case() {
      // test code
  }
}
```

#### Note

Clippy can only lint compiled code. For this lint to trigger, you must configure `cargo clippy`
to include test compilation, for instance, by using flags such as `--tests` or `--all-targets`.

---

### `redundant_type_annotations`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Warns about needless / redundant type annotations.

#### Why restrict this?

Code without type annotations is shorter and in most cases
more idiomatic and easier to modify.

#### Limitations

This lint doesn’t support:

- Generics
- Refs returned from anything else than a `MethodCall`
- Complex types (tuples, arrays, etc…)
- `Path` to anything else than a primitive type.

#### Example

```rust
let foo: String = String::new();
```

Use instead:

```rust
let foo = String::new();
```

---

### `ref_patterns`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.71.0 |

#### What it does

Checks for usages of the `ref` keyword.

#### Why restrict this?

The `ref` keyword can be confusing for people unfamiliar with it, and often
it is more concise to use `&` instead.

#### Example

```rust
let opt = Some(5);
if let Some(ref foo) = opt {}
```

Use instead:

```rust
let opt = Some(5);
if let Some(foo) = &opt {}
```

---

### `renamed_function_params`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.80.0 |

#### What it does

Lints when the name of function parameters from trait impl is
different than its default implementation.

#### Why restrict this?

Using the default name for parameters of a trait method is more consistent.

#### Example

```rust
struct A(u32);

impl PartialEq for A {
    fn eq(&self, b: &Self) -> bool {
        self.0 == b.0
    }
}
```

Use instead:

```rust
struct A(u32);

impl PartialEq for A {
    fn eq(&self, other: &Self) -> bool {
        self.0 == other.0
    }
}
```

#### Configuration

- `allow-renamed-params-for`:  List of trait paths to ignore when checking renamed function parameters.

#### Example

```toml
allow-renamed-params-for = [ "std::convert::From" ]
```

#### Noteworthy

- By default, the following traits are ignored: `From`, `TryFrom`, `FromStr`
- `".."` can be used as part of the list to indicate that the configured values should be appended to the
default configuration of Clippy. By default, any configuration will replace the default value.

(default: `["core::convert::From", "core::convert::TryFrom", "core::str::FromStr"]`)

---

### `rest_pat_in_fully_bound_structs`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.43.0 |

#### What it does

Checks for unnecessary ‘..’ pattern binding on struct when all fields are explicitly matched.

#### Why restrict this?

Correctness and readability. It’s like having a wildcard pattern after
matching all enum variants explicitly.

#### Example

```rust
let a = A { a: 5 };

match a {
    A { a: 5, .. } => {},
    _ => {},
}
```

Use instead:

```rust
match a {
    A { a: 5 } => {},
    _ => {},
}
```

---

### `return_and_then`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.86.0 |

#### What it does

Detect functions that end with `Option::and_then` or `Result::and_then`, and suggest using
the `?` operator instead.

#### Why is this bad?

The `and_then` method is used to chain a computation that returns an `Option` or a `Result`.
This can be replaced with the `?` operator, which is more concise and idiomatic.

#### Example

```rust
fn test(opt: Option<i32>) -> Option<i32> {
    opt.and_then(|n| {
        if n > 1 {
            Some(n + 1)
        } else {
            None
       }
    })
}
```

Use instead:

```rust
fn test(opt: Option<i32>) -> Option<i32> {
    let n = opt?;
    if n > 1 {
        Some(n + 1)
    } else {
        None
    }
}
```

---

### `same_name_method`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

It lints if a struct has two methods with the same name:
one from a trait, another not from a trait.

#### Why restrict this?

Confusing.

#### Example

```rust
trait T {
    fn foo(&self) {}
}

struct S;

impl T for S {
    fn foo(&self) {}
}

impl S {
    fn foo(&self) {}
}
```

---

### `self_named_module_files`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

Checks that module layout uses only `mod.rs` files.

#### Why restrict this?

Having multiple module layout styles in a project can be confusing.

#### Example

```text
src/
  stuff/
    stuff_files.rs
  stuff.rs
  lib.rs
```

Use instead:

```text
src/
  stuff/
    stuff_files.rs
    mod.rs
  lib.rs
```

---

### `semicolon_inside_block`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.68.0 |

#### What it does

Suggests moving the semicolon after a block to the inside of the block, after its last
expression.

#### Why restrict this?

For consistency it’s best to have the semicolon inside/outside the block. Either way is fine
and this lint suggests inside the block.
Take a look at `semicolon_outside_block` for the other alternative.

#### Example

```rust
unsafe { f(x) };
```

Use instead:

```rust
unsafe { f(x); }
```

#### Configuration

- `semicolon-inside-block-ignore-singleline`:  Whether to lint only if it’s multiline.

(default: `false`)

---

### `semicolon_outside_block`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.68.0 |

#### What it does

Suggests moving the semicolon from a block’s final expression outside of the block.

#### Why restrict this?

For consistency it’s best to have the semicolon inside/outside the block. Either way is fine
and this lint suggests outside the block.
Take a look at `semicolon_inside_block` for the other alternative.

#### Example

```rust
unsafe { f(x); }
```

Use instead:

```rust
unsafe { f(x) };
```

#### Configuration

- `semicolon-outside-block-ignore-multiline`:  Whether to lint only if it’s singleline.

(default: `false`)

---

### `separated_literal_suffix`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.58.0 |

#### What it does

Warns if literal suffixes are separated by an underscore.
To enforce separated literal suffix style,
see the `unseparated_literal_suffix` lint.

#### Why restrict this?

Suffix style should be consistent.

#### Example

```rust
123832_i32
```

Use instead:

```rust
123832i32
```

---

### `shadow_reuse`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for bindings that shadow other bindings already in
scope, while reusing the original value.

#### Why restrict this?

Some argue that name shadowing like this hurts readability,
because a value may be bound to different things depending on position in
the code.

See also `shadow_same` and `shadow_unrelated` for other restrictions on shadowing.

#### Example

```rust
let x = 2;
let x = x + 1;
```

use different variable name:

```rust
let x = 2;
let y = x + 1;
```

---

### `shadow_same`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for bindings that shadow other bindings already in
scope, while just changing reference level or mutability.

#### Why restrict this?

To require that what are formally distinct variables be given distinct names.

See also `shadow_reuse` and `shadow_unrelated` for other restrictions on shadowing.

#### Example

```rust
let x = &x;
```

Use instead:

```rust
let y = &x; // use different variable name
```

---

### `shadow_unrelated`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for bindings that shadow other bindings already in
scope, either without an initialization or with one that does not even use
the original value.

#### Why restrict this?

Shadowing a binding with a closely related one is part of idiomatic Rust,
but shadowing a binding by accident with an unrelated one may indicate a mistake.

Additionally, name shadowing in general can hurt readability, especially in
large code bases, because it is easy to lose track of the active binding at
any place in the code. If linting against all shadowing is desired, you may wish
to use the `shadow_same` and `shadow_reuse` lints as well.

#### Example

```rust
let x = y;
let x = z; // shadows the earlier binding
```

Use instead:

```rust
let x = y;
let w = z; // use different variable name
```

---

### `single_call_fn`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for functions that are only used once. Does not lint tests.

#### Why restrict this?

If a function is only used once (perhaps because it used to be used more widely),
then the code could be simplified by moving that function’s code into its caller.

However, there are reasons not to do this everywhere:

- Splitting a large function into multiple parts often improves readability
by giving names to its parts.
- A function’s signature might serve a necessary purpose, such as constraining
the type of a closure passed to it.
- Generic functions might call non-generic functions to reduce duplication
in the produced machine code.

If this lint is used, prepare to `#[allow]` it a lot.

#### Example

```rust
pub fn a<T>(t: &T)
where
    T: AsRef<str>,
{
    a_inner(t.as_ref())
}

fn a_inner(t: &str) {
    /* snip */
}
```

Use instead:

```rust
pub fn a<T>(t: &T)
where
    T: AsRef<str>,
{
    let t = t.as_ref();
    /* snip */
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `single_char_lifetime_names`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.60.0 |

#### What it does

Checks for lifetimes with names which are one character
long.

#### Why restrict this?

A single character is likely not enough to express the
purpose of a lifetime. Using a longer name can make code
easier to understand.

#### Known problems

Rust programmers and learning resources tend to use single
character lifetimes, so this lint is at odds with the
ecosystem at large. In addition, the lifetime’s purpose may
be obvious or, rarely, expressible in one character.

#### Example

```rust
struct DiagnosticCtx<'a> {
    source: &'a str,
}
```

Use instead:

```rust
struct DiagnosticCtx<'src> {
    source: &'src str,
}
```

---

### `std_instead_of_alloc`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Finds items imported through `std` when available through `alloc`.

#### Why restrict this?

Crates which have `no_std` compatibility and require alloc may wish to ensure types are imported from
alloc to ensure disabling `std` does not cause the crate to fail to compile. This lint is also useful
for crates migrating to become `no_std` compatible.

#### Example

```rust
use std::vec::Vec;
```

Use instead:

```rust
use alloc::vec::Vec;
```

---

### `std_instead_of_core`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.64.0 |

#### What it does

Finds items imported through `std` when available through `core`.

#### Why restrict this?

Crates which have `no_std` compatibility may wish to ensure types are imported from core to ensure
disabling `std` does not cause the crate to fail to compile. This lint is also useful for crates
migrating to become `no_std` compatible.

#### Example

```rust
use std::hash::Hasher;
```

Use instead:

```rust
use core::hash::Hasher;
```

---

### `str_to_string`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

This lint checks for `.to_string()` method calls on values of type `&str`.

#### Why restrict this?

The `to_string` method is also used on other types to convert them to a string.
When called on a `&str` it turns the `&str` into the owned variant `String`, which can be
more specifically expressed with `.to_owned()`.

#### Example

```rust
let _ = "str".to_string();
```

Use instead:

```rust
let _ = "str".to_owned();
```

---

### `string_add`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for all instances of `x + _` where `x` is of type
`String`, but only if [string_add_assign](#string_add_assign) does *not*
match.

#### Why restrict this?

This particular
`Add` implementation is asymmetric (the other operand need not be `String`,
but `x` does), while addition as mathematically defined is symmetric, and
the `String::push_str(_)` function is a perfectly good replacement.
Therefore, some dislike it and wish not to have it in their code.

That said, other people think that string addition, having a long tradition
in other languages is actually fine, which is why we decided to make this
particular lint `allow` by default.

#### Example

```rust
let x = "Hello".to_owned();
x + ", World";
```

Use instead:

```rust
let mut x = "Hello".to_owned();
x.push_str(", World");
```

---

### `string_lit_chars_any`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.73.0 |

#### What it does

Checks for `<string_lit>.chars().any(|i| i == c)`.

#### Why is this bad?

It’s significantly slower than using a pattern instead, like
`matches!(c, '\\' | '.' | '+')`.

Despite this being faster, this is not `perf` as this is pretty common, and is a rather nice
way to check if a `char` is any in a set. In any case, this `restriction` lint is available
for situations where that additional performance is absolutely necessary.

#### Example

```rust
"\\.+*?()|[]{}^$#&-~".chars().any(|x| x == c);
```

Use instead:

```rust
matches!(c, '\\' | '.' | '+' | '*' | '(' | ')' | '|' | '[' | ']' | '{' | '}' | '^' | '$' | '#' | '&' | '-' | '~');
```

---

### `string_slice`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.58.0 |

#### What it does

Checks for slice operations on strings

#### Why restrict this?

UTF-8 characters span multiple bytes, and it is easy to inadvertently confuse character
counts and string indices. This may lead to panics, and should warrant some test cases
containing wide UTF-8 characters. This lint is most useful in code that should avoid
panics at all costs.

#### Known problems

Probably lots of false positives. If an index comes from a known valid position (e.g.
obtained via `char_indices` over the same string), it is totally OK.

#### Example

```rust
&"Ölkanne"[1..];
```

---

### `suspicious_xor_used_as_pow`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.67.0 |

#### What it does

Warns for a Bitwise XOR (`^`) operator being probably confused as a powering. It will not trigger if any of the numbers are not in decimal.

#### Why restrict this?

It’s most probably a typo and may lead to unexpected behaviours.

#### Example

```rust
let x = 3_i32 ^ 4_i32;
```

Use instead:

```rust
let x = 3_i32.pow(4);
```

---

### `tests_outside_test_module`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.70.0 |

#### What it does

Triggers when a testing function (marked with the `#[test]` attribute) isn’t inside a testing module
(marked with `#[cfg(test)]`).

#### Why restrict this?

The idiomatic (and more performant) way of writing tests is inside a testing module (flagged with `#[cfg(test)]`),
having test functions outside of this module is confusing and may lead to them being “hidden”.

#### Example

```rust
#[test]
fn my_cool_test() {
    // [...]
}

#[cfg(test)]
mod tests {
    // [...]
}
```

Use instead:

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn my_cool_test() {
        // [...]
    }
}
```

---

### `todo`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks for usage of `todo!`.

#### Why restrict this?

The `todo!` macro indicates the presence of unfinished code,
so it should not be present in production code.

#### Example

```rust
todo!();
```

Finish the implementation, or consider marking it as explicitly unimplemented.

```rust
unimplemented!();
```

---

### `try_err`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.38.0 |

#### What it does

Checks for usage of `Err(x)?`.

#### Why restrict this?

The `?` operator is designed to allow calls that
can fail to be easily chained. For example, `foo()?.bar()` or
`foo(bar()?)`. Because `Err(x)?` can’t be used that way (it will
always return), it is more clear to write `return Err(x)`.

#### Example

```rust
fn foo(fail: bool) -> Result<i32, String> {
    if fail {
      Err("failed")?;
    }
    Ok(0)
}
```

Could be written:

```rust
fn foo(fail: bool) -> Result<i32, String> {
    if fail {
      return Err("failed".into());
    }
    Ok(0)
}
```

---

### `undocumented_unsafe_blocks`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.58.0 |

#### What it does

Checks for `unsafe` blocks and impls without a `// SAFETY: ` comment
explaining why the unsafe operations performed inside
the block are safe.

Note the comment must appear on the line(s) preceding the unsafe block
with nothing appearing in between. The following is ok:

```rust
foo(
    // SAFETY:
    // This is a valid safety comment
    unsafe { *x }
)
```

But neither of these are:

```rust
// SAFETY:
// This is not a valid safety comment
foo(
    /* SAFETY: Neither is this */ unsafe { *x },
);
```

#### Why restrict this?

Undocumented unsafe blocks and impls can make it difficult to read and maintain code.
Writing out the safety justification may help in discovering unsoundness or bugs.

#### Example

```rust
use std::ptr::NonNull;
let a = &mut 42;

let ptr = unsafe { NonNull::new_unchecked(a) };
```

Use instead:

```rust
use std::ptr::NonNull;
let a = &mut 42;

// SAFETY: references are guaranteed to be non-null.
let ptr = unsafe { NonNull::new_unchecked(a) };
```

#### Configuration

- `accept-comment-above-attributes`:  Whether to accept a safety comment to be placed above the attributes for the `unsafe` block

(default: `true`)
- `accept-comment-above-statement`:  Whether to accept a safety comment to be placed above the statement containing the `unsafe` block

(default: `true`)

---

### `unimplemented`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `unimplemented!`.

#### Why restrict this?

This macro, or panics in general, may be unwanted in production code.

#### Example

```rust
unimplemented!();
```

---

### `unnecessary_safety_comment`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.67.0 |

#### What it does

Checks for `// SAFETY: ` comments on safe code.

#### Why restrict this?

Safe code has no safety requirements, so there is no need to
describe safety invariants.

#### Example

```rust
use std::ptr::NonNull;
let a = &mut 42;

// SAFETY: references are guaranteed to be non-null.
let ptr = NonNull::new(a).unwrap();
```

Use instead:

```rust
use std::ptr::NonNull;
let a = &mut 42;

let ptr = NonNull::new(a).unwrap();
```

---

### `unnecessary_safety_doc`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.67.0 |

#### What it does

Checks for the doc comments of publicly visible
safe functions and traits and warns if there is a `# Safety` section.

#### Why restrict this?

Safe functions and traits are safe to implement and therefore do not
need to describe safety preconditions that users are required to uphold.

#### Examples

```rust
/// # Safety
///
/// This function should not be called before the horsemen are ready.
pub fn start_apocalypse_but_safely(u: &mut Universe) {
    unimplemented!();
}
```

The function is safe, so there shouldn’t be any preconditions
that have to be explained for safety reasons.

```rust
/// This function should really be documented
pub fn start_apocalypse(u: &mut Universe) {
    unimplemented!();
}
```

#### Configuration

- `check-private-items`:  Whether to also run the listed lints on private items.

(default: `false`)

---

### `unnecessary_self_imports`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.53.0 |

#### What it does

Checks for imports ending in `::{self}`.

#### Why restrict this?

In most cases, this can be written much more cleanly by omitting `::{self}`.

#### Known problems

Removing `::{self}` will cause any non-module items at the same path to also be imported.
This might cause a naming conflict (https://github.com/rust-lang/rustfmt/issues/3568). This lint makes no attempt
to detect this scenario and that is why it is a restriction lint.

#### Example

```rust
use std::io::{self};
```

Use instead:

```rust
use std::io;
```

---

### `unneeded_field_pattern`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for structure field patterns bound to wildcards.

#### Why restrict this?

Using `..` instead is shorter and leaves the focus on
the fields that are actually bound.

#### Example

```rust
let f = Foo { a: 0, b: 0, c: 0 };

match f {
    Foo { a: _, b: 0, .. } => {},
    Foo { a: _, b: _, c: _ } => {},
}
```

Use instead:

```rust
let f = Foo { a: 0, b: 0, c: 0 };

match f {
    Foo { b: 0, .. } => {},
    Foo { .. } => {},
}
```

---

### `unreachable`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks for usage of `unreachable!`.

#### Why restrict this?

This macro, or panics in general, may be unwanted in production code.

#### Example

```rust
unreachable!();
```

---

### `unseparated_literal_suffix`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Warns if literal suffixes are not separated by an
underscore.
To enforce unseparated literal suffix style,
see the `separated_literal_suffix` lint.

#### Why restrict this?

Suffix style should be consistent.

#### Example

```rust
123832i32
```

Use instead:

```rust
123832_i32
```

---

### `unused_result_ok`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.82.0 |

#### What it does

Checks for calls to `Result::ok()` without using the returned `Option`.

#### Why is this bad?

Using `Result::ok()` may look like the result is checked like `unwrap` or `expect` would do
but it only silences the warning caused by `#[must_use]` on the `Result`.

#### Example

```rust
some_function().ok();
```

Use instead:

```rust
let _ = some_function();
```

---

### `unused_trait_names`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.83.0 |

#### What it does

Checks for `use Trait` where the Trait is only used for its methods and not referenced by a path directly.

#### Why is this bad?

Traits imported that aren’t used directly can be imported anonymously with `use Trait as _`.
It is more explicit, avoids polluting the current scope with unused names and can be useful to show which imports are required for traits.

#### Example

```rust
use std::fmt::Write;

fn main() {
    let mut s = String::new();
    let _ = write!(s, "hello, world!");
    println!("{s}");
}
```

Use instead:

```rust
use std::fmt::Write as _;

fn main() {
    let mut s = String::new();
    let _ = write!(s, "hello, world!");
    println!("{s}");
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `unwrap_in_result`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.48.0 |

#### What it does

Checks for functions of type `Result` that contain `expect()` or `unwrap()`

#### Why restrict this?

These functions promote recoverable errors to non-recoverable errors,
which may be undesirable in code bases which wish to avoid panics,
or be a bug in the specific function.

#### Known problems

This can cause false positives in functions that handle both recoverable and non recoverable errors.

#### Example

Before:

```rust
fn divisible_by_3(i_str: String) -> Result<(), String> {
    let i = i_str
        .parse::<i32>()
        .expect("cannot divide the input by three");

    if i % 3 != 0 {
        Err("Number is not divisible by 3")?
    }

    Ok(())
}
```

After:

```rust
fn divisible_by_3(i_str: String) -> Result<(), String> {
    let i = i_str
        .parse::<i32>()
        .map_err(|e| format!("cannot divide the input by three: {}", e))?;

    if i % 3 != 0 {
        Err("Number is not divisible by 3")?
    }

    Ok(())
}
```

---

### `unwrap_used`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.45.0 |

#### What it does

Checks for `.unwrap()` or `.unwrap_err()` calls on `Result`s and `.unwrap()` call on `Option`s.

#### Why restrict this?

It is better to handle the `None` or `Err` case,
or at least call `.expect(_)` with a more helpful message. Still, for a lot of
quick-and-dirty code, `unwrap` is a good choice, which is why this lint is
`Allow` by default.

`result.unwrap()` will let the thread panic on `Err` values.
Normally, you want to implement more sophisticated error handling,
and propagate errors upwards with `?` operator.

Even if you want to panic on errors, not all `Error`s implement good
messages on display. Therefore, it may be beneficial to look at the places
where they may get displayed. Activate this lint to do just that.

#### Examples

```rust
option.unwrap();
result.unwrap();
```

Use instead:

```rust
option.expect("more helpful message");
result.expect("more helpful message");
```

If [expect_used](#expect_used) is enabled, instead:

```rust
option?;

// or

result?;
```

#### Past names

- option_unwrap_used
- result_unwrap_used

#### Configuration

- `allow-unwrap-in-consts`:  Whether `unwrap` should be allowed in code always evaluated at compile time

(default: `true`)
- `allow-unwrap-in-tests`:  Whether `unwrap` should be allowed in test functions or `#[cfg(test)]`

(default: `false`)
- `allow-unwrap-types`:  List of types to allow `unwrap()` and `expect()` on.

#### Example

```toml
allow-unwrap-types = [ "std::sync::LockResult" ]
```

(default: `[]`)

---

### `use_debug`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for usage of `Debug` formatting. The purpose of this
lint is to catch debugging remnants.

#### Why restrict this?

The purpose of the `Debug` trait is to facilitate debugging Rust code,
and [no guarantees are made about its output](https://doc.rust-lang.org/stable/std/fmt/trait.Debug.html#stability).
It should not be used in user-facing output.

#### Example

```rust
println!("{:?}", foo);
```

---

### `verbose_file_reads`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.44.0 |

#### What it does

Checks for usage of File::read_to_end and File::read_to_string.

#### Why restrict this?

`fs::{read, read_to_string}` provide the same functionality when `buf` is empty with fewer imports and no intermediate values.
See also: [fs::read docs](https://doc.rust-lang.org/std/fs/fn.read.html), [fs::read_to_string docs](https://doc.rust-lang.org/std/fs/fn.read_to_string.html)

#### Example

```rust
let mut f = File::open("foo.txt").unwrap();
let mut bytes = Vec::new();
f.read_to_end(&mut bytes).unwrap();
```

Can be written more concisely as

```rust
let mut bytes = fs::read("foo.txt").unwrap();
```

---

### `wildcard_enum_match_arm`

| 属性 | 值 |
|------|----|
| 分组 | restriction |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.34.0 |

#### What it does

Checks for wildcard enum matches using `_`.

#### Why restrict this?

New enum variants added by library updates can be missed.

#### Known problems

Suggested replacements may be incorrect if guards exhaustively cover some
variants, and also may not use correct path to enum if it’s not present in the current scope.

#### Example

```rust
match x {
    Foo::A(_) => {},
    _ => {},
}
```

Use instead:

```rust
match x {
    Foo::A(_) => {},
    Foo::B(_) => {},
}
```

---

## Nursery (52)

*实验性 - 仍在开发中的新 lint，默认 allow*

### `as_ptr_cast_mut`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.66.0 |

#### What it does

Checks for the result of a `&self`-taking `as_ptr` being cast to a mutable pointer.

#### Why is this bad?

Since `as_ptr` takes a `&self`, the pointer won’t have write permissions unless interior
mutability is used, making it unlikely that having it as a mutable pointer is correct.

#### Example

```rust
let mut vec = Vec::<u8>::with_capacity(1);
let ptr = vec.as_ptr() as *mut u8;
unsafe { ptr.write(4) }; // UNDEFINED BEHAVIOUR
```

Use instead:

```rust
let mut vec = Vec::<u8>::with_capacity(1);
let ptr = vec.as_mut_ptr();
unsafe { ptr.write(4) };
```

---

### `branches_sharing_code`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.53.0 |

#### What it does

Checks if the `if` and `else` block contain shared code that can be
moved out of the blocks.

#### Why is this bad?

Duplicate code is less maintainable.

#### Example

```rust
let foo = if … {
    println!("Hello World");
    13
} else {
    println!("Hello World");
    42
};
```

Use instead:

```rust
println!("Hello World");
let foo = if … {
    13
} else {
    42
};
```

---

### `clear_with_drain`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.70.0 |

#### What it does

Checks for usage of `.drain(..)` for the sole purpose of clearing a container.

#### Why is this bad?

This creates an unnecessary iterator that is dropped immediately.

Calling `.clear()` also makes the intent clearer.

#### Example

```rust
let mut v = vec![1, 2, 3];
v.drain(..);
```

Use instead:

```rust
let mut v = vec![1, 2, 3];
v.clear();
```

---

### `coerce_container_to_any`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.89.0 |

#### What it does

Protects against unintended coercion of references to container types to `&dyn Any` when the
container type dereferences to a `dyn Any` which could be directly referenced instead.

#### Why is this bad?

The intention is usually to get a reference to the `dyn Any` the value dereferences to,
rather than coercing a reference to the container itself to `&dyn Any`.

#### Example

Because `Box<dyn Any>` itself implements `Any`, `&Box<dyn Any>`
can be coerced to an `&dyn Any` which refers to *the Box itself*, rather than the
inner `dyn Any`.

```rust
let x: Box<dyn Any> = Box::new(0u32);
let dyn_any_of_box: &dyn Any = &x;

// Fails as we have a &dyn Any to the Box, not the u32
assert_eq!(dyn_any_of_box.downcast_ref::<u32>(), None);
```

Use instead:

```rust
let x: Box<dyn Any> = Box::new(0u32);
let dyn_any_of_u32: &dyn Any = &*x;

// Succeeds since we have a &dyn Any to the inner u32!
assert_eq!(dyn_any_of_u32.downcast_ref::<u32>(), Some(&0u32));
```

---

### `collection_is_never_read`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.70.0 |

#### What it does

Checks for collections that are never queried.

#### Why is this bad?

Putting effort into constructing a collection but then never querying it might indicate that
the author forgot to do whatever they intended to do with the collection. Example: Clone
a vector, sort it for iteration, but then mistakenly iterate the original vector
instead.

#### Example

```rust
let mut sorted_samples = samples.clone();
sorted_samples.sort();
for sample in &samples { // Oops, meant to use `sorted_samples`.
    println!("{sample}");
}
```

Use instead:

```rust
let mut sorted_samples = samples.clone();
sorted_samples.sort();
for sample in &sorted_samples {
    println!("{sample}");
}
```

---

### `debug_assert_with_mut_call`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.40.0 |

#### What it does

Checks for function/method calls with a mutable
parameter in `debug_assert!`, `debug_assert_eq!` and `debug_assert_ne!` macros.

#### Why is this bad?

In release builds `debug_assert!` macros are optimized out by the
compiler.
Therefore mutating something in a `debug_assert!` macro results in different behavior
between a release and debug build.

#### Example

```rust
debug_assert_eq!(vec![3].pop(), Some(3));

// or

debug_assert!(takes_a_mut_parameter(&mut x));
```

---

### `derive_partial_eq_without_eq`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.63.0 |

#### What it does

Checks for types that derive `PartialEq` and could implement `Eq`.

#### Why is this bad?

If a type `T` derives `PartialEq` and all of its members implement `Eq`,
then `T` can always implement `Eq`. Implementing `Eq` allows `T` to be used
in APIs that require `Eq` types. It also allows structs containing `T` to derive
`Eq` themselves.

#### Example

```rust
#[derive(PartialEq)]
struct Foo {
    i_am_eq: i32,
    i_am_eq_too: Vec<String>,
}
```

Use instead:

```rust
#[derive(PartialEq, Eq)]
struct Foo {
    i_am_eq: i32,
    i_am_eq_too: Vec<String>,
}
```

---

### `doc_link_code`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.87.0 |

#### What it does

Checks for links with code directly adjacent to code text:
`[`MyItem`]`<`[`u32`]`>``.

#### Why is this bad?

It can be written more simply using HTML-style `<code>` tags.

#### Example

```rust
//! [`first`](x)`second`
```

Use instead:

```rust
//! <code>[first](x)second</code>
```

---

### `equatable_if_let`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.57.0 |

#### What it does

Checks for pattern matchings that can be expressed using equality.

#### Why is this bad?

- It reads better and has less cognitive load because equality won’t cause binding.
- It is a [Yoda condition](https://en.wikipedia.org/wiki/Yoda_conditions). Yoda conditions are widely
criticized for increasing the cognitive load of reading the code.
- Equality is a simple bool expression and can be merged with `&&` and `||` and
reuse if blocks

#### Example

```rust
if let Some(2) = x {
    do_thing();
}
```

Use instead:

```rust
if x == Some(2) {
    do_thing();
}
```

---

### `fallible_impl_from`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for impls of `From<..>` that contain `panic!()` or `unwrap()`

#### Why is this bad?

`TryFrom` should be used if there’s a possibility of failure.

#### Example

```rust
struct Foo(i32);

impl From<String> for Foo {
    fn from(s: String) -> Self {
        Foo(s.parse().unwrap())
    }
}
```

Use instead:

```rust
struct Foo(i32);

impl TryFrom<String> for Foo {
    type Error = ();
    fn try_from(s: String) -> Result<Self, Self::Error> {
        if let Ok(parsed) = s.parse() {
            Ok(Foo(parsed))
        } else {
            Err(())
        }
    }
}
```

---

### `future_not_send`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.44.0 |

#### What it does

This lint requires Future implementations returned from
functions and methods to implement the `Send` marker trait,
ignoring type parameters.

If a function is generic and its Future conditionally implements `Send`
based on a generic parameter then it is considered `Send` and no warning is emitted.

This can be used by library authors (public and internal) to ensure
their functions are compatible with both multi-threaded runtimes that require `Send` futures,
as well as single-threaded runtimes where callers may choose `!Send` types
for generic parameters.

#### Why is this bad?

A Future implementation captures some state that it
needs to eventually produce its final value. When targeting a multithreaded
executor (which is the norm on non-embedded devices) this means that this
state may need to be transported to other threads, in other words the
whole Future needs to implement the `Send` marker trait. If it does not,
then the resulting Future cannot be submitted to a thread pool in the
end user’s code.

Especially for generic functions it can be confusing to leave the
discovery of this problem to the end user: the reported error location
will be far from its cause and can in many cases not even be fixed without
modifying the library where the offending Future implementation is
produced.

#### Example

```rust
async fn not_send(bytes: std::rc::Rc<[u8]>) {}
```

Use instead:

```rust
async fn is_send(bytes: std::sync::Arc<[u8]>) {}
```

---

### `imprecise_flops`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.43.0 |

#### What it does

Looks for floating-point expressions that
can be expressed using built-in methods to improve accuracy
at the cost of performance.

#### Why is this bad?

Negatively impacts accuracy.

#### Example

```rust
let a = 3f32;
let _ = a.powf(1.0 / 3.0);
let _ = (1.0 + a).ln();
let _ = a.exp() - 1.0;
```

Use instead:

```rust
let a = 3f32;
let _ = a.cbrt();
let _ = a.ln_1p();
let _ = a.exp_m1();
```

---

### `iter_on_empty_collections`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.65.0 |

#### What it does

Checks for calls to `iter`, `iter_mut` or `into_iter` on empty collections

#### Why is this bad?

It is simpler to use the empty function from the standard library:

#### Example

```rust
use std::{slice, option};
let a: slice::Iter<i32> = [].iter();
let f: option::IntoIter<i32> = None.into_iter();
```

Use instead:

```rust
use std::iter;
let a: iter::Empty<i32> = iter::empty();
let b: iter::Empty<i32> = iter::empty();
```

#### Known problems

The type of the resulting iterator might become incompatible with its usage

---

### `iter_on_single_items`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.65.0 |

#### What it does

Checks for calls to `iter`, `iter_mut` or `into_iter` on collections containing a single item

#### Why is this bad?

It is simpler to use the once function from the standard library:

#### Example

```rust
let a = [123].iter();
let b = Some(123).into_iter();
```

Use instead:

```rust
use std::iter;
let a = iter::once(&123);
let b = iter::once(123);
```

#### Known problems

The type of the resulting iterator might become incompatible with its usage

---

### `iter_with_drain`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.61.0 |

#### What it does

Checks for usage of `.drain(..)` on `Vec` and `VecDeque` for iteration.

#### Why is this bad?

`.into_iter()` is simpler with better performance.

#### Example

```rust
let mut foo = vec![0, 1, 2, 3];
let bar: HashSet<usize> = foo.drain(..).collect();
```

Use instead:

```rust
let foo = vec![0, 1, 2, 3];
let bar: HashSet<usize> = foo.into_iter().collect();
```

---

### `large_stack_frames`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for functions that use a lot of stack space.

This often happens when constructing a large type, such as an array with a lot of elements,
or constructing *many* smaller-but-still-large structs, or copying around a lot of large types.

This lint is a more general version of [large_stack_arrays](https://rust-lang.github.io/rust-clippy/master/#large_stack_arrays)
that is intended to look at functions as a whole instead of only individual array expressions inside of a function.

#### Why is this bad?

The stack region of memory is very limited in size (usually *much* smaller than the heap) and attempting to
use too much will result in a stack overflow and crash the program.
To avoid this, you should consider allocating large types on the heap instead (e.g. by boxing them).

Keep in mind that the code path to construction of large types does not even need to be reachable;
it purely needs to *exist* inside of the function to contribute to the stack size.
For example, this causes a stack overflow even though the branch is unreachable:

```rust
fn main() {
    if false {
        let x = [0u8; 10000000]; // 10 MB stack array
        black_box(&x);
    }
}
```

#### Known issues

False positives. The stack size that clippy sees is an estimated value and can be vastly different
from the actual stack usage after optimizations passes have run (especially true in release mode).
Modern compilers are very smart and are able to optimize away a lot of unnecessary stack allocations.
In debug mode however, it is usually more accurate.

This lint works by summing up the size of all variables that the user typed, variables that were
implicitly introduced by the compiler for temporaries, function arguments and the return value,
and comparing them against a (configurable, but high-by-default).

#### Example

This function creates four 500 KB arrays on the stack. Quite big but just small enough to not trigger `large_stack_arrays`.
However, looking at the function as a whole, it’s clear that this uses a lot of stack space.

```rust
struct QuiteLargeType([u8; 500_000]);
fn foo() {
    // ... some function that uses a lot of stack space ...
    let _x1 = QuiteLargeType([0; 500_000]);
    let _x2 = QuiteLargeType([0; 500_000]);
    let _x3 = QuiteLargeType([0; 500_000]);
    let _x4 = QuiteLargeType([0; 500_000]);
}
```

Instead of doing this, allocate the arrays on the heap.
This currently requires going through a `Vec` first and then converting it to a `Box`:

```rust
struct NotSoLargeType(Box<[u8]>);

fn foo() {
    let _x1 = NotSoLargeType(vec![0; 500_000].into_boxed_slice());
//                           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  Now heap allocated.
//                                                                The size of `NotSoLargeType` is 16 bytes.
//  ...
}
```

#### Configuration

- `allow-large-stack-frames-in-tests`:  Whether functions inside `#[cfg(test)]` modules or test functions should be checked.

(default: `true`)
- `stack-size-threshold`:  The maximum allowed stack size for functions in bytes

(default: `512000`)

---

### `literal_string_with_formatting_args`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.85.0 |

#### What it does

Checks if string literals have formatting arguments outside of macros
using them (like `format!`).

#### Why is this bad?

It will likely not generate the expected content.

#### Example

```rust
let x: Option<usize> = None;
let y = "hello";
x.expect("{y:?}");
```

Use instead:

```rust
let x: Option<usize> = None;
let y = "hello";
x.expect(&format!("{y:?}"));
```

---

### `missing_const_for_fn`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.34.0 |

#### What it does

Suggests the use of `const` in functions and methods where possible.

#### Why is this bad?

Not having the function const prevents callers of the function from being const as well.

#### Known problems

Const functions are currently still being worked on, with some features only being available
on nightly. This lint does not consider all edge cases currently and the suggestions may be
incorrect if you are using this lint on stable.

Also, the lint only runs one pass over the code. Consider these two non-const functions:

```rust
fn a() -> i32 {
    0
}
fn b() -> i32 {
    a()
}
```

When running Clippy, the lint will only suggest to make `a` const, because `b` at this time
can’t be const as it calls a non-const function. Making `a` const and running Clippy again,
will suggest to make `b` const, too.

If you are marking a public function with `const`, removing it again will break API compatibility.

#### Example

```rust
fn new() -> Self {
    Self { random_number: 42 }
}
```

Could be a const fn:

```rust
const fn new() -> Self {
    Self { random_number: 42 }
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `needless_collect`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.30.0 |

#### What it does

Checks for functions collecting an iterator when collect
is not needed.

#### Why is this bad?

`collect` causes the allocation of a new data structure,
when this allocation may not be needed.

#### Example

```rust
let len = iterator.collect::<Vec<_>>().len();
```

Use instead:

```rust
let len = iterator.count();
```

---

### `needless_pass_by_ref_mut`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.73.0 |

#### What it does

Check if a `&mut` function argument is actually used mutably.

Be careful if the function is publicly reexported as it would break compatibility with
users of this function, when the users pass this function as an argument.

#### Why is this bad?

Less `mut` means less fights with the borrow checker. It can also lead to more
opportunities for parallelization.

#### Example

```rust
fn foo(y: &mut i32) -> i32 {
    12 + *y
}
```

Use instead:

```rust
fn foo(y: &i32) -> i32 {
    12 + *y
}
```

#### Configuration

- `avoid-breaking-exported-api`:  Suppress lints whenever the suggested change would cause breakage for other crates.

(default: `true`)

---

### `needless_type_cast`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.94.0 |

#### What it does

Checks for bindings (constants, statics, or let bindings) that are defined
with one numeric type but are consistently cast to a different type in all usages.

#### Why is this bad?

If a binding is always cast to a different type when used, it would be clearer
and more efficient to define it with the target type from the start.

#### Example

```rust
const SIZE: u16 = 15;
let arr: [u8; SIZE as usize] = [0; SIZE as usize];
```

Use instead:

```rust
const SIZE: usize = 15;
let arr: [u8; SIZE] = [0; SIZE];
```

---

### `non_send_fields_in_send_ty`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

This lint warns about a `Send` implementation for a type that
contains fields that are not safe to be sent across threads.
It tries to detect fields that can cause a soundness issue
when sent to another thread (e.g., `Rc`) while allowing `!Send` fields
that are expected to exist in a `Send` type, such as raw pointers.

#### Why is this bad?

Sending the struct to another thread effectively sends all of its fields,
and the fields that do not implement `Send` can lead to soundness bugs
such as data races when accessed in a thread
that is different from the thread that created it.

See:

- [The Rustonomicon about Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html)
- [The documentation of Send](https://doc.rust-lang.org/std/marker/trait.Send.html)

#### Known Problems

This lint relies on heuristics to distinguish types that are actually
unsafe to be sent across threads and `!Send` types that are expected to
exist in  `Send` type. Its rule can filter out basic cases such as
`Vec<*const T>`, but it’s not perfect. Feel free to create an issue if
you have a suggestion on how this heuristic can be improved.

#### Example

```rust
struct ExampleStruct<T> {
    rc_is_not_send: Rc<String>,
    unbounded_generic_field: T,
}

// This impl is unsound because it allows sending `!Send` types through `ExampleStruct`
unsafe impl<T> Send for ExampleStruct<T> {}
```

Use thread-safe types like [std::sync::Arc](https://doc.rust-lang.org/std/sync/struct.Arc.html)
or specify correct bounds on generic type parameters (`T: Send`).

#### Configuration

- `enable-raw-pointer-heuristic-for-send`:  Whether to apply the raw pointer heuristic to determine if a type is `Send`.

(default: `true`)

---

### `nonstandard_macro_braces`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.55.0 |

#### What it does

Checks that common macros are used with consistent bracing.

#### Why is this bad?

Having non-conventional braces on well-stablished macros can be confusing
when debugging, and they bring incosistencies with the rest of the ecosystem.

#### Example

```rust
vec!{1, 2, 3};
```

Use instead:

```rust
vec![1, 2, 3];
```

#### Configuration

- `standard-macro-braces`:  Enforce the named macros always use the braces specified.

A `MacroMatcher` can be added like so `{ name = "macro_name", brace = "(" }`. If the macro
could be used with a full path two `MacroMatcher`s have to be added one with the full path
`crate_name::macro_name` and one with just the macro name.

(default: `[]`)

---

### `option_if_let_else`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.47.0 |

#### What it does

Lints usage of `if let Some(v) = ... { y } else { x }` and
`match .. { Some(v) => y, None/_ => x }` which are more
idiomatically done with `Option::map_or` (if the else bit is a pure
expression) or `Option::map_or_else` (if the else bit is an impure
expression).

#### Why is this bad?

Using the dedicated functions of the `Option` type is clearer and
more concise than an `if let` expression.

#### Notes

This lint uses a deliberately conservative metric for checking if the
inside of either body contains loop control expressions `break` or
`continue` (which cannot be used within closures). If these are found,
this lint will not be raised.

#### Example

```rust
let _ = if let Some(foo) = optional {
    foo
} else {
    5
};
let _ = match optional {
    Some(val) => val + 1,
    None => 5
};
let _ = if let Some(foo) = optional {
    foo
} else {
    let y = do_complicated_function();
    y*y
};
```

should be

```rust
let _ = optional.map_or(5, |foo| foo);
let _ = optional.map_or(5, |val| val + 1);
let _ = optional.map_or_else(||{
    let y = do_complicated_function();
    y*y
}, |foo| foo);
```

---

### `or_fun_call`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for calls to `.or(foo(..))`, `.unwrap_or(foo(..))`,
`.or_insert(foo(..))` etc., and suggests to use `.or_else(|| foo(..))`,
`.unwrap_or_else(|| foo(..))`, `.unwrap_or_default()` or `.or_default()`
etc. instead.

#### Why is this bad?

The function will always be called. This is only bad if it allocates or
does some non-trivial amount of work.

#### Known problems

If the function has side-effects, not calling it will change the
semantic of the program, but you shouldn’t rely on that.

The lint also cannot figure out whether the function you call is
actually expensive to call or not.

#### Example

```rust
foo.unwrap_or(String::from("empty"));
```

Use instead:

```rust
foo.unwrap_or_else(|| String::from("empty"));
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `path_buf_push_overwrite`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.36.0 |

#### What it does

- Checks for [push](https://doc.rust-lang.org/std/path/struct.PathBuf.html#method.push)
calls on `PathBuf` that can cause overwrites.

#### Why is this bad?

Calling `push` with a root path at the start can overwrite the
previous defined path.

#### Example

```rust
use std::path::PathBuf;

let mut x = PathBuf::from("/foo");
x.push("/bar");
assert_eq!(x, PathBuf::from("/bar"));
```

Could be written:

```rust
use std::path::PathBuf;

let mut x = PathBuf::from("/foo");
x.push("bar");
assert_eq!(x, PathBuf::from("/foo/bar"));
```

---

### `read_zero_byte_vec`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.63.0 |

#### What it does

This lint catches reads into a zero-length `Vec`.
Especially in the case of a call to `with_capacity`, this lint warns that read
gets the number of bytes from the `Vec`’s length, not its capacity.

#### Why is this bad?

Reading zero bytes is almost certainly not the intended behavior.

#### Known problems

In theory, a very unusual read implementation could assign some semantic meaning
to zero-byte reads. But it seems exceptionally unlikely that code intending to do
a zero-byte read would allocate a `Vec` for it.

#### Example

```rust
use std::io;
fn foo<F: io::Read>(mut f: F) {
    let mut data = Vec::with_capacity(100);
    f.read(&mut data).unwrap();
}
```

Use instead:

```rust
use std::io;
fn foo<F: io::Read>(mut f: F) {
    let mut data = Vec::with_capacity(100);
    data.resize(100, 0);
    f.read(&mut data).unwrap();
}
```

---

### `redundant_clone`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.32.0 |

#### What it does

Checks for a redundant `clone()` (and its relatives) which clones an owned
value that is going to be dropped without further use.

#### Why is this bad?

It is not always possible for the compiler to eliminate useless
allocations and deallocations generated by redundant `clone()`s.

#### Known problems

False-negatives: analysis performed by this lint is conservative and limited.

#### Example

```rust
{
    let x = Foo::new();
    call(x.clone());
    call(x.clone()); // this can just pass `x`
}

["lorem", "ipsum"].join(" ").to_string();

Path::new("/a/b").join("c").to_path_buf();
```

---

### `redundant_pub_crate`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.44.0 |

#### What it does

Checks for items declared `pub(crate)` that are not crate visible because they
are inside a private module.

#### Why is this bad?

Writing `pub(crate)` is misleading when it’s redundant due to the parent
module’s visibility.

#### Example

```rust
mod internal {
    pub(crate) fn internal_fn() { }
}
```

This function is not visible outside the module and it can be declared with `pub` or
private visibility

```rust
mod internal {
    pub fn internal_fn() { }
}
```

---

### `search_is_some`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for an iterator or string search (such as `find()`,
`position()`, or `rposition()`) followed by a call to `is_some()` or `is_none()`.

#### Why is this bad?

Readability, this can be written more concisely as:

- `_.any(_)`, or `_.contains(_)` for `is_some()`,
- `!_.any(_)`, or `!_.contains(_)` for `is_none()`.

#### Example

```rust
let vec = vec![1];
vec.iter().find(|x| **x == 0).is_some();

"hello world".find("world").is_none();
```

Use instead:

```rust
let vec = vec![1];
vec.iter().any(|x| *x == 0);

!"hello world".contains("world");
```

---

### `set_contains_or_insert`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.81.0 |

#### What it does

Checks for usage of `contains` to see if a value is not present
in a set like `HashSet` or `BTreeSet`, followed by an `insert`.

#### Why is this bad?

Using just `insert` and checking the returned `bool` is more efficient.

#### Known problems

In case the value that wants to be inserted is borrowed and also expensive or impossible
to clone. In such a scenario, the developer might want to check with `contains` before inserting,
to avoid the clone. In this case, it will report a false positive.

#### Example

```rust
use std::collections::HashSet;
let mut set = HashSet::new();
let value = 5;
if !set.contains(&value) {
    set.insert(value);
    println!("inserted {value:?}");
}
```

Use instead:

```rust
use std::collections::HashSet;
let mut set = HashSet::new();
let value = 5;
if set.insert(&value) {
    println!("inserted {value:?}");
}
```

---

### `significant_drop_in_scrutinee`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.60.0 |

#### What it does

Checks for temporaries returned from function calls in a match scrutinee that have the
`clippy::has_significant_drop` attribute.

#### Why is this bad?

The `clippy::has_significant_drop` attribute can be added to types whose Drop impls have
an important side-effect, such as unlocking a mutex, making it important for users to be
able to accurately understand their lifetimes. When a temporary is returned in a function
call in a match scrutinee, its lifetime lasts until the end of the match block, which may
be surprising.

For `Mutex`es this can lead to a deadlock. This happens when the match scrutinee uses a
function call that returns a `MutexGuard` and then tries to lock again in one of the match
arms. In that case the `MutexGuard` in the scrutinee will not be dropped until the end of
the match block and thus will not unlock.

#### Example

```rust
let mutex = Mutex::new(State {});

match mutex.lock().unwrap().foo() {
    true => {
        mutex.lock().unwrap().bar(); // Deadlock!
    }
    false => {}
};

println!("All done!");
```

Use instead:

```rust
let mutex = Mutex::new(State {});

let is_foo = mutex.lock().unwrap().foo();
match is_foo {
    true => {
        mutex.lock().unwrap().bar();
    }
    false => {}
};

println!("All done!");
```

---

### `significant_drop_tightening`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MaybeIncorrect |
| 引入版本 | 1.69.0 |

#### What it does

Searches for elements marked with `#[clippy::has_significant_drop]` that could be early
dropped but are in fact dropped at the end of their scopes. In other words, enforces the
“tightening” of their possible lifetimes.

#### Why is this bad?

Elements marked with `#[clippy::has_significant_drop]` are generally synchronizing
primitives that manage shared resources, as such, it is desired to release them as soon as
possible to avoid unnecessary resource contention.

#### Example

```rust
fn main() {
  let lock = some_sync_resource.lock();
  let owned_rslt = lock.do_stuff_with_resource();
  // Only `owned_rslt` is needed but `lock` is still held.
  do_heavy_computation_that_takes_time(owned_rslt);
}
```

Use instead:

```rust
fn main() {
    let owned_rslt = some_sync_resource.lock().do_stuff_with_resource();
    do_heavy_computation_that_takes_time(owned_rslt);
}
```

---

### `single_option_map`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.87.0 |

#### What it does

Checks for functions with method calls to `.map(_)` on an arg
of type `Option` as the outermost expression.

#### Why is this bad?

Taking and returning an `Option<T>` may require additional
`Some(_)` and `unwrap` if all you have is a `T`.

#### Example

```rust
fn double(param: Option<u32>) -> Option<u32> {
    param.map(|x| x * 2)
}
```

Use instead:

```rust
fn double(param: u32) -> u32 {
    param * 2
}
```

---

### `string_lit_as_bytes`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for the `as_bytes` method called on string literals
that contain only ASCII characters.

#### Why is this bad?

Byte string literals (e.g., `b"foo"`) can be used
instead. They are shorter but less discoverable than `as_bytes()`.

#### Known problems

`"str".as_bytes()` and the suggested replacement of `b"str"` are not
equivalent because they have different types. The former is `&[u8]`
while the latter is `&[u8; 3]`. That means in general they will have a
different set of methods and different trait implementations.

```rust
fn f(v: Vec<u8>) {}

f("...".as_bytes().to_owned()); // works
f(b"...".to_owned()); // does not work, because arg is [u8; 3] not Vec<u8>

fn g(r: impl std::io::Read) {}

g("...".as_bytes()); // works
g(b"..."); // does not work
```

The actual equivalent of `"str".as_bytes()` with the same type is not
`b"str"` but `&b"str"[..]`, which is a great deal of punctuation and not
more readable than a function call.

#### Example

```rust
let bstr = "a byte string".as_bytes();
```

Use instead:

```rust
let bstr = b"a byte string";
```

---

### `suboptimal_flops`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.43.0 |

#### What it does

Looks for floating-point expressions that
can be expressed using built-in methods to improve both
accuracy and performance.

#### Why is this bad?

Negatively impacts accuracy and performance.

#### Example

```rust
use std::f32::consts::E;

let a = 3f32;
let _ = (2f32).powf(a);
let _ = E.powf(a);
let _ = a.powf(1.0 / 2.0);
let _ = a.log(2.0);
let _ = a.log(10.0);
let _ = a.log(E);
let _ = a.powf(2.0);
let _ = a * 2.0 + 4.0;
let _ = if a < 0.0 {
    -a
} else {
    a
};
let _ = if a < 0.0 {
    a
} else {
    -a
};
```

is better expressed as

```rust
use std::f32::consts::E;

let a = 3f32;
let _ = a.exp2();
let _ = a.exp();
let _ = a.sqrt();
let _ = a.log2();
let _ = a.log10();
let _ = a.ln();
let _ = a.powi(2);
let _ = a.mul_add(2.0, 4.0);
let _ = a.abs();
let _ = -a.abs();
```

---

### `suspicious_operation_groupings`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.50.0 |

#### What it does

Checks for unlikely usages of binary operators that are almost
certainly typos and/or copy/paste errors, given the other usages
of binary operators nearby.

#### Why is this bad?

They are probably bugs and if they aren’t then they look like bugs
and you should add a comment explaining why you are doing such an
odd set of operations.

#### Known problems

There may be some false positives if you are trying to do something
unusual that happens to look like a typo.

#### Example

```rust
struct Vec3 {
    x: f64,
    y: f64,
    z: f64,
}

impl Eq for Vec3 {}

impl PartialEq for Vec3 {
    fn eq(&self, other: &Self) -> bool {
        // This should trigger the lint because `self.x` is compared to `other.y`
        self.x == other.y && self.y == other.y && self.z == other.z
    }
}
```

Use instead:

```rust
// same as above except:
impl PartialEq for Vec3 {
    fn eq(&self, other: &Self) -> bool {
        // Note we now compare other.x to self.x
        self.x == other.x && self.y == other.y && self.z == other.z
    }
}
```

---

### `too_long_first_doc_paragraph`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.82.0 |

#### What it does

Checks if the first paragraph in the documentation of items listed in the module page is too long.

#### Why is this bad?

Documentation will show the first paragraph of the docstring in the summary page of a
module. Having a nice, short summary in the first paragraph is part of writing good docs.

#### Example

```rust
/// A very short summary.
/// A much longer explanation that goes into a lot more detail about
/// how the thing works, possibly with doclinks and so one,
/// and probably spanning a many rows.
struct Foo {}
```

Use instead:

```rust
/// A very short summary.
///
/// A much longer explanation that goes into a lot more detail about
/// how the thing works, possibly with doclinks and so one,
/// and probably spanning a many rows.
struct Foo {}
```

---

### `trailing_empty_array`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.58.0 |

#### What it does

Displays a warning when a struct with a trailing zero-sized array is declared without a `repr` attribute.

#### Why is this bad?

Zero-sized arrays aren’t very useful in Rust itself, so such a struct is likely being created to pass to C code or in some other situation where control over memory layout matters (for example, in conjunction with manual allocation to make it easy to compute the offset of the array). Either way, `#[repr(C)]` (or another `repr` attribute) is needed.

#### Example

```rust
struct RarelyUseful {
    some_field: u32,
    last: [u32; 0],
}
```

Use instead:

```rust
#[repr(C)]
struct MoreOftenUseful {
    some_field: usize,
    last: [u32; 0],
}
```

---

### `trait_duplication_in_bounds`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.47.0 |

#### What it does

Checks for cases where generics or trait objects are being used and multiple
syntax specifications for trait bounds are used simultaneously.

#### Why is this bad?

Duplicate bounds makes the code
less readable than specifying them only once.

#### Example

```rust
fn func<T: Clone + Default>(arg: T) where T: Clone + Default {}
```

Use instead:

```rust
fn func<T: Clone + Default>(arg: T) {}

// or

fn func<T>(arg: T) where T: Clone + Default {}
```

```rust
fn foo<T: Default + Default>(bar: T) {}
```

Use instead:

```rust
fn foo<T: Default>(bar: T) {}
```

```rust
fn foo<T>(bar: T) where T: Default + Default {}
```

Use instead:

```rust
fn foo<T>(bar: T) where T: Default {}
```

---

### `transmute_undefined_repr`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.60.0 |

#### What it does

Checks for transmutes between types which do not have a representation defined relative to
each other.

#### Why is this bad?

The results of such a transmute are not defined.

#### Known problems

This lint has had multiple problems in the past and was moved to `nursery`. See issue
[#8496](https://github.com/rust-lang/rust-clippy/issues/8496) for more details.

#### Example

```rust
struct Foo<T>(u32, T);
let _ = unsafe { core::mem::transmute::<Foo<u32>, Foo<i32>>(Foo(0u32, 0u32)) };
```

Use instead:

```rust
#[repr(C)]
struct Foo<T>(u32, T);
let _ = unsafe { core::mem::transmute::<Foo<u32>, Foo<i32>>(Foo(0u32, 0u32)) };
```

---

### `trivial_regex`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for trivial [regex](https://crates.io/crates/regex)
creation (with `Regex::new`, `RegexBuilder::new`, or `RegexSet::new`).

#### Why is this bad?

Matching the regex can likely be replaced by `==` or
`str::starts_with`, `str::ends_with` or `std::contains` or other `str`
methods.

#### Known problems

If the same regex is going to be applied to multiple
inputs, the precomputations done by `Regex` construction can give
significantly better performance than any of the `str`-based methods.

#### Example

```rust
Regex::new("^foobar")
```

Use instead:

```rust
str::starts_with("foobar")
```

---

### `tuple_array_conversions`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.72.0 |

#### What it does

Checks for tuple<=>array conversions that are not done with `.into()`.

#### Why is this bad?

It may be unnecessary complexity. `.into()` works for converting tuples<=> arrays of up to
12 elements and conveys the intent more clearly, while also leaving less room for hard to
spot bugs!

#### Known issues

The suggested code may hide potential asymmetry in some cases. See
[#11085](https://github.com/rust-lang/rust-clippy/issues/11085) for more info.

#### Example

```rust
let t1 = &[(1, 2), (3, 4)];
let v1: Vec<[u32; 2]> = t1.iter().map(|&(a, b)| [a, b]).collect();
```

Use instead:

```rust
let t1 = &[(1, 2), (3, 4)];
let v1: Vec<[u32; 2]> = t1.iter().map(|&t| t.into()).collect();
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `type_repetition_in_bounds`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.38.0 |

#### What it does

This lint warns about unnecessary type repetitions in trait bounds

#### Why is this bad?

Repeating the type for every bound makes the code
less readable than combining the bounds

#### Example

```rust
pub fn foo<T>(t: T) where T: Copy, T: Clone {}
```

Use instead:

```rust
pub fn foo<T>(t: T) where T: Copy + Clone {}
```

#### Configuration

- `max-trait-bounds`:  The maximum number of bounds a trait can have to be linted

(default: `3`)
- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)

---

### `uninhabited_references`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.76.0 |

#### What it does

It detects references to uninhabited types, such as `!` and
warns when those are either dereferenced or returned from a function.

#### Why is this bad?

Dereferencing a reference to an uninhabited type would create
an instance of such a type, which cannot exist. This constitutes
undefined behaviour. Such a reference could have been created
by `unsafe` code.

#### Example

The following function can return a reference to an uninhabited type
(`Infallible`) because it uses `unsafe` code to create it. However,
the user of such a function could dereference the return value and
trigger an undefined behavior from safe code.

```rust
fn create_ref() -> &'static std::convert::Infallible {
    unsafe { std::mem::transmute(&()) }
}
```

---

### `unnecessary_struct_initialization`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.70.0 |

#### What it does

Checks for initialization of an identical `struct` from another instance
of the type, either by copying a base without setting any field or by
moving all fields individually.

#### Why is this bad?

Readability suffers from unnecessary struct building.

#### Example

```rust
struct S { s: String }

let a = S { s: String::from("Hello, world!") };
let b = S { ..a };
```

Use instead:

```rust
struct S { s: String }

let a = S { s: String::from("Hello, world!") };
let b = a;
```

The struct literal `S { ..a }` in the assignment to `b` could be replaced
with just `a`.

#### Known Problems

Has false positives when the base is a place expression that cannot be
moved out of, see [#10547](https://github.com/rust-lang/rust-clippy/issues/10547).

Empty structs are ignored by the lint.

---

### `unused_peekable`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.65.0 |

#### What it does

Checks for the creation of a `peekable` iterator that is never `.peek()`ed

#### Why is this bad?

Creating a peekable iterator without using any of its methods is likely a mistake,
or just a leftover after a refactor.

#### Example

```rust
let collection = vec![1, 2, 3];
let iter = collection.iter().peekable();

for item in iter {
    // ...
}
```

Use instead:

```rust
let collection = vec![1, 2, 3];
let iter = collection.iter();

for item in iter {
    // ...
}
```

---

### `unused_rounding`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | 1.63.0 |

#### What it does

Detects cases where a whole-number literal float is being rounded, using
the `floor`, `ceil`, or `round` methods.

#### Why is this bad?

This is unnecessary and confusing to the reader. Doing this is probably a mistake.

#### Example

```rust
let x = 1f32.ceil();
```

Use instead:

```rust
let x = 1f32;
```

---

### `use_self`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | MachineApplicable |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for unnecessary repetition of structure name when a
replacement with `Self` is applicable.

#### Why is this bad?

Unnecessary repetition. Mixed use of `Self` and struct
name
feels inconsistent.

#### Known problems

- Unaddressed false negative in fn bodies of trait implementations

#### Example

```rust
struct Foo;
impl Foo {
    fn new() -> Foo {
        Foo {}
    }
}
```

could be

```rust
struct Foo;
impl Foo {
    fn new() -> Self {
        Self {}
    }
}
```

#### Configuration

- `msrv`:  The minimum rust version that the project supports. Defaults to the `rust-version` field in `Cargo.toml`

(default: `current version`)
- `recursive-self-in-type-definitions`:  Whether the type itself in a struct or enum should be replaced with `Self` when encountering recursive types.

(default: `true`)

---

### `useless_let_if_seq`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | HasPlaceholders |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks for variable declarations immediately followed by a
conditional affectation.

#### Why is this bad?

This is not idiomatic Rust.

#### Example

```rust
let foo;

if bar() {
    foo = 42;
} else {
    foo = 0;
}

let mut baz = None;

if bar() {
    baz = Some(42);
}
```

should be written

```rust
let foo = if bar() {
    42
} else {
    0
};

let baz = if bar() {
    Some(42)
} else {
    None
};
```

---

### `volatile_composites`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.92.0 |

#### What it does

This lint warns when volatile load/store operations
(`write_volatile`/`read_volatile`) are applied to composite types.

#### Why is this bad?

Volatile operations are typically used with memory mapped IO devices,
where the precise number and ordering of load and store instructions is
important because they can have side effects. This is well defined for
primitive types like `u32`, but less well defined for structures and
other composite types. In practice it’s implementation defined, and the
behavior can be rustc-version dependent.

As a result, code should only apply `write_volatile`/`read_volatile` to
primitive types to be fully well-defined.

#### Example

```rust
struct MyDevice {
    addr: usize,
    count: usize
}

fn start_device(device: *mut MyDevice, addr: usize, count: usize) {
    unsafe {
        device.write_volatile(MyDevice { addr, count });
    }
}
```

Instead, operate on each primtive field individually:

```rust
struct MyDevice {
    addr: usize,
    count: usize
}

fn start_device(device: *mut MyDevice, addr: usize, count: usize) {
    unsafe {
        (&raw mut (*device).addr).write_volatile(addr);
        (&raw mut (*device).count).write_volatile(count);
    }
}
```

---

### `while_float`

| 属性 | 值 |
|------|----|
| 分组 | nursery |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.80.0 |

#### What it does

Checks for while loops comparing floating point values.

#### Why is this bad?

If you increment floating point values, errors can compound,
so, use integers instead if possible.

#### Known problems

The lint will catch all while loops comparing floating point
values without regarding the increment.

#### Example

```rust
let mut x = 0.0;
while x < 42.0 {
    x += 1.0;
}
```

Use instead:

```rust
let mut x = 0;
while x < 42 {
    x += 1;
}
```

---

## Cargo (5)

*Cargo - Cargo manifest 相关的 lint，默认 allow*

### `cargo_common_metadata`

| 属性 | 值 |
|------|----|
| 分组 | cargo |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.32.0 |

#### What it does

Checks to see if all common metadata is defined in
`Cargo.toml`. See: https://rust-lang-nursery.github.io/api-guidelines/documentation.html#cargotoml-includes-all-common-metadata-c-metadata

#### Why is this bad?

It will be more difficult for users to discover the
purpose of the crate, and key information related to it.

#### Example

```toml
[package]
name = "clippy"
version = "0.0.212"
repository = "https://github.com/rust-lang/rust-clippy"
readme = "README.md"
license = "MIT OR Apache-2.0"
keywords = ["clippy", "lint", "plugin"]
categories = ["development-tools", "development-tools::cargo-plugins"]
```

Should include a description field like:

```toml
[package]
name = "clippy"
version = "0.0.212"
description = "A bunch of helpful lints to avoid common pitfalls in Rust"
repository = "https://github.com/rust-lang/rust-clippy"
readme = "README.md"
license = "MIT OR Apache-2.0"
keywords = ["clippy", "lint", "plugin"]
categories = ["development-tools", "development-tools::cargo-plugins"]
```

#### Configuration

- `cargo-ignore-publish`:  For internal testing only, ignores the current `publish` settings in the Cargo manifest.

(default: `false`)

---

### `multiple_crate_versions`

| 属性 | 值 |
|------|----|
| 分组 | cargo |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | pre 1.29.0 |

#### What it does

Checks to see if multiple versions of a crate are being
used.

#### Why is this bad?

This bloats the size of targets, and can lead to
confusing error messages when structs or traits are used interchangeably
between different versions of a crate.

#### Known problems

Because this can be caused purely by the dependencies
themselves, it’s not always possible to fix this issue.
In those cases, you can allow that specific crate using
the `allowed-duplicate-crates` configuration option.

#### Example

```toml
[dependencies]
ctrlc = "=3.1.0"
ansi_term = "=0.11.0"
```

#### Configuration

- `allowed-duplicate-crates`:  A list of crate names to allow duplicates of

(default: `[]`)

---

### `negative_feature_names`

| 属性 | 值 |
|------|----|
| 分组 | cargo |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

Checks for negative feature names with prefix `no-` or `not-`

#### Why is this bad?

Features are supposed to be additive, and negatively-named features violate it.

#### Example

```toml
[features]
default = []
no-abc = []
not-def = []
```

Use instead:

```toml
[features]
default = ["abc", "def"]
abc = []
def = []
```

---

### `redundant_feature_names`

| 属性 | 值 |
|------|----|
| 分组 | cargo |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.57.0 |

#### What it does

Checks for feature names with prefix `use-`, `with-` or suffix `-support`

#### Why is this bad?

These prefixes and suffixes have no significant meaning.

#### Example

```toml
[features]
default = ["use-abc", "with-def", "ghi-support"]
use-abc = []  // redundant
with-def = []   // redundant
ghi-support = []   // redundant
```

Use instead:

```toml
[features]
default = ["abc", "def", "ghi"]
abc = []
def = []
ghi = []
```

---

### `wildcard_dependencies`

| 属性 | 值 |
|------|----|
| 分组 | cargo |
| 默认级别 | allow |
| Applicability | Unspecified |
| 引入版本 | 1.32.0 |

#### What it does

Checks for wildcard dependencies in the `Cargo.toml`.

#### Why is this bad?

[As the edition guide says](https://rust-lang-nursery.github.io/edition-guide/rust-2018/cargo-and-crates-io/crates-io-disallows-wildcard-dependencies.html),
it is highly unlikely that you work with any possible version of your dependency,
and wildcard dependencies would cause unnecessary breakage in the ecosystem.

#### Example

```toml
[dependencies]
regex = "*"
```

Use instead:

```toml
[dependencies]
some_crate_1 = "~1.2.3"

some_crate_2 = "=1.2.3"
```

---

