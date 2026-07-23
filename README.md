# long-video-deep-study

把一条高信息密度长视频（深度访谈 / 课程 / video podcast）当"书"读：抽逐字稿 → AI 讲难点 → 关键词定位 → 中英翻译 → 导出笔记沉淀，最后 Build in Public 分享。

核心不是"看完"，是"**结构化吸收 + 可检索复用**"。

> 源自张咋啦 LongCut 方法论 + 与 `video-understand-skill` 能力互补（本 skill 负责"观看侧"深加工，video-understand 负责"下载 + 抽帧 + 粗转写"）。

## 工作流（5 步）

1. **获取视频源** — `yt-dlp` 下载（cookie 被拦则走本地文件，永不出网）；或用户直接给本地文件。
2. **转写逐字稿（SRT）** — `faster-whisper` 转写，输出带时间戳 SRT；含模型档位选择、专有名词纠错、长视频提速指引。
3. **结构化** — 按话题/静音切分章节，LLM 给每章摘要 + 时间戳。
4. **交互深化（四件套 + 视觉补强，按需触发）** — `explain` 讲解难点 / `search` 关键词定位 / `translate` 中英对照 / `frames` 抽关键帧补视觉 / `export` 导出笔记。
5. **沉淀与分享** — 笔记进记忆库；建议 Build in Public 发心得。

## 安装

```bash
# 作为 WorkBuddy / Claude Code 等 Agent 的 skill 使用
cp -r long-video-deep-study ~/.workbuddy/skills/
# 或 git clone
git clone https://github.com/joysinleung/long-video-deep-study.git
```

## 依赖

- `yt-dlp`（下载）
- `faster-whisper`（转写；`pip install faster-whisper`）
- `ffmpeg`（抽帧）
- 可选：`video-understand-skill`（互补的下载 + 抽帧 + 粗转写）

## 资源

- `references/workflow.md` — 命令模板（下载 / 转写 / 章节 / 抽帧）
- `references/prompts.md` — explain / translate / 章节 / 笔记 / 术语纠错 / 视觉结合讲解 的 prompt 模板

## License

MIT
