---
name: ultra-mv-studio
description: 仅使用现有 Google AI Pro会员或者Ultra 20× 网页会员权益与 Antigravity 内置 Browser Agent，在 Google Flow 以纯 720p（禁止超分辨率）首尾帧连贯生成多段视频，结合 Demucs + stable-ts 人声歌词毫秒级强制对齐与本地 FFmpeg 剪辑压制精装字幕，端到端制作完整音乐 MV。当用户要求制作音乐 MV、配置 Antigravity Flow 工作流、批量生成首尾帧连贯视频或精准对齐 MV 歌词字幕时触发。
---

# Ultra MV Studio Skill（720P 经济高质 + 精准字幕对齐全流程）

本 Skill 用于指导 Agent 在 **Google Antigravity App（`v2.30.0+`）** 中，端到端完成从内置浏览器环境检查、Google Flow 首尾帧连贯视频生成、开源歌词强制对齐到 FFmpeg 合成导出的完整音乐 MV 制作。

---

## 最高优先级硬性规则

1. **零付费 API**：仅使用现有 Google AI Pro会员或者Ultra 20× 网页会员权益（`https://gemini.google.com/` 与 `https://flow.google.com/`）。严禁调用独立计费的 Gemini / Veo / Lyria / Vertex AI API，严禁创建 GCP 付费项目或购买额外积分。
2. **严格锁定 `720p` 原生分辨率，禁止 Flow 超分辨率**：
   - 在 Google Flow 生成视频时，分辨率设为 **`720p`**（`16:9`、`x1`）。
   - 下载视频片段时，**必须且只能选择 `720p Original size`（额外 0 积分）**。
   - **严禁点击 `1080p Upscaled` 或 `4K Upscaled`**（禁止消耗任何超分辨率积分与等待时间）。
3. **仅用内置浏览器与合规交互**：优先使用 Antigravity 内置 `browser` subagent；禁止导出 Cookie、读取凭据、自动填密码或绕过官方使用限制。
4. **消耗积分前必获用户明确批准**：生成前必须先向用户汇报总镜头数、选定模型、单段时长与精确积分预估，获得批准后再提交生成任务。
5. **诚实汇报边界**：明确区分“已实测验证”与“规划预估”，不宣称能保证 100% 人物绝对一致或绕过官方速率/配额限制。

---

## 标准十步端到端执行流程

### Step 1：检查环境与本地工具链
1. 检查本机 `ffmpeg`、`ffprobe`、`python3`（3.11+ 推荐）及 `uv` 是否可用，确认 FFmpeg 支持 `libass`、`xfade`、`afade` 及硬件编码（macOS 优先 `h264_videotoolbox`，否则回退 `libx264`）。
2. 创建独立项目目录 `~/Movies/Antigravity-MV/projects/<project-slug>/`，包含子目录：`music/`、`lyrics/`、`references/fonts/`、`storyboard/`、`prompts/`、`clips/`、`previews/qa_frames/`、`output/`、`logs/`，并初始化断点账本 `progress.json`。

### Step 2：检查内置浏览器登录状态
1. 启动 `browser` subagent（默认使用 `flash` 模型）访问 `https://flow.google.com/`（如需要创作歌曲则同时检查 `https://gemini.google.com/`）。
2. 如遇到 `accounts.google.com` 登录拦截或未登录状态：保留标签页、**暂停并提示用户手动在内置浏览器完成登录**；用户确认后通过 `send_message` 唤醒同一个 `browser` subagent 继续，不新建多余浏览器代理。
3. 核实账户 `PRO` / `ULTRA` 会员标识、当前可用积分余额、可用视频模型（如 `Omni 1.2 Flash`、`Veo 3.1 - Lite [Lower Priority]`）以及 `Visible watermarking` 是否关闭。

### Step 3：准备音频、歌词与主角视觉设定
1. **音频**：导入用户提供的原唱音频（或经确认后通过 Gemini 网页端 `Create music` 生成并下载）至 `music/`，用 `ffprobe` 获取精确总时长（秒）。
2. **角色锁定**：将用户提供的主角参考图保存至 `references/`，撰写 `references/character_sheet.md`，提炼非敏感外观锚点（脸型、发色发饰、服饰纹样与配色）及统一画风前缀（如 3D 国漫 UE5 写实光影风格）。
3. **无文字画面约束**：所有 Flow 视频提示词末尾统一追加 `no text, no watermark, no subtitles.`，确保画面干净，字幕统一由后期本地精准烧录。

### Step 4：规划首尾帧连贯分镜与精确积分预算
1. 结合歌曲精确总时长与乐句段落（Intro / Verse / Chorus / Interlude / Bridge / Outro）切分镜头：
   - **主力推荐模型**：`Omni 1.2 Flash`（`720p`、`x1`、`720p Original size` 下载），单段支持 `3s`–`15s`。
   - **省积分原则**：`Omni 1.2 Flash` 在 `≤ 10s` 为 **3 积分/秒**，`11s`–`15s` 超出部分为 **6 积分/秒**；如预算敏感优先拆成 **`10s` 每段**（例：4分32秒约 28 段 = `840` 积分）。
   - **零积分备选**：若网页提供 `Veo 3.1 - Lite [Lower Priority]`（`≤ 8s`、`0` 积分），一并列出供用户选择。
2. 生成结构化 `storyboard/storyboard.json` 与 `prompts/flow_prompts.md`，同步写入 `progress.json`。
3. 向用户展示分镜摘要、选定模型（明确标注**仅生成并下载 `720p`，不超分**）、预估总积分与执行节奏（先样片审核或直接全片批跑），等待用户明确批准。

