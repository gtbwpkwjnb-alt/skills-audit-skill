---
name: skills-audit
version: "4.3.0"
description: 技能审查 → 画像驱动·四维评分·规划追踪
user-invocable: true
---

# Skills Audit v4.3.0 — 画像驱动 · 四维评分 · 描述精炼 · 5块报告

> **四大核心能力**: ①项目角色画像驱动 ②四维评分(含外部⭐+描述质量) ③自动精炼技能描述为触发词→功能简介 ④5块永不塌缩报告。
>
> **核心原则**: 画像驱动一切。有分才有据。描述必查。报告从不塌。

---

## 触发方式

**ZCode**: `技能审查`（主）/ `技能审计` `项目审查` `项目审计` / `skills-audit` · **Claude Code**: `/skills-audit` · **Codex/Cursor**: `技能审查` / `skills-audit`

**参数**: 项目路径可选。不传则用当前工作目录。
**触发规则**: 独立词触发。嵌入句子中不触发（如"帮我做一下技能审查"不触发）。

---

## 执行流程

### ⓪ 加载配置

读取同目录下 `config.yaml`，确定：
- `scan_paths` — 要扫描的技能/工具存放目录
- `project_scan` — 项目扫描配置（忽略模式、最大深度等）
- `version_check` — 版本检查源 URL
- `data_dir` — 持久数据目录
- `external_signals_cache` — 外部信号缓存 TTL 配置

然后读取 `user-profile.md`（含用户高频技能偏好），用于个性化 T2/T3 判定和推荐。
然后检查 `config.yaml` 与 `platforms/*.yaml` 的一致性（trigger_words / scan_paths 是否冲突），不一致则输出警告。

> references 文件（project-types.yaml / skill-registry.yaml）由步骤③加载。

### ① 项目扫描

扫描用户指定的项目目录（或当前工作目录），提取：

**基础信息:**
- 项目名称（从目录名或 `package.json` `name` 字段）
- 项目根路径

**文件特征（尊重 `.gitignore`，不扫描 node_modules/.git/.venv 等）:**
- 语言文件：`.py`, `.js`, `.ts`, `.rs`, `.go`, `.java`, `.cs`, `.rb`, `.c`, `.cpp` 等
- 配置文件：`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle`, `CMakeLists.txt`, `.csproj`, `Gemfile`, `Dockerfile`, `docker-compose.yml`, `.github/`, `.gitlab-ci.yml`
- 框架标识：在 `package.json` dependencies 中查找 React/Vue/Angular/Express/Nest/Next/Nuxt 等
- 测试文件：`*.test.*`, `*.spec.*`, `__tests__/`, `test/`

**推断项目类型并构建项目角色画像：**

依次匹配 `references/project-types.yaml`。然后根据文件特征充实为**项目角色画像**（这是后续所有分析的核心锚点）：

```
┌─ 项目角色画像 ─────────────────────────────────┐
│ 类型    <从 project-types.yaml 匹配>              │
│ 阶段    <从文件特征推断：立项/开发/测试/部署/维护>  │
│ 活动    <从文件特征推断的 3-5 个核心开发活动>      │
│ 产物    <主要文件类型和输出格式>                   │
│ 痛点    <从项目结构推断的 2-3 个潜在效率瓶颈>      │
└───────────────────────────────────────────────┘
```

画像示例（React 项目）：
```
类型   Web前端 (React + TypeScript)
活动   组件开发 · 状态管理 · 路由配置 · 构建优化 · 测试编写
产物   .tsx · .css · .test.tsx · dist/
痛点   状态管理复杂度 · 组件拆分粒度 · SSR 调试
```

画像示例（AI Agent 技能开发项目）：
```
类型   AI Agent 工作区配置 / 技能管理
活动   技能编写 · Agent 编排 · 工具链优化 · MCP 集成 · 规则维护
产物   SKILL.md · AGENTS.md · YAML · .mjs · 安装脚本
痛点   跨项目技能复用 · 版本追踪 · 冗余清理 · 新技能发现
```

> **画像即锚点**：后续所有步骤（分层、评分、搜索）都以此为尺度。

### ② 已安装技能清单

扫描 `config.yaml` 中所有 `scan_paths`，提取每个技能的：
- `name`、`description`、路径、来源（user/system）、版本号（从 SKILL.md frontmatter 或 VERSION 文件读取）、可编辑性

