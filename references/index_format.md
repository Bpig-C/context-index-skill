# 项目结构索引 - 格式参考

## 格式规则

- `##` = 顶层目录（backend/、frontend/、docs/、scripts/）
- `###` = 二级目录（processors/、api/、services/ 等）
- 文件名直接列出，无标题符号
- 文件节点后接缩进的签名列表（class/function）

## 标记规则

- `⚠️` 前缀 = 该签名从未生成过描述，需补充（Phase 2 自动处理）
- 无标记 = 已有描述
- `[模块描述]` = 用方括号包裹的是目录级的手动描述
- `★` = 扩展点或集成边界标识

## 描述风格指南

### 模块描述（目录级）
应包含：
- 模块职责（1-2句话）
- 设计模式（如有）
- 扩展点标识（如有）

**示例**：
```markdown
### /processors/   [数据解析层 - 扩展点★]
### /api/   [REST API端点层 - 集成边界★]
### /services/   [业务逻辑层]
```

### 函数/类描述
应包含：
- 动词开头（获取、构建、解析、导出等）
- 核心功能（1句话）
- 关键参数或返回值（如有必要）

**示例**：
```markdown
- def get_processors() → List[dict]  获取所有可用处理器列表，返回 [{name, display_name}]
- class QueryEngine  图谱查询引擎，支持多处理器路由
- def build_graph(json_data: dict)  构建知识图谱，插入节点和关系到数据库
```

### 描述长度
- 类描述：10-20字
- 函数描述：15-30字
- 避免过于简单（如"处理数据"）或过于冗长

## 完整示例

```markdown
## /backend/   [后端服务层]
 ### /app/   [应用核心]
  ### /processors/   [数据解析层 - 扩展点★]
     base.py
       - class BaseProcessor  处理器抽象基类（扩展点★）
         - abstract method parse_excel(file_path, output_dir) → str  解析Excel文件，返回JSON路径
         - abstract method extract_entities(json_data) → dict  提取实体数据
         - abstract method build_entity_text(entity) → str  构建实体文本描述（用于语料导出）
     __init__.py
       - PROCESSOR_REGISTRY  处理器注册表，包含 {"kf": KFProcessor, "qms": QMSProcessor}
       - def get_processor(name: str) → BaseProcessor  根据名称获取处理器实例（工厂函数）
   ### /kf/   [快反问题记录处理器]
      __init__.py
        - class KFProcessor(BaseProcessor)  快反问题记录处理器，处理内嵌图片的Excel
        - def parse_excel(file_path: str, output_dir: str) → str  解析快反Excel文件，提取内嵌图片，返回JSON路径
  ### /api/   [REST API端点层 - 集成边界★]
     graph_api.py
       - def get_graph_data(processor: str)  获取图谱数据（节点+边），路由到 QueryEngine
       - def get_statistics(processor: str)  获取统计数据，路由到 QueryEngine
  ### /services/   [业务逻辑层]
     corpus_exporter.py
       - class CorpusExporter  语料导出器，委托处理器的 build_entity_text() 构建文本
       - def export_entity_text(processor: BaseProcessor, output_dir: str) → str  导出实体文本格式，返回输出路径
```

## 生成描述时的注意事项

1. **参考同目录已有描述**：保持风格一致
2. **避免重复信息**：函数名已经说明了功能，描述应补充关键细节
3. **突出关键参数**：如果参数对理解功能很重要，在描述中提及
4. **标识设计模式**：如工厂函数、抽象基类、策略模式等
5. **标识扩展点**：如果是可扩展的插件点，用 ★ 标识
6. **保持简洁**：一句话说清楚，不要写成段落

## 错误示例 vs 正确示例

### 错误示例
```markdown
- def get_processors  获取处理器  ❌ 太简单
- class QueryEngine  这是一个查询引擎类，用于查询图谱数据  ❌ 太冗长
- def build_graph  构建图谱  ❌ 缺少关键信息
```

### 正确示例
```markdown
- def get_processors() → List[dict]  获取所有可用处理器列表，返回 [{name, display_name}]  ✅
- class QueryEngine  图谱查询引擎，支持多处理器路由  ✅
- def build_graph(json_data: dict)  构建知识图谱，插入节点和关系到数据库  ✅
```
