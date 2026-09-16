---
name: french-interpretation-learning
description: "法语口译学习统一入口。封装已有的「译前准备(interpreter-terminology)」与「法语口译与视译训练(french-interpretation-training)」两个 Skill，提供译前准备、口译练习、视译练习、学习记录、数据导入导出等统一学习流程。"
---

# 法语口译学习（french-interpretation-learning）

一个**统一学习入口**。本 Skill 不重新实现任何专业功能，只做编排：

- 首次进入介绍
- 提供统一入口
- 收集参数
- 判断是否需要译前准备
- **调用**已有「译前准备 Skill」
- **调用**已有「口译与视译训练 Skill」
- 统一保存学习数据
- 学习数据导入 / 导出（统一 `.md`）

```
本 Skill
├── 用户入口
├── 参数收集
├── 流程判断（是否需译前准备）
├── Skill 调用
│     ├── → 已有「interpreter-terminology」（译前准备）
│     └── → 已有「french-interpretation-training」（口译/视译训练）
├── 学习记录
├── 数据导入
└── 数据导出（.md）
```

**不复制、不重写两个已有 Skill 的任何专业逻辑。**

---

## 0. 铁律

| # | 铁律 | 说明 |
|---|------|------|
| R1 | 不重造逻辑 | 译前准备与训练功能一律调用已有 Skill，不在此复制实现。 |
| R2 | 不虚构调用 | 调用已有 Skill 用真实标识（`interpreter-terminology`、`french-interpretation-training`）；无法调用时明说，不得假装调用或假装成功。 |
| R3 | 不伪造数据 | 不伪造学习记录、来源、链接、评分。 |
| R4 | 不假装保存 | 仅用当前环境真实支持的机制持久化；不支持时如实说明并提示用 `.md` 导出备份。 |
| R5 | 导入不重复 | 重复导入同一 `.md` 不产生重复记录；无法解析的内容不静默丢弃，明确告知。 |
| R6 | 只做本 Skill 该做的 | 只做入口/参数/判断/调用/记录/导入导出；专业训练细节交给两个依赖 Skill。 |

---

## 1. 首次进入

安装启动后先显示介绍：

> 这是一个法语口译学习 Skill，支持译前准备、口译练习和视译练习。
> 你可以先针对一个主题进行译前准备，也可以直接开始练习。对于专业性较强的练习主题，系统会在需要时自动调用译前准备功能。
> 所有学习结果都会保存，并支持导入和导出 Markdown 学习数据。

然后显示入口菜单：

- 译前准备
- 口译练习
- 视译练习
- 学习记录
- 导入学习数据
- 导出学习数据

> 这是 Agent Skill 的交互入口，**不是**独立 Web App / 独立工作台。

---

## 2. 入口：译前准备

收集：
- **主题**（自由输入：人工智能 / 量子计算 / 气候变化 / 国际贸易 / 欧洲能源政策…）
- **语言方向**：法→中 / 中→法 / 法↔中
- **可选要求**：CEFR 水平、是否首次接触、重点整理术语、口译或视译、是否重点介绍法国背景…

然后**直接调用已有「interpreter-terminology」**，不复制其逻辑。得到结果后由本 Skill 保存学习记录。

---

## 3. 入口：口译练习 / 视译练习

### 口译练习参数
- 语言方向：法→中 / 中→法
- 难度：A1–C2
- 主题：自定义 / 随机
- 材料：AI 搜索 / 上传材料

### 视译练习参数
- 语言方向：法→中 / 中→法
- 难度：A1–C2
- 主题：自定义 / 随机
- 材料：AI 搜索 / 上传材料

材料搜索、音频/视频、ASR、评分等专业训练功能由 **「french-interpretation-training」** 负责（详见第 4、5 节编排）。

---

## 4. 核心编排：判断是否需要译前准备

进入口译/视译练习前判断。

### 判断依据

