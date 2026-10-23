# HyperFrames Vibe 视频大赏（16:9 横版）· 豆包工作第三方技能

> **第三方技能声明**：本项目为**非官方**的第三方自定义技能，与 HeyGen 及其官方产品无任何隶属、背书或合作关系。
> 技能所驱动的上游框架 **HyperFrames** 由 **HeyGen** 开源（Apache-2.0），仓库地址：https://github.com/heygen-com/hyperframes
> 本仓库仅包含重新编写的提示词/技能文档与示例代码，未复制上游任何源码或官方技能原文。

基于 [HyperFrames](https://github.com/heygen-com/hyperframes) 框架，为**豆包工作「本地电脑」模式**量身定制：一句分镜脚本 → 自动生成 16:9 横版「Vibe 视频大赏」科普短视频工程（HTML + GSAP 时间线）→ 写入本地 `index.html` → `lint` 校验 → `preview` 预览 → `render` 导出 MP4。

## 特性

- ✅ 豆包工作本地 Agent 可直接读写本地 HyperFrames 项目文件
- ✅ 自动执行 `npx hyperframes lint / preview / render`
- ✅ 16:9 横版知识卡片风：深色低饱和、左右分栏（左文字右图形）、克制动效
- ✅ 根据分镜文案核心意象自动匹配右侧图形与动效（覆盖基础几何 / 数据图表 / 分形生成艺术 / 模拟与元胞自动机 / 程序化纹理 / 3D 图形 / 结构关系图 / 文字图形 / 动态图形九大类）
- ✅ 附完整可渲染示例（40 秒熵增科普），开箱即测
- ✅ 规范依据官方 `hyperframes-core` 契约，规避常见 lint 错误

## 目录结构

```
hyperframes-vibe-169/
├── SKILL.md                          # 技能主体（豆包工作自定义技能）
├── references/
│   ├── hyperframes-contract.md       # HyperFrames 组合契约精编（写码前通读）
│   └── documentary-director.md       # 纪录片导演规范（可选风格：锚词/音乐对位/无口播时触发）
├── examples/
│   └── entropy-16-9/                 # 40秒 16:9 熵增科普示例（模块化子组合，0错误0警告）
│       ├── index.html                #   宿主：挂载 5 个分镜子组合
│       └── compositions/             #   5 个场景子组合文件
├── LICENSE                           # Apache-2.0
└── 豆包技能包-hyperframes-vibe-169.zip  # 已打包，可直接上传豆包技能库
```

## 安装（豆包工作｜本地电脑模式）

### 1. 本机环境（豆包不代装，一次性装好即可）

环境只需装一次，之后所有项目共用。按你的系统选一条命令复制粘贴即可。

**① 安装 Node.js（≥ 22）**

Windows（任选其一）：
```powershell
winget install OpenJS.NodeJS.LTS
```
或去 [nodejs.org](https://nodejs.org) 下载 **LTS 版**安装包，一路「下一步」装完。

macOS：
```bash
brew install node@22 && brew link --overwrite node@22
```

验证（应显示 v22.x.x 或更高）：
```bash
node -v
```

**② 安装 FFmpeg**

Windows：
```powershell
winget install Gyan.FFmpeg
```
macOS：
```bash
brew install ffmpeg
```

验证：
```bash
ffmpeg -version
```

**③ 一键自检（全绿即可继续）**

复制下面整行到终端运行，出现 3 行版本/诊断信息且无报错即环境就绪：
```bash
node -v && ffmpeg -version | head -1 && npx -y hyperframes@latest doctor
```

**④ 初始化 HyperFrames 项目（每个项目执行一次）**
```bash
npx hyperframes init <项目名> && cd <项目名>
```

### 2. 导入技能（二选一）
- **方式 A（上传 zip）**：豆包 PC 客户端 → 「插件·技能·伙伴」→「技能」→「+ 添加」→「上传技能」→ 选择 `豆包技能包-hyperframes-vibe-169.zip`
- **方式 B（对话新建）**：打开「与豆包对话新建技能」，粘贴 `SKILL.md` 全文，保存

### 3. 新建工作任务
1. 「新工作任务」→ 工作设备选 **本地电脑**
2. 授权你的 HyperFrames 项目文件夹
3. 启用技能 `hyperframes-vibe-169`

### 4. 下达任务示例
```
使用 hyperframes-vibe-169 技能，制作 40 秒 16:9 Vibe 视频大赏，分镜：
0-4s 标题：熵增是什么？
4-12s 观点一：封闭系统会越来越混乱
12-22s 观点二：混乱不是无序，而是可能性增加
22-32s 观点三：成长就是主动建立秩序
32-40s 结尾金句：对抗熵增，是每个人的长期修行

按标准流程执行：读取 index.html → 生成并写入代码 → lint 校验 → 启动 preview。完成后告诉我结果。
```
豆包会自动：读写 `index.html` → 执行 `npx hyperframes lint` → 修复报错 → 启动 `preview`；确认后下达 `npx hyperframes render` 导出 MP4。

## 快速测试示例（不装技能也能跑）

```bash
npx hyperframes init vibe-demo && cd vibe-demo
cp -r /path/to/examples/entropy-16-9/* .   # 覆盖 index.html，并带入 compositions/ 子组合
npx hyperframes lint      # 0 errors, 0 warnings
npx hyperframes preview   # 浏览器预览动画
npx hyperframes render    # 导出 MP4
```

## 注意事项

- 示例代码使用通用字族（`system-ui, sans-serif`）规避 lint；如需更优中文渲染，请将本地字体放入项目并配置 `@font-face`。
- 旁白与 BGM 请在 `assets/` 放入音频文件后，取消示例中注释掉的 `<audio>` 标签（每个 audio 必须有 `id` 和 `src`）。
- 本技能面向豆包工作「本地电脑」模式；不支持 Claude Code 的 `/hyperframes` 斜杠命令体系。

## License

本仓库内容（技能文档、示例代码）采用 [Apache-2.0](LICENSE)。上游 HyperFrames 由 HeyGen 以 Apache-2.0 协议开源，使用本技能即视为同时遵守上游协议要求（保留版权声明、注明来源）。如需商用，请自行核对 HeyGen 上游许可与商标使用规则。
