# Clapper Top50 监控中心

主播收入 & 大R用户的统一监控平台。

## 目录结构

```
clapper-dashboards/
├── index.html              # 门户首页
├── streamer.html           # 主播收入Top50看板
├── whale.html              # 大R用户Top50看板
├── data/
│   ├── db-schema.sql       # Phase 2 数据库表结构设计
│   └── archive/            # 历史数据JSON存档
│       ├── streamer_w5_20260222.json
│       └── whale_w5_20260222.json
└── README.md
```

## Phase 1: GitHub Pages 部署 (当前)

### 首次部署

1. 登录 GitHub，点右上角 `+` → `New repository`
2. 仓库名填 `clapper-dashboards`，选 **Private**（仅团队可见）
3. 创建后，点 `uploading an existing file`，把本文件夹内所有文件拖入上传
4. 进入 Settings → Pages → Source 选 `Deploy from a branch`
5. Branch 选 `main`，文件夹选 `/ (root)`，点 Save
6. 约1分钟后，页面顶部出现访问链接

> 访问地址: `https://<你的用户名>.github.io/clapper-dashboards/`

### 每周更新流程

1. 用 Claude 生成本周新的 `streamer.html` 和 `whale.html`
2. 同时让 Claude 导出JSON存档到 `data/archive/` (文件名带周次和日期)
3. 在 GitHub 仓库页面，直接上传替换这两个HTML + 新增JSON文件
4. GitHub Pages 自动更新，约1分钟后生效

### 权限管理

- Private仓库: 只有被邀请的Collaborator能访问
- 邀请方式: Settings → Collaborators → Add people
- GitHub Pages 对 Private 仓库需要 Pro/Team 计划
- 替代方案: 设为 Public 但不在任何地方公开链接 (security through obscurity)

## Phase 2: 后端服务 + 数据库 (计划中)

### 架构

```
用户浏览器
    ↓
Render.com (Node.js 后端)
    ├── GET /api/streamer?weeks=all  → 返回历史数据
    ├── GET /api/whale?weeks=all     → 返回历史数据
    ├── POST /api/upload/streamer    → 上传Excel入库
    └── POST /api/upload/whale       → 上传Excel入库
    ↓
Supabase (PostgreSQL)
    ├── streamer_weekly  (每周50行，持续累积)
    ├── whale_weekly     (每周50行，持续累积)
    ├── streamers        (基础信息)
    ├── whales           (基础信息)
    └── upload_logs      (上传记录)
```

### 数据库设计

见 `data/db-schema.sql`，核心思路:
- 每周数据插入新行，历史永久保留
- 主播周数据包含: 收入、观众、开播、评分、分型、归因标签
- 大R周数据包含: 充值、行为、分型、健康状态、风险评分
- 支持的分析: 长期趋势、生命周期、季节性、流失预测

### 部署步骤

1. **Supabase**: 注册 → 创建项目 → 执行 `db-schema.sql` 建表
2. **GitHub**: 添加后端代码 (Node.js + Express)
3. **Render**: 连接 GitHub → 创建 Web Service → 设置环境变量 `SUPABASE_URL` + `SUPABASE_KEY`
4. 批量导入 `data/archive/` 中的历史JSON到数据库
5. 前端改为从API读取数据

### 数据积累后的分析能力

| 数据深度 | 可解锁的分析 |
|---------|------------|
| 4-8周   | 主播收入趋势稳定性、大R充值周期基线建立 |
| 3个月   | 季节性规律、主播生命周期阶段判断、流失预测模型 |
| 6个月   | 大R迁移路径分析、主播-大R关系演变、LTV预估 |
| 1年     | 年度对比、完整生命周期、精准节奏偏离检测 |

## 数据周期

- 周数据覆盖: W1(01.19) - W5(02.16)
- 日明细覆盖: 02.09 - 02.15
- 最后更新: 2026-02-25
