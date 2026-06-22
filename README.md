# Skills Audit v4.3.0 — 项目技能推荐引擎

> **Project Skill Audit** — 扫描项目文件夹，分析技术栈和项目类型，生成全面的技能/插件/工具推荐报告。画像驱动·四维评分·描述精炼。
>
> **跨平台**: ZCode · Claude Code · Codex · Cursor · Windsurf

[![Version](https://img.shields.io/badge/version-4.3.0-blue)](VERSION)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-ZCode%20%7C%20Claude%20Code%20%7C%20Codex-lightgrey)]()

---

## Overview

**EN**: Scans your active project folder, profiles the tech stack, then scores installed skills on 4 dimensions (Fit/Value/Fresh/Community), checks description quality, and searches the web for better alternatives. Each suggestion includes a 3-part explanation: **原因** (why), **效果** (effect), **场景** (use case).

**CN**: 扫描开发中项目，构建项目角色画像，四维评分已安装技能，检查描述规范，搜索网络备选。每项建议附带三段解释：**原因**、**效果**、**场景**。

---

## Trigger / 触发

| Platform | Trigger | Example |
|----------|---------|---------|
| **ZCode** | `技能审查`（主）/ `技能审计` `项目审查` `项目审计` / `skills-audit` | `> 技能审查 ./my-project` |
| **Claude Code** | `/skills-audit` | `> /skills-audit ./my-project` |
| **Codex** | `技能审查` / `skills-audit` | `> skills-audit ./my-project` |
| **Cursor** | `@skills-audit` | `> @skills-audit ./my-project` |

**Parameter**: Optional project path. Defaults to current working directory.

---

## How It Works / 工作流程

```
⓪ Load config → ① Scan project → ② List installed skills
③ Load reference maps → ④ 3-tier + 3-dim scoring + desc check
⑤ Web search + external signals → ⑥ Fill Community + final report
⑦ User confirms → Execute actions → ⑧ Persist data
```

---

## Config / 配置

Edit `config.yaml` (唯一配置源) to customize scan paths, project scan settings, and version check behavior.

See [SKILL.md](SKILL.md) for full reference.

---

## Installation / 安装

```bash
curl -sL https://raw.githubusercontent.com/gtbwpkwjnb-alt/skills-audit-skill/main/install.sh | bash
```

Windows PowerShell:
```powershell
iwr https://raw.githubusercontent.com/gtbwpkwjnb-alt/skills-audit-skill/main/install.ps1 | iex
```

---

## Changelog

### v4.3.0 (2026-06-22) — 全维度优化

- 🔄 **评分流程修正**：Community 维度步骤④暂用默认值，步骤⑤回填真实数据
- 📉 **步骤③精简**：移除冗余初筛，仅加载参考数据到内存
- 🎯 **Value 维度重构**：从固定类别分改为项目匹配加成（T1 +10）
- 📊 **健康度行重设计**：活跃数(含分层) + 建议归档数，均分仅计活跃技能
- 🧹 **边界规则去重**：15 条合并为 10 条
- 🗑️ **去重逻辑**：同 name 技能优先用户版→版本号高者
- ↩️ **归档/描述修改增加 undo**：复制归档+备份描述
- 🔍 **搜索模板明确化**：中英各一次、取前5、去重规则
- ⚡ **外部信号增量缓存**：首次全量获取，后续仅增量（GitHub 7天/npm 30天过期）
- 📝 **简评限制放宽**：≤12字 → ≤16字
- 📁 **project-types 扩展**：新增 Flutter/C++/Svelte/数据科学 4 种类型

### v4.2.0 (2026-06-20) — 触发词·语言·自检

- 🏷️ 触发词优化：`技能审查` 为主，`技能审计/项目审查/项目审计` 为同义词
- 🌐 output_language 三态驱动（auto/zh/en）
- 🔍 自检锚点：审计完成后必须检查自身 description
- ✏️ 描述优化询问 + 修改描述执行操作

### v4.0.0 (2026-06-19) — 项目技能推荐引擎

- 🎯 项目角色画像驱动 · 四维评分 · 5块永不塌缩报告
- 📁 项目扫描 + 15+ 项目类型模式
- 🌐 网络搜索备选技能

---

## Development Self-Test / 开发自测

每次修改后验证：

1. **项目扫描**：在 2 个不同类型的项目上运行，确认类型推断正确
2. **匹配测试**：已知项目类型→检查推荐列表是否合理
3. **分层测试**：添加不匹配技能，确认归入 T3 且标注建议归档
4. **持久化测试**：确认 `activity-log.jsonl` 可写且格式正确

## Feedback

🐛 [GitHub Issues](https://github.com/gtbwpkwjnb-alt/skills-audit-skill/issues/new)

## License

MIT