**分类**：
- **用户技能**（editable=true，用户可自行增删改）
- **系统/插件技能**（editable=false，来自插件缓存）

**输出:**
```
🔍 已安装技能: 用户 2 个 + 系统/插件 17 个
```

**去重**：同 `name` 的技能保留一个——优先 editable=true（用户版）→ 同 editable 则选版本号高者。去重数在输出中标注（如"去重 2 个"）。

### ③ 加载参考数据

从 disk 读取 `references/project-types.yaml`（项目类型→推荐技能映射）和 `references/skill-registry.yaml`（已知技能版本号+分类），加载到内存供步骤④使用。

> 找不到 references 文件时报 warning，继续执行（降级为仅靠内置默认规则+web 搜索推荐）。
> 此步不做任何筛选判定，仅为后续步骤准备数据。

### ④ 三级分层 + 四维评分 + 描述检查

**第一步：三级分层（以项目角色画像的"活动"为尺子）。**

| 层级 | 规则 | 动作 |
|------|------|------|
| **T1 核心** | 技能的主要用途直接命中"核心活动"中的某一项 | 保留（自动） |
| **T2 通用** | 技能不局限于特定项目类型，当前项目可用，无负面影响 | 保留 |
| **T3 专业** | 技能为特定项目类型设计，与"核心活动"零交集 | 建议归档 |

> 判定密钥：把技能的 description 和项目画像的"活动"做交集。有交集 → T1；无交集但通用类别(debug/docs/review/管理/验证/设计等) → T2；明确服务其他类型(Android/iOS/特定迁移) → T3。
> T1 技能之间做功能重叠检测：description 关键词相似度 >60% 则标 ⚠️ 建议二选一（如 subagent-driven-development vs dispatching-parallel-agents）。

**第二步：四维评分（仅已安装技能，每技能 0-100）。Community 维度暂用默认值 25，步骤⑤获取真实数据后回填。**

| 维度 | 权重 | 计算方式 |
|------|------|---------|
| **Fit** 项目匹配 | 40% | T1 → 85–100；T2 → 50–84；T3 → 0–49。与"核心活动"重合项数每多 1 项 +3，无重合 −5 |
| **Value** 项目价值+描述质量 | 20% | ①项目匹配加成：T1 +10，T2 持平，T3 −10。②类别微调（±5）：debug/验证/审查 +5；文档/笔记/管理 持平；设计/规划 −0；纯平台工具 −5。③使用频率（读取 `usage-*.json`）：高频调用 +5；低频 −5；零调用 −10。④描述质量分：中文且格式合格 → +10；仅英文 → −10；超长/多行 → −10 |
| **Fresh** 版本时效 | 20% | 版本 = registry 最新 → 100；落后 1 大版本 → 60；registry 无记录 → 50；落后 2+ → 30；无版本号 → 40 |
| **Community** 外部信号 | 20% | 步骤⑤获取前暂用默认值（无公开 repo → 25）。步骤⑤回填规则：GitHub ⭐≥1000→100，≥100→80，≥10→60，<10→30；npm 📥≥1000/w→100，≥100/w→70，<100→40；最近更新 1 月内→100，6 月内→70，1 年内→50，>1 年→20 |

**总分 = Fit×0.4 + Value×0.2 + Fresh×0.2 + Community×0.2**

分数 → 建议：≥70 ✅保留 · 40–69 🟡保留(低频) · <40 ❌建议归档

> Community 维度在步骤④先用默认值，步骤⑤获取真实外部信号后回填到评分表。分层（T1/T2/T3）不受 Community 分影响。

**第三步：描述质量检查（每个已安装的用户可编辑技能）。**

检查项与优化规则：

| 检查项 | 规则 | 不合格的处理 |
|--------|------|-------------|
| 语言 | 根据 `output_language` 设置决定（见下表） | 自动生成目标语言版本 |
| 格式 | 必须遵循 `触发词 → 功能简介` | 自动重排为规范格式 |
| 长度 | 单行，≤40 字符 | 截断并保留核心语义 |
| 双语 | `auto` 模式建议双语，其他模式尊重设置 | 单语也接受，不扣分 |

**语言判定规则（根据 `output_language`）：**

