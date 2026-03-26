# AeroGrow Pro Demo 部署指南

## 方案 1：GitHub Pages（推荐，永久免费）

1. 访问 https://github.com/new
2. 仓库名填 `aerogrow-pro-demo`
3. 选择 **Public**（公开）
4. 点击 **Create repository**
5. 点击 **uploading an existing file**
6. 上传 `index.html` 文件
7. 点击 **Commit changes**
8. 进入 **Settings** → 左侧 **Pages**
9. Branch 选择 **main**，点击 **Save**
10. 等 1-2 分钟，访问 `https://你的用户名.github.io/aerogrow-pro-demo/`

## 方案 2：Netlify Drop（最快，无需注册）

1. 访问 https://app.netlify.com/drop
2. 直接把 `index.html` 文件拖拽到页面
3. 立刻获得在线链接，如 `https://abc123.netlify.app`
4. 链接永久有效

## 方案 3：Vercel（推荐，国内访问快）

1. 访问 https://vercel.com/new
2. 用 GitHub 登录
3. 导入你的仓库
4. 自动部署，获得 `https://项目名.vercel.app` 链接

## 手机访问

部署完成后，手机 Safari 直接访问链接即可：
- 点击分享按钮 → "添加到主屏幕" → 像 App 一样使用

## 当前文件位置

所有需要部署的文件在：
```
/Users/ryan/.openclaw/workspace/AeroGrow-Pro/
├── index.html      ← 主要文件
├── AeroGrow Pro.app
└── README.md
```