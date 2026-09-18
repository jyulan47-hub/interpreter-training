# 数据与 I/O（data-and-io.md）：结构、Markdown 格式、去重、持久化

## 1. 学习记录统一结构（YAML 字段）

每次行为（译前准备 / 口译 / 视译）保存一条：

```yaml
id: 20260916-001
created_at: 2026-09-16T09:00:00+08:00
type: preparation            # preparation | interpreting | sight_translation
topic: 人工智能
language_direction: fr→zh   # fr→zh | zh→fr | fr↔zh
cefr: B2                    # 仅训练类有；准备类可为 null
title:                       # 材料/主题标题
source:                      # 来源机构/平台
source_url:
prep_required: true
prep_called: true
prep_record_id: 20260916-000   # 已调用译前准备时引用其记录，不复制全文
material:                      # 材料元信息或引用（见口译训练 Skill）
original_asr:
corrected_transcript:
score:                         # 0-100；无评分则 null
score_breakdown: {}
strengths: []
errors: []
suggestions: []
reference_normal:
reference_advanced:
notes:
tags: []
```

规则：
- `original_asr` 与 `corrected_transcript` 分开保存；评分以 corrected 为准。
- 调用过译前准备：存 `prep_called: true` + `prep_record_id`，**不整份复制**译前准备内容。仅当宿主无法跨 Skill 引用时，按能力存必要快照。

## 2. 单条记录导出模板（.md）

一个文件一条记录，使用：
- 顶部 Markdown 标题
- YAML frontmatter（`---` 包裹）承载全部可机读字段
- 正文为人类可读的完整内容（含背景/术语/材料/反馈/参考译文）

模板见 `templates/learning-record.md`。每条记录的**机读字段真值以 YAML frontmatter 为准**，保证可再导入。

## 3. 多记录 / 全部 / 筛选导出（.md）

一个文件含多条记录时，用明确的记录标记切分，便于机器解析：

```markdown
# 法语口译学习记录
> 导出来源：法语口译学习 Skill
> 导出时间：…
> 记录数：N

<!-- RECORD BEGIN id=20260916-000 -->
---
id: 20260916-000
type: preparation
…
---
## 正文…完整内容…
<!-- RECORD END -->

<!-- RECORD BEGIN id=20260916-001 -->
---
id: 20260916-001
…
---
## 正文…
<!-- RECORD END -->
```

文件名建议：
- 当前记录：`2026-09-16_人工智能_B2_口译.md`
- 全部记录：`法语口译学习记录.md`
- 筛选后记录：`B2口译学习记录.md`

> Markdown 须含足够完整数据，使之后可重新导入（完整字段 + 正文）。

## 4. 导入（.md）流程

1. **切分**：按 `<!-- RECORD BEGIN id=… -->` … `<!-- RECORD END -->` 切分记录。
2. **解析**：读取每条 frontmatter 为 YAML；缺失 frontmatter 的记录尝试从正文提取，取不到的字段留空。
3. **校验**：必需字段 `id / created_at / type` 缺失 → 判为"不可解析记录"。
4. **去重**：
   - 先按 `id` 查重：已存在 → 跳过（保留已有记录，不覆盖用户改动）。
   - 无 `id` 或冲突，用"内容指纹"（frontmatter+正文 归一化后哈希）判断重复。
   - 同一文件重复导入、跨文件同 `id` 均不产生重复记录。
5. **合并**：新记录追加；已存在记录不动。
6. **报告**：明确告知成功导入 N 条、因重复跳过 M 条、无法解析 K 条及各自原因。**不静默丢弃**。

## 5. 持久化

- 优先使用宿主真实支持的持久化（文件系统 / 数据目录 / 本地存储）。
- 不支持长期持久化 → 向用户明示当前为临时存储及风险，并引导用 `.md` 导出做长期备份/迁移。
- 绝不假装保存成功。

### 5.1 固定目录与命名（R7）

Skill 数据根目录下建立两个文件夹：

| 文件夹 | 内容 | 命名 |
|---|---|---|
| `译前准备/` | 专业术语数据（主题背景/术语表/双语表达/来源），一次译前准备一份 | `{主题}_{日期}.md` 例：`人工智能_2026-09-18.md` |
| `练习数据/` | 一份练习数据 = 素材（视频链接）+ 用户翻译（ASR/修正稿）+ 评分 + 修正 + 提升建议 | `{素材名}_{日期}.md` 例：`EU_AI_Act_10min_2026-09-18.md` |

规则：
- 术语数据源自 `interpreter-terminology` 结果整理；练习数据在调用 `french-interpretation-training` 后整理。
- 日期取当次创建日 `YYYY-MM-DD`。
- **覆盖保护**：同名已存在 → 追加序号（`_2`、`_3`…）或询问用户，不直接覆盖（R7）。
- 不得混放；宿主不支持目录持久化时如实说明，并按同样的两套结构产出 `.md` 留给用户手动保存。