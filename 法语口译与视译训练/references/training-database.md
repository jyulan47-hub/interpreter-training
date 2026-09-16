# 训练数据库与历史复盘（training-database.md）

## 1. 单次训练记录结构

```text
training_id               # 唯一 ID（如 YYYYMMDD-序号）
date
mode                      # interpreting / sight_translation
direction                 # fr→zh / zh→fr
CEFR_level                # A1..C2
topic
title
source
source_url
duration
material                  # 材料元数据（见 materials.md）或指向材料引用
original_transcript       # 原材料文字稿（若可得）
recording_file            # 录音文件路径（未持久化则记 null + 说明）
original_asr
corrected_transcript      # USER_CORRECTED_TRANSCRIPT
score
score_breakdown
strengths
errors
suggestions
reference_translation_normal
reference_translation_advanced
```

存为机器可读格式（JSON 为宜，宿主支持再补文档/表格展示）。

## 2. Personal Error Bank（个人错误库）

自动归纳用户长期反复出现的问题，用于个性化反馈与复盘，例如：

- 漏译
- 数字错误
- 时间错误
- 因果关系错误
- 转折关系错误
- 术语错误
- 法语介词
- 法语时态
- 中文欧化
- 法语逐字翻译
- 不会重组长句
- 信息压缩能力不足

**归纳规则**：以出现频率与影响程度累积；后续评分时结合历史错误给出个性化提示。**不覆盖既有归纳**（R7），只增量更新。

## 3. 记录 / 复盘能力

| 能力 | 说明 |
|---|---|
| save_training_record | 持久化单次记录 |
| review_training_history | 按日期/模式/主题/方向读取记录 |
| analyze_personal_errors | 汇总高频/顽固问题 |
| compare_attempts | 同一材料多次训练的对比（上/本次分数、提升项、遗留问题、新问题、表达进步） |

## 4. 持久化约束

- 环境支持持久化 → 落盘并关联录音/转写。
- 不支持 → **明示"无法持久化训练记录/录音"**（R3/R8），仅会话内呈现，不谎称已保存。
- 任何查询都只返回**真实存在的**记录，不存在的内容不虚构或默认填 0/空而不标注。