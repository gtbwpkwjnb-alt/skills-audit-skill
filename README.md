# Skills Audit v3.0 — 工具画像 · 评分 · 优化（多平台通用）

> **Profile · Score · Optimize** — 一键审查你的全部代理工具，找出短板、冗余和优化空间。
> One-command audit: profile, score, and optimize all your agent tools across platforms.
> **跨平台**: ZCode · Claude Code · Codex · Cursor · Windsurf

[![Version](https://img.shields.io/badge/version-3.0.0-blue)](VERSION)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-ZCode%20%7C%20Claude%20Code%20%7C%20Codex-lightgrey)]()

---

## Overview

**EN**: A multi-platform agent that profiles you and your project, scores every installed tool/skill/plugin across 4 dimensions (project fit, retention trend, description quality, maintainability), suggests optimizations, and executes cleanup. One command — full audit.

**CN**: 跨平台工具审查系统。扫描你的用户画像和项目画像，对每个已安装工具/技能/插件做四维评分（项目适配、留存趋势、描述质量、可维护性），给出优化建议并执行清理。一条命令，完整审查。

### Core Capabilities

| Feature | 功能 |
|---------|------|
| 👤 **User Profile** | 用户画像 — 工作类型、高频工具、模式偏好 |
| 📁 **Project Profile** | 项目画像 — 类型、技术栈 |
| 📊 **4D Scoring** | 四维评分 — 适配 35% + 留存 30% + 质量 20% + 维护 15% |
| 🔧 **Auto Optimize** | 自动优化 — 修复描述、归档低分工具、搭配建议 |

---

## Trigger / 触发

| Platform | Trigger | Example |
|----------|---------|---------|
| **ZCode** | `skills-audit` or `技能审查` or `审查技能` | `> skills-audit` |
| **Claude Code** | `/skills-audit` | `> /skills-audit` |
| **Codex** | `skills-audit` or `审查技能` | `> skills-audit` |
| **Cursor** | `@skills-audit` | `> @skills-audit` |

> Standalone word only. Not triggered in sentences.

---

## Scoring Model / 评分模型

| Dimension | Weight | Basis |
|-----------|--------|-------|
| 🎯 **Project Fit** | 35% | Tool's function vs project type |
| 📈 **Retention** | 30% | Historical usage + score stability |
| 📝 **Description** | 20% | Clarity, completeness, bilingual |
| 🔧 **Maintainability** | 15% | User-editable + valid path + complete format |

### Rating / 分类

- 🟢 **Keep** (≥70) — Good to go
- 🟡 **Watch** (40-69) — Needs improvement
- 🔴 **Archive** (<40) — Remove or replace

---

## Config / 配置

Edit `config.yaml` to set your platform's scan paths and archive directory. Platform presets available in `platforms/`.

```yaml
scan_paths:
  - name: "用户工具"
    path: "~/.zcode/skills/*/"
    type: user
  - name: "插件工具"
    path: "~/.zcode/plugins/*/"
    type: system
```

See [SKILL.md](SKILL.md) for full config reference.

---

## Installation / 安装

```bash
curl -sL https://raw.githubusercontent.com/gtbwpkwjnb-alt/skills-audit-skill/main/install.sh | bash
```

Windows PowerShell:
```powershell
iwr https://raw.githubusercontent.com/gtbwpkwjnb-alt/skills-audit-skill/main/install.ps1 | iex
```

### Manual

```bash
# ZCode
git clone git@github.com:gtbwpkwjnb-alt/skills-audit-skill.git ~/.zcode/skills/skills-audit

# Claude Code
git clone git@github.com:gtbwpkwjnb-alt/skills-audit-skill.git ~/.claude/skills/skills-audit

# Codex
git clone git@github.com:gtbwpkwjnb-alt/skills-audit-skill.git ~/.codex/skills/skills-audit
```

---

## Changelog

### v3.0.0 (2026-06-18) — 多平台通用化

- 🌍 **Multi-platform** — ZCode / Claude Code / Codex / Cursor / Windsurf
- 📋 **config.yaml** — 可配置 scan paths, archive dir, custom rules
- 🎯 **New trigger** — `skills-audit` / `技能审查` / `审查技能` (zero false positives)
- 🔧 **Generic scoring** — 4D model with configurable quality checks
- 📚 **Platform presets** — `platforms/zcode.yaml`, `platforms/default.yaml`

### v2.3.0 — ZCode 专用版 (原名 skill-manager)

- Profile, 4D scoring, optimization suggestions, auto-persistence

---

## Feedback

🐛 [GitHub Issues](https://github.com/gtbwpkwjnb-alt/skills-audit-skill/issues/new)

## License

MIT