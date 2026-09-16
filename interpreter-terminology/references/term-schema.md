# 术语卡片数据结构（term-schema.md）

本文件定义术语的**动态语言、结构化卡片、mastery/版本字段**。语言字段一律动态（R1）；核心字段不得删除，可扩展。

---

## 1. 语言字段（动态，禁止写死三语）

```json
{
  "source_language": "zh",
  "target_language": "fr",
  "term_language": "fr",
  "definition_language": "zh",
  "learning_language": "zh"
}
```

- `source_language` / `target_language`：本次口译方向。
- `term_language`：`source_term`/`target_term` 所在语言标签（用于标记真实语料与搭配的语言）。
- `definition_language`：定义所用的语言。
- `learning_language`：测试与提示用语（通常等于用户母语）。

`zh / fr / en / …` 仅为示例；任何语言组合都允许。**不要为每个词强行补齐"中英法"。**

---

## 2. 术语卡片完整结构

```json
{
  "id": "",
  "source_language": "",
  "target_language": "",

  "source_term": "",
  "target_term": "",

  "field": "",
  "subfield": "",
  "topic": "",
  "priority": "P1|P2|P3",

  "part_of_speech": "",
  "definition": "",
  "definition_language": "",

  "usage_note": "",
  "collocations": [],
  "variants": [],
  "synonyms": [],
  "abbreviations": [],

  "source_name": "",
  "source_institution": "",
  "source_url": "",
  "source_quote": "",
  "source_date": "",

  "verification_status": "",
  "confidence": 0,

  "ai_version": "",
  "user_version": "",
  "user_note": "",

  "examples": [],

  "mastery": 0,
  "error_count": 0,
  "last_reviewed": "",
  "next_review": ""
}
```

**说明：**
- `verification_status` 取值见 verification-rules.md（verified / multi-source / single-source / conflict / pending / ai-candidate）。
- `confidence`：0–100，confidence 与 verification_status 应相互印证；`ai-candidate` 的 confidence 不应虚高。
- `ai_version`：Skill 原始产出；`user_version`：用户修订后内容。两者并存，用户修订时只填 `user_version`，不覆盖 AI 原始记录（R3）。
- `examples` 为真实语料与 AI 例句混合数组，每条带 `example_type`（R5）。
- 可自行扩展字段，但**不得删除以上核心字段**。

---

## 3. 例句（examples）条目

```json
{
  "example_type": "real|ai_generated",
  "text": "",
  "source": "",
  "url": "",
  "notice": ""      // 仅 ai_generated 必填："AI生成，仅用于练习，不代表真实语料。"
}
```

---

## 4. 掌握度与复习字段

```json
{
  "mastery": 0,
  "error_count": 0,
  "last_reviewed": "",
  "next_review": ""
}
```

- `mastery`：0–100，档位见 testing-rules.md。
- `error_count`：累计错误次数，随每次测试增减。
- `last_reviewed` / `next_review`：复习计划用（ISO 日期）。

---

## 5. 建议（不强求）

为使库可长期维护，额外建议但非必需字段：`created_at`、`updated_at`、`tags`（动态标签）、`origin`（首次来源：用户手动/检索入库/导入）。