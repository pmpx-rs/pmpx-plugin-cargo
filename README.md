# pmpx-plugin-cargo

> 中文版见 [README_CN.md](README_CN.md)

The **cargo** backend for [pmpx](https://crates.io/crates/pmpx). It maps pmpx's verbs onto
cargo commands, and does nothing else: no file reads, no environment, no network.

```console
$ pmpx install serde      # in a Rust project → cargo add serde
$ pmpx run -- --version   #                   → cargo run -- --version
$ pmpx test -- --nocapture #                  → cargo test -- --nocapture
```

## The mapping

| pmpx | cargo |
| ---- | ----- |
| `install` | `cargo fetch` |
| `install <pkg>...` | `cargo add <pkg>...` |
| `remove <pkg>...` | `cargo remove <pkg>...` |
| `run [arg]...` | `cargo run -- [arg]...` |
| `build [arg]...` | `cargo build [arg]...` |
| `test [arg]...` | `cargo test -- [arg]...` |
| `update` | `cargo update` |
| `update <pkg>...` | `cargo update <pkg>...` |
| `exec` | unsupported |

**Two of these insert a `--`.** `cargo run` and `cargo test` both have something sitting
behind them -- a program, a test harness -- and cargo refuses anything it does not recognise
as an option of its own instead of forwarding it. `pmpx test -- --nocapture` is the concrete
case: without the `--` it fails with `unexpected argument '--nocapture' found`. `cargo build`
has nothing behind it, so its arguments go straight through.

**`exec` is deliberately unsupported.** pmpx degrades it to running the command itself, which
beats inventing a cargo subcommand that does not exist.

**`install` with no arguments is `cargo fetch`**, not a build: cargo has no separate
"install the dependencies" step, and `fetch` is the closest thing -- it resolves and downloads
without compiling.

## Install

```console
$ pmpx plugin add cargo
```

Every release also publishes prebuilt assets for the common targets -- Linux x64, Windows
x64 and both macOS architectures. `crate-plugin-kit` downloads them from the release of the
same tag, so an install usually takes a second instead of a build; a target without assets
falls back to compiling from source, which is slower, not broken.

## Detection

From `pmpx-plugin.toml`, which travels with this crate:

| File | Weight | What it proves |
| ---- | ------ | -------------- |
| `Cargo.lock` | strong (100) | the project was actually resolved by cargo |
| `Cargo.toml` | weak (10) | the ecosystem, not the tool |

The gap between the two tiers is the point. A library crate that gitignores its lockfile
scores 10 and can lose to another ecosystem in the same tree -- that is what the pins in
`.pmpx.toml` are for.

## Requirements

Rust **1.82+**, which is the contract crate's floor. `cargo add` and `cargo remove`
themselves need only 1.62.

## Repository

<https://github.com/pmpx-rs/pmpx-plugin-cargo>

## License

MIT — see [LICENSE](LICENSE).