### Step 5：创建专属 Flow 项目并完成唯一一次参考图上传
1. 由 `browser` subagent 在 `https://flow.google.com/` 点击 **`+ New project`** 创建独立项目，记录项目 URL 至 `progress.json`（不改动用户已有项目）。
2. 如内置浏览器沙箱拦截对 `flow.google.com` 的自动文件上传（`setInputFiles`），暂停并提示用户**手动一次性将主角参考图拖入已打开的 Flow 项目画布**。
3. 确认参考图进入项目 `Uploads` 库后，后续所有镜头均在 Flow 内部调用素材，无需再做任何本地上传。

### Step 6：分批执行首尾帧连贯生成与 `720p` 原生下载
1. **防上下文截断**：将全部提示词复制到 `<ArtifactDir>/browser_sandbox/references/all_shots.md`，每次只给 `browser` subagent 派发 **3～5 段为一批**的简短指令，让其通过 `read_file` 读取精确提示词。
2. **每段标准闭环（严格顺序执行）**：
   - 设置 **`Video` → `Ingredients`**、模型 **`Omni 1.2 Flash`**（或用户批准模型）、**`16:9`**、**`720p`**、**`x1`**。
   - 第 1 段挂载主角参考图；第 `N` 段（`N ≥ 2`）挂载 **`第 N-1 段保存的尾帧`** + **本段出镜主角原始参考图**。
   - 设置时长、粘贴提示词并提交等待生成完成。
   - 打开生成片段进入 Flow **Video Editor**：
     1. 将进度条拖至**最后一帧** → 点击 **`Save frame`**（除最后一段外）；
     2. 点击 **Download → `720p Original size`**（**严禁点击 `1080p Upscaled` 或 `4K Upscaled`**！），等待文件完整落盘到 `browser_sandbox/downloads/`。
3. **每批本地核验与断点落盘**：
   - 复制归档至 `clips/shot_XXX.mp4`；
   - 运行 `ffprobe` 校验分辨率（`1280×720`）、时长与非空文件；
   - 立即更新 `progress.json`（记录状态与实际消耗积分）。若遇漏下载，下一批首步直接补下已有视频（0 积分），**绝不重复生成已完成镜头**。

### Step 7：开源人声分离 + 逐字歌词强制对齐（可选/需字幕时必做）
为避免网传 LRC 存在数秒漂移，使用独立隔离环境 `~/Movies/Antigravity-MV/tools/align-venv` 执行三阶段精准对齐：
1. **安装隔离环境（若尚未安装）**：
   ```bash
   uv venv --python 3.11 ~/Movies/Antigravity-MV/tools/align-venv
   uv pip install --python ~/Movies/Antigravity-MV/tools/align-venv/bin/python stable-ts demucs soundfile scipy
   ```
2. **Demucs 干声分离**：使用 `htdemucs`（macOS 开启 MPS 加速）提取纯人声轨 `vocals.wav`，剥离鼓点与伴奏干扰。
3. **分段强制对齐 + 10ms RMS 能量精修**：
   - 通过纯人声静默间隙（`> 2.5s`）与一次快速转写锁定每个真实演唱段落边界（防止跨越长间奏对齐失败）；
   - 对每段分别调用 `stable-ts`（Whisper `turbo`）执行 `model.align(..., language="zh", original_split=True, suppress_silence=True)`；
   - 结合 10ms 纯人声 RMS 能量校准起唱（加约 `0.08s` 视觉提前量，避开呼吸与吟唱停顿）与长拖音尾音收束。
4. **生成精装底部 `.ass` 字幕与 `.lrc`**：
   - 选用适配画风的中文字体（如古风 `ZCOOL XiaoWei`），主歌月白银辉、副歌高潮浅金流光，配柔化墨色描边与 `\fad(160,220)` 淡入淡出，`Alignment=2` 固定居中在画面最下方。

### Step 8：本地 FFmpeg 两阶段无漂移剪辑与字幕烧录
1. **补偿 `xfade` 耗时**：相邻片段采用 `xfade=0.35s` 平滑转场；除最后一段外，每段裁剪长度设为 `target_duration_sec + 0.35`（若原生片段略短，用极微幅 `setpts` 1.01x–1.03x 补齐，确保全片总时长毫秒级贴合音频）。
2. **两阶段分组渲染**：
   - 阶段一：每 5 段编为 1 组，统一 `1280×720`（保持 `720p` 原生清晰度，不做虚高超分）、`30fps`、`yuv420p`、静音 Flow 原声（`-an`）输出高码率临时分段；
   - 阶段二：拼接各分组，添加片头黑场淡入与片尾淡出，通过 `ass=filename='...':fontsdir='...'` 烧录底部 `.ass` 字幕，混入原唱母带音频（`AAC 48kHz 256kbps`）并启用 `+faststart` 导出至 `output/`。

### Step 9：自动化技术与目视 QA 验收
1. 用 `ffprobe` 校验最终 MP4 时长、编码（`h264` + `aac`）、分辨率（`1280×720`）、帧率（`30fps`）与文件大小。
2. 抽取片头、主歌、副歌高潮、终章 4 帧图片至 `previews/qa_frames/` 并使用 `read_file` 目视检查：主角一致性、画面连贯度、底部字幕字形与居中排版、无乱码水印。
3. 清理临时人声 WAV，更新 `progress.json` 为 `completed`。

### Step 10：断点恢复与成果交付
- 若中途遇到 5 小时额度耗尽、排队限流或会话过期：立即记录已失败镜头原因并保存 `progress.json` 停止提交，绝不宣称可无人值守自动恢复或绕过限制；待用户再次唤醒时，先读取 `progress.json` 与 `clips/`，仅继续未完成镜头。
- 全部完成后，向用户交付最终成片路径、关键抽检截图、字幕文件、Flow 项目链接及精确积分消耗明细。
