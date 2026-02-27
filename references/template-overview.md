# 技术方案文档结构模板

本文档定义技术设计方案的标准结构、章节说明和格式规范。

## 完整章节结构

```
# [功能名称] - 技术方案

## 0. 设计范围与协作边界
### 0.1 当前模块范围
### 0.2 外部模块依赖与协作说明

## 1. 数据库设计
### 1.1 表清单
### 1.2 ER 图
### 1.3 表关系说明
### 1.4 建表 SQL

## 2. 架构设计
### 2.1 系统架构图
### 2.2 技术栈说明
### 2.3 模块划分

## 3. 数据模型
### 3.1 表关系图
### 3.2 数据权限
### 3.3 多租户隔离
### 3.4 软删除

## 4. 组件接口设计
### 4.1 Controller 接口
### 4.2 Service 接口
### 4.3 Mapper 接口

## 5. 核心业务流程
### 5.1 [流程1名称]
### 5.2 [流程2名称]
### 5.3 [流程3名称]

## 6. 功能权限配置
### 6.1 数据权限规则
### 6.2 操作权限
```

## 章节说明

### 0. 设计范围与协作边界

**目的**: 明确当前文档的设计边界，支持多团队并行开发时的职责隔离。

**必须包含**:
- 当前模块范围：本次方案覆盖的业务域、对象、接口边界
- 外部模块清单：仅列出依赖模块名称与责任团队
- 外部协作清单：接入方式（HTTP/RPC/消息/任务）、协作接口/事件、关键入参与出参、状态

**约束要求**:
- 仅详细设计当前模块
- 外部模块仅保留协作边界与对接信息，不展开内部数据库、Service/Mapper、流程实现

### 1. 数据库设计

**目的**: 定义数据存储结构，确保数据模型清晰、关系合理。

**必须包含**:
- 表清单：序号、表名、说明的表格
- ER 图：使用 Mermaid erDiagram 绘制（**默认使用详细格式**，包含完整表结构、字段类型、约束）
- 表关系说明：描述表之间的关联关系（1:1, 1:N, N:M）
- 建表 SQL：完整的 CREATE TABLE 语句

**ER 图格式要求**:
- **默认使用详细格式**：包含所有字段、类型、约束（PK/FK/UK）、注释
- 字段顺序：主键 → 业务字段 → 标准字段
- 标准字段必须包含：creator, create_time, updater, update_time, deleted, tenant_id

**示例格式**:
```markdown
### 1.1 表清单

| 序号 | 表名 | 说明 |
|------|------|------|
| 1 | user | 用户表 |
| 2 | order | 订单表 |

### 1.2 ER 图

​```mermaid
erDiagram
    user {
        bigint id PK "雪花算法"
        varchar username UK "用户名"
        varchar password "密码(加密)"
        varchar creator "创建者"
        datetime create_time "创建时间"
        varchar updater "更新者"
        datetime update_time "更新时间"
        tinyint deleted "是否删除:0-否,1-是"
        bigint tenant_id "租户编号"
    }

    order {
        bigint id PK "雪花算法"
        varchar order_no UK "订单编号"
        bigint user_id FK "用户ID(user.id)"
        decimal total_amount "订单总金额"
        varchar order_status "订单状态:PENDING/PAID/COMPLETED"
        varchar creator "创建者"
        datetime create_time "创建时间"
        varchar updater "更新者"
        datetime update_time "更新时间"
        tinyint deleted "是否删除:0-否,1-是"
        bigint tenant_id "租户编号"
    }

    user ||--o{ order : "1:N 一个用户多个订单"
​```
```

### 2. 架构设计

**目的**: 说明系统整体架构、技术选型和模块划分。

**必须包含**:
- 系统架构图：展示系统分层、模块关系
- 技术栈说明：列出使用的框架、中间件、工具
- 模块划分：说明各模块职责

### 3. 数据模型

**目的**: 详细说明数据关系、权限控制和数据隔离策略。

**必须包含**:
- 表关系图：使用 Mermaid 绘制详细的实体关系
- 数据权限：说明数据访问权限规则
- 多租户隔离：如何实现租户数据隔离
- 软删除：软删除字段和逻辑

### 4. 组件接口设计

**目的**: 定义各层接口规范，确保代码结构清晰。

**必须包含**:
- Controller 接口：HTTP 接口定义（路径、方法、参数、响应）
- Service 接口：业务逻辑接口定义
- Mapper 接口：数据访问接口定义

### 5. 核心业务流程

**目的**: 描述关键业务流程的执行逻辑。

**必须包含**:
- 流程图：使用 Mermaid flowchart 绘制
- 流程说明：文字描述流程步骤
- 异常处理：说明异常情况的处理方式

### 6. 功能权限配置

**目的**: 定义功能的权限控制规则。

**必须包含**:
- 数据权限规则：谁可以访问哪些数据
- 操作权限：谁可以执行哪些操作

## 标题层级规范

- **一级标题 (#)**: 文档标题，仅用于文档开头
- **二级标题 (##)**: 主要章节（数据库设计、架构设计等）
- **三级标题 (###)**: 子章节（表清单、ER 图等）
- **四级标题 (####)**: 详细说明（可选，用于复杂内容）

## 格式规范

### 表格格式

使用 Markdown 表格，必须包含表头：

```markdown
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| 值1 | 值2 | 值3 |
```

### 代码块格式

必须标注语言类型：

```markdown
​```sql
CREATE TABLE user (
  id BIGINT PRIMARY KEY,
  name VARCHAR(100)
);
​```
```

### 图表格式

使用 Mermaid 代码块：

**ER 图（默认使用详细格式）**:

```markdown
​```mermaid
erDiagram
    user {
        bigint id PK "雪花算法"
        varchar username UK "用户名"
        varchar password "密码(加密)"
        varchar creator "创建者"
        datetime create_time "创建时间"
        varchar updater "更新者"
        datetime update_time "更新时间"
        tinyint deleted "是否删除:0-否,1-是"
        bigint tenant_id "租户编号"
    }

    order {
        bigint id PK "雪花算法"
        varchar order_no UK "订单编号"
        bigint user_id FK "用户ID(user.id)"
        decimal total_amount "订单总金额"
        varchar creator "创建者"
        datetime create_time "创建时间"
        varchar updater "更新者"
        datetime update_time "更新时间"
        tinyint deleted "是否删除:0-否,1-是"
        bigint tenant_id "租户编号"
    }

    user ||--o{ order : "1:N 一个用户多个订单"
​```
```

## 文档命名规范

- 文件名使用中文或英文，清晰表达功能
- 示例：`线索管理-技术方案.md`、`Lead-Management-Tech-Proposal.md`

## 版本控制

建议在文档开头添加版本信息：

```markdown
| 版本 | 日期 | 作者 | 变更说明 |
|------|------|------|----------|
| 1.0 | 2024-01-01 | 张三 | 初始版本 |
| 1.1 | 2024-01-15 | 李四 | 新增权限配置 |
```

## 跨模块协作模板（业务化表达）

```markdown
### 0.2 外部模块依赖与协作说明

| 模块 | 接入方式 | 协作接口/事件 | 关键入参 | 关键出参 | 责任团队 | 状态 |
|------|----------|----------|----------|----------|----------|------|
| customer | RPC | CustomerQueryFacade#getById | customerId | customerName,status | 客户域团队 | 待对接 |
| workflow | 事件 | lead.approved | leadId,operatorId | approveResult | 流程域团队 | 待对接 |
```
