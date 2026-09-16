# interpreter-terminology —— 口译术语研究与记忆训练（Agent Skill）

一个供 AI Agent 调用的专业 Skill：**口译术语研究 → 权威来源核验 → 个人术语库积累 → 记忆训练 → 场景口译 → 错题复习**。

- **不是网页应用**，不提供固定 UI / 前端 / 独立 App；只输出指令与结构化数据，由宿主 Agent 执行。
- **不强制中英法三语**：语言方向完全由用户的实际口译方向决定，字段动态配置。
- **严格反幻觉**：找不到可靠来源时标记 `ai-candidate`，从不为编造来源背书。

---

## 1. 目录结构

```
interpreter-terminology/
├── SKILL.md                     # 入口：触发条件、铁律、语言方向、工作流、工具调用
├── README.md                    # 本文件：安装与跨平台适配
│
├── references/
│   ├── terminology-sources.md   # 权威来源清单、检索优先级、来源记录字段
│   ├── verification-rules.md    # 反幻觉、核验状态、冲突处理、真实语料分离、来源一一对应
│   ├── topic-overview.md        # 主题知识概览：内容模块 + 逐条来源核验
│   ├── term-schema.md           # 术语卡片 JSON schema（动态语言/版本/mastery）
│   ├── term-library.md          # 个人术语库：检索/增改/导入导出/合并去重
│   └── testing-rules.md         # 测试题型、口译评价、掌握度、智能复习
│
├── templates/
│   └── terminology-report.md    # 术语研究/核验报告 与 记忆测试报告 模板
│
└── examples/
    └── example-workflow.md      # 完整流程示例 + 6 个验收用例 + 可移植性自检
```

> 设计原则：**不为凑结构而建文件。** 每个文件承担一类清晰职责；规则不重复、可执行。SKILL.md 只做编排，细节下沉到 references。

---

## 2. 核心闭环（长期使用）

```
研究 → 核验 → 入库 → 修改 → 积累 → 测试 → 发现薄弱项 → 复习 → 再次口译 → 继续积累
```

核心承诺：
- 先查个人术语库，命中优先展示你的个人版本；
- 你的任何修改永不被打字机自动覆盖；
- AI 候选 / 官方译法 / 真实语料三者界限分明，绝不混淆；
- 记忆测试针对薄弱项，而非随机抽题。

**研究交付结构**（先懂主题、再学表达，来源逐条可点）：
```
主题知识概览 → 核心术语 → 易混淆术语 → 高频搭配/表达 → 真实语料 → 来源
```
- 主题概览每个事实/数据/政策/事件紧跟在来源链接；每个术语旁直接给可点击原始来源，支持多个来源。
- 结构为"具体内容 → 对应来源 → 点击即可核验"，**不是**"大量内容 → 底部统一列一堆来源"。

## 3. 触发方式

直接向 Agent 用自然语言提出即可，例如：
- "帮我准备这个主题的口译术语"
- "研究这个会议涉及哪些专业词汇"
- "查这个词在法语里的专业译法" / "检查这个译法是否权威"
- "加入我的术语库" / "搜索我的术语库"
- "测试我今天的术语" / "只测试我容易错的词"
- "根据我的术语库做一套中法口译模拟"
- "重新核验这些术语" / "导出这个主题的术语表"

---

## 4. 如何调用外部能力

本 Skill **不内置** Web Search / 数据库 / 文件系统 / 浏览器。它只做两件事：
1. 判断某一步该调用宿主的哪种能力；
2. 处理宿主返回的结果（入库、核验、生成报告）。

宿主有 Web Search / Browser / PDF / 文件系统 / 文档表格 → 合理调用；没有 → **明确告知"本环境无法执行 X"**，不假装。（详见 SKILL.md §5）

---

## 5. 如何维护个人术语库

- 采用机器可读结构（默认 JSON，可按宿主能力用 CSV/MD/TXT，宿主支持时可用 Excel）。
- 研究前先查库；新增去重合并；保留来源；`ai_version`/`user_version` 并存，`user_version` 永不覆盖。
- 支持导入导出，导入时检测重复并合并、保护个人版本。（references/term-library.md）

---

## 6. 如何做记忆测试与场景口译

- 8 种题型（源→目标、目标→源、定义↔术语、完形、搭配、易混淆、场景）。
- 12 项前提：只按词条的方向与语言出题（不掺第三语）。
- 8 维口译评价；掌握度 0–100 六档；结构化错误记录；智能复习按"低掌握度＋近期错误＋高频＋易混淆＋近期任务相关"抽取。（references/testing-rules.md）

---

## 7. 安装 / 加载到各宿主

本 Skill 是一个**可整体拷贝的文件夹**，目录名 `interpreter-terminology` 即 Skill 名。

| 宿主 | 放置位置 | 说明 |
|---|---|---|
| **Claude Code** | 项目内 `~/.claude/skills/` 或 project 的 `.claude/skills/interpreter-terminology/` | 标准 Agent Skills 格式 |
| **Codex / OpenAI env** | 按宿主 skills 目录约定放置（通常 `skills/interpreter-terminology/`） | SKILL.md frontmatter 兼容 |
| **Trae** | 项目内 `.trae/skills/interpreter-terminology/` | 本目录已按此放置 |
| **WorkBuddy / 其他** | 读取该宿主的技能/skill 加载路径，放入同名目录 | 见下"适配" |

**适配方式**：
- 核心入口 `SKILL.md` 采用跨平台的 YAML frontmatter（`name` + `description`），绝大多数 Agent Skill/技能机制可直接读取。
- 若某平台要求额外字段（如 frontmatter 需加 `version` / `metadata` / `allowed-tools`），在拷贝后按该平台惯例补充即可，**不影响正文逻辑**。
- references / templates / examples 为纯 Markdown 说明文件，与格式无关，可原样携带。

---

## 8. 版本与变更

**v1.1（本次升级）**
- 新增**主题知识概览**：交付前先给出该领域背景知识的检索式总结（背景/现状/核心议题/机构政策/口译重点）。
- 交付结构调整为：`主题知识概览 → 核心术语 → 易混淆术语 → 高频搭配/表达 → 真实语料 → 来源`。
- 来源**与内容一一对应**：概览每条事实、每个术语的权威注记，内联可点击原始链接（可多个），不走文末统一列表；无可靠来源标"未核验/AI生成"。
- 更新 references/topic-overview.md（新增）、verification-rules.md §3bis、terminology-report.md、SKILL.md 工作流与 description、example-workflow.md（新增 Case 7 与检查项）。

---

## 9. 验证说明（本环境）

- **已实际创建**并在当前 Trae 工程放置于 `.trae/skills/interpreter-terminology/`，符合宿主加载结构。
- **逻辑级自检**：已完成 R1–R8 铁律对照、可移植性检查项（见 examples/example-workflow.md 第三节）。
- **安装后是否立即可被调用**：取决于当前 Trae 会话是否在启动时扫描 `SKILL.md`。若需立即生效，通常需重启/重开会话让宿主重新加载技能清单。若本环境未提供自动化"技能加载验证"入口，请以新建会话并触发一次"帮我准备某主题口译术语"来实测。
- 验收用例见 examples/example-workflow.md，可用于在每个宿主上人工验证。

---

## 10. 已知边界

- 完全依赖宿主能力，**无联网时不做在线核验**。
- 不存储任何密文 / API Key / 用户隐私于 Skill 本身（隐私原则）。
- 不绑定任何单一厂商产品或数据库。