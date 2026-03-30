+++
type = "notecard"
title = "Create a Vector of Owned Strings from &str Collection in Rust"
description = "Concise One-Liners for `Vec<String>` from `[&str, n]`"

slug = "jetbrains-ide-launcher-script"

tags = ['JetBrains', 'Linux', 'Shell']

date = 2026-01-25
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