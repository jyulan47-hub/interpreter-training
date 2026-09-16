# 端到端流程用例与自检（example-workflow.md）

## 用例 A — 译前准备
进入 Skill → 选「译前准备」→ 输入主题"欧洲能源政策"、方向"法→中" → 调用 `interpreter-terminology` → 保存 `type: preparation` 记录 → 导出该记录为 `.md`。
检查：记录含 `prep_called/页面级`，可导出单条 `.md`。

## 用例 B — 专业主题 + 口译（自动调用译前准备）
进入 →「口译练习」→ 方向法→中、B2、自定义主题"量子计算" → 判定专业 → 提示"我先帮你做一个简短的译前准备" → 用户选"先做" → 调 `interpreter-terminology` → 调 `french-interpretation-training` → 训练 → 保存完整记录（含 `prep_called: true`、`prep_record_id`）。
检查：自动判断 + 调用链 + 完整数据落库。

## 用例 C — 普通主题 + 口译（无需译前准备）
进入 →「口译练习」→ 普通主题"巴黎城市生活" → 判定无需准备 → 直接调 `french-interpretation-training` → 保存记录（`prep_called: false`）。
检查：跳过译前准备，不产生多余调用。

## 用例 D — 视译练习
进入 →「视译练习」→ 选方向/难度/主题/材料 → 判是否需要准备 → 必要时调用 `interpreter-terminology` → 调 `french-interpretation-training` 视译模式 → 保存。
检查：复用同一判断逻辑，`type: sight_translation`。

## 用例 E — 导出 → 迁移 → 导入恢复
导出 `.md` 全部/单条 → 清空学习数据 → 导入该 `.md` → 记录恢复、完整可读。
检查：导出文件含完整数据，导入后字段完整、正文完整。

## 用例 F — 重复导入
重复导入同一 `.md` → 不产生重复记录。
检查：按 `id` 与内容指纹去重；跳过 M 条并报告；不可解析 K 条明确告知（不静默丢弃）。

---

## 可移植性自检

- [ ] 只做编排，未复制 `interpreter-terminology` 或 `french-interpretation-training` 的专业逻辑（R1）。
- [ ] 调用用真实标识，不虚构 Skill ID（R2）。
- [ ] 不伪造记录/来源/链接/评分（R3）。
- [ ] 持久化如实；不支持则明示 + 引导 `.md` 备份（R4）。
- [ ] 导入去重、不可解析项明确报告（R5）。
- [ ] 既可拆分（用户已装两个基础 Skill）也能独立作为统一入口运行（当基础 Skill 缺失时告知用户需安装）。

## 依赖 Skill 缺失时的行为
若 `interpreter-terminology` 或 `french-interpretation-training` 未安装：入口仍可用，但在需要调用其功能时，明确告诉用户"缺少该功能 Skill，请先安装"，并提供**有限降级**（如仅保存/导出用户自备内容），不假装已执行该类功能。