# Rust-LaTeX-Parser

LaTeX math parser for Rust. Takes a string, gives you an AST. No dependencies.


## Install

```toml
[dependencies]
rust-latex-parser = { git = "https://github.com/William-Selna/Rust-LaTeX-Parser" }
```

## Usage

```rust
use rust_latex_parser::{parse_equation, EqNode};

let tree = parse_equation("\\frac{-b \\pm \\sqrt{b^2 - 4ac}}{2a}");

match tree {
    EqNode::Frac(numerator, denominator) => {
        // do whatever you want with it
    }
    _ => {}
}
```

You get back an `EqNode` tree. Render it, convert it, inspect it — that part's on you.

## What it handles

Fractions (`\frac{a}{b}`, `a/b`), roots (`\sqrt{x}`), super/subscripts, matrices, cases, big operators with limits (`\sum`, `\int`, `\prod`), trig and log functions, accents (`\hat`, `\vec`, `\bar`), `\left...\right` delimiters, math fonts (`\mathbb`, `\mathcal`, etc.), binomials, over/underbraces, stacked relations, and 130+ symbols (Greek, arrows, set theory, relations, the usual).

You can also skip backslashes for common stuff — `pi` works, `sqrt(x)` works, `int_0^1` works. Parens after barewords act as grouping, so `sqrt(x+1)` grabs the whole `x+1`.

## The tree

19 variants:

```rust
EqNode::Text(String)                    // "x", "42", "α"
EqNode::Space(f32)                      // inserted around operators
EqNode::Seq(Vec<EqNode>)               // horizontal sequence
EqNode::Sup(base, superscript)
EqNode::Sub(base, subscript)
EqNode::SupSub(base, sup, sub)
EqNode::Frac(numerator, denominator)
EqNode::Sqrt(contents)
EqNode::BigOp { symbol, lower, upper }
EqNode::Accent(contents, kind)
EqNode::Limit { name, lower }
EqNode::TextBlock(String)               // \text{...}
EqNode::MathFont { kind, content }
EqNode::Delimited { left, right, content }
EqNode::Matrix { kind, rows }
EqNode::Cases { rows }
EqNode::Binom(top, bottom)
EqNode::Brace { content, label, over }
EqNode::StackRel { base, annotation, over }
```

There's also `EqMetrics` (width/ascent/descent) if you need it for layout.

## Symbol lookup

`latex_to_unicode` works standalone if you just need that:

```rust
use rust_latex_parser::latex_to_unicode;

latex_to_unicode("alpha");      // Some("α")
latex_to_unicode("rightarrow"); // Some("→")
latex_to_unicode("garbage");    // None
```

## Error handling

It doesn't error. Bad input gives you a best-effort tree — unmatched braces get skipped, unknown commands turn into text nodes. I needed this for a live editor where the input is always half-finished.

## Tests

```sh
cargo test
```

95 tests — parser correctness, adversarial input (deep nesting, emoji, 10k tokens), real-world expressions (quadratic formula, Gaussian integral, Schrodinger, Maxwell), Greek alphabet coverage.

## License

MIT
