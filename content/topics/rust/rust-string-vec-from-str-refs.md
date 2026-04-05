+++
type = "notecard"
title = "Create a Vector of Owned Strings from &str Collection in Rust"
description = "Concise One-Liners for `Vec<String>` from `[&str, n]`"

slug = "rust-string-vec-from-str-refs"

tags = ['Rust']

date = 2026-03-29
+++

## Problem

Creating a `Vec<String>` from `&str` values, such as those in test fixtures,
can waste quite a bit of space on `String::from` or `"some-string".into()` calls:

```rust
let input = String::from(" {% comment %} {% endcomment %} ");
let expected = vec![
    String::from(" "),
    String::from("{% comment %}"),
    String::from(" "),
    String::from("{% endcomment %}"),
    String::from(" "),
];
```

## Quick Solution

### Option 1: `map` + `to_vec`

This is the most concise:

```rust
let expected = [" ", "{% comment %}", " ", "{% endcomment %}", " "]
    .map(String::from)
    .to_vec();
```

### Option 2: `iter` + `map` + `collect`

This is a bit more verbose but can be applied to more iterable types than just `vec`:

```rust
let expected: Vec<String> = [" ", "{% comment %}", " ", "{% endcomment %}", " "]
    .iter()
    .map(|&s| s.into())
    .collect();
```

## Q.E.D.

Stay Rusty.

## References

* [Rust docs: `slice.to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
* [Rust docs: `Iterator.collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)