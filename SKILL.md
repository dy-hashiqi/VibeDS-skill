---
name: hyperframes-vibe-169
description: 使用 HyperFrames 框架制作 16:9 横版「Vibe 视频大赏」科普短视频。当用户要求生成知识科普、观点讲解、金句科普类短视频，或要求生成/修改 HyperFrames 视频工程（HTML+GSAP 时间线）、读写项目 index.html、执行 npx hyperframes lint / preview / render 时使用。适配豆包工作「本地电脑」模式。第三方技能，非 HeyGen 官方出品；上游 HyperFrames 由 HeyGen 开源（Apache-2.0，仓库 https://github.com/heygen-com/hyperframes）。
---

# HyperFrames Vibe 视频大赏（16:9 横版）

为豆包工作「本地电脑」模式设计的第三方技能：基于 HyperFrames 框架制作 16:9 横版 Vibe 视频大赏科普短视频，自动读写本地项目文件、调用本机终端执行 hyperframes CLI。不依赖 Claude Code / Cursor 的 `/hyperframes` 斜杠命令。

## 前置条件（用户负责，Agent 不代装）
- 本机已装 Node.js ≥22 与 FFmpeg；
- hyperframes CLI 可用，项目已执行 `npx hyperframes init <name>` 初始化；
- 项目文件夹已授权给当前豆包工作本地电脑任务。

## 收到任务后执行流程
1. 读取项目目录下的 `index.html`（若存在），理解现有时间线、分镜、布局、样式；
2. 按用户提供的分镜脚本（总时长、时间戳、文案、可选配色）编写**完整独立**的 HyperFrames 组合代码，写入 `index.html`；
3. 执行 `npx hyperframes lint` 校验；有报错则读取报错信息、修复代码并重新写入，直到通过；
4. 执行 `npx hyperframes preview` 启动本地预览；
5. 汇报：文件已更新、lint 结果、预览状态、下一步可执行命令；
6. 用户确认后按指令执行 `npx hyperframes render` 导出 MP4（优先 1920×1080 @ 30fps，受阻自动降级、绝不升级渲染，可用 `--output <name>.mp4` 指定文件名）。

## 输入格式
用户提供：总时长、比例（默认 16:9 横版）、分镜时间戳+文案、可选配色与风格。
示例：
> 40 秒 16:9 Vibe 视频大赏，分镜：
> 0-4s 标题：熵增是什么？
> 4-12s 观点一：封闭系统会越来越混乱
> 12-22s 观点二：混乱不是无序，而是可能性增加
> 22-32s 观点三：成长就是主动建立秩序
> 32-40s 结尾金句：对抗熵增，是每个人的长期修行

## 编写组合的硬性规则
- 严格遵循 HyperFrames 组合契约：根节点带 `data-composition-id`、`data-width`、`data-height`、`data-duration`；片段用 `class="clip"` + `data-start` + `data-duration`；注册**一条 paused 的 GSAP 时间线**到 `window.__timelines["<id>"]`（key 与根节点 `data-composition-id` 一致）。
- 16:9 横版：`<meta name="viewport" content="width=1920, height=1080">`，根节点同尺寸；默认 1920×1080 高清输出。
- 渲染规格优先使用标准的 1920×1080 @ 30fps，此为渲染上限，不得升级（如 60fps/4K）；若遇到不可解决的阻碍（如环境不支持 30fps），自动降级渲染（如 24fps 或 720p）以完成任务，但绝不升级渲染规格。
- 写码前先通读 `references/hyperframes-contract.md`，规避全部常见 lint 错误与 GSAP 陷阱。
- 不使用 `<template>` 包裹独立根节点；不输出代码片段，必须完整独立 HTML。
- 分镜 ≥2 时采用**模块化子组合**架构：每个场景一个 `compositions/scene-*.html`（`<template>` 内包含该场景的 style/script/根节点），主 `index.html` 用 `data-composition-src` 挂载各场景；宿主 `data-composition-id`、子组合文件内根节点 `data-composition-id`、`window.__timelines` key 三者必须一致。详见 `references/hyperframes-contract.md` 第 8 节。

## 分镜图形/动画选择指南（自动匹配）

制作时**根据每个分镜文案的核心意象自动选择右侧图形与动效**，不固定使用示例中的网格/散点/柱条。