| output_language | 行为 |
|-----------------|------|
| `auto`（默认） | 保持原文语言。纯英文→建议同时提供中文版（`\|` 分隔双语）；纯中文→建议同时提供英文版 |
| `zh` | 中文优先。纯英文→生成中文优化版 |
| `en` | 英文优先。纯中文→生成英文优化版 |

**精炼生成规则（不合格时自动生成优化版）：**

| 步骤 | 规则 | 示例 |
|------|------|------|
| ① 提取触发词 | 从技能 name/frontmatter 取最常用中文调用名，无中文则用英文名 | `summarize` → `总结`；`brainstorming` → `头脑风暴` |
| ② 构建简介 | `动词+宾语` 结构，2-4 个核心功能用 `·` 分隔。动词优先：创建/编辑/分析/提取/搜索/审查/追踪/管理 | `会话精炼·进度追踪·自动迭代` |
| ③ 截断 | 超 40 字符时优先级：触发词 > 核心动词 > 宾语 > 修饰词。逐词删除直到 ≤40 | `完整的 DOCX 文档创建、编辑与分析能力…` → `DOCX 创建·编辑·分析` |
| ④ 翻译 | 意译优先，功能对等 > 字面翻译；保持目标语言 ≤40 字符 | `Use when...` → `需求澄清·创意发散·方案对比` |
| ⑤ 双语拼接 | `output_language` 的主语言在前，` \| ` 分隔，两侧各自 ≤40 字符 | `技能审查 → 画像驱动·四维评分·规划追踪 \| Skill Audit → profile-driven scoring` |

> 系统/插件技能（editable=false）只标注问题，不自动修改。
> 审计完成后必须回头检查 skills-audit 自身的 description 是否符合以上规范，不符合则自动修正（解决自指盲区）。

**评分表行格式（已安装技能）：**
```
#N  技能名           分  层级  ⭐外部信号  简评(≤16字)
```

示例：
```
#1  skill-creator    89  T1    内置·官方   技能创作核心
#5  sub-agents       78  T1    ⭐44·6月新  跨LLM子代理
```

### ⑤a 外部信号获取（缓存优先）

从 `external-signals-cache.json` 加载缓存（首次运行为空）。

对每个已安装且 registry 中有 homepage 的技能，判定：
- **有缓存且未过期** → 直接用缓存值（GitHub ⭐ 缓存 7 天，npm 📥 缓存 30 天）
- **无缓存或已过期** → WebFetch 获取后写入缓存，标注 `fetched_at` 时间戳
- **网络不可用** → 用缓存值（忽略过期），无缓存则用默认值 25

```
GitHub: WebFetch api.github.com/repos/{owner}/{repo}
        → stargazers_count, pushed_at, description
npm:    WebFetch api.npmjs.org/{package}/latest
        → version, weekly downloads
PyPI:   WebFetch pypi.org/pypi/{package}/json
        → info.version, downloads
```

> 同一 GitHub org 下多个 repo 合并为单次 API 请求。缓存写入 `external-signals-cache.json`。
> 首次运行全量获取较慢，后续仅增量获取新技能或过期条目，大幅提速。
> 获取完成后将真实 Community 分回填到步骤④的评分表。

### ⑤b 画像驱动的网络搜索

**搜索关键词从画像提取：**
```
从画像.活动 提取英文关键词 → 组合为搜索串
从画像.痛点 提取英文关键词 → 补充为第二搜索串
示例: "skill authoring, agent orchestration"
     → "AI coding agent skill management tool 2026"
```

**搜索执行规则：**
- 中英文各 1 次搜索（从画像.活动提取 3 个英文 + 3 个中文关键词）
- 每次取前 5 条结果
- 同名技能去重（保留最高分来源）
- 与已安装技能冲突时：优先展示已安装版本，网络结果标 🆕 仅作备选参考

**结果格式：**

网络推荐的备选技能（未安装）→ 单独 `🌐 网络推荐` 区域：
```
🆕 [来源] · 技能名  ⭐X · 更新状态
    <≤16字描述> · 当前项目适配理由
```

对外部技能的描述同样执行描述质量检查，不符合规范的在推荐时就给出优化版。

### ⑥ 生成报告

报告固定 5 块，每块必定输出。禁止宽表格。

