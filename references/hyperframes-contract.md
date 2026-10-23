# HyperFrames 组合契约（精编版）

> 依据 HeyGen 官方 `skills/hyperframes-core` 规范整理，供 Agent 编写组合前通读。官方完整文档：https://hyperframes.heygen.com/introduction

## 目录
1. 组合结构（两种根形态）
2. 时间属性与片段（clips）
3. GSAP 时间线契约
4. 常见 lint 报错清单（写码先避）
5. 媒体（video/audio）规则
6. 确定性规则（determinism）
7. 16:9 横版 Vibe 视频大赏布局要点

---

## 1. 组合结构

HyperFrames 组合就是一个 HTML 文件，DOM 用 `data-*` 属性声明时间。

**独立组合（顶层 `index.html`）**：根 `<div>` 直接放在 `<body>` 中，**不要**用 `<template>` 包裹（`standalone_composition_wrapped_in_template` 是 lint 错误）。

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=1280, height=720" />
    <title>组合标题</title>
    <script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/gsap.min.js"></script>
    <style>/* 样式 */</style>
  </head>
  <body>
    <div id="root" data-composition-id="main" data-width="1280" data-height="720" data-duration="40">
      <!-- 片段们 -->
    </div>
    <script>
      const tl = gsap.timeline({ paused: true });
      window.__timelines["main"] = tl;
    </script>
  </body>
</html>
```

- 画布尺寸由 `data-width`/`data-height` 决定；`#root` 写 `width:100%; height:100%`，**不要**硬编码像素尺寸到 `#root`。
- 时长来源：根节点 `data-duration`（推荐），或时间线/媒体推断。
- 根节点 `data-start="0"` 是惯例（运行时会自动补齐，非必须）。

## 2. 时间属性与片段

- `data-start` + 时长属性（`data-duration`）是元素成为「片段」的唯一判据；`class="clip"` 只是布局与工具约定（推荐保留）。
- 片段示例：`<section id="s1" class="clip" data-start="0" data-duration="4">`
- `data-track-index` 只是 Studio 展示轨道，不是时序约束。
- 片段结束超过根 `data-duration` 会被截断（lint 警告 `clip_ends_past_root_duration`），同一次编辑里同步延长根时长。

## 3. GSAP 时间线契约

- 每个组合注册**恰好一条** `gsap.timeline({ paused: true })`，key 必须与根节点 `data-composition-id` 一致。
- 渲染长度 = 根 `data-duration`，不是时间线长度：时间线跑过根时长会被截断，提前结束则保持最后一帧。
- 不要在片段（`.clip`）上做场景退出的 `tl.set(..., {visibility:"hidden"})`，运行时会自动隐藏已结束的片段；淡出只作用于内部节点。
- 内部节点可用 `tl.fromTo(el, {...}, {...}, 绝对时间)` 精确控制入场/退场。

## 4. 常见 lint 报错清单（首次构建必踩，先写好）

| 报错 | 原因 | 正确写法 |
|---|---|---|
| `gsap_css_transform_conflict` | CSS 初始 `transform` 与 GSAP 同属性 tween 冲突 | 用 `gsap.fromTo(el, {x:-40},{x:0})` 表达初始态，CSS 不写 transform |
| `gsap_animates_clip_element` | 直接 tween 了 `.clip` 的 `autoAlpha`/`visibility`/`display` | 动画 `.clip` 的子元素 |
| `media_crossorigin_breaks_preview` | `<video>`/`<audio>` 加了 `crossorigin` | 去掉 `crossorigin`（无例外） |
| `video_nested_in_timed_element` | `<video data-start>` 的祖先也带 `data-start` | 时序只写在 wrapper 或 video 之一 |
| `media_missing_id` | `<audio>` 没有 `id` | 每个 `<audio>` 必须有 `id`（否则渲染无声） |
| `font_family_without_font_face` | 命名 `font-family` 无 `@font-face` | 用 `system-ui, sans-serif` 等通用族，或给本地字体文件配 `@font-face` |
| `standalone_composition_wrapped_in_template` | 独立组合被 `<template>` 包裹 | 根节点直接放 `<body>` |

## 5. 媒体（video/audio）规则

- 媒体在任意嵌套深度都能被框架发现（扁平 `querySelectorAll("video, audio")`）。
- 带声音的视频：声音留在 `<video>` 上（`data-has-audio="true"`，不 `muted`）；音乐、旁白、替换音轨用**单独** `<audio>`。
- `<audio>` 必需 `id` + `src`；音量基线用 `data-volume`（如 BGM 0.2，人声 1）；可用 `data-track-index` 分轨道。
- 不要在组合代码里调用 `play()`/`pause()`/seek，框架拥有播放权。
- 示例：
```html
<audio id="voice" src="assets/voiceover.wav" data-start="0" data-duration="40" data-volume="1" data-track-index="1"></audio>
<audio id="bgm" src="assets/bgm.mp3" data-start="0" data-duration="40" data-volume="0.2" data-track-index="2"></audio>
```

## 6. 确定性规则（determinism）

