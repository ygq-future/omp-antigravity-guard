# omp-antigravity-guard 🛡️

[English](README.md) | [中文说明](README_CN.md)

针对 [oh-my-pi (omp)](https://github.com/can1357/oh-my-pi) 的免配置外挂扩展插件，彻底解决使用 Google Antigravity（Google AI Pro / Gemini Cloud Code）模型时遇到的虚假 **HTTP 429 `RESOURCE_EXHAUSTED`** 错误。

---

## ⚡ 极速安装

只需一行终端命令克隆至扩展目录即可：

### macOS / Linux
```bash
git clone https://github.com/ygq-future/omp-antigravity-guard.git ~/.omp/agent/extensions/omp-antigravity-guard
```

### Windows (PowerShell)
```powershell
git clone https://github.com/ygq-future/omp-antigravity-guard.git "$env:USERPROFILE\.omp\agent\extensions\omp-antigravity-guard"
```

重启 `omp` 后，Google Antigravity 渠道的模型（`gemini-3.8-flash`、`gemini-3-pro`、`claude-sonnet-4-6` 等）即可秒级正常返回 **HTTP 200 OK**！

---

## 🔍 问题根因与背景

在 `omp`（v18.1.17、v18.1.18 及更新版本）中使用 Google Antigravity 时，即使账号额度剩余 90% 以上，发送第一条消息也会立刻报错：

```
Error: Cloud Code Assist API error (429): {
  "error": {
    "code": 429,
    "message": "Resource has been exhausted (e.g. check quota).",
    "status": "RESOURCE_EXHAUSTED"
  }
}
Error: Retry failed after 1 attempts: Provider requested 1800000ms wait, exceeds retry.maxDelayMs (300000ms).
```

### 为什么会发生？
根据开源社区在 [Issue #11689](https://github.com/can1357/oh-my-pi/issues/11689) 和 [Issue #11809](https://github.com/can1357/oh-my-pi/issues/11809) 的严格二分排查：
1. **`requestType: "agent"` 触发审查开关**：不同于 Google 官方 IDE 客户端（不发送 `requestType` 字段），`omp` 在请求信封中硬编码了 `requestType: "agent"`。
2. **定向提示词指纹拦截**：在 `requestType: "agent"` 开启下，Google 服务端会扫描系统提示词开头。一旦命中 `omp` 独有的 `<system-conventions>\nRFC 2119...` 文本特征，Google 就会直接返回伪装的 `429 RESOURCE_EXHAUSTED`。

### 为什么官方仓库不修复？
在 [Issue #11809](https://github.com/can1357/oh-my-pi/issues/11809) 中，官方维护者公开声明：**为了避免招致与 Google 的反爬对抗升级，官方不会合入或发布任何绕过补丁**，相关 PR（如 [#11742](https://github.com/can1357/oh-my-pi/pull/11742)）已全部搁置不予发布。

---

## 🛠️ 本扩展的修复机制（双重拦截）

本扩展采用纯用户态的零依赖双重拦截策略：

1. **生命周期提示词脱敏（`before_agent_start` Hook）**：
   在会话启动时，自动将 `<system-conventions>` 及 `</system-conventions>` 标签规范化为 `<conventions>`。大语言模型对指令的理解和工具调用能力**100% 保持一致**，但 Google 的文本特征匹配将彻底扑空。
2. **底层请求信封脱敏（`fetch` 拦截）**：
   动态拦截发往 `*cloudcode-pa*.googleapis.com` 的请求，剔除 `requestType: "agent"` 字段并自动重整 `content-length`，完全对齐 Google 官方 IDE 客户端行为。

---

## ✨ 核心优势

- **跨版本持久生效**：存放在 `~/.omp/agent/extensions/` 用户目录，后续执行 `bun update -g` 升级 `omp` 不会被任何方式覆盖或抹除。
- **零配置开箱即用**：不需要搭建任何中转代理、反向代理或侧车守护进程。
- **零性能开销**：纳秒级字符串处理与进程内透明调用，无任何网络延迟。
- **安全容错机制**：遇到非 Antigravity 目标或异常数据结构自动放行，绝不阻断正常业务。

---

## 🗑️ 卸载方式

如需卸载，直接删除该扩展文件夹即可：

```bash
# macOS / Linux
rm -rf ~/.omp/agent/extensions/omp-antigravity-guard

# Windows (PowerShell)
Remove-Item -Recurse -Force "$env:USERPROFILE\.omp\agent\extensions\omp-antigravity-guard"
```

---

## 许可证

[MIT](LICENSE) © 2026 ygq-future
