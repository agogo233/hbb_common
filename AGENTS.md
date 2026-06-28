# hbb_common

Rust 公共库，归属 [RustDesk](https://rustdesk.com) 项目。无独立可运行产物，仅作为库被上层仓库引用。

## 关键命令

- `cargo build` — 默认构建。
- `cargo test` — 运行所有内联单元测试（`#[cfg(test)]`）。
- `cargo run --example <name>` — 运行示例。
- `cargo run --example webrtc --features webrtc -- --offer <endpoint>` — 带 WebRTC 特性运行示例。
- `cargo run --example webrtc --features webrtc` — 启动 offer 端，按提示在另一终端运行 answer 端。

## 构建注意事项

- 编译前必须生成 Protobuf 代码：`build.rs` 会在 `OUT_DIR/protos/` 下自动生成 `rendezvous.proto` 和 `message.proto` 的 Rust 绑定。修改 `.proto` 后无需手动操作，触发重新编译即可。
- `protos/mod.rs` 通过 `include!(concat!(env!("OUT_DIR"), "/protos/mod.rs"));` 引入生成代码，不要直接编辑。
- 通过 `protobuf_codegen`（纯 Rust，无 protoc 依赖）生成代码。

## 特性（Features）

- `default = []` — 默认无 WebRTC。
- `webrtc` — 启用 `src/webrtc.rs` 模块及 `webrtc` 依赖。WebRTC 示例和库功能均依赖此特性。

## 结构速览

| 模块 | 说明 |
|------|------|
| `src/lib.rs` | 库入口，导出各模块及常用第三方 crate（`tokio`、`protobuf`、`sodiumoxide` 等）。 |
| `src/protos/` | Protobuf 生成代码目录，**代码自动生成，勿手动编辑**。 |
| `src/config.rs` | 核心配置模块（`PeerConfig`、`Config` 等）。 |
| `src/platform/` | 平台适配（`linux.rs`/`macos.rs`/`windows.rs`），条件编译。 |
| `src/bytes_codec.rs`, `src/tcp.rs`, `src/udp.rs`, `src/socket_client.rs` | 网络与编解码基础设施。 |
| `src/websocket.rs`, `src/webrtc.rs` | WebSocket / WebRTC 传输层。 |
| `src/fs.rs`, `src/stream.rs`, `src/tls.rs` | 文件、流、TLS 工具。 |
| `examples/` | 三个示例：`config`、`system_message`、`webrtc`（需 `webrtc` feature）。 |
| `protos/` | 源 `.proto` 文件定义。 |

## 特殊依赖

- 多个依赖指向 `rustdesk-org` 组织下的 fork（`confy`、`tokio-socks`、`sysinfo`、`default_net`、`machine-uid`），不要使用 crates.io 原版替代。
- `sysinfo` 锁定在 `rlim_max` 分支，原因见 [rustdesk PR #6330](https://github.com/rustdesk/rustdesk/pull/6330#issuecomment-2270871442)。
- `flexi_logger` 固定版本 `0.27`，因为新版在 `rustc 1.75` 构建失败。

## 平台差异

- 部分模块在 `android` 和 `ios` 上被排除（`#[cfg(not(any(target_os = "android", target_os = "ios")))]`）：`mac_address`、`default_net`、`machine-uid`、`dlopen`。
- Linux 额外依赖 `smithay-client-toolkit`（`sctk`）和 `x11`。macOS 额外依赖 `osascript`。Windows 额外依赖 `winapi`。

## 测试

- 全部为单元测试，分布在各模块 `#[cfg(test)]` 中。
- 无 `tests/` 目录，无集成测试。
- 运行时无需额外服务（如数据库、网络服务器）。

## 代码风格

- Rust edition 2018。
- 使用 `cargo fmt` / `cargo clippy` 等标准工具即可。