### 选择流程
1. 解析每个分镜文案，提取核心意象（如"增长/混乱/可能性/秩序/网络/周期/氛围"）；
2. 按下表匹配图形类别与动效；同一分镜只用 **1 种主图形**，最多 1 种辅助点缀；
3. 图形效果已放开：分形/元胞自动机/3D/Shader/纹理等复杂图形可用 CSS/HTML 或 Canvas/SVG 实现，允许使用 `Math.random`、实时时钟等动态效果（网络请求除外），数据图表用示意性数据；
4. 图形必须落在 16:9 安全框内（四周 ≥64px），不得溢出画布，保持克制质感。

### 语义 → 图形 → 动效 映射（常用）
| 分镜意象 | 推荐图形 | 推荐动效 |
|---|---|---|
| 增长 / 建立秩序 | 柱状条、阶梯、树形生长、黄金螺旋 | 从底生长、逐步升高 |
| 混乱 / 熵增 / 无序 | 网格方块、散点、Voronoi 碎块、分形裂纹 | 漂移、扩散、碎裂 |
| 可能性 / 分支 | 树图、散点、弦线、思维导图 | 向外发散、分支生长 |
| 周期 / 循环 | 环形图、圆环、波形、摆线 | 旋转、脉冲、往返 |
| 关系 / 网络 | 网络拓扑、知识图谱、力导向图 | 节点连线、渐进点亮 |
| 时间 / 过程 | 折线图、面积图、时间轴 | 笔划生长、轨迹延伸 |
| 对比 / 分布 | 饼图、环形图、雷达图、箱线图 | 扇区展开、逐项呈现 |
| 空间 / 结构 | 等距网格、点云、3D 线框 | 逐层搭建、旋转入场 |
| 氛围 / 抽象 | 噪声纹理、波干涉、驻波、粒子流 | 流动、起伏、消散 |

完整候选库（9 大类：基础几何、数据图表、分形生成艺术、模拟与元胞自动机、像素位图与程序化纹理、3D 图形、结构图/关系图、文字类图形、动态图形）及确定性实现约束见 `references/hyperframes-contract.md` 第 7 节「图形选择库」。

## Vibe 视频大赏风格（16:9 横版）
- 深色低饱和背景，简约高级知识卡片；建议左右分栏：左侧标题/观点文字，右侧图形图示（按分镜语义自动匹配，见上节）；转场干净克制；
- **小标签（kicker / 章节锚点）固定在画面左上角** `left:80px; top:64px`，独立于垂直居中的文字块，不随标题上下居中；主标题与副文仍在左侧文字区垂直居中；
- 动画：文字淡入+轻微上移，图形按选择指南匹配（生长/扩散/旋转/脉冲等，分步出现）；允许玻璃拟态、噪点、故障光效、花哨 shader 与特效堆叠等效果；
- 中文可读性优先：文字置于画面安全框内（四周留 ≥64px），不得溢出画布；不引入 `Math.random`、实时时钟等非确定性内容；
- 音频：旁白与 BGM 用单独 `<audio>`（每个 audio 必须有 `id` 和 `src`），BGM `data-volume` 约 0.2 不盖人声；本机暂无音频文件时，以 HTML 注释形式给出音频标签模板。

## 可选风格：纪录片导演模式

若用户要求纪录片风格、锚词叙事、音乐重拍对位或无口播短片：先读 `references/documentary-director.md` 并严格遵循其叙事与节奏规范。

## 修改迭代
用户提出修改时：读取当前 `index.html` → 修改对应时间轴/布局/样式 → 写回 → lint → 修复 → 重启 preview。

## 汇报格式
完成任务后输出简短报告：1) index.html 已更新；2) lint 结果（通过/失败及修复情况）；3) 预览状态；4) 建议的下一步（render 或继续修改）。

## 禁止事项
1. 不直接 tween `.clip` 元素本身（片段可见性由框架管理，lint 拒绝 `gsap_animates_clip_element`），只动画子元素；
2. 不在 body 正文中使用 `<br>`（分行用独立元素）；
3. 不给 `<video>`/`<audio>` 加 `crossorigin`；不调用 `video.play()`/`audio.play()`/seek；
4. 不添加用户未要求的特效，保持 Vibe 视频大赏克制质感；
5. 不改动与任务无关的时间线、id、变量、媒体路径。
