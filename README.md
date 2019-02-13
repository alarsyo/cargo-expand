# cargo-expand

[![Build Status](https://travis-ci.org/dtolnay/cargo-expand.svg?branch=master)](https://travis-ci.org/dtolnay/cargo-expand)
[![Latest Version](https://img.shields.io/crates/v/cargo-expand.svg)](https://crates.io/crates/cargo-expand)

Once installed, the following command prints out the result of macro expansion
and `#[derive]` expansion applied to the current crate.

```console
$ cargo expand
```

This is a wrapper around the more verbose compiler command:

```console
$ cargo rustc --profile=check -- -Zunstable-options --pretty=expanded
```

## Installation

Install with **`cargo install cargo-expand`**.

This command optionally uses [rustfmt] to format the expanded output. The
resulting code is typically much more readable than what you get from the
compiler. If rustfmt is not available, the expanded code is not formatted.
Install rustfmt with **`rustup component add rustfmt`**.

Cargo expand relies on unstable compiler flags so it requires a nightly
toolchain to be installed, though does not require nightly to be the default
toolchain or the one with which cargo expand itself is executed. If the default
toolchain is one other than nightly, running `cargo expand` will find and use
nightly anyway.

[rustfmt]: https://github.com/rust-lang/rustfmt

## Example

#### `$ cat src/main.rs`

```rust
#[derive(Debug)]
struct S;

fn main() {
    println!("{:?}", S);
}
```

#### `$ cargo expand`

```rust
#![feature(prelude_import)]
#![no_std]
#[prelude_import]
use std::prelude::v1::*;
#[macro_use]
extern crate std as std;
struct S;
#[automatically_derived]
#[allow(unused_qualifications)]
impl $crate::fmt::Debug for S {
    fn fmt(&self, f: &mut $crate::fmt::Formatter) -> $crate::fmt::Result {
        match *self {
            S => {
                let mut debug_trait_builder = f.debug_tuple("S");
                debug_trait_builder.finish()
            }
        }
    }
}
fn main() {
    {
        $crate::io::_print($crate::fmt::Arguments::new_v1_formatted(
            &["", "\n"],
            &match (&S,) {
                (arg0,) => [$crate::fmt::ArgumentV1::new(
                    arg0,
                    $crate::fmt::Debug::fmt,
                )],
            },
            &[$crate::fmt::rt::v1::Argument {
                position: $crate::fmt::rt::v1::Position::At(0usize),
                format: $crate::fmt::rt::v1::FormatSpec {
                    fill: ' ',
                    align: $crate::fmt::rt::v1::Alignment::Unknown,
                    flags: 0u32,
                    precision: $crate::fmt::rt::v1::Count::Implied,
                    width: $crate::fmt::rt::v1::Count::Implied,
                },
            }],
        ));
    };
}
```

## Options

*See `cargo expand --help` for a complete list of options, most of which are
consistent with other Cargo subcommands. Here are a few that are common in the
context of cargo expand.*

To expand a particular test target:

`$ cargo expand --test test_something`

To expand without rustfmt:

`$ cargo expand --ugly`

To expand a specific module or type or function only:

`$ cargo expand path::to::module`

[![cargo expand punctuated::printing][punctuated.png]][syn]
[![cargo expand token::FatArrow][fatarrow.png]][syn]

[punctuated.png]: https://raw.githubusercontent.com/dtolnay/cargo-expand/screenshots/punctuated.png
[fatarrow.png]: https://raw.githubusercontent.com/dtolnay/cargo-expand/screenshots/fatarrow.png
[syn]: https://github.com/dtolnay/syn

## Configuration

The cargo expand command reads the `[expand]` section of $CARGO_HOME/config if
there is one. Currently the only configuration option is the syntax highlighting
theme.

```toml
[expand]
theme = "TwoDark"
```

Run `cargo expand --themes` to print a list of available themes. Use `theme =
"none"` to disable coloring.

## Disclaimer

Be aware that macro expansion to text is a lossy process. This is a debugging
aid only. There should be no expectation that the expanded code can be compiled
successfully, nor that if it compiles then it behaves the same as the original
code.

For instance the following function returns `3` when compiled ordinarily by Rust
but the expanded code compiles and returns `4`.

```rust
fn f() -> i32 {
    let x = 1;

    macro_rules! first_x {
        () => { x }
    }

    let x = 2;

    x + first_x!()
}
```

Refer to [The Book] for more on the considerations around macro hygiene.

[The Book]: https://doc.rust-lang.org/book/first-edition/macros.html#hygiene

<br>

#### License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>
