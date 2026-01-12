+++
date = '2026-01-12T22:59:20+08:00'
draft = false
title = '配置hugo+vercel自动化环境'
+++

想把 Hugo 博客托管到 Vercel，并在每次推送后自动构建发布？下面用本仓库的配置做个完整示例。

## 前置条件
- 已有 Vercel 账号，并能从 GitHub/GitLab/Bitbucket 导入仓库
- 本地安装 Git、curl，推荐先安装 Hugo Extended 版（`brew install hugo`）
- 代码里有一个 Hugo 站点，本例主题是 `ananke`，基础配置在 `hugo.toml`

## 本地准备
1) 初始化站点：`hugo new site quickstart && cd quickstart`
2) 添加主题并配置：将 `theme = 'ananke'` 写入 `hugo.toml`，按需调整 `baseURL`、`title`
3) 写一篇文章：`hugo new posts/my-first-post.md`，填入正文
4) 本地验证：`hugo server -D` 打开 <http://localhost:1313> 检查页面

## 自动化构建脚本
仓库提供了 `build.sh` 作为 Vercel 的构建入口：
- 安装 Dart Sass、Go、Hugo、Node.js 固定版本，保证 Vercel 环境一致
- 最终执行：`hugo --gc --minify --baseURL "https://${VERCEL_PROJECT_PRODUCTION_URL}"`  
  `VERCEL_PROJECT_PRODUCTION_URL` 由 Vercel 自动注入，可让生成站点指向生产域名
- 如需调整版本，修改脚本开头的几个 `*_VERSION` 变量即可

本地想模拟 Vercel，可以先导出域名再运行脚本：
```bash
export VERCEL_PROJECT_PRODUCTION_URL=your-project.vercel.app
chmod a+x build.sh && ./build.sh
```
生成好的静态文件会落在 `public/` 目录。

## Vercel 配置
1) 仓库根目录放置 `vercel.json`，内容示例：
   ```json
   {
     "$schema": "https://openapi.vercel.sh/vercel.json",
     "buildCommand": "chmod a+x build.sh && ./build.sh",
     "outputDirectory": "public"
   }
   ```
2) 在 Vercel 仪表盘点击「Add New… → Project」，导入仓库
3) Build Command 设置为上面一行，Output Directory 填 `public`
4) 默认环境变量即可（生产域名会自动注入），若需要自定义时区等，可在项目 Settings → Environment Variables 里添加
5) 保存后触发首个构建，构建日志里可看到 Hugo 版本、构建耗时等信息

## 自动化发布流程
- 推送到主分支 → Vercel 自动构建发布到生产域名（如 `*.vercel.app` 或自定义域名）
- 提交 Pull Request → 自动生成 Preview 预览链接，合并后再发布生产
- 日志与回滚：在 Vercel 的 Deployments 页面可以查看日志，必要时回滚到任意历史部署

## 常见问题
- 构建失败且提示版本不兼容：在 `build.sh` 更新对应版本号或精简依赖
- 页面资源 404：确认 `baseURL` 是否正确，尤其是本地手动运行时要设置 `VERCEL_PROJECT_PRODUCTION_URL`
- 本地和线上效果不一致：本地用 `./build.sh` 生成 `public/` 再 `hugo server --renderToDisk` 对比输出
