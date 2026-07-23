# 长视频深度学习 · 命令模板（渐进式披露）

> 本文件是 `long-video-deep-study` 的"动手"层。SKILL.md 只给流程与判据，具体命令放这里，按需查阅，避免主文件膨胀。

## 1. 下载（yt-dlp）

```bash
# URL 下载（无 cookie 优先试）
yt-dlp -f "bestvideo+bestaudio" -o "%(title)s.%(ext)s" "<URL>"

# 只抽音频（转写够用，省空间/显存）
yt-dlp -x --audio-format wav -o "%(title)s.%(ext)s" "<URL>"
```

**回退铁律**：cookie 被拦（`Fresh cookies needed` / `401` / `403`）→ 让用户发本地视频文件 → 走本地路径模式，**永不需 cookie、永不出网**。判定法：先 `yt-dlp -s <url>` 无 cookie 试；报错即需 cookie → 转回退。

## 2. 转写（faster-whisper）

```bash
pip install faster-whisper
```

### 模型选择
| 模型 | 精度 | 速度 / 显存 | 何时用 |
|------|------|-------------|--------|
| tiny / base / small | 低–中 | 快、CPU 友好 | 草稿、长视频快速预览 |
| **medium** | 中高 | 中、建议 GPU | **生产默认（精度/速度平衡）** |
| large-v3 | 最高 | 慢、需 GPU / 大内存 | 专有名词密集（安全 / 医疗 / 代码 / 人名） |

```python
from faster_whisper import WhisperModel
import srt

# device="auto" 优先 GPU；纯 CPU 用 compute_type="int8" 省内存
model = WhisperModel("medium", device="auto", compute_type="auto")
segments, _ = model.transcribe("input.wav", language="zh", word_timestamps=True)
subs = [srt.Subtitle(i, s.start, s.end, s.text) for i, s in enumerate(segments, 1)]
with open("transcript.srt", "w", encoding="utf-8") as f:
    f.write(srt.compose(subs))
```

### 长视频提速（CPU 耗时偏长时）
- 设备：`device="auto"` 让 faster-whisper 自动选 GPU；无 GPU 时 `small` + `int8` 最稳。
- 切分：用 `ffmpeg -ss/-to` 按章节把长音频切成数段，分别转写后按时间轴拼接，避免单次 OOM / 超时。
- 备选引擎：`whisper.cpp`（C++，纯 CPU 比 Python 版快数倍）。

### 术语纠错（专有名词补正）
专有名词常被误听。实测案例：`checkAutoType`→"Check Out Of Type"、`miscCodec`→"miscodec"、`expectClass`→"Express Class"、`FuzzJson`→"Fuzz Jason"。两种补正：
1. 用户给术语表 → 转写后做字符串替换（如把 "check out of type" 还原为 `checkAutoType`）。
2. 无术语表 → 用 `references/prompts.md` 的「术语纠错」模板，把 SRT 片段 + 已知主题交给模型校对。

备选 openai-whisper：
```bash
pip install whisper
whisper input.wav --model medium --language zh --output_format srt
```

## 3. 章节切分（LLM 辅助）

把 SRT 文本喂给模型，用 `references/prompts.md` 的「章节生成」模板，输出 JSON：
```json
[{"t_start":"00:00","t_end":"03:12","title":"...","summary":"一句话摘要"}]
```
要求：章节不重叠、覆盖全片、标题具体。

## 4. 关键词定位（search）

SRT 是纯文本，直接 `grep -n "关键词" transcript.srt` 或用模型做语义检索（"找出所有讨论 X 的段落"），返回时间戳 + 上下文。

## 5. 抽帧（视觉补强）

纯音频转写会丢掉视频中的流程图、公式、代码、画面文字。在讲解 / 笔记涉及的章节时间戳处抽关键帧：

```bash
# 在指定时间戳抽 1 帧（如 3:00 处的 checkAutoType 流程图）
ffmpeg -ss 00:03:00 -i input.mp4 -frames:v 1 -q:v 2 frames/ch03_checkAutoType.png

# 按固定间隔批量抽（每 30 秒 1 帧）
ffmpeg -i input.mp4 -vf "fps=1/30" -q:v 2 frames/frame_%04d.png
```

- 抽帧后可交 `video-understand-skill` 做画面文字 / 内容识别，或作为笔记配图直接引用。
- 帧文件建议落到 `frames/` 子目录，笔记里用相对路径引用，便于归档与 Build in Public 分享。
