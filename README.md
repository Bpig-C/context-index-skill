# context-index

**项目代码签名索引管理工具。** 使用 repomix 自动生成代码签名索引，配合手动维护的架构文档，让 AI 和开发者快速建立对项目结构的全局认知。

---

## 解决什么问题

在大型项目中，AI 每次对话都需要重新摸索项目结构，效率低、token 消耗高。这个 skill 建立一套持久化的"项目地图"：

- **代码签名索引**（自动）：repomix 压缩生成，包含所有函数/类签名，供 AI 快速扫描
- **架构文档**（手动维护）：技术栈、模块职责、数据流，AI 读文档比读代码快 10 倍

---

## 前置条件

| 依赖 | 说明 |
|------|------|
| Node.js | 运行 `npx repomix` |
| repomix | 自动安装：`npx repomix@latest` |
| bash | 运行辅助脚本（Windows 使用 Git Bash） |

---

## 触发词

| 场景 | 触发词示例 |
|------|-----------|
| 初始化 | "初始化项目索引"、"设置项目索引"、"配置项目索引" |
| 更新 | "更新项目索引"、"同步项目索引" |
| 查看变更 | "查看项目变更"、"项目有什么变化"、"比对项目索引" |
| 适配新项目 | "配置项目索引"、"适配项目索引" |

---

## 工作流程

```
初始化阶段
├── 扫描文件结构（bash init_index.sh）
├── 配置 repomix.config.json（ignore 规则）
├── 生成代码签名索引（bash init_index.sh --run）
├── 创建架构文档（PROJECT_INDEX/architecture.md）
└── 更新 CLAUDE.md 添加索引引用

日常更新
└── bash update_index.sh  →  自动比对 diff，提示手动文档是否需更新
```

---

## 产物

```
PROJECT_INDEX/
├── architecture.md          # 架构文档（手动维护）
├── database_schema.md       # 数据库表结构（手动维护，可选）
└── history/
    └── {项目名}_{timestamp}.md  # 代码签名索引（自动生成）

repomix.config.json          # repomix 配置（项目根目录）
init_index.sh                # 初始化脚本
update_index.sh              # 更新脚本
```

---

## 快速开始

```bash
# 1. 扫描文件结构，查看需要 ignore 的文件类型
bash init_index.sh

# 2. 根据输出配置 repomix.config.json

# 3. 生成索引
bash init_index.sh --run

# 4. 后续日常更新
bash update_index.sh
```

---

## 设计原则

- **自动化的交给 repomix**：文件列表、函数签名——准确且无需维护
- **手动的只写值得写的**：模块职责、架构决策——不需要函数级别注释
- **不做的事**：不用复杂脚本比对签名，不依赖模型生成描述（容易幻觉）

---

## 被哪些 skill 依赖

- [`repo-analysis`](../repo-analysis/README.md)：在 Phase 0 中调用本 skill 初始化代码签名索引