- 允许渲染期时钟（`Date.now`、`requestAnimationFrame` 计时）与 `Math.random` 等动态效果；禁止网络请求与输入态；
- `repeat: -1` 只在根 `data-duration` 有限时使用（导出片段受限）；否则用有限次数；
- 正文不用 `<br>`；被 transform 的元素必须是块级且已定尺寸；
- 每个 `id` 在整页唯一。

## 7. 16:9 横版 Vibe 视频大赏布局要点

- 1280×720：四周安全边距 ≥64px，左右分栏（左 45-50% 文字，右 40% 图形），底部可留字幕条；
- 渲染输出优先标准 1920×1080 @ 30fps（渲染上限，不得升级至 60fps/4K）；遇到不可解决的阻碍时自动降级（如 720p 或 24fps）继续完成，绝不升级渲染规格；
- 字号参考：主标题 56-72px、副文 26-34px、小标签 18-24px；深色背景 `#0e1116` 类低饱和色，强调色统一（如 `#63a4ff`）；
- 小标签（kicker / age-tag，如"18—30 岁 · 相遇"这类章节标识）**固定在画面左上角**：`position:absolute; left:80px; top:64px;`，独立于垂直居中的文字块，不随标题/副文一起上下居中；主标题与副文仍在左侧文字区垂直居中，二者层级分明——小标签是常驻左上的章节锚点，标题是画面主体；
- 图形示例：网格方块（有序→漂移表示混乱）、散点（向外扩散表示可能性）、柱状条（渐次升高表示建立秩序）；制作时**按分镜语义自动匹配图形与动效**，候选库与映射见下方 7.2；
- 动效节奏：每镜入场文字 0.6-0.9s 淡入上移，图形分步 0.3s 间隔，避免 0.2s 以下生硬闪切。

### 7.2 图形选择库（按分镜语义自动匹配）

制作时根据分镜文案的核心意象自动选图，不固定使用三种示例图形。优先 CSS/HTML 实现；复杂图形（分形、元胞自动机、3D、Shader、程序化纹理）可用 Canvas/SVG 实现，允许使用 `Math.random`/`Date.now` 等动态效果（网络请求除外），数据图表只作示意、不编造业务数据含义。

**候选库（9 大类）**

1. **基础几何矢量**：点、直线、折线、矩形、正方形、圆形、椭圆、圆弧、扇形、三角形/五边形/六边形等多边形、二次/三次贝塞尔曲线、自定义闭合路径轮廓
2. **数据可视化图表**：折线图、柱状图、条形图、饼图、环形图、散点图、热力图、雷达图、桑基图、树图、词云、等高线图；补充面积图、堆叠面积图、箱线图、小提琴图、误差棒图、双轴图、极坐标玫瑰图、和弦图、平行坐标图；3D 散点、3D 曲面、3D 柱状图
3. **分形 & 生成艺术纹样**：曼德博集合、朱利亚集、科赫雪花、谢尔宾斯基三角形、L-system 分形树/草丛/珊瑚、黄金螺旋、阿基米德螺旋、心脏线、摆线、外摆线、内摆线、弦线艺术、极坐标万花筒、几何镶嵌纹样、Voronoi 泰森多边形（蜂窝、碎块、细胞裂纹）
4. **模拟与元胞自动机**：康威生命游戏、沙堆模型、森林火灾模拟、反应扩散斑纹（斑马纹、生物皮肤纹理）、向量场、电场/磁场线、波干涉、驻波、流体粒子、烟雾粒子图形
5. **像素位图 & 程序化纹理**：像素画、图片合成、图像滤镜；Perlin/Simplex 噪声纹理（云朵、大理石、木纹、无缝贴图）；GLSL Shader 特效（辉光、色散、屏幕扭曲、溶解、全息线框）
6. **3D 图形**：立方体、球体、圆锥、圆柱、圆环、多面体、莫比乌斯环；程序化地形山脉、高度图地貌、网格细分、四维几何体 3D 投影、点云、星空
7. **结构图 / 关系图**：流程图、时序图、甘特图、思维导图、UML 图、网络拓扑、知识图谱、电路原理图、依存树形图
8. **文字类图形**：文字轮廓、点阵像素字体、文字沿曲线排列、文字粒子打散效果
9. **动态图形 & 动画**：图形位移、旋转、形变、生长动画；粒子动画、流体动态图案

**语义匹配表（分镜意象 → 推荐图形 → 推荐动效）**

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

**使用约束**
- 每镜 1 种主图形 + 至多 1 种辅助点缀；保持 16:9 安全框（四周 ≥64px）与克制质感；
- 复杂图形（分形、元胞自动机、3D、Shader、纹理）允许使用 `Math.random`、`Date.now` 等动态效果（网络请求除外）；`repeat: -1` 仅限根 `data-duration` 有限时使用；
- 数据图表仅作示意，不赋予虚假业务含义；文字类图形保证中文可读性、不溢出安全框；
- 3D/Shader 若实现成本过高，降级为同类 2D 图形（如 3D 柱状→2D 柱状、全息线框→网格线框），不得牺牲 lint 通过性。