1. **专业性**：主题属明显专业知识门槛领域 → 倾向需准备。如：医学、法律、金融、国际贸易、国际关系、AI、量子计算、能源、气候政策、航空、科技、经济、公共政策等。
2. **用户主动要求**：用户明确说"先做译前准备 / 我不了解这个主题 / 我不熟悉专业术语 / 先帮我了解背景" → **必须**调用译前准备。
3. **用户水平**：CEFR 较低 + 主题专业强 → 建议先译前准备。
4. **历史学习数据**：历史记录显示该主题反复出现术语/专名/背景/概念/信息遗漏错误 → 自动建议译前准备。

### 调用关系

```
本 Skill
  ↓ 判断需译前准备
调用「interpreter-terminology」
  ↓ 获得：主题背景 + 术语 + 双语表达 + 来源
作为本次训练上下文
  ↓
调用「french-interpretation-training」
  ↓
开始练习
```

### 用户提示（自然语言，不暴露技术细节）

> 这个主题涉及一些专业背景，我先帮你做一个简短的译前准备，再开始训练。

并给用户选择：
- **先做译前准备**
- **直接开始练习**

用户选"直接练习" → **不调用**译前准备 Skill。

---

## 5. Skill 调用（真实标识）

- 已有「译前准备 Skill」ID：`interpreter-terminology`
- 已有「口译与视译训练 Skill」ID：`french-interpretation-training`

通过宿主实际支持的 Skill 发现 / invocation 机制调用（Skill tool、manifest、dependency 等由宿主决定）。调用失败或目标 Skill 在本环境不可用 → 明确告知用户，不虚构成功（R2）。

---

## 6. 学习数据统一保存

所有行为（译前准备 / 口译 / 视译）都保存一条记录。字段与 YAML 结构见 references/data-and-io.md。核心字段：

```yaml
id:
created_at:
type:            # preparation | interpreting | sight_translation
topic:
language_direction:
cefr:
title:
source:
source_url:
prep_required:
prep_called:
prep_record_id:
material:
original_asr:
corrected_transcript:
score:
score_breakdown:
strengths:
errors:
suggestions:
reference_normal:
reference_advanced:
notes:
tags:
```

**译前准备数据**：若已调用过「interpreter-terminology」，不整份复制其内容，优先保存 `prep_called` + `prep_record_id`（引用）。仅当宿主无法跨 Skill 引用记录时，按实际能力决定是否存必要快照。

---

## 7. 学习记录

显示已完成记录，支持按条件筛选：日期 / 类型 / 主题 / 语言方向 / CEFR。每条可查看完整内容。

---

## 8. 数据导出（统一 `.md`）

导出格式见 references/data-and-io.md。支持：

- **导出当前记录**：如 `2026-09-16_人工智能_B2_口译.md`
- **导出全部记录**：如 `法语口译学习记录.md`
- **导出筛选后记录**：如 `B2口译学习记录.md`

Markdown 需含足够完整数据，便于之后重新导入。

---

## 9. 数据导入（`.md`）

导入流程：
1. 解析 Markdown
2. 识别学习记录（按记录标记切分 + YAML frontmatter 解析）
3. 校验字段
4. 检查重复（按 `id` 与内容指纹）
5. 合并已有数据

**去重**：同一文件重复导入不产生重复记录；跨文件同 `id` 也按去重处理（以已有记录为准，保留用户改动）。
**部分不可解析**：不静默丢弃，明确告知哪些内容解析失败。

---

## 10. 数据持久化

用宿主当前真实支持的持久化机制（文件系统 / 本地存储等）。不得假装保存（R4）。

- 支持持久化 → 落盘并记录路径。
- 不支持长期持久化 → 明确说明限制，并通过 `.md` 导出提供长期备份/迁移方式。

---

## 11. 规则索引

| 主题 | 文件 |
|---|---|
| 编排决策（是否需要译前准备、调用链、用户提示） | references/orchestration.md |
| 数据结构 + Markdown 导入/导出格式 + 去重 + 持久化 | references/data-and-io.md |
| 单条学习记录导出模板 | templates/learning-record.md |
| 端到端流程用例（A–F）与自检 | examples/example-workflow.md |

---

## 12. 边界

本 Skill 不是新的译前准备系统，也不是新的口译训练系统；只是**把两个已有 Skill 封装成统一用户入口与学习流程**。禁止创建第三套重复的专业训练逻辑（R1/R6）。