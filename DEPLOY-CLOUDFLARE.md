# 部署到 Cloudflare Workers + Access 登录保护

本仓库通过 **Cloudflare Workers（静态资源）+ Cloudflare Access** 发布。只有被邀请的邮箱通过验证后才能访问，未登录的人连图片直链都拿不到。

---

## 为什么不用 GitHub Pages

GitHub Pages 是**纯静态托管，没有服务器**，无法做真正的登录：

- 任何"登录"都只是浏览器里的 JS 判断，看源码即可绕过
- 就算页面被挡住，所有图片直链依然公开可访问
- 把仓库设为私有也没用——免费/Pro 账号下私有仓库的 Pages 站点同样是公开的

> ⚠️ **所以部署到 Cloudflare 之后，务必关掉 GitHub Pages**
> （仓库 → Settings → Pages → Source 选 `None`）。
> 否则 `https://kawanagi123456.github.io/self-use-prompt/` 会把 3026 张图重新公开出去，
> Access 的保护就完全失效了。

---

## 免费额度够用吗

够。这个站点用的都是免费额度：

| 项目 | 免费额度 | 本站用量 |
| --- | --- | --- |
| 静态资源文件数 / 每次部署 | 20,000 | **3,050** |
| 单个文件大小 | 25 MiB | 最大 **714 KB** |
| **静态资源请求数** | **免费且不限量** | 全部请求都是静态资源 |
| Cloudflare Access 用户数 | 50 | 你自己 + 朋友 |

> 关键点：Cloudflare 官方文档写明「Requests to static assets are free and unlimited」。
> 本站没有 Worker 脚本（纯静态），所以**不消耗每天 10 万次的 Worker 请求额度**。
> 参考：[Static Assets 计费与限制](https://developers.cloudflare.com/workers/static-assets/billing-and-limitations/)

---

## 部署步骤

### 0. 前置

- Node.js（本机已有 v24）
- 一个 Cloudflare 账号（免费注册）

### 1. 注册 Cloudflare

<https://dash.cloudflare.com/sign-up>

### 2. 登录 wrangler

```powershell
cd D:\aki-ComfyUI\prompt\self-use-prompt
npx wrangler login
```

会自动打开浏览器让你授权。第一次运行会提示安装 wrangler，同意即可。

### 3. 部署

```powershell
npx wrangler deploy
```

部署完成后会输出一个地址：

```
https://self-use-prompt.<你的账号子域>.workers.dev
```

**此时这个地址还是公开的**，任何人都能访问 —— 下一步才加锁。

### 4. 开启 Access 登录保护（关键）

1. 打开 <https://dash.cloudflare.com> → 左侧 **Workers & Pages**
2. 点进 `self-use-prompt`
3. 切到 **Access** 标签页
4. 启用登录保护（Require sign-in / Protect this Worker）
5. 添加策略 **Policy**：
   - **Action**: `Allow`
   - **Include**: `Emails` → 填入允许访问的邮箱（自己和朋友的）
6. 选择保护范围：仅预览环境，或预览 + 生产环境（选后者）

> Access 的策略是**挂在 Worker 本身**上的，所以 `workers.dev` 地址和以后可能加的
> 自定义域名会一起受保护，不需要逐个添加。参考：
> [Cloudflare 更新日志](https://developers.cloudflare.com/changelog/post/2026-08-14-workers-access/)

### 5. 验证是否生效

用**无痕窗口**打开 `https://self-use-prompt.<你的账号子域>.workers.dev`：

- ✅ 应该跳到 Cloudflare 的登录页
- ✅ 输入被允许的邮箱 → 收到一次性验证码 → 登录后才能看到站点
- ❌ 如果直接看到图库，说明 Access 没启用成功

再试一下图片直链，例如 `.../眼睛tag与示例图/images/0001_blue_eyes.jpg` —— 未登录时应被拦截。

### 6. 关闭 GitHub Pages

GitHub 仓库 → **Settings** → **Pages** → **Source** 选 `None` → Save。

这一步不能省。

### 7. 以后更新内容

改完本地文件后重新部署即可：

```powershell
cd D:\aki-ComfyUI\prompt\self-use-prompt
npx wrangler deploy
```

---

## 增删可访问的用户

Dashboard → **Zero Trust** → **Access** → **Applications** → 找到对应策略 → 编辑 **Emails** 列表。

支持三种放行方式：

- **Emails** —— 指定具体邮箱（最常用）
- **Emails ending in** —— 放行整个域名（如 `@yourcompany.com`）
- **Cloudflare account members** —— 按 Cloudflare 账号成员

登录方式默认支持邮箱一次性验证码，也可以在 Zero Trust → Settings → Authentication 里接 Google / GitHub 等 SSO。

---

## 两个不能删的文件

| 文件 | 作用 |
| --- | --- |
| `wrangler.jsonc` | 部署配置，指定 `assets.directory = "./"` |
| `.assetsignore` | 排除不该上传的文件 |

`.assetsignore` 尤其重要：仓库里的 `.git` 目录有约 500 MB、3400+ 个对象文件。没有它，wrangler 会把这些 git 对象一起当成静态资源上传——不仅慢，还会**把完整提交历史公开出去**。

已实测确认：加上 `.assetsignore` 后，实际会上传 **3,050 个文件 / 268 MB**，`.git` 下 **0 个文件**被上传。

---

## 已知限制

- **内容此前已经公开过**。这些图片从第一次推送到公开仓库起就在公网，可能已被搜索引擎或爬虫抓取、缓存。现在加锁能阻止后续访问，但无法保证"从未被看到"。
- 免费版 Access 上限 **50 个用户**。
