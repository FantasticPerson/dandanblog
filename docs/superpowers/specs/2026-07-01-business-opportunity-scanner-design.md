# 商业机会扫描器 — 设计文档

## 项目概述

自动从多个信息源搜集新产品/项目信息，从 6 个维度分析商业可行性和实现难度，通过网页可视化展示分析报告。

---

## 技术栈

| 层 | 技术 |
|---|---|
| 前端 | Vue3 + Vite + ECharts |
| 后端 | NestJS + TypeScript |
| 数据库 | MongoDB + Mongoose |
| 采集 | Playwright + Cheerio |
| 队列 | Bull + Redis |
| 分析 | DeepSeek / Claude API |
| 部署 | 前端 Vercel，后端 2C4G 服务器 |

---

## 整体架构

```
Frontend (Vercel)          Backend (服务器)
Vue3 + ECharts             NestJS + Mongoose
    │                            │
    └──────── REST API ──────────┘
                                 │
                    ┌────────────┼────────────┐
                    │            │            │
              采集调度模块    分析评估模块    API 服务层
              Bull Queue      LLM DeepSeek   CRUD + 搜索
                    │            │
                    ▼            ▼
               MongoDB      Redis (Bull)
```

### 数据流

1. 定时任务/手动触发 → 采集调度
2. Bull 队列分发给 7+ 源 Worker → 爬取 → 存 MongoDB
3. 采集完成后触发分析队列 → LLM 评估 6 个维度 → 报告落地
4. 前端查询 API → 展示报告

---

## 数据模型（MongoDB Collections）

### projects

```typescript
interface Project {
  _id: ObjectId
  title: string
  description: string
  source: 'manual' | 'auto'
  sourceUrl: string | null
  status: 'draft' | 'collecting' | 'analyzing' | 'completed' | 'failed'
  createdAt: Date
  updatedAt: Date
}
```

### raw_pages

```typescript
interface RawPage {
  _id: ObjectId
  projectId: ObjectId
  source: 'product_hunt' | 'github' | '36kr' | 'hacker_news' | 'reddit' | 'twitter' | 'app_store'
  title: string
  url: string
  content: string
  summary: string
  tags: string[]
  metrics: {
    stars?: number
    votes?: number
    comments?: number
  }
  collectedAt: Date
}
```

### analysis_reports

```typescript
interface AnalysisReport {
  _id: ObjectId
  projectId: ObjectId
  dimensions: {
    market: { score: number; analysis: string; evidence: string[] }
    techFeasibility: { score: number; analysis: string; evidence: string[] }
    commercialPotential: { score: number; analysis: string; evidence: string[] }
    compliance: { score: number; analysis: string; evidence: string[] }
    overseas: { score: number; analysis: string; evidence: string[] }
    indieFriendliness: { score: number; analysis: string; evidence: string[] }
  }
  overallScore: number
  verdict: string
  riskAlerts: string[]
  createdAt: Date
}
```

### source_status

```typescript
interface SourceStatus {
  _id: ObjectId
  source: string
  lastRunAt: Date
  status: 'idle' | 'running' | 'failed'
  errorMessage: string | null
  itemsCollected: number
}
```

---

## 采集模块

### 7 个信源与采集方式

| 源 | 方式 | 频率 |
|---|---|---|
| Product Hunt | Playwright / RSS | 每天 |
| GitHub Trending | Playwright | 每天 |
| 36氪 | Playwright | 每天 |
| Hacker News | RSS / API | 每天 |
| Reddit | RSS | 每天 |
| Twitter/X | 第三方 API / RSS | 每天 |
| App Store | RSS | 每周 |

### 去重

采集前查 MongoDB 是否有相同 `url` 的记录，已有则跳过。

### 采集流程

```
cron: 每天早8点 / 晚8点
  → Scheduler（按源分类触发）
    → Bull Queue "collection-queue"
      → 7 个独立 Worker（每个源一个）
        → 入库 raw_pages
          → 触发分析队列
```

---

## 分析模块

### 流程

```
raw_pages 入库
  → Analysis Queue
    → LLM Analyzer (DeepSeek/Claude)
      → 6 维度评分 + 综合结论 + 风险提醒
        → analysis_reports 落地
```

### Prompt 设计

- 每个维度独立打分（1-10），附理由和证据来源
- LLM 引用具体 raw_pages 内容作为依据
- 风险提醒覆盖：法律、资质、竞争、技术、成本

### Token 预算

- 每个项目约 10-20K token
- 每天约 20 个项目 → 200-400K token
- 使用 DeepSeek 成本约几毛钱/天

---

## 前端页面

### 页面一：仪表盘（Dashboard）

- 综合统计卡（总项目、本周新增、待分析、平均分）
- 推荐项目 Top 5 排行
- 趋势看板：雷达图、词云、新增趋势、信源占比

### 页面二：项目详情

- 综合评分 + 风险提醒
- 六维评分雷达图 + 柱状图
- 每个维度可展开查看分析详情和引用来源
- 原始采集数据列表

### 页面三：项目管理

- 项目列表 + 筛选（全部/待采集/待分析/已完成/失败）
- 排序（综合评分/来源/时间）
- 新增想法、批量分析、导出

---

## 审核状态

- [x] 架构设计
- [x] 数据模型
- [x] 采集模块
- [x] 分析模块
- [x] 前端页面
- [ ] 实现计划（下一步）
