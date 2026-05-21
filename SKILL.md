---
name: cover-gen
description: Generate platform-specific content cover images and thumbnails for YouTube, Xiaohongshu, Douyin, Bilibili, WeChat, X/Twitter, Instagram, Zhihu, LinkedIn, and other publishing platforms. Use when the user asks to generate a cover, thumbnail, video cover, content cover, social media cover, or batch platform-specific cover variants.
---

# 封面生成 Skill

## 这个 skill 是干什么的

帮作者快速生成各平台的视频/内容封面。作者只需回答三个问题,skill 生成第一张主图,经作者确认后,再批量生成其他平台对应尺寸的版本。

人物素材(可选)由作者预先放在 `assets/persona/` 目录,作为生成时的人物参考,保持形象一致性。素材类型不限——真人照片、像素风、动漫形象、3D 渲染、手绘 IP 都可以。如果目录为空,跳过人物参考,直接按风格生成。

---

## 启动方式

作者在 Codex 里说"生成封面"(或类似指令)时,按本文档流程执行。无需配置任何 API key,使用 Codex 内置的图像生成能力即可。

---

## Step 1:抛出三个问题

向作者提问,等待回答。问题如下,逐字照抄:

```
请回答以下三个问题,我将根据你的回答生成封面:

1、你的标题是:
2、你希望的图片风格是:(例如:科幻、温柔、治愈、赛博朋克、暖色生活流、极简等,自由填写)
3、你希望发布的平台是:(可填多个,用顿号分隔。例如:YouTube、小红书、抖音)
```

收到回答后,把三项内容复述一遍给作者确认,确认通过再进入 Step 2。

---

## Step 2:检测人物素材

扫描 `assets/persona/` 目录:

- **有图片**(`.png` `.jpg` `.jpeg` `.webp` 任意一种或多种):全部读取作为人物参考。自动识别素材类型(真人照、像素风、动漫、3D、手绘 IP 等),在后续 prompt 中说明"保持参考图中的形象特征、风格、配色一致"。
- **目录为空或不存在**:跳过人物参考,直接按风格关键词生成无固定主体的封面。不要向作者询问,不要中断流程。

素材风格的识别交给图像生成模型自己处理,本 skill 不做预处理。

---

## Step 3:确定主图平台和尺寸

平台 → 封面尺寸对照表(单位:像素):

| 平台 | 比例 | 尺寸 |
|---|---|---|
| YouTube | 16:9 | 1280×720 |
| YouTube Shorts | 9:16 | 1080×1920 |
| Bilibili | 16:10 | 1146×717 |
| 小红书 | 3:4 | 1242×1660 |
| 抖音 | 9:16 | 1080×1920 |
| 公众号头图 | 2.35:1 | 900×383 |
| 公众号封面 | 1:1 | 900×900 |
| X / Twitter | 16:9 | 1600×900 |
| Instagram Feed | 1:1 | 1080×1080 |
| Instagram Reels | 9:16 | 1080×1920 |
| 知乎 | 16:9 | 1280×720 |
| LinkedIn | 1.91:1 | 1200×627 |

规则:

- 作者列出的第一个平台 = 主图平台,先出这张
- 如果作者写了表里没有的平台,问一次"该平台的封面建议尺寸/比例是多少?",拿到答案后按此尺寸生成,本会话内记住该平台
- 如果作者只写了一个平台,出完确认即结束,不主动扩展

---

## Step 4:生成主图

调用 Codex 内置的图像生成能力,把以下 prompt 模板填好后传入:

```
A high-quality content cover image.

Title text rendered on the image (must be clearly readable, large, bold, 
properly positioned, no garbled characters): "{标题}"

Visual style: {风格}

{如果有人物参考图,追加:}
The character/person shown in the reference image must appear as the main 
subject. Strictly preserve their facial features, style, color palette, 
and overall visual identity from the reference. The reference may be a 
real photo, pixel art, anime illustration, 3D render, or hand-drawn IP — 
match the reference's style exactly.

{如果没有参考图,追加:}
No specific character required. Focus on the visual style and title 
typography to create an eye-catching cover.

Composition for {平台} ({比例}):
{根据比例追加的构图指引,见下}

High production value, professional cover design, eye-catching, strong 
visual hierarchy. Title text and main subject (if any) are the focal points.

Output size: {宽}×{高} pixels.
```

构图指引按比例追加:

- **16:9 / 16:10 / 1.91:1 / 2.35:1(横屏)**:主体置于画面右侧 1/3,标题文字占据左侧 2/3 空间,字号大、粗体,可加描边或阴影增强对比。
- **9:16(竖屏)**:标题置于画面上 1/3,字号极大,主体占画面下 2/3,表情或姿态突出。
- **1:1(方图)**:主体居中或偏下,标题置于上方或叠加在主体上方,留白克制。
- **3:4(竖图,小红书)**:标题置于顶部,主体占画面主体,整体氛围感强,配色契合风格关键词。

标题语言保持作者输入的原文(中文就是中文,英文就是英文,不翻译)。

---

## Step 5:保存与展示

输出目录:`outputs/{YYYYMMDD}_{标题前10字符}/`

文件命名:`{序号}_{平台名}_{尺寸}.png`

例如:
- `outputs/20260520_如何快速学英语/01_youtube_1280x720.png`
- `outputs/20260520_如何快速学英语/02_xiaohongshu_1242x1660.png`

主图生成后,在对话里告知作者文件路径,让作者打开查看。

---

## Step 6:等待确认

主图生成后,问作者:

```
主图已生成:{文件路径}

请查看后回复:
- "通过" → 我将自动生成其他平台的版本
- "重做" → 告诉我要调整什么(标题/风格/构图/配色等)
```

---

## Step 7:批量生成其他平台

收到"通过"后,按作者列表中剩余的平台逐个生成。每个平台:

- 用同一组参考图(若有)
- prompt 模板的"标题、风格、人物"部分保持不变
- 只切换"平台、比例、构图指引、输出尺寸"
- 保存到同一个 outputs 子目录,序号递增

全部生成完后,列出所有文件路径给作者。

---

## 重做流程

收到"重做"后,根据作者描述的调整方向修改对应字段:

- 改标题 → 改 prompt 里的 title 文本
- 改风格 → 改 prompt 里的 style 关键词
- 改构图 → 在 prompt 末尾追加具体构图要求
- 改配色 → 在 prompt 末尾追加配色要求

重新生成,覆盖主图文件(同名),再次进入 Step 6。

---

## 注意事项

- 三问之外不向作者追加问题(只有"通过/重做"和未识别平台尺寸两种例外)
- 生成失败时如实告诉作者错误信息,不要假装成功
- 人物素材目录为空不是错误,直接继续
- 中间产物用完即删,不污染目录

---

## 目录结构(参考)

```
cover-gen/
├── SKILL.md             # 本文件
├── agents/openai.yaml   # Codex UI 元数据
├── assets/persona/      # 人物参考图(可选,任意风格)
└── outputs/             # 自动创建,按日期+标题分子目录
```
