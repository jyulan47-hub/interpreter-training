# 工具调用规范与依赖降级（tool-roster.md）

本 Skill **只判断"何时调用宿主什么能力"，不实现任何工具**。每次执行前先确认当前宿主实际可用的能力，按需调用；缺失的能力如实降级。

## 宿主能力地图

| 宿主能力 | 本 Skill 用于 | 缺失时降级方案 |
|---|---|---|
| Web Search | 自动搜索训练材料、核验来源 | 明说"无法联网搜索"，改走用户上传 |
| Browser / 网页抓取 | 读取具体文章/视频页、字幕 | 能搜不能读时改引元数据；都不能则仅上传 |
| 音频/视频播放 | 口译听辨 | 提供文字稿改做视译式训练并明确说明 |
| 语音输入 / ASR | 转写用户口译 | 改为文字录入，标注"非语音转写" |
| 文件系统 / 持久化 | 保存录音+转写+训练记录 | 明说"无法保存文件"，仅会话内呈现 |
| 字幕 / PDF / DOCX 解析 | 解析用户上传材料 | 不支持的格式说明并建议转 TXT/字幕 |
| 计算 | 评分/误差统计/复盘对比 | 直接算，无需外部 |

## 调用原则

1. 先探测能力，再进行对应动作；探测不到就按降级走。
2. **不假装**：不假装可联网、不假装已保存、不假装已验证来源、不假装读过未读取的内容。
3. 来源链接必须真实可点；拿不到真实链接就显式标注"无来源/未核验"。
4. ASR 结果若可疑，提示用户人工确认，不替用户做最终定稿。

## 暴露给 Agent 的能力接口（建议）

按宿主 skill 架构调整命名，本 Skill 内部以这些逻辑单元编排：

```
enter_mode(mode)                 # interpreting / sight_translation / record / review / material
collect_parameters()             # direction, CEFR, source, length
search_material(params)          # 自动搜索真实材料
analyze_material(material)       # 识别语言/方向/主题/难度/时长/字数
prepare_topic_background(params) # 显式调用 interpreter-terminology
prepare_terminology(params)      # 返回术语表（带一一对应来源）
start_session(...)               # 打开一次训练会话，记录状态
submit_performance(text)         # 用户提交口译/视译文本
transcribe_audio(audio)          # 生成 ORIGINAL_ASR
apply_correction(...)            # 生成 USER_CORRECTED_TRANSCRIPT（不覆盖 ORIGINAL_ASR）
evaluate(transcript, meta)       # 评分（随 CEFR 变化）
generate_reference(...)          # 普通版/进阶版参考译文
save_training_record(record)     # 持久化
review_history(query)            # 读取/对比历史
update_personal_error_bank(err)  # 归纳个人错误
```