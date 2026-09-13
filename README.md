# omp-antigravity-guard 🛡️

[English](README.md) | [中文说明](README_CN.md)

Zero-configuration, drop-in extension for [oh-my-pi (omp)](https://github.com/can1357/oh-my-pi) that completely resolves the false **HTTP 429 `RESOURCE_EXHAUSTED`** error when using Google Antigravity (Google AI Pro / Gemini Cloud Code) models.

---

## ⚡ Quick Start

Install with a single command:

### macOS / Linux
```bash
git clone https://github.com/ygq-future/omp-antigravity-guard.git ~/.omp/agent/extensions/omp-antigravity-guard
```

### Windows (PowerShell)
```powershell
git clone https://github.com/ygq-future/omp-antigravity-guard.git "$env:USERPROFILE\.omp\agent\extensions\omp-antigravity-guard"
```

Restart `omp`, and Google Antigravity models (`gemini-3.8-flash`, `gemini-3-pro`, `claude-sonnet-4-6`, etc.) will work immediately with **HTTP 200 OK**!

---

## 🔍 The Problem

When using Google Antigravity as the provider in `omp` (v18.1.17, v18.1.18, and later), requests fail on the very first prompt with a fake quota error despite having 90%+ quota remaining:

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

### Why does this happen?
As thoroughly investigated and verified in [Issue #11689](https://github.com/can1357/oh-my-pi/issues/11689) and [Issue #11809](https://github.com/can1357/oh-my-pi/issues/11809):
1. **`requestType: "agent"` arms upstream content inspection**: Unlike official Google IDE clients (which omit `requestType`), `omp` sends `requestType: "agent"`.
2. **Targeted fingerprint blocking**: Under `requestType: "agent"`, Google Cloud Code's backend scans for `omp`'s system prompt opening tag (`<system-conventions>\nRFC 2119...`). When matched, Google immediately returns a fake `429 RESOURCE_EXHAUSTED`.

### Why doesn't upstream `omp` fix this?
In [Issue #11809](https://github.com/can1357/oh-my-pi/issues/11809), the maintainers explicitly decided **not to ship any in-tree bypass or release fix**, citing concerns about escalating anti-bot countermeasures with Google. PRs such as [#11742](https://github.com/can1357/oh-my-pi/pull/11742) remain unmerged.

---

## 🛠️ How This Extension Fixes It

This extension provides a dual-layer, zero-dependency defense:

1. **Prompt Sanitization (`before_agent_start`)**:
   Normalizes `<system-conventions>` and `</system-conventions>` to `<conventions>` and `</conventions>`. The LLM's comprehension of instructions and tool capabilities is **100% unaffected**, but Google's literal regex fingerprint match fails cleanly.
2. **Wire-Level Envelope Stripping (`fetch` interceptor)**:
   Intercepts requests dispatched to `*cloudcode-pa*.googleapis.com` and drops the `requestType: "agent"` field, perfectly aligning the request with the official Antigravity client.

---

## ✨ Features

- **Survives `omp` Upgrades**: Placed in `~/.omp/agent/extensions/`, so running `bun update -g` will never overwrite or disable your fix.
- **Zero Configuration**: No proxy, sidecar daemon, or server setup required.
- **Zero Performance Impact**: Nanosecond-level string substitution and non-blocking in-process fetch wrapping.
- **Safe Fallback**: If an unexpected payload is encountered, the original request passes through unchanged.

---

## 🗑️ Uninstall

Simply delete the extension folder:

```bash
# macOS / Linux
rm -rf ~/.omp/agent/extensions/omp-antigravity-guard

# Windows (PowerShell)
Remove-Item -Recurse -Force "$env:USERPROFILE\.omp\agent\extensions\omp-antigravity-guard"
```

---

## License

[MIT](LICENSE) © 2026 ygq-future
