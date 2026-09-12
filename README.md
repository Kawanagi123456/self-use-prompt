# Self-Use Prompt Studio

一个用于 SD / Anima（Pony 系）提示词拼装的单页工具，内置 **2827 条带示例图的 Danbooru tag 图鉴**。

**在线访问：** https://kawanagi123456.github.io/self-use-prompt/

---

## 功能

- **23 个标签分类 · 2827 条 tag · 3026 张示例图**，图片全部随仓库内置，不依赖外部图床
  - 二次元属性 59 · 发型 134 · 发饰 174 · 头饰 168 · 套装 142 · 基础姿势 55 · 手臂姿势 78 · 手部配饰 96 · 泳装 88 · 礼服 70 · 耳饰 69 · 背景 336 · 衬衫 97 · 上衣 96 · 袜子 147 · 裙子 224 · 裤子 91 · 身份设定 204 · 面部表情 186 · 眼睛 61 · 鞋子 121 · 风格化 73 · 画师串 58
- **提示词拼装**：左侧点选 tag → 右侧实时生成 Prompt，支持复制、随机排序、手动直接编辑
- **图鉴卡片**：按「子类」分组展示，侧栏可一键展开 / 收起某个分类的全部子类，单个子类标题也能单独点开关
- **悬停预览**：鼠标停在右侧已选标签上，弹出示例图 + 中文名 + Danbooru tag + 备注说明
- **多图灯箱**：风格化分类每条 tag 有多张示例图，点开可浏览全部
- **三语界面**：简体 / 繁体 / English
- **NSFW 模式**：默认隐藏，需确认成人声明后才开启
- 搜索、玻璃拟态深色 UI、选择状态本地持久化（localStorage）

## 目录结构

```
.
├── index.html            # 整个应用（全部 tag 数据内联，单文件即可运行）
├── .nojekyll             # 必须保留，见下方说明
├── README.md
└── <分类名>tag与示例图/
    ├── *.md              # 该分类的 tag 原始文档
    └── images/*.jpg      # 示例图
```

## 部署与访问控制

站点通过 **Cloudflare Workers（静态资源）+ Cloudflare Access** 发布，**只有被邀请的邮箱登录后才能访问**。

完整步骤见 **[DEPLOY-CLOUDFLARE.md](./DEPLOY-CLOUDFLARE.md)**。

> ⚠️ **不要启用 GitHub Pages。**
> GitHub Pages 是纯静态托管，没有登录能力；一旦开启，
> `https://kawanagi123456.github.io/self-use-prompt/` 会把全部 3026 张图重新公开出去，
> Cloudflare Access 的保护就形同虚设。如果之前开过，请在仓库
> Settings → Pages → Source 选 `None` 关掉。

日常更新：

```powershell
cd self-use-prompt
npx wrangler deploy
```

### `wrangler.jsonc` / `.assetsignore` 不能删

- `wrangler.jsonc` —— 部署配置
- `.assetsignore` —— 排除 `.git`（约 500 MB、3400+ 个对象文件）。没有它，wrangler 会把 git 对象也当成静态资源上传，既慢又会**把完整提交历史公开出去**

已实测：加上 `.assetsignore` 后实际上传 **3,050 个文件 / 268 MB**，`.git` 下 0 个文件。

### 图片已压缩

原始示例图共 1196 MB，已统一重新编码为**长边最大 1280px、JPEG 质量 82（mozjpeg / progressive）**，压缩后约 266 MB。

页面中图片的最大显示宽度约 1150px，因此 1280px 对图鉴展示和放大查看都足够。本地原始未压缩图片不包含在本仓库内。

新增分类时只需压缩新图，已处理过的图片会直接复用，不会二次编码造成画质损失。

### `.nojekyll` 现已无用但保留

它原本是给 GitHub Pages 关掉 Jekyll 用的（本仓库有 84 张图以 `_` 开头）。
改用 Cloudflare Workers 后不再需要，留着不影响。

## 更新内容

修改 `index.html` 或替换示例图之后：

```bash
git add .
git commit -m "更新 tag 图鉴"
git push
```

推送只更新 GitHub 仓库，线上站点需要另外执行 `npx wrangler deploy` 才会更新。
