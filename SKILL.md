---
name: long-video-deep-study
description: "Deep-learn a high-density long video (interview / course / video podcast) like reading a book: download, transcribe to timestamped SRT, AI-explain hard parts, keyword-locate, translate CN/EN side-by-side, and export notes into the memory bank. Use when the user wants to study a long video deeply, break down a podcast, or turn a talk into retrievable notes."
version: "0.2.1"
agent_created: true
slug: long-video-deep-study
displayName: "长视频深度学习（像读书一样看长视频）"
---

# 长视频深度学习（像读书一样看长视频）

把一条高信息密度长视频（深度访谈 / 课程 / video podcast）当"书"读：抽逐字稿 → AI 讲难点 → 关键词定位 → 中英翻译 → 导出笔记沉淀，最后 Build in Public 分享。核心不是"看完"，是"结构化吸收 + 可检索复用"。

源自张咋啦 LongCut 方法论 + 与 `video-understand-skill` 能力互补（本 skill 负责"观看侧"深加工，video-understand 负责"下载 + 抽帧 + 粗转写"）。

## 何时用
- "深度学习这个长视频 / 拆解这条 podcast / 把这条访谈做成笔记"
- 用户拿到一条一小时以上、信息密度高的访谈 / 课程，想吃透并沉淀

## 工作流（5 步，逐步带完成判据）

### 1. 获取视频源
- URL → 用 `yt-dlp` 下载音频 / 视频（**回退铁律**：若平台要 cookie 被拦，让用户发本地文件，绝不绕 cookie 出网；详见 `video-understand-skill`）。
- 本地文件 → 直接进第 2 步。
- ✅ 完成判据 = 本地存在可读的音 / 视频文件，已确认时长与来源；未拿到源绝不继续。

### 2. 转写逐字稿（带时间戳 SRT）
- 抽音频 → `faster-whisper` 转写，输出 SRT（含时间轴）。**模型档位见 `references/workflow.md`「模型选择」表**（small=快/CPU、medium=平衡、large-v3=最高精度）。
- **专有名词纠错**：安全/医疗/法律/代码等领域术语极易被误听（实测 `checkAutoType`→"Check Out Of Type"、`miscCodec`→"miscodec"）。转写后用 `references/prompts.md` 的「术语纠错」模板做一次校对，或加载用户术语表做字符串替换。
- **长视频提速**：`device="auto"` 优先用 GPU；纯 CPU 跑 1 小时+ 视频很慢，按章节切分后分批转写（见 workflow.md「长视频提速」）；可选 `whisper.cpp` 进一步加速。
- 缺依赖：先给安装命令（`pip install faster-whisper`），或复用 `video-understand-skill` 的转写路径；都不行则**明确告知用户缺依赖，不假装已完成**。
- ✅ 完成判据 = 生成 SRT 且时间轴覆盖全片、可与视频对齐，且已对明显误听术语做过一次校对；未生成 SRT 前不得声称"已转写"。

### 3. 结构化（章节 / 分段）
- 按话题或静音切分章节，LLM 给每章一句摘要 + 时间戳。
- ✅ 完成判据 = 产出章节列表（时间戳 + 标题 + 一句话摘要），覆盖全片主要段落。

### 4. 交互深化（四件套，按需触发）
按用户需求选做；每做完一项即交付，不堆到最后：
- **explain**：选中某段 → AI 讲解难点 / 背景 / 延伸（prompt 见 `references/prompts.md`）。
- **search**：关键词 → 定位所有出现位置（时间戳 + 上下文）。
- **translate**：整篇或段落 → 中英对照。
- **frames（视觉补强）**：讲解/笔记涉及的画面，用 `video-understand-skill` 或 ffmpeg 在对应时间戳抽 1–3 帧（命令见 workflow.md「抽帧」），作为笔记配图，弥补纯音频转写丢视觉的短板（流程图 / 公式 / 代码 / 画面文字）。抽帧后可交 video-understand-skill 做画面识别，或用 `references/prompts.md` 的「视觉结合讲解」模板把帧 + 转写一起讲。
- **export**：导出 md / txt 逐字稿 + 笔记（+ 抽帧图），落盘到你的笔记目录。
- ✅ 完成判据 = 用户指定的交互动作已完成并交付（讲解文本 / 定位结果 / 译文 / 抽帧图 / 导出文件均可见或落盘）。

### 5. 沉淀与分享
- 笔记存入你的个人知识库（如 Obsidian / WorkBuddy 记忆库，按内容性质分类）；建议 Build in Public：看完发心得。
- ✅ 完成判据 = 笔记已存入你指定的笔记位置，且已提示用户"可分享心得（Build in Public）"。

## 资源（渐进式披露）
- `references/workflow.md` — 命令模板：yt-dlp 下载（含回退）、whisper / faster-whisper 转写、SRT 切分与章节生成。
- `references/prompts.md` — explain / translate / 章节生成 / 笔记生成 的 prompt 模板，直接复用。

## 关联
- `video-understand-skill`（下载 + 抽帧 + 粗转写，本 skill 的前置 / 互补）
- 参考方法：张咋啦 LongCut 长视频精读法、行业入门「获取 → 过滤 → 存储 → 决策」四步法等公开资料（四步与本 skill 对齐）。
