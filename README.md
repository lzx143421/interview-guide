<div align="center">

**智能 AI 面试官平台** - 基于大语言模型的简历分析和模拟面试系统

[![Java](https://img.shields.io/badge/Java-21-orange?logo=openjdk)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0-green?logo=springboot)](https://spring.io/projects/spring-boot)
[![React](https://img.shields.io/badge/React-18.3-blue?logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.6-blue?logo=typescript)](https://www.typescriptlang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-pgvector-336791?logo=postgresql)](https://www.postgresql.org/)


</div>


---

## 项目介绍

InterviewGuide 是一个集成了简历分析、模拟面试和知识库管理的智能面试辅助平台。系统利用大语言模型（LLM）和向量数据库技术，为求职者和 HR 提供智能化的简历评估和面试练习服务。

## 系统架构

**提示**：架构图采用 draw.io 绘制，导出为 svg 格式，在 Github Dark 模式下的显示效果会有问题。

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/interview-guide-architecture-diagram.svg)

**异步处理流程**：

简历分析、知识库向量化和面试报告生成采用 Redis Stream 异步处理，这里以简历分析和知识库向量化为例介绍一下整体流程：

```
上传请求 → 保存文件 → 发送消息到 Stream → 立即返回
                              ↓
                      Consumer 消费消息
                              ↓
                    执行分析/向量化任务
                              ↓
                      更新数据库状态
                              ↓
                   前端轮询获取最新状态
```

状态流转： `PENDING` → `PROCESSING` → `COMPLETED` / `FAILED`。

## 配套教程

本项目承诺**完整功能免费开源**，也不会做所谓的 Pro 版或“付费解锁核心功能”之类的设计。

如果你想学习这个项目，或者希望把它作为个人项目经历 / 毕设选题，我也整理了一套相对细致的教程：从基础设施搭建、核心业务实现，到最后如何在面试中讲清楚思路与亮点，尽量把容易卡住的地方讲透。

如果你确实需要更系统的辅导，可以点这里了解详情（**教程为付费内容**，主要是想覆盖一些时间成本，望理解，感谢支持）：[《SpringAI 智能面试平台+RAG 知识库》](https://javaguide.cn/zhuanlan/interview-guide.html)。

## 技术栈

### 后端技术

| 技术                  | 版本  | 说明                      |
| --------------------- | ----- | ------------------------- |
| Spring Boot           | 4.0   | 应用框架                  |
| Java                  | 21    | 开发语言                  |
| Spring AI             | 2.0   | AI 集成框架               |
| PostgreSQL + pgvector | 14+   | 关系数据库 + 向量存储     |
| Redis                 | 6+    | 缓存 + 消息队列（Stream） |
| Apache Tika           | 2.9.2 | 文档解析                  |
| iText 8               | 8.0.5 | PDF 导出                  |
| MapStruct             | 1.6.3 | 对象映射                  |
| Gradle                | 8.14  | 构建工具                  |

技术选型常见问题解答：

1. 数据存储为什么选择 PostgreSQL + pgvector？PG 的向量数据存储功能够用了，精简架构，不想引入太多组件。
2. 为什么引入 Redis？
   - Redis 替代 `ConcurrentHashMap` 实现面试会话的缓存。
   - 基于 Redis Stream 实现简历分析、知识库向量化等场景的异步（还能解耦，分析和向量化可以使用其他编程语言来做）。不使用 [Kafka](https://javaguide.cn/high-performance/message-queue/kafka-questions-01.html) 这类成熟的消息队列，也是不想引入太多组件。
3. 构建工具为什么选择 Gradle？个人更喜欢用 Gradle，也写过相关的文章：[Gradle核心概念总结](https://javaguide.cn/tools/gradle/gradle-core-concepts.html)。

### 前端技术

| 技术          | 版本  | 说明     |
| ------------- | ----- | -------- |
| React         | 18.3  | UI 框架  |
| TypeScript    | 5.6   | 开发语言 |
| Vite          | 5.4   | 构建工具 |
| Tailwind CSS  | 4.1   | 样式框架 |
| React Router  | 7.11  | 路由管理 |
| Framer Motion | 12.23 | 动画库   |
| Recharts      | 3.6   | 图表库   |
| Lucide React  | 0.468 | 图标库   |

## 功能特性

### 简历管理模块

- **多格式解析**：支持 PDF、DOCX、DOC、TXT 等多种简历格式。
- **异步处理流**：基于 Redis Stream 实现异步简历分析，支持实时查看处理进度（待分析/分析中/已完成/失败）。
- **稳定性保障**：内置分析失败自动重试机制（最多 3 次）与基于内容哈希的重复检测。
- **分析报告导出**：支持将 AI 分析结果一键导出为结构化的 PDF 简历分析报告。

### 模拟面试模块

- **个性化出题**：基于简历内容智能生成针对性的面试题目，支持实时问答交互。
- **智能追问流**：支持配置多轮智能追问（默认 1 条），构建模拟真实场景的线性问答流。
- **分批评估机制**：创新性采用分批评估策略，有效规避大模型 Token 溢出风险，确保长文本评估稳定性。
- **智能汇总建议**：对分批评估结果进行二次汇总，提供多维度的改进建议、表现趋势与统计信息。
- **报告一键导出**：支持异步生成并导出详细的 PDF 模拟面试评估报告。

### 知识库管理模块

- **文档智能处理**：支持 PDF、DOCX、Markdown 等多种格式文档的自动上传、分块与异步向量化。
- **RAG 检索增强**：集成向量数据库，通过检索增强生成（RAG）提升 AI 问答的准确性与专业度。
- **流式响应交互**：基于 SSE（Server-Sent Events）技术实现打字机式流式响应。
- **智能问答对话**：支持基于知识库内容的智能问答，并提供直观的知识库统计信息。

### TODO

- [x] 问答助手的 Markdown 展示优化
- [x] 知识库管理页面的知识库下载
- [x] 异步生成模拟面试评估报告
- [x] Docker 快速部署
- [x] 添加 API 限流保护
- [x] 前端性能优化（RAG 聊天 - 虚拟列表）
- [x] 模拟面试增加追问功能
- [ ] 打通模拟面试和知识库

## 效果展示

### 简历与面试

简历库：

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-resume-history.png)

简历上传分析：

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-resume-upload-analysis.png)

简历分析详情：

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-resume-analysis-detail.png)

面试记录：

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-interview-history.png)

面试详情：

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-interview-detail.png)

模拟面试：

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-mock-interview.png)

### 知识库

知识库管理：

![](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-knowledge-base-management.png)

问答助手：

