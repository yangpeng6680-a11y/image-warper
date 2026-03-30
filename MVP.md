# 🌸 批量图片拉伸编辑器 — MVP 需求文档

> 版本：v1.0 | 日期：2026-03-30 | 状态：规划中

---

## 1. 产品概述

### 1.1 是什么
一款**四角透视变形工具**，支持背景合成与批量导出。用户上传图片后，通过拖拽四个控制点对图片进行透视变形，可叠加背景图，批量处理后一键导出 PNG。

### 1.2 核心用户
- 电商卖家（制作商品主图）
- 设计师（快速做透视效果图）
- 普通用户（图片创意编辑）

### 1.3 核心价值
**一句话：** 打开网页就能用，无需安装，无需注册登录基础功能，专业级透视变形效果。

---

## 2. 功能范围

### 2.1 已完成（当前版本）
- [x] 单图片四角透视变形（拖拽控制点）
- [x] 数值精确调节（X/Y 坐标输入）
- [x] 背景图底板合成
- [x] 批量多图处理
- [x] 一键下载 PNG
- [x] 重置功能
- [x] 响应式移动端适配

### 2.2 MVP 必须有（本次开发）
- [ ] **Google 账号登录**
  - 一键 Google 登录，保护用户数据
  - 登录用户可保存历史操作记录
- [ ] **用户操作记录**
  - 保存最近 10 次透视参数配置
  - 快速复用历史参数
- [ ] **历史记录管理**
  - 查看、删除、清空历史记录
  - 离线可用（localStorage）

### 2.3 未来版本（S2/S3）
- [ ] 微信登录
- [ ] 社交分享（生成链接）
- [ ] 用户中心（个人主页、我的作品）
- [ ] 导出历史记录（JSON/CSV）
- [ ] 批量上传云端处理
- [ ] 付费会员（高清原图下载）

---

## 3. 技术方案

### 3.1 当前技术栈
| 层级 | 技术 | 说明 |
|------|------|------|
| 前端 | HTML + CSS + JS | 单文件，~800行 |
| 托管 | GitHub Pages | imagedistortion.shop |
| 登录 | — | 暂未接入 |

### 3.2 MVP 技术栈
| 层级 | 技术 | 说明 |
|------|------|------|
| 前端框架 | Next.js 16 | App Router |
| 登录鉴权 | NextAuth.js | Google OAuth |
| 部署平台 | Vercel | 免费 Tier |
| 数据存储 | localStorage | 用户端本地存储 |
| 图床/CDN | 暂用本地 | 未来迁移云存储 |

### 3.3 部署架构
```
GitHub (main branch)
       ↓ push
   Vercel CI/CD
       ↓ auto deploy
   *.vercel.app (临时)
       ↓ DNS 指向
   imagedistortion.shop (正式域名)
```

---

## 4. UI/UX 说明

### 4.1 页面结构
```
├── 登录区（右上角）
│   ├── 未登录：Google 登录按钮
│   └── 已登录：用户名 + 登出按钮
│
├── 顶部标题区
│   └── 🌸 批量图片拉伸编辑器 🌸
│
├── 第一区块：准备素材
│   ├── 背景图上传区（可开关）
│   └── 前景图上传区（支持多选）
│   └── 缩略图预览队列
│
├── 第二区块：四角拉伸调节
│   ├── 可视化画布（棋盘格背景 + 控制点）
│   ├── 四角坐标控制面板（X/Y 数值输入）
│   ├── 下载全部按钮
│   └── 重置按钮
│
└── 页脚（可选）
```

### 4.2 登录流程
```
用户点击"Google 登录"
       ↓
弹出 Google 授权窗口
       ↓
用户授权
       ↓
跳转回网站，显示用户名
       ↓
自动保存登录态（localStorage）
```

### 4.3 历史记录流程
```
用户完成一次变形操作
       ↓
自动保存参数到 localStorage
（包含：四角坐标、是否启用背景、背景图数据URL）
       ↓
用户点击"历史"按钮
       ↓
弹出历史记录列表
       ↓
用户选择一条记录
       ↓
自动填充参数到画布
```

---

## 5. 数据模型

### 5.1 localStorage 结构
```typescript
// 用户登录态
interface UserSession {
  id: string
  name: string
  email: string
  image: string
  loginAt: number  // 时间戳
}

// 操作记录
interface HistoryRecord {
  id: string
  createdAt: number
  corners: Array<{ x: number; y: number }>  // 四角坐标
  useBackground: boolean
  backgroundDataUrl?: string  // base64 背景图
  imageCount: number
}
```

### 5.2 localStorage Key 规划
| Key | 类型 | 说明 |
|-----|------|------|
| `iw_session` | UserSession | 当前登录用户 |
| `iw_history` | HistoryRecord[] | 操作历史（最多10条） |

---

## 6. Google OAuth 配置（需完成）

### 6.1 Google Cloud Console
- [x] 创建 OAuth 2.0 Client ID
- [ ] 添加 `https://imagedistortion.shop` 到 Authorized JavaScript origins
- [ ] 添加 `https://imagedistortion.shop/api/auth/callback/google` 到 Authorized redirect URIs
- [ ] 配置 OAuth Consent Screen（Publishing status → In production 或添加测试用户）

### 6.2 Vercel 环境变量
| Name | Value |
|------|-------|
| `GOOGLE_CLIENT_ID` | 来自 Google Cloud Console |
| `GOOGLE_CLIENT_SECRET` | 来自 Google Cloud Console |
| `NEXTAUTH_URL` | `https://imagedistortion.shop` |
| `NEXTAUTH_SECRET` | 随机字符串 |

---

## 7. 里程碑

### M1 ✅ 基础功能（已完成）
单页 HTML 版本，原生实现四角变形 + 背景合成 + 批量导出

### M2 🔨 登录系统（本次 MVP）
- [ ] Next.js 项目创建完成（✅ 代码已推送 GitHub）
- [ ] Vercel 部署 + 环境变量配置
- [ ] Google OAuth 登录功能
- [ ] 登录态持久化

### M3 🔨 历史记录（本次 MVP）
- [ ] 操作记录自动保存
- [ ] 历史列表查看/加载/删除
- [ ] localStorage 管理

### M4 �的未来
- 社交登录扩展
- 用户中心
- 批量云端处理
- 付费功能

---

## 8. 非功能需求

### 8.1 性能
- 首屏加载 < 3 秒（3G 网络）
- 变形计算 < 500ms

### 8.2 兼容性
- Chrome / Firefox / Safari / Edge 最新两个版本
- 移动端 iOS Safari / Android Chrome

### 8.3 安全
- Google OAuth Token 不在 localStorage 明文存储
- 背景图数据仅本地处理，不上传服务器

---

## 9. 项目仓库

| 仓库 | 说明 |
|------|------|
| `yangpeng6680-a11y/image-warper` | 原始单页 HTML 版本（当前线上版本） |
| `yangpeng6680-a11y/image-warper-nextjs` | Next.js 重构版本（MVP 开发分支） |

---

*本文档为 MVP 规划文档，功能范围以实际开发为准。*
