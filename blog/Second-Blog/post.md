---
title: 更新！太强了DS v4.1 Flash ！
subtitle: 添加了可视化blog编写页面，带有一键发布/预览文章等功能
top_image: top.png
---

![屏幕截图 2026-09-20 161639](屏幕截图 2026-09-20 161639.png)
## 交付清单

| 文件 | 大小 | 职责 |
|---|---|---|
| `gitfolio\studio.html` | 18.9 KB | BlogStudio 界面（编辑区 + 实时预览 + 日志） |
| `gitfolio\studio-server.js` | 13.8 KB | 本地服务端，零第三方依赖，只监听 127.0.0.1 |
| `gitfolio\md2post.ps1` | 11.8 KB | 文章渲染器：Markdown + 前言 → gitfolio 格式的 HTML |
| `gitfolio\blog.ps1` | 10.5 KB | 命令行入口：`new` / `list` / `edit` / `build` / `preview` / `publish` |
| `gitfolio\sync-blog.ps1` | 10.9 KB | 发布流水线：生成 → 校验 → 同步 → 提交 → 推送 |
| `gitfolio\BlogStudio.cmd`、`sync-blog.cmd` | 150 / 194 B | 双击入口 |
| `bin\blog.cmd`、`bin\BlogStudio.cmd` | 205 / 55 B | **全局命令**（`D:\Projects\bin` 已加进你的用户 PATH） |

**核心设计原则：一个渲染器、一个发布器。** 三个入口（图形界面 / `blog` 命令 / `sync-blog.cmd`）最终都走 `md2post.ps1` 和 `sync-blog.ps1` 这一套 —— 改一处逻辑，三条路同时生效，不会出现"界面里是对的、命令行里是错的"。

## 内置的七道保护（都是今天实际踩过的坑）

1. **博客目录大小写检查** —— Windows 不区分大小写、GitHub Pages 是 Linux 区分，不一致会 404。现在会直接报错并给出修法
2. **快照是否真的重新生成** —— 比对 `dist\index.html` 时间戳，没变就中止并提示"可能是 GitHub API 限流"，避免把旧快照推上线
3. **`Copy-Item` 只增不删的漂移检测** —— 列出发布仓库里已删除文章的残留（要 `-Clean` 才真删）
4. **旧文章自动备份** —— 网页编辑器建的 HTML 文章第一次转 Markdown 时，原 `index.html` 自动备份成 `.bak`
5. **`.nojekyll` 自动补** —— 让 GitHub Pages 跳过 Jekyll
6. **安全防护** —— 网址名白名单 + 路径穿越拦截（实测 403），服务端只绑本机
7. **编码修复** —— `.ps1` 带 UTF-8 BOM、`.cmd` 用 CRLF + `chcp 65001`、Node 调 PowerShell 时强制 UTF-8 输出

## 测试覆盖

| 类型 | 内容 | 结果 |
|---|---|---|
| 静态检查 | 脚本语法、HTML 内联 JS 语法、25 个 `$('id')` 引用一致性 | 全过 |
| 服务端联调 | 首页/state/上传/去重/保存/备份/渲染/预览/穿越防护/发布链路 | 12 项全过 |
| 中文端到端 | 中文标题、副标题、正文、中文图片名、冒号边界、转义 | 13 项全过 |
| 启动器实测 | 四个 `.cmd` 走真实 cmd.exe（等价双击） | 全过 |

测试全程在沙箱文章上做，跑完删除并**按备份逐字节还原** `blog.json`，你的真实数据零污染。
