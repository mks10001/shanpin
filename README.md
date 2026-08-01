# shanpin
创新招聘网站
flashhire/
├── frontend/                      # Vue 3 前端
│   ├── src/
│   │   ├── components/
│   │   │   ├── VideoFeed.vue     # 主视频流
│   │   │   ├── LeftPanel.vue     # 左滑人才成型模块
│   │   │   ├── RightPanel.vue    # 右滑筛选导航
│   │   │   └── GestureHandler.ts # 手势控制器
│   │   ├── stores/
│   │   │   ├── userStore.ts      # 用户状态管理
│   │   │   └── videoStore.ts     # 视频状态管理
│   │   ├── api/
│   │   │   └── client.ts         # API 调用模块
│   │   └── App.vue
│   ├── package.json
│   └── vite.config.ts
├── backend/                       # Node.js (NestJS) 后端
│   ├── src/
│   │   ├── modules/
│   │   │   ├── jobs/
│   │   │   ├── users/
│   │   │   ├── applications/
│   │   │   └── matching/
│   │   ├── database/
│   │   │   └── schema.sql
│   │   └── main.ts
│   ├── package.json
│   └── .env.example
└── README.md