![page-qa-assistant](https://oss.javaguide.cn/xingqiu/pratical-project/interview-guide/page-qa-assistant.png)

## 项目结构

```
interview-guide/
├── app/                              # 后端应用
│   ├── src/main/java/interview/guide/
│   │   ├── App.java                  # 主启动类
│   │   ├── common/                   # 通用模块
│   │   │   ├── config/               # 配置类
│   │   │   ├── exception/            # 异常处理
│   │   │   └── result/               # 统一响应
│   │   ├── infrastructure/           # 基础设施
│   │   │   ├── export/               # PDF 导出
│   │   │   ├── file/                 # 文件处理
│   │   │   ├── redis/                # Redis 服务
│   │   │   └── storage/              # 对象存储
│   │   └── modules/                  # 业务模块
│   │       ├── interview/            # 面试模块
│   │       ├── knowledgebase/        # 知识库模块
│   │       └── resume/               # 简历模块
│   └── src/main/resources/
│       ├── application.yml           # 应用配置
│       └── prompts/                  # AI 提示词模板
│
├── frontend/                         # 前端应用
│   ├── src/
│   │   ├── api/                      # API 接口
│   │   ├── components/               # 公共组件
│   │   ├── pages/                    # 页面组件
│   │   ├── types/                    # 类型定义
│   │   └── utils/                    # 工具函数
│   ├── package.json
│   └── vite.config.ts
│
└── README.md
```

## 快速开始

环境要求：

| 依赖          | 版本 | 必需 |
| ------------- | ---- | ---- |
| JDK           | 21+  | 是   |
| Node.js       | 18+  | 是   |
| PostgreSQL    | 14+  | 是   |
| pgvector 扩展 | -    | 是   |
| Redis         | 6+   | 是   |
| S3 兼容存储   | -    | 是   |

### 1. 克隆项目

```bash
git clone https://github.com/Snailclimb/interview-guide.git
cd interview-guide
```

### 2. 配置数据库

```sql
-- 创建数据库
CREATE DATABASE interview_guide;

-- 连接数据库并启用 pgvector 扩展（可选，启动后端SpringAI框架底层会自动创建）
CREATE EXTENSION vector;
```

### 3. 配置环境变量

```bash
# AI API 密钥（阿里云 DashScope）
export AI_BAILIAN_API_KEY=your_api_key
```

### 4. 修改应用配置

编辑 `app/src/main/resources/application.yml`：

```yaml
spring:
  # PostgreSQL数据库配置
  datasource:
    url: jdbc:postgresql://${POSTGRES_HOST:localhost}:${POSTGRES_PORT:5432}/${POSTGRES_DB:interview_guide}
    username: ${POSTGRES_USER:postgres}
    password: ${POSTGRES_PASSWORD:123456}
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: create #首次启动用 create，表创建成功后，改回 update

  # Redisson配置 (使用 spring.redis.redisson，参考官方文档)
  redis:
    redisson:
      config: |
        singleServerConfig:
          address: "redis://${REDIS_HOST:localhost}:${REDIS_PORT:6379}"
          database: 0
          connectionMinimumIdleSize: 10
          connectionPoolSize: 64
          subscriptionConnectionMinimumIdleSize: 1
          subscriptionConnectionPoolSize: 50

# RustFS (S3兼容) 存储配置
app:
  # 面试配置
  interview:
    follow-up-count: ${APP_INTERVIEW_FOLLOW_UP_COUNT:1}    # 每个主问题生成追问数量
    evaluation:
      batch-size: ${APP_INTERVIEW_EVALUATION_BATCH_SIZE:8} # 回答评估分批大小
  storage:
    endpoint: ${APP_STORAGE_ENDPOINT:http://localhost:9000}
    access-key: ${APP_STORAGE_ACCESS_KEY:wr45VXJZhCxc6FAWz0YR}
    secret-key: ${APP_STORAGE_SECRET_KEY:GtKxV57WJkpw4CvASPBzTy2DYElLnRqh8dIXQa0m}
    bucket: ${APP_STORAGE_BUCKET:interview-guide}
    region: ${APP_STORAGE_REGION:us-east-1}



```

⚠️**注意**：

1. JPA 的 `ddl-auto` 首次启动用 `create`，表创建成功后，改回 `update`。
2. 如果本地有 Minio 的话，可以用其替换 RusfFS。

### 5. 启动服务

**后端：**

```bash
./gradlew bootRun
```

后端服务启动于 `http://localhost:8080`

**前端：**

```bash
cd frontend
pnpm install
pnpm dev
```

前端服务启动于 `http://localhost:5173`


## Docker 快速部署

本项目提供了完整的 Docker 支持，可以一键启动所有服务（前后端、数据库、中间件）。

### 1. 前置准备
- 安装 [Docker](https://www.docker.com/products/docker-desktop/) 和 Docker Compose
- 申请阿里云百炼 API Key（用于 AI 对话功能）

### 2. 快速启动
在项目根目录下执行：

```bash
# 1. 复制环境变量配置文件
cp .env.example .env

# 2. 编辑 .env 文件，填入 AI 配置
# vim .env
# 必填：AI_BAILIAN_API_KEY=your_key_here
# 可选：AI_MODEL=qwen-plus        # 默认值为 qwen-plus
#        # 也可以改为 qwen-max、qwen-long 等其他可用模型
#
# 面试参数配置（可选）：
# APP_INTERVIEW_FOLLOW_UP_COUNT=1         # 每个主问题生成追问数量（默认 1）
# APP_INTERVIEW_EVALUATION_BATCH_SIZE=8   # 回答评估分批大小（默认 8）

# 3. 构建并启动所有服务
docker-compose up -d --build
```

### 3. 服务访问
启动完成后，您可以通过以下地址访问各个服务：

| 服务             | 地址                                           | 默认账号     | 默认密码     | 说明                   |
| ---------------- | ---------------------------------------------- | ------------ | ------------ | ---------------------- |
| **前端应用**     | [http://localhost](http://localhost)           | -            | -            | 用户访问入口           |
| **后端 API**     | [http://localhost:8080](http://localhost:8080) | -            | -            | Swagger/接口文档       |
| **MinIO 控制台** | [http://localhost:9001](http://localhost:9001) | `minioadmin` | `minioadmin` | 对象存储管理           |
| **MinIO API**    | `localhost:9000`                               | -            | -            | S3 兼容接口            |
| **PostgreSQL**   | `localhost:5432`                               | `postgres`   | `password`   | 数据库 (包含 pgvector) |
| **Redis**        | `localhost:6379`                               | -            | -            | 缓存与消息队列         |

### 4. 常用运维命令

```bash
# 查看服务状态
docker-compose ps

# 查看后端日志
docker-compose logs -f app

# 停止并移除所有服务
docker-compose down

# 清理无用镜像（构建产生的中间层）
docker image prune -f
```

## 使用场景

| 用户角色        | 使用场景                               |
| --------------- | -------------------------------------- |
| **求职者**      | 上传简历获取分析建议，进行模拟面试练习 |
| **HR/招聘人员** | 批量分析简历，评估候选人能力           |
| **培训机构**    | 提供面试培训服务，管理知识库资源       |

## 常见问题

### Q: 数据库表创建失败/数据丢失

这大概率是 JPA 的 `ddl-auto` 配置不对的原因。`ddl-auto` 模式对比：

| 模式     | 行为                            | 适用场景      |
| -------- | ------------------------------- | ------------- |
| create   | 无条件删除并重建所有表          | 开发/测试环境 |
| update   | 对比现有 schema，只执行增量更新 | 开发环境      |
| validate | 只验证，不修改                  | 生产环境      |
| none     | 什么都不做                      | 生产环境      |

对于新数据库，推荐：

```yaml
# 首次启动用 create
jpa:
  hibernate:
    ddl-auto: create

# 表创建成功后，改回 update
jpa:
  hibernate:
    ddl-auto: update
```

记得改回 **update**，否则每次重启都会删除所有数据！

### Q: 知识库向量化失败

当 `initialize-schema: false` 时，Spring AI **不会自动创建** `vector_store` 表。

```java
spring:
  ai:
    vectorstore:
      pgvector:
        initialize-schema: true 

```

建议开发环境设置为 true，方便快速启动。生产环境设置为 false，手动管理数据库 schema，避免意外变更。

### Q: 简历分析失败

检查一下阿里云 DashScope API KEY 是否配置正确（申请地址：<https://bailian.console.aliyun.com/>）。

### Q: 简历分析一直显示"分析中"？

检查 Redis 连接和 Stream Consumer 是否正常运行。查看后端日志确认是否有错误。

### Q: PDF 导出失败或中文显示异常？

项目已内置中文字体（珠圆玉润仿宋），支持跨平台导出。如遇到问题，请检查：
- 字体文件是否存在：`app/src/main/resources/fonts/ZhuqueFangsong-Regular.ttf`
- 检查日志中的字体加载信息
- 确认 iText 依赖是否正确

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

AGPL-3.0 License（只要通过网络提供服务，就必须向用户公开修改后的源码）

---

## 项目架构详解

### 整体架构设计

本项目采用**前后端分离架构**，后端基于 Spring Boot 4.0 + Java 21，前端基于 React 18.3 + TypeScript。整体架构分为以下层次：

#### 1. 技术架构分层

```
┌─────────────────────────────────────────────────────────┐
│                    前端层 (Frontend)                      │
│  React 18 + TypeScript + Vite + Tailwind CSS + Recharts  │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP/REST + SSE
┌────────────────────────▼────────────────────────────────┐
│                  API 网关层 (Controller)                  │
│  ResumeController | InterviewController | KnowledgeBase  │
│  Controller | RagChatController | RateLimitAspect        │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                  业务逻辑层 (Service)                     │
│  ResumeUploadService | InterviewSessionService |         │
│  KnowledgeBaseQueryService | RagChatSessionService       │
└───────┬──────────────────────┬──────────────────────────┘
        │                      │
┌───────▼──────┐    ┌──────────▼──────────┐
│  异步任务层   │    │   AI 集成层         │
│ Redis Stream │    │ Spring AI 2.0       │
│ Producer/    │    │ - ChatClient (LLM)  │
│ Consumer     │    │ - VectorStore       │
└───────┬──────┘    │ - EmbeddingModel    │
        │           └──────────┬──────────┘
        │                      │
┌───────▼──────────────────────▼──────────────────────────┐
│                  数据存储层 (Infrastructure)              │
│  PostgreSQL + pgvector | Redis | S3兼容存储(RustFS)      │
└─────────────────────────────────────────────────────────┘
```

#### 2. 模块划分

后端采用**模块化设计**，主要分为四大模块：

**① 简历管理模块 (resume)**
- **职责**：简历上传、解析、AI分析、去重、历史管理、PDF导出
- **核心组件**：
  - `ResumeController`：REST API接口
  - `ResumeUploadService`：上传与异步分析触发
  - `ResumeGradingService`：AI简历评分与分析
  - `ResumePersistenceService`：数据持久化
  - `ResumeHistoryService`：历史记录查询
  - `AnalyzeStreamProducer/Consumer`：Redis Stream异步处理

**② 模拟面试模块 (interview)**
- **职责**：面试会话管理、智能出题、答案提交、分批评估、报告生成
- **核心组件**：
  - `InterviewController`：REST API接口
  - `InterviewSessionService`：会话管理与问答流程
  - `InterviewHistoryService`：历史记录与PDF导出
  - `InterviewPersistenceService`：数据持久化
  - `EvaluateStreamConsumer`：异步评估消费者

**③ 知识库管理模块 (knowledgebase)**
- **职责**：文档上传、分块向量化、RAG检索、智能问答
- **核心组件**：
  - `KnowledgeBaseController`：知识库管理API
  - `RagChatController`：RAG聊天会话API
  - `KnowledgeBaseUploadService`：上传与向量化触发
  - `KnowledgeBaseQueryService`：RAG查询与流式响应
  - `KnowledgeBaseVectorService`：向量存储与检索
  - `RagChatSessionService`：聊天会话管理
  - `VectorizeStreamProducer/Consumer`：异步向量化处理

**④ 通用基础设施 (common + infrastructure)**
- **职责**：提供全局配置、异常处理、限流、异步框架、文件处理、存储等
- **核心组件**：
  - `RateLimit + RateLimitAspect`：基于Redis的分布式限流
  - `GlobalExceptionHandler`：统一异常处理
  - `Result<T>`：统一响应封装
  - `AbstractStreamProducer/Consumer`：Redis Stream异步处理抽象基类
  - `RedisService`：Redis操作封装
  - `DocumentParseService`：Apache Tika文档解析
  - `PdfExportService`：iText 8 PDF导出
  - `StorageService`：S3兼容对象存储

#### 3. 异步处理架构

项目采用 **Redis Stream** 实现异步任务处理，核心流程：

```
上传请求 → 保存文件 → 发送消息到Stream → 立即返回任务ID
                              ↓
                    Consumer后台消费消息
                              ↓
                  执行AI分析/向量化任务
                              ↓
                    更新数据库状态字段
                              ↓
                  前端轮询获取最新状态
```

**支持的异步任务**：
- 简历分析：`RESUME_ANALYZE_STREAM`
- 知识库向量化：`KB_VECTORIZE_STREAM`
- 面试评估：`INTERVIEW_EVALUATE_STREAM`

**状态流转**：`PENDING` → `PROCESSING` → `COMPLETED` / `FAILED`

**重试机制**：失败任务自动重新入队，最多重试3次

#### 4. 数据模型

**核心实体关系**：
```
ResumeEntity (1) ──────< (N) ResumeAnalysisEntity
    │
    └──────< (N) InterviewSessionEntity
                │
                └──────< (N) InterviewAnswerEntity

KnowledgeBaseEntity (独立实体，通过向量元数据关联)
    │
    └──────< (N) RagChatSessionEntity
                │
                └──────< (N) RagChatMessageEntity
```

---

## API 接口文档

### 通用说明

**基础路径**：`http://localhost:8080`

**统一响应格式**：
```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

**错误响应格式**：
```json
{
  "code": 500,
  "message": "错误描述",
  "data": null
}
```

**限流说明**：部分接口标注了 `@RateLimit`，表示启用了限流保护。

---

### 一、简历管理接口

#### 1.1 上传简历并分析

```
POST /api/resumes/upload
Content-Type: multipart/form-data
限流: 全局+IP，5次/秒
```

**请求参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| file | File | 是 | 简历文件（支持PDF、DOCX、DOC、TXT、MD） |

**响应示例**：
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "filename": "张三-Java开发工程师.pdf",
    "fileSize": 204800,
    "contentType": "application/pdf",
    "analyzeStatus": "PENDING",
    "duplicate": false
  }
}
```

#### 1.2 获取简历列表

```
GET /api/resumes
```

**响应示例**：
```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "filename": "张三-简历.pdf",
      "fileSize": 204800,
      "uploadedAt": "2024-01-15T10:30:00",
      "accessCount": 5,
      "analyzeStatus": "COMPLETED"
    }
  ]
}
```

#### 1.3 获取简历详情

```
GET /api/resumes/{id}/detail
```

**路径参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | Long | 是 | 简历ID |

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "id": 1,
    "filename": "张三-简历.pdf",
    "fileSize": 204800,
    "contentType": "application/pdf",
    "storageUrl": "http://...",
    "uploadedAt": "2024-01-15T10:30:00",
    "accessCount": 5,
    "resumeText": "解析后的简历文本...",
    "analyzeStatus": "COMPLETED",
    "analyzeError": null,
    "analyses": [
      {
        "id": 1,
        "overallScore": 85,
        "contentScore": 88,
        "structureScore": 82,
        "skillMatchScore": 90,
        "expressionScore": 80,
        "projectScore": 85,
        "summary": "简历整体质量较好...",
        "analyzedAt": "2024-01-15T10:31:00",
        "strengths": ["技术栈匹配度高", "项目经验丰富"],
        "suggestions": [...]
      }
    ],
    "interviews": []
  }
}
```

#### 1.4 导出简历分析报告PDF

```
GET /api/resumes/{id}/export
```

**响应**：返回PDF文件流（application/pdf）

#### 1.5 删除简历

```
DELETE /api/resumes/{id}
```

#### 1.6 重新分析简历

```
POST /api/resumes/{id}/reanalyze
限流: 全局+IP，2次/秒
```

用于分析失败后手动重试。

#### 1.7 健康检查

```
GET /api/resumes/health
```

---

### 二、模拟面试接口

#### 2.1 创建面试会话

```
POST /api/interview/sessions
限流: 全局+IP，5次/秒
```

**请求体**：
```json
{
  "resumeId": 1,
  "questionCount": 5
}
```

**请求参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| resumeId | Long | 是 | 简历ID |
| questionCount | Integer | 是 | 题目数量 |

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "sessionId": "uuid-string",
    "resumeId": 1,
    "totalQuestions": 5,
    "status": "CREATED"
  }
}
```

#### 2.2 获取会话信息

```
GET /api/interview/sessions/{sessionId}
```

#### 2.3 获取当前问题

```
GET /api/interview/sessions/{sessionId}/question
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "questionIndex": 0,
    "question": "请介绍一下你在Spring Boot项目中的经验...",
    "category": "技术深度",
    "totalQuestions": 5
  }
}
```

#### 2.4 提交答案

```
POST /api/interview/sessions/{sessionId}/answers
限流: 全局，10次/秒
```

**请求体**：
```json
{
  "questionIndex": 0,
  "answer": "我在Spring Boot项目中有3年经验..."
}
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "nextQuestionIndex": 1,
    "hasNext": true
  }
}
```

#### 2.5 暂存答案

```
PUT /api/interview/sessions/{sessionId}/answers
```

暂存答案但不进入下一题。

#### 2.6 提前交卷

```
POST /api/interview/sessions/{sessionId}/complete
```

#### 2.7 获取面试报告

```
GET /api/interview/sessions/{sessionId}/report
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "sessionId": "uuid",
    "overallScore": 82,
    "overallFeedback": "整体表现良好...",
    "strengths": ["基础知识扎实", "表达清晰"],
    "improvements": ["深度学习框架经验不足"],
    "questions": [...],
    "referenceAnswers": [...]
  }
}
```

#### 2.8 查找未完成会话

```
GET /api/interview/sessions/unfinished/{resumeId}
```

#### 2.9 获取面试详情

```
GET /api/interview/sessions/{sessionId}/details
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "id": 1,
    "sessionId": "uuid",
    "totalQuestions": 5,
    "status": "EVALUATED",
    "evaluateStatus": "COMPLETED",
    "overallScore": 82,
    "overallFeedback": "整体表现良好...",
    "createdAt": "2024-01-15T14:00:00",
    "completedAt": "2024-01-15T14:30:00",
    "questions": [...],
    "strengths": [...],
    "improvements": [...],
    "referenceAnswers": [...],
    "answers": [
      {
        "questionIndex": 0,
        "question": "请介绍Spring Boot经验...",
        "category": "技术深度",
        "userAnswer": "我有3年经验...",
        "score": 85,
        "feedback": "回答较为全面...",
        "referenceAnswer": "标准答案...",
        "keyPoints": ["项目经验", "技术细节"],
        "answeredAt": "2024-01-15T14:05:00"
      }
    ]
  }
}
```

#### 2.10 导出面试报告PDF

```
GET /api/interview/sessions/{sessionId}/export
```

**响应**：返回PDF文件流

#### 2.11 删除面试会话

```
DELETE /api/interview/sessions/{sessionId}
```

---

### 三、知识库管理接口

#### 3.1 获取知识库列表

```
GET /api/knowledgebase/list?sortBy=time&vectorStatus=COMPLETED
```

**查询参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| sortBy | String | 否 | 排序方式（time/name/size） |
| vectorStatus | String | 否 | 向量化状态（PENDING/PROCESSING/COMPLETED/FAILED） |

**响应示例**：
```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "name": "Java面试宝典",
      "category": "Java面试",
      "originalFilename": "java-interview.pdf",
      "fileSize": 1024000,
      "contentType": "application/pdf",
      "uploadedAt": "2024-01-15T10:00:00",
      "accessCount": 10,
      "questionCount": 5,
      "vectorStatus": "COMPLETED",
      "vectorError": null,
      "chunkCount": 45
    }
  ]
}
```

#### 3.2 获取知识库详情

```
GET /api/knowledgebase/{id}
```

#### 3.3 删除知识库

```
DELETE /api/knowledgebase/{id}
```

#### 3.4 查询知识库（RAG）

```
POST /api/knowledgebase/query
限流: 全局+IP，10次/秒
```

**请求体**：
```json
{
  "knowledgeBaseIds": [1, 2],
  "question": "什么是Spring Bean的生命周期？"
}
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "answer": "Spring Bean的生命周期包括...",
    "sourceDocuments": [...]
  }
}
```

#### 3.5 查询知识库（流式SSE）

```
POST /api/knowledgebase/query/stream
Content-Type: application/json
Accept: text/event-stream
限流: 全局+IP，5次/秒
```

**请求体**：同上

**响应**：SSE流式响应，逐字返回AI回答

#### 3.6 获取所有分类

```
GET /api/knowledgebase/categories
```

#### 3.7 按分类获取知识库

```
GET /api/knowledgebase/category/{category}
```

#### 3.8 获取未分类知识库

```
GET /api/knowledgebase/uncategorized
```

#### 3.9 更新知识库分类

```
PUT /api/knowledgebase/{id}/category
```

**请求体**：
```json
{
  "category": "Java面试"
}
```

#### 3.10 上传知识库文件

```
POST /api/knowledgebase/upload
Content-Type: multipart/form-data
限流: 全局+IP，3次/秒
```

**请求参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| file | File | 是 | 文档文件（支持PDF、DOCX、MD等） |
| name | String | 否 | 自定义名称 |
| category | String | 否 | 分类名称 |

#### 3.11 下载知识库文件

```
GET /api/knowledgebase/{id}/download
```

**响应**：返回文件流

#### 3.12 搜索知识库

```
GET /api/knowledgebase/search?keyword=spring
```

#### 3.13 获取统计信息

```
GET /api/knowledgebase/stats
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "totalDocuments": 10,
    "totalChunks": 450,
    "totalQuestions": 25,
    "vectorizedDocuments": 9
  }
}
```

#### 3.14 重新向量化

```
POST /api/knowledgebase/{id}/revectorize
限流: 全局+IP，2次/秒
```

---

### 四、RAG聊天会话接口

#### 4.1 创建聊天会话

```
POST /api/rag-chat/sessions
```

**请求体**：
```json
{
  "knowledgeBaseIds": [1, 2],
  "title": "Java技术问答"
}
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "id": 1,
    "title": "Java技术问答",
    "knowledgeBaseIds": [1, 2],
    "createdAt": "2024-01-15T10:00:00"
  }
}
```

#### 4.2 获取会话列表

```
GET /api/rag-chat/sessions
```

**响应示例**：
```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "title": "Java技术问答",
      "messageCount": 10,
      "knowledgeBaseNames": ["Java面试宝典", "Spring实战"],
      "updatedAt": "2024-01-15T11:00:00",
      "isPinned": true
    }
  ]
}
```

#### 4.3 获取会话详情

```
GET /api/rag-chat/sessions/{sessionId}
```

**响应示例**：
```json
{
  "code": 200,
  "data": {
    "id": 1,
    "title": "Java技术问答",
    "knowledgeBases": [...],
    "messages": [
      {
        "id": 1,
        "type": "user",
        "content": "什么是Spring Bean？",
        "createdAt": "2024-01-15T10:05:00"
      },
      {
        "id": 2,
        "type": "assistant",
        "content": "Spring Bean是Spring容器管理的对象...",
        "createdAt": "2024-01-15T10:05:01"
      }
    ],
    "createdAt": "2024-01-15T10:00:00",
    "updatedAt": "2024-01-15T10:05:01"
  }
}
```

#### 4.4 更新会话标题

```
PUT /api/rag-chat/sessions/{sessionId}/title
```

**请求体**：
```json
{
  "title": "新标题"
}
```

#### 4.5 置顶/取消置顶会话

```
PUT /api/rag-chat/sessions/{sessionId}/pin
```

#### 4.6 更新会话知识库

```
PUT /api/rag-chat/sessions/{sessionId}/knowledge-bases
```

**请求体**：
```json
{
  "knowledgeBaseIds": [1, 3, 5]
}
```

#### 4.7 删除会话

```
DELETE /api/rag-chat/sessions/{sessionId}
```

#### 4.8 发送消息（流式SSE）

```
POST /api/rag-chat/sessions/{sessionId}/messages/stream
Accept: text/event-stream
```

**请求体**：
```json
{
  "question": "解释一下依赖注入的原理"
}
```

**响应**：SSE流式响应，实时返回AI回答内容

---

### 五、错误码说明

| 错误码 | 说明 |
|--------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 404 | 资源不存在 |
| 429 | 请求过于频繁（触发限流） |
| 500 | 服务器内部错误 |

---

### 六、关键技术实现细节

#### 1. 限流机制

基于 **Redis + Lua脚本** 实现令牌桶限流：
- 支持多维度限流：全局、IP、用户
- 可组合使用：`@RateLimit(dimensions = {GLOBAL, IP}, count = 5)`
- 滑动时间窗口算法，精准控制请求频率

#### 2. 异步任务处理

基于 **Redis Stream** 实现可靠的异步消息队列：
- Producer发送任务到Stream
- Consumer独立线程消费，支持重试
- 消息ACK机制保证不丢失
- 失败任务自动重新入队（最多3次）

#### 3. RAG检索增强

采用**智能检索策略**：
- 查询重写：使用LLM优化用户问题
- 动态Top-K：根据查询长度调整检索数量
  - 短查询（≤4字）：Top-20，最低相似度0.18
  - 中等查询：Top-12
  - 长查询：Top-8，最低相似度0.28
- 多知识库支持：可同时检索多个知识库

#### 4. 面试评估策略

采用**分批评估机制**：
- 避免大模型Token溢出
- 每批评估8个答案（可配置）
- 最终二次汇总生成完整报告
- 异步评估，不阻塞用户交互

#### 5. 流式响应（SSE）

基于 **Spring WebFlux Flux** 实现：
- 打字机效果，实时返回AI生成内容
- 虚拟线程支持（Java 21），提升并发性能
- 错误处理：流式中断时保存已生成内容

#### 6. 文档解析与向量化

- **文档解析**：Apache Tika支持PDF、DOCX、TXT、MD等多种格式
- **文本分块**：TokenTextSplitter，每块约500 tokens，重叠50 tokens
- **向量化**：阿里云text-embedding-v3模型，1024维向量
- **向量存储**：PostgreSQL + pgvector，HNSW索引，余弦距离

#### 7. 文件存储

- 使用 **S3兼容存储**（RustFS/MinIO）
- 文件去重：SHA-256内容哈希
- 支持文件下载与URL访问

---

### 七、数据库表结构

#### 核心表清单

| 表名 | 说明 | 关键字段 |
|------|------|----------|
| resumes | 简历表 | id, fileHash, originalFilename, analyzeStatus |
| resume_analyses | 简历分析结果表 | id, resume_id, overallScore, contentScore, structureScore... |
| interview_sessions | 面试会话表 | id, sessionId, resume_id, totalQuestions, status, evaluateStatus |
| interview_answers | 面试答案表 | id, session_id, questionIndex, question, userAnswer, score |
| knowledge_bases | 知识库表 | id, fileHash, name, category, vectorStatus, chunkCount |
| rag_chat_sessions | RAG聊天会话表 | id, title, knowledgeBaseIds, isPinned |
| rag_chat_messages | RAG聊天消息表 | id, session_id, type, content |
| vector_store | 向量存储表（SpringAI管理） | id, embedding, metadata, content |

---

### 八、配置文件说明

#### application.yml 核心配置

```yaml
server:
  port: 8080
  threads:
    virtual:
      enabled: true  # 启用Java 21虚拟线程

spring:
  ai:
    openai:
      base-url: https://dashscope.aliyuncs.com/compatible-mode
      api-key: ${AI_BAILIAN_API_KEY}
      chat:
        options:
          model: ${AI_MODEL:qwen-plus}
          temperature: 0.2
      embedding:
        options:
          model: text-embedding-v3
    vectorstore:
      pgvector:
        index-type: HNSW
        distance-type: COSINE_DISTANCE
        dimensions: 1024

app:
  interview:
    follow-up-count: ${APP_INTERVIEW_FOLLOW_UP_COUNT:1}
    evaluation:
      batch-size: ${APP_INTERVIEW_EVALUATION_BATCH_SIZE:8}
  ai:
    rag:
      rewrite:
        enabled: ${APP_AI_RAG_REWRITE_ENABLED:true}
      search:
        short-query-length: ${APP_AI_RAG_SHORT_QUERY_LENGTH:4}
        topk-short: ${APP_AI_RAG_TOPK_SHORT:20}
        topk-medium: ${APP_AI_RAG_TOPK_MEDIUM:12}
        topk-long: ${APP_AI_RAG_TOPK_LONG:8}
```

---

### 九、部署架构

```
                    ┌─────────────┐
                    │   Nginx     │
                    │ (前端+反向  │
                    │  代理)      │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
        ┌─────▼─────┐          ┌───────▼───────┐
        │  Frontend │          │  Backend      │
        │  React    │          │  Spring Boot  │
        │  :5173    │          │  :8080        │
        └───────────┘          └───┬───┬───┬───┘
                                   │   │   │
                    ┌──────────────┘   │   └──────────────┐
                    │                  │                   │
              ┌─────▼─────┐   ┌───────▼──────┐   ┌───────▼──────┐
              │ PostgreSQL│   │    Redis     │   │   S3存储     │
              │ +pgvector │   │  Stream+Cache│   │  RustFS/MinIO│
              └───────────┘   └──────────────┘   └──────────────┘
```

**Docker部署**：支持一键启动所有服务（详见快速开始-Docker部署章节）

---

## 前端项目架构详解

### 前端技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| React | 18.3 | UI框架 |
| TypeScript | 5.6 | 类型安全 |
| Vite | 5.4 | 构建工具 |
| Tailwind CSS | 4.1 | 样式框架 |
| React Router | 7.11 | 路由管理 |
| Axios | 1.7 | HTTP客户端 |
| Framer Motion | 12.23 | 动画库 |
| Recharts | 3.6 | 图表库 |
| React Markdown | 9.0 | Markdown渲染 |
| React Virtuoso | 4.18 | 虚拟列表 |
| Lucide React | 0.468 | 图标库 |

### 前端项目结构

```
frontend/
├── src/
│   ├── api/                          # API接口层
│   │   ├── request.ts                # Axios封装与拦截器
│   │   ├── index.ts                  # 统一导出
│   │   ├── resume.ts                 # 简历相关API
│   │   ├── interview.ts              # 面试相关API
│   │   ├── knowledgebase.ts          # 知识库相关API
│   │   ├── history.ts                # 历史记录相关API
│   │   └── ragChat.ts                # RAG聊天相关API
│   │
│   ├── components/                   # 公共组件
│   │   ├── Layout.tsx                # 主布局（侧边栏+内容区）
│   │   ├── AnalysisPanel.tsx         # 分析报告面板
│   │   ├── CodeBlock.tsx             # 代码高亮组件
│   │   ├── ConfirmDialog.tsx         # 确认对话框
│   │   ├── DeleteConfirmDialog.tsx   # 删除确认对话框
│   │   ├── FileUploadCard.tsx        # 文件上传卡片
│   │   ├── HistoryList.tsx           # 历史记录列表
│   │   ├── InterviewChatPanel.tsx    # 面试聊天面板
│   │   ├── InterviewConfigPanel.tsx  # 面试配置面板
│   │   ├── InterviewDetailPanel.tsx  # 面试详情面板
│   │   ├── InterviewPanel.tsx        # 面试问答面板
│   │   ├── RadarChart.tsx            # 雷达图组件
│   │   └── ScoreProgressBar.tsx      # 分数进度条
│   │
│   ├── pages/                        # 页面组件
│   │   ├── UploadPage.tsx            # 简历上传页
│   │   ├── HistoryPage.tsx           # 简历库列表页
│   │   ├── ResumeDetailPage.tsx      # 简历详情页
│   │   ├── InterviewPage.tsx         # 模拟面试页
│   │   ├── InterviewHistoryPage.tsx  # 面试记录页
│   │   ├── KnowledgeBaseManagePage.tsx   # 知识库管理页
│   │   ├── KnowledgeBaseUploadPage.tsx   # 知识库上传页
│   │   └── KnowledgeBaseQueryPage.tsx    # 问答助手页
│   │
│   ├── types/                        # 类型定义
│   │   ├── resume.ts                 # 简历相关类型
│   │   └── interview.ts              # 面试相关类型
│   │
│   ├── hooks/                        # 自定义Hooks
│   │   └── useTheme.ts               # 主题切换Hook
│   │
│   ├── utils/                        # 工具函数
│   │   ├── date.ts                   # 日期格式化
│   │   └── score.ts                  # 分数计算
│   │
│   ├── App.tsx                       # 根组件（路由配置）
│   ├── main.tsx                      # 入口文件
│   └── index.css                     # 全局样式
│
├── public/                           # 静态资源
│   └── favicon.svg
│
├── package.json                      # 依赖配置
├── vite.config.ts                    # Vite配置
├── tsconfig.json                     # TypeScript配置
└── tailwind.config.js                # Tailwind配置
```

### 前端架构设计

#### 1. 分层架构

```
┌─────────────────────────────────────────┐
│          页面层 (Pages)                  │
│  8个页面组件，每个页面对应一个路由         │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│         组件层 (Components)              │
│  13个可复用UI组件                         │
│  - 布局组件：Layout                      │
│  - 业务组件：AnalysisPanel, InterviewPanel│
│  - 通用组件：ConfirmDialog, CodeBlock    │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│          API层 (API)                     │
│  6个API模块，封装所有后端接口调用          │
│  - request.ts: Axios统一封装             │
│  - resume.ts: 简历API                    │
│  - interview.ts: 面试API                 │
│  - knowledgebase.ts: 知识库API           │
│  - history.ts: 历史记录API               │
│  - ragChat.ts: RAG聊天API                │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│         类型层 (Types)                   │
│  TypeScript类型定义，保证类型安全          │
└─────────────────────────────────────────┘
```

#### 2. 路由设计

采用 **React Router 7** 实现路由管理，支持懒加载：

```typescript
路由结构：
/                           → 重定向到 /upload
/upload                     → 简历上传页
/history                    → 简历库列表页
/history/:resumeId          → 简历详情页
/interviews                 → 面试记录列表页
/interview/:resumeId        → 模拟面试页
/knowledgebase              → 知识库管理页
/knowledgebase/upload       → 知识库上传页
/knowledgebase/chat         → 问答助手页
```

**懒加载优化**：所有页面组件使用 `lazy()` 动态导入，配合 `Suspense` 实现代码分割。

#### 3. API请求架构

**统一请求封装** (`request.ts`)：

```typescript
// Axios实例配置
const instance = axios.create({
  baseURL: 'http://localhost:8080',  // 开发环境
  timeout: 60000,
});

// 响应拦截器：统一处理后端Result格式
instance.interceptors.response.use(
  (response) => {
    const result = response.data as Result;
    if (result.code === 200) {
      response.data = result.data;  // 提取data字段
      return response;
    }
    return Promise.reject(new Error(result.message));
  },
  (error) => {
    // 统一错误处理
    return Promise.reject(new Error(getErrorMessage(error)));
  }
);
```

**支持的请求方法**：
- `request.get<T>()` - GET请求
- `request.post<T>()` - POST请求
- `request.put<T>()` - PUT请求
- `request.delete<T>()` - DELETE请求
- `request.upload<T>()` - 文件上传（120秒超时）
- `request.getInstance()` - 获取原始Axios实例（用于Blob下载）

#### 4. SSE流式响应实现

项目中有两处使用SSE流式响应：

**① 知识库查询流式响应** (`knowledgebase.ts`)：
```typescript
async queryKnowledgeBaseStream(
  req: QueryRequest,
  onMessage: (chunk: string) => void,
  onComplete: () => void,
  onError: (error: Error) => void
): Promise<void>
```

**② RAG聊天流式响应** (`ragChat.ts`)：
```typescript
async sendMessageStream(
  sessionId: number,
  question: string,
  onMessage: (chunk: string) => void,
  onComplete: () => void,
  onError: (error: Error) => void
): Promise<void>
```

**实现原理**：
- 使用 Fetch API 的 `ReadableStream` 读取响应
- 使用 `TextDecoder` 解码二进制数据
- 解析SSE格式（`data: content\n\n`）
- 通过回调函数逐块返回内容

#### 5. 状态管理

采用 **React Hooks + Props** 进行状态管理，未引入Redux等状态管理库：

- 使用 `useState` 管理组件内部状态
- 使用 `useEffect` 处理副作用（数据加载、轮询等）
- 使用 `useCallback` 和 `useMemo` 优化性能
- 通过Props传递状态和回调函数

#### 6. 性能优化

**代码分割**：
```typescript
// vite.config.ts
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        'react-vendor': ['react', 'react-dom', 'react-router-dom'],
        'ui-vendor': ['framer-motion', 'lucide-react'],
        'syntax-highlighter': ['react-syntax-highlighter'],
      },
    },
  },
}
```

**虚拟列表**：使用 `react-virtuoso` 优化长列表渲染（问答助手消息列表）

**懒加载**：所有页面组件使用 `React.lazy()` 延迟加载

**超时配置**：
- 普通请求：60秒
- 文件上传：120秒
- AI生成（创建会话/提交答案/获取报告）：180秒

#### 7. 主题系统

支持深色/浅色主题切换：

```typescript
// hooks/useTheme.ts
export function useTheme() {
  const [theme, setTheme] = useState<'light' | 'dark'>(() => {
    return localStorage.getItem('theme') as 'light' | 'dark' || 'light';
  });

  const toggleTheme = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';
    setTheme(newTheme);
    localStorage.setItem('theme', newTheme);
    document.documentElement.classList.toggle('dark', newTheme === 'dark');
  };

  return { theme, toggleTheme };
}
```

---

## 前端接口调用详情

### 一、简历模块接口调用

#### 1.1 上传简历

**API文件**：`api/resume.ts`

```typescript
// 调用方法
resumeApi.uploadAndAnalyze(file: File): Promise<UploadResponse>

// 接口调用
POST /api/resumes/upload
Content-Type: multipart/form-data

// 响应类型
interface UploadResponse {
  analysis?: ResumeAnalysisResponse;  // 异步模式可能为空
  storage: StorageInfo;
  duplicate?: boolean;
  message?: string;
}
```

**使用场景**：`UploadPage.tsx` - 用户上传简历文件

#### 1.2 健康检查

```typescript
resumeApi.healthCheck(): Promise<{ status: string; service: string }>

// 接口调用
GET /api/resumes/health
```

---

### 二、历史记录模块接口调用

**API文件**：`api/history.ts`

#### 2.1 获取简历列表

```typescript
historyApi.getResumes(): Promise<ResumeListItem[]>

// 接口调用
GET /api/resumes

// 响应类型
interface ResumeListItem {
  id: number;
  filename: string;
  fileSize: number;
  uploadedAt: string;
  accessCount: number;
  latestScore?: number;
  lastAnalyzedAt?: string;
  interviewCount: number;
  analyzeStatus?: AnalyzeStatus;  // PENDING | PROCESSING | COMPLETED | FAILED
  analyzeError?: string;
  storageUrl?: string;
}
```

**使用场景**：`HistoryPage.tsx` - 简历库列表展示

#### 2.2 获取简历详情

```typescript
historyApi.getResumeDetail(id: number): Promise<ResumeDetail>

// 接口调用
GET /api/resumes/{id}/detail

// 响应类型
interface ResumeDetail {
  id: number;
  filename: string;
  fileSize: number;
  contentType: string;
  storageUrl: string;
  uploadedAt: string;
  accessCount: number;
  resumeText: string;
  analyzeStatus?: AnalyzeStatus;
  analyzeError?: string;
  analyses: AnalysisItem[];      // 分析历史
  interviews: InterviewItem[];   // 面试历史
}
```

**使用场景**：`ResumeDetailPage.tsx` - 展示简历详情、分析结果、面试历史

#### 2.3 导出简历分析PDF

```typescript
historyApi.exportAnalysisPdf(resumeId: number): Promise<Blob>

// 接口调用
GET /api/resumes/{resumeId}/export
Response-Type: blob
```

**使用场景**：`ResumeDetailPage.tsx` - 导出PDF报告

#### 2.4 删除简历

```typescript
historyApi.deleteResume(id: number): Promise<void>

// 接口调用
DELETE /api/resumes/{id}
```

#### 2.5 重新分析简历

```typescript
historyApi.reanalyze(id: number): Promise<void>

// 接口调用
POST /api/resumes/{id}/reanalyze
```

**使用场景**：分析失败后手动重试

---

### 三、面试模块接口调用

**API文件**：`api/interview.ts`

#### 3.1 创建面试会话

```typescript
interviewApi.createSession(req: CreateInterviewRequest): Promise<InterviewSession>

// 请求类型
interface CreateInterviewRequest {
  resumeText: string;
  questionCount: number;
  resumeId?: number;
  forceCreate?: boolean;  // 强制创建新会话
}

// 接口调用
POST /api/interview/sessions
Timeout: 180000 (3分钟)

// 响应类型
interface InterviewSession {
  sessionId: string;
  resumeText: string;
  totalQuestions: number;
  currentQuestionIndex: number;
  questions: InterviewQuestion[];
  status: 'CREATED' | 'IN_PROGRESS' | 'COMPLETED' | 'EVALUATED';
}
```

**使用场景**：`InterviewPage.tsx` - 开始模拟面试

#### 3.2 获取会话信息

```typescript
interviewApi.getSession(sessionId: string): Promise<InterviewSession>

// 接口调用
GET /api/interview/sessions/{sessionId}
```

#### 3.3 获取当前问题

```typescript
interviewApi.getCurrentQuestion(sessionId: string): Promise<CurrentQuestionResponse>

// 响应类型
interface CurrentQuestionResponse {
  completed: boolean;
  question?: InterviewQuestion;
  message?: string;
}
```

**使用场景**：`InterviewPanel.tsx` - 展示当前面试问题

#### 3.4 提交答案

```typescript
interviewApi.submitAnswer(req: SubmitAnswerRequest): Promise<SubmitAnswerResponse>

// 请求类型
interface SubmitAnswerRequest {
  sessionId: string;
  questionIndex: number;
  answer: string;
}

// 接口调用
POST /api/interview/sessions/{sessionId}/answers
Timeout: 180000 (3分钟)

// 响应类型
interface SubmitAnswerResponse {
  hasNextQuestion: boolean;
  nextQuestion: InterviewQuestion | null;
  currentIndex: number;
  totalQuestions: number;
}
```

**使用场景**：`InterviewPanel.tsx` - 用户提交答案后获取下一题

#### 3.5 暂存答案

```typescript
interviewApi.saveAnswer(req: SubmitAnswerRequest): Promise<void>

// 接口调用
PUT /api/interview/sessions/{sessionId}/answers
```

**使用场景**：用户暂存答案但不进入下一题

#### 3.6 提前交卷

```typescript
interviewApi.completeInterview(sessionId: string): Promise<void>

// 接口调用
POST /api/interview/sessions/{sessionId}/complete
```

#### 3.7 查找未完成会话

```typescript
interviewApi.findUnfinishedSession(resumeId: number): Promise<InterviewSession | null>

// 接口调用
GET /api/interview/sessions/unfinished/{resumeId}
```

**使用场景**：恢复未完成的面试

#### 3.8 获取面试报告

```typescript
interviewApi.getReport(sessionId: string): Promise<InterviewReport>

// 接口调用
GET /api/interview/sessions/{sessionId}/report
Timeout: 180000 (3分钟)

// 响应类型
interface InterviewReport {
  sessionId: string;
  totalQuestions: number;
  overallScore: number;
  categoryScores: CategoryScore[];
  questionDetails: QuestionEvaluation[];
  overallFeedback: string;
  strengths: string[];
  improvements: string[];
  referenceAnswers: ReferenceAnswer[];
}
```

**使用场景**：`InterviewDetailPanel.tsx` - 展示面试评估报告

#### 3.9 获取面试详情

```typescript
historyApi.getInterviewDetail(sessionId: string): Promise<InterviewDetail>

// 接口调用
GET /api/interview/sessions/{sessionId}/details

// 响应类型
interface InterviewDetail {
  id: number;
  sessionId: string;
  totalQuestions: number;
  status: string;
  evaluateStatus?: EvaluateStatus;
  evaluateError?: string;
  overallScore: number | null;
  overallFeedback: string | null;
  createdAt: string;
  completedAt: string | null;
  answers: AnswerItem[];
  questions?: unknown[];
  strengths?: string[];
  improvements?: string[];
  referenceAnswers?: unknown[];
}
```

**使用场景**：`InterviewHistoryPage.tsx` - 面试记录详情

#### 3.10 导出面试报告PDF

```typescript
historyApi.exportInterviewPdf(sessionId: string): Promise<Blob>

// 接口调用
GET /api/interview/sessions/{sessionId}/export
Response-Type: blob
```

#### 3.11 删除面试记录

```typescript
historyApi.deleteInterview(sessionId: string): Promise<void>

// 接口调用
DELETE /api/interview/sessions/{sessionId}
```

---

### 四、知识库模块接口调用

**API文件**：`api/knowledgebase.ts`

#### 4.1 上传知识库文件

```typescript
knowledgeBaseApi.uploadKnowledgeBase(
  file: File,
  name?: string,
  category?: string
): Promise<UploadKnowledgeBaseResponse>

// 接口调用
POST /api/knowledgebase/upload
Content-Type: multipart/form-data

// 响应类型
interface UploadKnowledgeBaseResponse {
  knowledgeBase: {
    id: number;
    name: string;
    category: string;
    fileSize: number;
    contentLength: number;
  };
  storage: {
    fileKey: string;
    fileUrl: string;
  };
  duplicate: boolean;
}
```

**使用场景**：`KnowledgeBaseUploadPage.tsx` - 上传知识文档

#### 4.2 下载知识库文件

```typescript
knowledgeBaseApi.downloadKnowledgeBase(id: number): Promise<Blob>

// 接口调用
GET /api/knowledgebase/{id}/download
Response-Type: blob
```

#### 4.3 获取知识库列表

```typescript
knowledgeBaseApi.getAllKnowledgeBases(
  sortBy?: SortOption,
  vectorStatus?: VectorStatus
): Promise<KnowledgeBaseItem[]>

// 接口调用
GET /api/knowledgebase/list?sortBy=time&vectorStatus=COMPLETED

// 响应类型
interface KnowledgeBaseItem {
  id: number;
  name: string;
  category: string | null;
  originalFilename: string;
  fileSize: number;
  contentType: string;
  uploadedAt: string;
  lastAccessedAt: string;
  accessCount: number;
  questionCount: number;
  vectorStatus: VectorStatus;  // PENDING | PROCESSING | COMPLETED | FAILED
  vectorError: string | null;
  chunkCount: number;
}
```

**使用场景**：`KnowledgeBaseManagePage.tsx` - 知识库管理列表

#### 4.4 获取知识库详情

```typescript
knowledgeBaseApi.getKnowledgeBase(id: number): Promise<KnowledgeBaseItem>

// 接口调用
GET /api/knowledgebase/{id}
```

#### 4.5 删除知识库

```typescript
knowledgeBaseApi.deleteKnowledgeBase(id: number): Promise<void>

// 接口调用
DELETE /api/knowledgebase/{id}
```

#### 4.6 查询知识库（普通）

```typescript
knowledgeBaseApi.queryKnowledgeBase(req: QueryRequest): Promise<QueryResponse>

// 请求类型
interface QueryRequest {
  knowledgeBaseIds: number[];  // 支持多知识库
  question: string;
}

// 接口调用
POST /api/knowledgebase/query
Timeout: 180000 (3分钟)

// 响应类型
interface QueryResponse {
  answer: string;
  knowledgeBaseId: number;
  knowledgeBaseName: string;
}
```

#### 4.7 查询知识库（流式SSE）

```typescript
knowledgeBaseApi.queryKnowledgeBaseStream(
  req: QueryRequest,
  onMessage: (chunk: string) => void,
  onComplete: () => void,
  onError: (error: Error) => void
): Promise<void>

// 接口调用
POST /api/knowledgebase/query/stream
Accept: text/event-stream
```

**使用场景**：`KnowledgeBaseQueryPage.tsx` - 问答助手流式回答

**实现细节**：
- 使用 Fetch API 获取ReadableStream
- 逐块解析SSE格式数据
- 通过回调函数实时更新UI
- 支持Markdown渲染

#### 4.8 分类管理

```typescript
// 获取所有分类
knowledgeBaseApi.getAllCategories(): Promise<string[]>
GET /api/knowledgebase/categories

// 按分类获取
knowledgeBaseApi.getByCategory(category: string): Promise<KnowledgeBaseItem[]>
GET /api/knowledgebase/category/{category}

// 获取未分类
knowledgeBaseApi.getUncategorized(): Promise<KnowledgeBaseItem[]>
GET /api/knowledgebase/uncategorized

// 更新分类
knowledgeBaseApi.updateCategory(id: number, category: string | null): Promise<void>
PUT /api/knowledgebase/{id}/category
```

#### 4.9 搜索知识库

```typescript
knowledgeBaseApi.search(keyword: string): Promise<KnowledgeBaseItem[]>

// 接口调用
GET /api/knowledgebase/search?keyword=spring
```

#### 4.10 获取统计信息

```typescript
knowledgeBaseApi.getStatistics(): Promise<KnowledgeBaseStats>

// 接口调用
GET /api/knowledgebase/stats

// 响应类型
interface KnowledgeBaseStats {
  totalCount: number;
  totalQuestionCount: number;
  totalAccessCount: number;
  completedCount: number;
  processingCount: number;
}
```

#### 4.11 重新向量化

```typescript
knowledgeBaseApi.revectorize(id: number): Promise<void>

// 接口调用
POST /api/knowledgebase/{id}/revectorize
```

**使用场景**：向量化失败后手动重试

---

### 五、RAG聊天模块接口调用

**API文件**：`api/ragChat.ts`

#### 5.1 创建聊天会话

```typescript
ragChatApi.createSession(
  knowledgeBaseIds: number[],
  title?: string
): Promise<RagChatSession>

// 接口调用
POST /api/rag-chat/sessions

// 响应类型
interface RagChatSession {
  id: number;
  title: string;
  knowledgeBaseIds: number[];
  createdAt: string;
}
```

**使用场景**：`KnowledgeBaseQueryPage.tsx` - 创建新聊天会话

#### 5.2 获取会话列表

```typescript
ragChatApi.listSessions(): Promise<RagChatSessionListItem[]>

// 接口调用
GET /api/rag-chat/sessions

// 响应类型
interface RagChatSessionListItem {
  id: number;
  title: string;
  messageCount: number;
  knowledgeBaseNames: string[];
  updatedAt: string;
  isPinned: boolean;
}
```

#### 5.3 获取会话详情

```typescript
ragChatApi.getSessionDetail(sessionId: number): Promise<RagChatSessionDetail>

// 接口调用
GET /api/rag-chat/sessions/{sessionId}

// 响应类型
interface RagChatSessionDetail {
  id: number;
  title: string;
  knowledgeBases: KnowledgeBaseItem[];
  messages: RagChatMessage[];
  createdAt: string;
  updatedAt: string;
}

interface RagChatMessage {
  id: number;
  type: 'user' | 'assistant';
  content: string;
  createdAt: string;
}
```

#### 5.4 更新会话标题

```typescript
ragChatApi.updateSessionTitle(sessionId: number, title: string): Promise<void>

// 接口调用
PUT /api/rag-chat/sessions/{sessionId}/title
```

#### 5.5 更新会话知识库

```typescript
ragChatApi.updateKnowledgeBases(
  sessionId: number,
  knowledgeBaseIds: number[]
): Promise<void>

// 接口调用
PUT /api/rag-chat/sessions/{sessionId}/knowledge-bases
```

#### 5.6 置顶/取消置顶

```typescript
ragChatApi.togglePin(sessionId: number): Promise<void>

// 接口调用
PUT /api/rag-chat/sessions/{sessionId}/pin
```

#### 5.7 删除会话

```typescript
ragChatApi.deleteSession(sessionId: number): Promise<void>

// 接口调用
DELETE /api/rag-chat/sessions/{sessionId}
```

#### 5.8 发送消息（流式SSE）

```typescript
ragChatApi.sendMessageStream(
  sessionId: number,
  question: string,
  onMessage: (chunk: string) => void,
  onComplete: () => void,
  onError: (error: Error) => void
): Promise<void>

// 接口调用
POST /api/rag-chat/sessions/{sessionId}/messages/stream
Accept: text/event-stream
```

**使用场景**：`KnowledgeBaseQueryPage.tsx` - 聊天消息发送与实时响应

**SSE解析流程**：
1. 使用Fetch API发起POST请求
2. 获取ReadableStream reader
3. 使用TextDecoder解码二进制数据
4. 按`\n\n`分割SSE事件
5. 提取`data:`字段内容
6. 还原转义的换行符（`\\n` → `\n`）
7. 通过onMessage回调逐块更新UI

---

### 六、前端组件调用流程图

#### 1. 简历分析流程

```
UploadPage
  │
  ├─ 选择文件
  │
  ├─ resumeApi.uploadAndAnalyze(file)
  │   └─ POST /api/resumes/upload
  │
  ├─ 获取响应 { id, analyzeStatus: 'PENDING' }
  │
  └─ 导航到 /history (带newResumeId)
        │
        └─ HistoryPage
            │
            ├─ historyApi.getResumes()  // 轮询获取列表
            │
            └─ 显示分析状态（PENDING/PROCESSING/COMPLETED）
```

#### 2. 模拟面试流程

```
ResumeDetailPage
  │
  ├─ 点击"开始面试"
  │
  ├─ 导航到 /interview/:resumeId
  │
  └─ InterviewPage
        │
        ├─ interviewApi.createSession({ resumeText, questionCount })
        │   └─ POST /api/interview/sessions
        │
        ├─ 获取 sessionId
        │
        └─ InterviewPanel
            │
            ├─ interviewApi.getCurrentQuestion(sessionId)
            │   └─ GET /api/interview/sessions/{sessionId}/question
            │
            ├─ 用户输入答案
            │
            ├─ interviewApi.submitAnswer({ sessionId, questionIndex, answer })
            │   └─ POST /api/interview/sessions/{sessionId}/answers
            │
            ├─ 获取下一题或完成
            │
            └─ interviewApi.getReport(sessionId)
                └─ GET /api/interview/sessions/{sessionId}/report
```

#### 3. RAG聊天流程

```
KnowledgeBaseQueryPage
  │
  ├─ 选择知识库
  │
  ├─ ragChatApi.createSession(knowledgeBaseIds, title)
  │   └─ POST /api/rag-chat/sessions
  │
  ├─ 获取 sessionId
  │
  └─ 用户输入问题
        │
        ├─ ragChatApi.sendMessageStream(sessionId, question, callbacks)
        │   └─ POST /api/rag-chat/sessions/{sessionId}/messages/stream
        │
        ├─ onMessage(chunk) → 实时更新AI回答
        │
        ├─ onComplete() → 标记完成
        │
        └─ onError(error) → 显示错误
```

---

### 七、前端关键技术实现

#### 1. 异步状态轮询

简历分析和知识库向量化采用异步处理，前端需要轮询状态：

```typescript
// HistoryPage.tsx 示例
useEffect(() => {
  const interval = setInterval(async () => {
    const resumes = await historyApi.getResumes();
    setResumes(resumes);
    
    // 检查是否还有处理中的任务
    const hasProcessing = resumes.some(
      r => r.analyzeStatus === 'PENDING' || r.analyzeStatus === 'PROCESSING'
    );
    
    if (!hasProcessing) {
      clearInterval(interval);  // 停止轮询
    }
  }, 3000);  // 每3秒轮询一次
  
  return () => clearInterval(interval);
}, []);
```

#### 2. 文件下载处理

```typescript
// 导出PDF
const handleExport = async () => {
  const blob = await historyApi.exportAnalysisPdf(resumeId);
  const url = window.URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = `简历分析报告_${resumeId}.pdf`;
  link.click();
  window.URL.revokeObjectURL(url);
};
```

#### 3. Markdown渲染

使用 `react-markdown` + `remark-gfm` + `react-syntax-highlighter`：

```typescript
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter';

<ReactMarkdown
  remarkPlugins={[remarkGfm]}
  components={{
    code({ className, children, ...props }) {
      const match = /language-(\w+)/.exec(className || '');
      return match ? (
        <SyntaxHighlighter language={match[1]} {...props}>
          {String(children).replace(/\n$/, '')}
        </SyntaxHighlighter>
      ) : (
        <code className={className} {...props}>
          {children}
        </code>
      );
    }
  }}
>
  {markdownContent}
</ReactMarkdown>
```

#### 4. 虚拟列表优化

使用 `react-virtuoso` 优化长列表：

```typescript
import { Virtuoso } from 'react-virtuoso';

<Virtuoso
  ref={virtuosoRef}
  data={messages}
  itemContent={(index, message) => (
    <MessageItem key={message.id} message={message} />
  )}
  followOutput="smooth"
/>
```

#### 5. 动画效果

使用 `framer-motion` 实现页面切换动画：

```typescript
import { motion } from 'framer-motion';

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -20 }}
  transition={{ duration: 0.3 }}
>
  {children}
</motion.div>
```

---

### 八、前端开发命令

```bash
# 安装依赖
pnpm install

# 启动开发服务器（自动代理到后端）
pnpm dev

# 构建生产版本
pnpm build

# 预览生产构建
pnpm preview
```

**开发服务器配置**（`vite.config.ts`）：
```typescript
server: {
  host: '0.0.0.0',
  port: 5173,
  proxy: {
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true,
    },
  },
}
```

---

### 九、前端部署

**生产环境构建**：
```bash
pnpm build
```

生成 `dist/` 目录，包含静态资源文件。

**Nginx配置示例**：
```nginx
server {
    listen 80;
    
    # 前端静态资源
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }
    
    # 后端API代理
    location /api {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Docker部署**：前端打包为Docker镜像，通过docker-compose统一启动（详见快速开始-Docker部署章节）