```
════════════════════════════════════════════════
📋 项目角色画像
════════════════════════════════════════════════
类型   <画像.类型>
阶段   <画像.阶段>
活动   <画像.活动>
产物   <画像.产物>
痛点   <画像.痛点>
════════════════════════════════════════════════
已安装 N个 · 活跃 M (T1:X T2:Y) · 建议归档 Z (T3)
活跃均分 XX · Community: 缓存命中 X/Y · 实时获取 Z
📈 对比上次审计：均分 ±N · 新增活跃 K · 归档建议 ±P
════════════════════════════════════════════════

📊 已安装技能评分（仅已安装，按分降序）
#  技能              分  层级  ⭐外部信号    简评
 1 skill-creator    89  T1   内置·官方     技能创作核心
 2 summarize        75  T2   本地·无repo   会话精炼，日常高频
 3 docx             65  T2   内置·官方     文档工具，偶尔用
 ...
22 android-dev      36  T3   内置·官方     专为Android，无交集
════════════════════════════════════════════════

📝 描述优化（N个）—仅用户可编辑技能
#  技能              问题              建议
 1 brainstorming     纯英文，格式不符   头脑风暴 → 需求澄清·创意发散·方案对比
 2 systematic-debug   纯英文             系统化调试 → 假设驱动·二分定位·根因分析
 ...
── 描述全部合格，无需优化 ── （无不合格项时的兜底）

──────────────────────────────────────────
✏️ 是否更新以上描述？ [回复"是"全部更新，或回复编号如"1 3"指定更新]
──────────────────────────────────────────
════════════════════════════════════════════════

🌐 网络推荐备选（未安装，供评估）
🆕 AllAgents (npm)       v1.11.9 · 📥577/w   跨23客户端技能同步
🆕 Mega-Mind (pypi)      v0.4.0 · ⭐?          42技能+自进化工作流
 ...
── 本次未搜索到推荐备选 ── （无结果时的兜底）
════════════════════════════════════════════════

📌 操作
[1] 归档 android-dev       T3/36分  专为Android，无交集
[2] 优化描述 brainstorming   纯英文→中文化
[3] 评估 🆕 AllAgents        跨平台技能管理
 ...
── ✅ 无需调整，当前搭配合理 ──

💡 建议搭配（T1 技能间的协同关系）
  TDD + systematic-debugging → 先写测试再调试，互补增效
  TDD + verification-before-completion → 测试+验证双保险
  skill-creator + summarize → 创作+精炼闭环
════════════════════════════════════════════════
```

**规则**：
- 画像+健康度 → 必定输出。
- 📊 评分表 → 仅已安装技能，按分降序。每条含外部信号(⭐/📥/更新状态)。未安装的不在此列。
- 📝 描述优化 → 必定输出。检查语言/格式/长度。无不合格时显示兜底文本。
- 🌐 网络推荐 → 必定输出。未安装的外部推荐，每条 ≤1 行。无结果时显示兜底。
- 📌 操作 → 必定输出。有序号 [1][2]...，操作可以是归档/优化描述/安装/关注。每条含 `🔍 原因:` `💡 效果:` `🎯 场景:` 三段（≤1行/段，≤40字/句）。保留项不加三段解释。禁止空泛词。
- 💡 建议搭配 → T1 技能间的协同关系（3-5 组），标注互补原因。

### ⑦ 执行（用户确认后）

**操作范围**：仅用户层技能（editable=true）。系统/插件层技能（editable=false）只读——可评分、标注、建议，禁止任何 Edit/Write/Bash 修改操作。

- **归档**：技能目录**复制**到 `config.yaml` 的 `archive_dir`，原路径重命名为 `{原名}.archived`（非删除，可手动恢复）
- **修改描述**：① 同目录创建 `.description.bak` 备份文件 ② Edit 对应技能 SKILL.md frontmatter 的 `description:` 行，替换为优化版描述
- **安装**：给出安装命令示例（clone / 下载 / 安装脚本）
- **更新**：给出更新方式
- **搜索安装**：给出来源 URL
- **回滚**：报告末尾附回滚命令——`还原描述: cp/mv (Linux) 或 copy/move (PowerShell)` / `还原归档: mv {name}.archived → {name} (Linux) 或 move (PowerShell)`

