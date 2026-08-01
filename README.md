# shanpin
创新招聘网站
以下是为该项目定制的 `README.md`（自述文件）模板。你可以直接将其复制到项目的根目录中，并根据实际开发进度进行微调。

***

# 闪聘 (FlashHire) - 抖音模式短视频招聘系统

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Vue](https://img.shields.io/badge/vue-3.x-green.svg)](https://vuejs.org/)
[![Node](https://img.shields.io/badge/node->=16.0.0-darkgreen.svg)](https://nodejs.org/)

**闪聘 (FlashHire)** 是一款将“短视频信息流”与“精准人才画像”深度结合的创新招聘系统。平台采用类似抖音的交互模式，通过直观的视频展示企业环境与岗位职责，配以极简的滑动操作，实现求职者与招聘方的高效对接。

## 📱 核心交互设计 (Gestures & Interaction)

项目采用**三屏滑动架构**，提供沉浸式的求职体验：

*   **⚡ 主屏 (上下滑动):** 浏览企业岗位介绍、真实工作环境的短视频。
*   **👈 左滑 (人才成型面板):** 
    *   实时显示应聘者与当前岗位的匹配度（是否通过初步筛选）。
    *   展示【相关课程】、【相关技能】、【相关测试】、【相关经验】、【相关作品】。
    *   展示【真实工作场景】（图片/轮播）、公司联系方式及物理地址。
    *   底部固定 **【发送人才成型模块】** 按钮，一键投递结构化简历。
*   **👉 右滑 (侧边导航栏):** 快速按【岗位】、【公司】、【推荐】、【城市】等维度进行分类筛选。

---

## 🛠️ 技术栈 (Tech Stack)

### 前端 (Frontend)
*   **核心框架:** Vue 3 (Composition API)
*   **构建工具:** Vite
*   **滑动与手势:** Swiper.js (用于视频流及左右面板滑动切换)
*   **样式框架:** Tailwind CSS
*   **组件库:** Vant UI (移动端适配)
*   **状态管理:** Pinia

### 后端 (Backend)
*   **开发语言:** Node.js (NestJS) 或 Go (GoFrame)
*   **数据库:** PostgreSQL (主数据库) + Redis (缓存与视频热度)
*   **文件存储:** 阿里云 OSS (视频、作品集托管) + CDN 加速

---

## 📁 目录结构 (Directory Structure)

```text
flashhire-platform/
├── client/                     # 前端项目 (Vue 3)
│   ├── src/
│   │   ├── assets/             # 静态资源
│   │   ├── components/         # 公共组件 (视频播放器、滑动容器)
│   │   ├── views/
│   │   │   ├── Feed.vue        # 视频主页
│   │   │   ├── LeftPanel.vue   # 人才成型模块 (左滑)
│   │   │   └── RightPanel.vue  # 分类导航栏 (右滑)
│   │   ├── store/              # Pinia 状态管理
│   │   └── router/             # 路由配置
├── server/                     # 后端项目 (Node.js/NestJS)
│   ├── src/
│   │   ├── jobs/               # 职位与视频模块
│   │   ├── users/              # 用户画像与技能模块
│   │   ├── match/              # 匹配度计算引擎
│   │   └── apply/              # 投递与人才成型模块发送
└── README.md                   # 项目自述文件
```

---

## 🚀 快速开始 (Quick Start)

### 1. 克隆项目
```bash
git clone https://github.com/your-username/flashhire.git
cd flashhire
```

### 2. 前端启动
```bash
cd client
npm install
npm run dev
```
打开浏览器访问 `http://localhost:5173` 并开启**移动端调试模式**。

### 3. 后端启动
```bash
cd server
npm install
# 配置 .env 环境变量（数据库连接、OSS密钥等）
npm run start:dev
```

---

## 📝 核心业务逻辑说明

### 匹配度计算 (Matching Engine)
当用户滑动到特定职位时，后端会通过 `/api/user/match` 接口，对比用户的 `user_qualifications`（技能、课程、测试成绩）与岗位的 `job_requirements`，计算出匹配百分比，并在左滑面板顶部直观呈现“已通过/未通过”状态。

### 发送人才成型模块 (Send Profile)
点击左滑面板底部的 **【发送人才成型模块】**，系统不会发送传统 PDF 简历，而是将求职者在该岗位下的**匹配度、技能图谱、作品案例及测试成绩**打包成一个动态的“人才画像页面”，直接推送给 HR 端的管理后台，实现秒级评估。
