# pmpx-plugin-cargo

> English see [README.md](README.md)

[pmpx](https://crates.io/crates/pmpx) 的 **cargo** 后端。它只做一件事：把 pmpx 的动词映射成
cargo 命令 —— 不读文件、不看环境变量、不联网。

```console
$ pmpx install serde        # Rust 项目里 → cargo add serde
$ pmpx run -- --version     #              → cargo run -- --version
$ pmpx test -- --nocapture  #              → cargo test -- --nocapture
```

## 映射表

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
| `exec` | 不支持 |

**其中两条要自己插一个 `--`。** `cargo run` 和 `cargo test` 后面都挂着东西（一个程序、
一个测试 harness），而 cargo 对自己不认识的参数**不是转发、是拒绝**。
`pmpx test -- --nocapture` 就是这个情况的具体样子 —— 少了 `--` 它会报：

```text
error: unexpected argument '--nocapture' found
  tip: a similar argument exists: '--features'
```

`cargo build` 后面没有东西，参数直接过。

**`exec` 是刻意不支持的。** pmpx 会把它降级成自己跑那条命令 —— 这比编一个 cargo 里不存在的
子命令要好。

**`install` 无参时是 `cargo fetch`，不是构建。** cargo 没有独立的"装依赖"这一步，而 `fetch`
是最接近的：它解析并下载，但不编译。

## 安装

```console
$ pmpx plugin add cargo
```

## 检测

依据随这个 crate 一起发布的 `pmpx-plugin.toml`：

| 文件 | 权重 | 能证明什么 |
| ---- | ---- | ---------- |
| `Cargo.lock` | 强（100） | 这个项目确实被 cargo 解析过 |
| `Cargo.toml` | 弱（10） | 只证明属于这个生态，不证明用了哪个工具 |

**两档之间的差距才是重点。** 一个 gitignore 掉锁文件的库 crate 只有 10 分，在同一棵树里
可能输给别的生态 —— 这正是 `.pmpx.toml` 里那些固化项存在的理由。

## 环境要求

Rust **1.82+**，这是契约 crate 的地板。`cargo add` / `cargo remove` 本身只需要 1.62。

## 仓库

<https://github.com/pmpx-rs/pmpx-plugin-cargo>

## 许可

MIT —— 见 [LICENSE](LICENSE)。
