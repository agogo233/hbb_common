# Changelog

> 本次对比基线：`origin/main`（commit 387603f）  
> 对比分支：`Mod`（HEAD e4d9715）  
> 用户/贡献者：agogo233

---

## 合并注意事项

1. **此分支包含品牌重命名（RustDesk → MyDesk）及默认端口变更，与上游 origin/main 在业务定位上存在差异，直接合并将覆盖上游默认配置。** 请先评估是否需要保留 MyDesk 定制逻辑，或将其提取为独立的 feature/配置项。
2. **端口变更影响范围大**：涉及 rendezvous/relay/ws 四大默认端口，合并后所有依赖默认端口的服务端、客户端、测试用例均需同步调整。
3. **常量更名会破坏 API**：`OPTION_ALLOW_HTTPS_21114` 已更名为 `OPTION_ALLOW_HTTPS_31114`，任何外部引用该常量的代码都需要同步修改。

---

## feat

### 默认端口由 211XX 迁移至 311XX

- `RENDEZVOUS_PORT`: `21116` → `31116`
- `RELAY_PORT`: `21117` → `31117`
- `WS_RENDEZVOUS_PORT`: `21118` → `31118`
- `WS_RELAY_PORT`: `21119` → `31119`

**涉及文件**：`src/config.rs`

---

### 应用名称及品牌标识由 RustDesk 统一替换为 MyDesk

- `APP_NAME` 默认值由 `"RustDesk"` 改为 `"MyDesk"`
- `VER_TYPE_RUSTDESK_CLIENT` / `VER_TYPE_RUSTDESK_SERVER` 常量分别更名为 `VER_TYPE_MYDESK_CLIENT` / `VER_TYPE_MYDESK_SERVER`
- Linux 平台环境变量由 `RUSTDESK_FORCED_DISPLAY_SERVER` 改为 `MYDESK_FORCED_DISPLAY_SERVER`
- 崩溃信号处理器弹窗标题由 `"RustDesk"` 改为 `"MyDesk"`
- 测试临时文件前缀由 `rustdesk_validation_{id}` 改为 `mydesk_validation_{id}`

**涉及文件**：`src/config.rs`、`src/lib.rs`、`src/fs.rs`、`src/platform/linux.rs`、`src/platform/mod.rs`

---

### WebSocket 单元测试端口同步更新

所有 WebSocket 测试用例中的硬编码端口 `21115~21119` 同步调整为 `31115~31119`，以保持与默认端口变更一致。

**涉及文件**：`src/websocket.rs`

---

## fix

### `get_or` 增加空值过滤，修复配置回退链断裂

**问题现象**：当 `CONFIG2.options` 中存在残留的空值键（如 `custom-rendezvous-server: ""`）时，`HashMap::get` 会返回 `Some("")`。由于原实现仅使用 `.or()` 做回退，`Some("")` 会阻断后续向 `DEFAULT_SETTINGS` 的回退链，导致默认值被忽略。

**修复方式**：在回退链中增加 `.filter(|x| !x.is_empty())`，使空字符串等价于 `None`，确保能继续向后回退到 `DEFAULT_SETTINGS`。

**涉及文件**：`src/config.rs`

---

### 常量 `OPTION_ALLOW_HTTPS_21114` 重命名为 `OPTION_ALLOW_HTTPS_31114`

- 同步更新常量名称及其在 `REUSABLE_SETTINGS` 数组中的引用，保持与 HTTPS 端口变更一致。

**涉及文件**：`src/config.rs`

---

## 文件变更汇总

| 文件 | 变更类型 | 变更说明 |
|------|----------|----------|
| `src/config.rs` | 修改 | 端口常量、APP_NAME、get_or 空值过滤、OPTION_ALLOW_HTTPS 常量更名 |
| `src/fs.rs` | 修改 | 测试临时文件前缀 rustdesk → mydesk |
| `src/lib.rs` | 修改 | VER_TYPE 常量更名、测试端口调整 |
| `src/platform/linux.rs` | 修改 | 环境变量名 rustdesk → mydesk |
| `src/platform/mod.rs` | 修改 | 崩溃弹窗标题 rustdesk → mydesk |
| `src/websocket.rs` | 修改 | 测试用例端口 211XX → 311XX |