### ⑧ 持久化（始终执行）

1. 项目画像 → `project-profiles.json`（以项目 hash 为 key，含阶段+技能使用历史字段，覆盖不追加）
2. 运行统计 → `stats.json`（仅存汇总统计，不含每条明细）
3. 审计日志 → `activity-log.jsonl`（保留最近 50 条，超出自动截断。每条记录含本次审计评分+各技能调用次数（从 `usage-*.json` 读取））
4. 外部信号缓存 → `external-signals-cache.json`（以技能 name 为 key，含 fetched_at 时间戳。仅供下次运行增量获取用）
5. 审计快照 → 上次审计数据从 `activity-log.jsonl` 倒数第 2 条读取，写入报告生成 `📈 对比上次审计` 行

**持久化失败不阻塞主流程**。

---

## 项目类型识别规则

详见 `references/project-types.yaml`。核心规则：

| 特征 | 推断类型 | 推荐优先级 |
|------|---------|-----------|
| package.json 含 react/react-dom | Web前端-React | 高 |
| package.json 含 vue/vue-router | Web前端-Vue | 高 |
| package.json 含 @angular/core | Web前端-Angular | 高 |
| package.json 含 express/koa/fastify | Node后端 | 高 |
| package.json 含 next | Web全栈-Next.js | 高 |
| Cargo.toml | Rust | 中 |
| pyproject.toml / requirements.txt | Python | 中 |
| go.mod | Go | 中 |
| pom.xml / build.gradle | Java | 中 |
| SKILL.md (多文件) | AI Agent 技能开发 | 高 |
| .csproj | C#/.NET | 中 |
| Gemfile | Ruby | 中 |
| pubspec.yaml 含 flutter | Flutter | 高 |
| CMakeLists.txt | C/C++ | 中 |
| package.json 含 svelte | Web前端-Svelte | 高 |
| requirements.txt 含 pandas/numpy | 数据科学 | 高 |
| Dockerfile | 容器化 | 低（辅助） |

多特征叠加时合并推断（如 package.json 含 react + express → "全栈: React+Express"）。

---

## config.yaml 格式

完整配置见同目录 `config.yaml` 文件。核心结构：

```yaml
# skills-audit v4.3 配置
scan_paths: [...]            # 技能扫描路径（用户+系统/插件）
project_scan: { max_depth, max_files, ignore_patterns }
version_check: { enabled, registry_url }
output_language: "auto"      # auto / zh / en
external_signals_cache: { github_ttl_days: 7, npm_ttl_days: 30 }
```

---

## 边界规则

- **触发**：仅独立打出触发词（`技能审查`/`技能审计`/`项目审查`/`项目审计`/`skills-audit`）；嵌入句子不触发
- **路径**：支持绝对路径和相对路径；路径不存在时报错终止
- **分层**：T1/T2/T3 以项目角色画像"核心活动"为唯一尺度，禁止退回全局/领域二分
- **评分**：每个已安装技能必须有分数+层级+外部信号+简评（≤16字）；Community 维度步骤⑤回填；未安装技能不进评分表
- **描述**：每个已安装技能 description 检查语言/格式/长度，不合格自动生成优化版；系统/插件技能只标注不修改
- **报告5块**：画像·评分·描述优化·网络推荐·操作，每块必定输出，空区兜底；已安装技能不进网络推荐区
- **外部信号**：首次全量获取 GitHub⭐/npm📥/更新时间，后续增量（GitHub 7天/npm 30天 TTL）；缓存命中直接使用；网络不可用忽略 TTL 用缓存兜底，无缓存用默认值 25
- **确认执行**：所有归档/安装/更新/描述修改操作必须用户确认后执行（步骤⑦）；系统/插件层技能（editable=false）仅只读——可评分、标注、建议，禁止任何修改操作
- **降级**：Web 不可用跳过步骤⑤；references 缺失用内置默认规则（Node.js→TDD+debug / Python→debug / Rust→debug / AI Skill→skill-creator）；持久化失败不阻塞主流程
- **输出**：禁用 >60 字符宽表格；评分表每条 ≤1 行；同类栏目不分散

## 平台配置

参见 `platforms/zcode.yaml` 和 `platforms/default.yaml`（多平台支持，ZCode 和通用回退）。
