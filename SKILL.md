---
name: context-index
description: 项目索引管理。使用 repomix 生成代码签名索引，配合手动维护的架构文档，帮助快速了解项目结构。触发词："更新项目索引"、"配置项目索引"、"初始化项目索引"。
---

# Project Index Skill

使用 repomix 生成代码签名索引，配合手动维护的架构文档，帮助快速了解项目结构。

## 设计理念

**简单可靠**：
- ✅ repomix 自动生成代码签名（准确、可靠）
- ✅ 架构文档手动维护（灵活、可控）
- ✅ 职责清晰，零维护成本

**不做的事**：
- ❌ 不使用复杂的脚本比对新旧签名
- ❌ 不依赖模型生成描述（容易偷懒）
- ❌ 不尝试自动保留已有描述（容易出 bug）

## 文件结构

```
PROJECT_INDEX/
├── README.md              # 使用说明
├── architecture.md        # 架构文档（手动维护）
├── database_schema.md     # 数据库表结构（手动维护，可选）
├── repomix_config_guide.md # repomix 配置指南
└── history/               # 代码签名索引（repomix 自动生成）
    └── {项目名}_{timestamp}.md

repomix.config.json        # repomix 配置文件（项目根目录）
repomix.config.template.json # 配置模板（可选）
update_index.sh            # 快捷更新脚本（可选）
```

## 使用场景

### 场景1：初始化项目索引

**触发词**："初始化项目索引"、"设置项目索引"

**步骤**：

1. **创建辅助脚本**

   在项目根目录创建以下两个脚本（内容固定，直接写入）：

   **`init_index.sh`**（首次初始化用）：
   ```bash
   #!/bin/bash
   # 初始化项目索引
   # 不带参数运行：扫描文件结构，供配置 ignore 规则使用
   # 带 --run 参数运行：正式生成索引并检查配置

   echo "=== 初始化项目索引 ==="
   echo "创建目录结构..."
   mkdir -p PROJECT_INDEX/history

   echo ""
   echo "=== 项目文件结构扫描 ==="
   echo "（供配置 repomix.config.json 的 ignore 规则使用）"
   echo ""

   find . \
     -not -path '*/.*' \
     -not -path '*/node_modules/*' \
     -not -path '*/PROJECT_INDEX/*' \
     -not -path '*/__pycache__/*' \
     -not -path '*/venv/*' \
     -not -path '*/.venv/*' \
     -not -path '*/env/*' \
     -not -path '*/dist/*' \
     -not -path '*/build/*' \
     -not -path '*/target/*' \
     -not -path '*/vendor/*' \
     -type f | sort

   echo ""
   echo "--- 文件类型统计 ---"
   find . \
     -not -path '*/.*' \
     -not -path '*/node_modules/*' \
     -not -path '*/PROJECT_INDEX/*' \
     -not -path '*/__pycache__/*' \
     -not -path '*/venv/*' \
     -not -path '*/.venv/*' \
     -not -path '*/env/*' \
     -not -path '*/dist/*' \
     -not -path '*/build/*' \
     -not -path '*/target/*' \
     -not -path '*/vendor/*' \
     -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

   echo ""
   echo "--- 顶层目录结构 ---"
   find . -maxdepth 2 \
     -not -path '*/.*' \
     -not -path '*/node_modules/*' \
     -not -path '*/PROJECT_INDEX/*' \
     -not -path '*/__pycache__/*' \
     -not -path '*/venv/*' \
     -not -path '*/.venv/*' \
     | sort

   echo ""
   echo "=========================================="
   echo "请根据以上文件清单配置 repomix.config.json"
   echo "确认 ignore.customPatterns 覆盖了不需要的文件类型后"
   echo "再继续运行: bash init_index.sh --run"
   echo "=========================================="

   if [ "$1" != "--run" ]; then
       exit 0
   fi

   if [ ! -f "repomix.config.json" ]; then
       echo "❌ 错误：repomix.config.json 不存在，请先创建配置文件"
       exit 1
   fi

   if ! grep -q "{datetime}" repomix.config.json; then
       echo "⚠️  警告：repomix.config.json 中未找到 {datetime} 占位符"
       echo "   filePath 应该类似: \"PROJECT_INDEX/history\\\\ProjectName_{datetime}.md\""
       echo ""
   fi

   echo ""
   echo "=== 生成索引 ==="
   ts=$(date +"%Y-%m-%d_%H-%M-%S")
   sed -i.bak "s/{datetime}/$ts/" repomix.config.json && rm -f repomix.config.json.bak
   npx repomix@latest --config repomix.config.json --compress
   sed -i.bak "s/$ts/{datetime}/" repomix.config.json && rm -f repomix.config.json.bak

   latest=$(ls -t PROJECT_INDEX/history/*.md 2>/dev/null | head -1)
   if [ -z "$latest" ]; then
       echo "❌ 错误：未找到生成的索引文件"
       exit 1
   fi

   echo ""
   echo "✅ 生成完成: $latest"
   echo ""
   echo "=== 配置检查 ==="
   total_files=$(grep -c "^## File:" "$latest" 2>/dev/null; true)
   echo "总文件数: $total_files"
   echo ""
   echo "文件类型分布:"
   grep "^## File:" "$latest" | sed 's/.*\.//' | sort | uniq -c | sort -rn

   echo ""
   echo "=== 优化建议 ==="
   should_ignore=""
   md_count=$(grep "^## File:" "$latest" | grep -c "\.md$" 2>/dev/null; true)
   [ "$md_count" -gt 0 ] && should_ignore="${should_ignore}\n  \"**/*.md\","
   json_count=$(grep "^## File:" "$latest" | grep -c "\.json$" 2>/dev/null; true)
   [ "$json_count" -gt 0 ] && should_ignore="${should_ignore}\n  \"**/*.json\","
   log_count=$(grep "^## File:" "$latest" | grep -c "\.log$" 2>/dev/null; true)
   [ "$log_count" -gt 0 ] && should_ignore="${should_ignore}\n  \"**/*.log\","
   img_count=$(grep "^## File:" "$latest" | grep -cE "\.(png|jpg|jpeg|gif|svg|webp)$" 2>/dev/null; true)
   [ "$img_count" -gt 0 ] && should_ignore="${should_ignore}\n  \"**/*.png\", \"**/*.jpg\", \"**/*.gif\","
   data_count=$(grep "^## File:" "$latest" | grep -cE "\.(csv|xlsx|db|sqlite)$" 2>/dev/null; true)
   [ "$data_count" -gt 0 ] && should_ignore="${should_ignore}\n  \"**/*.csv\", \"**/*.xlsx\", \"**/*.db\","

   if [ -n "$should_ignore" ]; then
       echo "⚠️  建议在 ignore.customPatterns 中添加："
       echo -e "$should_ignore"
       echo "添加后运行: bash update_index.sh"
   else
       echo "✅ 配置良好，未发现需要忽略的文件类型"
   fi

   echo ""
   echo "=== 下一步 ==="
   echo "1. 检查索引文件: less $latest"
   echo "2. 创建架构文档: PROJECT_INDEX/architecture.md"
   echo "3. 创建数据库文档: PROJECT_INDEX/database_schema.md（如有数据库）"
   echo "4. 更新 CLAUDE.md 添加索引引用"
   echo "5. 后续更新使用: bash update_index.sh"
   ```

   **`update_index.sh`**（日常更新用）：
   ```bash
   #!/bin/bash
   # 更新项目索引
   echo "=== 更新项目索引 ==="
   echo "运行 repomix..."

   ts=$(date +"%Y-%m-%d_%H-%M-%S")
   sed -i.bak "s/{datetime}/$ts/" repomix.config.json && rm -f repomix.config.json.bak
   npx repomix@latest --config repomix.config.json --compress
   sed -i.bak "s/$ts/{datetime}/" repomix.config.json && rm -f repomix.config.json.bak

   echo ""
   echo "✅ 索引已更新"
   echo ""
   echo "=== 检测变更 ==="

   latest=$(ls -t PROJECT_INDEX/history/*.md 2>/dev/null | head -1)
   previous=$(ls -t PROJECT_INDEX/history/*.md 2>/dev/null | head -2 | tail -1)

   if [ -z "$previous" ]; then
       echo "这是首次生成索引，没有历史版本可比对"
       echo "最新索引: $latest"
   elif [ "$latest" = "$previous" ]; then
       echo "只有一个历史版本，没有变更"
   else
       echo "比对: $previous"
       echo "  vs: $latest"
       echo ""
       diff -u "$previous" "$latest" > PROJECT_INDEX/latest_changes.diff
       added=$(grep -c "^+[^+]" PROJECT_INDEX/latest_changes.diff 2>/dev/null; true)
       removed=$(grep -c "^-[^-]" PROJECT_INDEX/latest_changes.diff 2>/dev/null; true)
       echo "变更统计:"
       echo "  新增: $added 行"
       echo "  删除: $removed 行"
       echo ""
       echo "详细变更: PROJECT_INDEX/latest_changes.diff"
   fi

   echo ""
   echo "提示："
   echo "- 架构文档: PROJECT_INDEX/architecture.md（需手动维护）"
   echo "- 数据库表结构: PROJECT_INDEX/database_schema.md（需手动维护）"
   echo "- 代码签名索引: PROJECT_INDEX/history/（自动生成）"
   ```

2. **扫描项目文件结构**

   运行（不带参数）：
   ```bash
   bash init_index.sh
   ```

   脚本会输出：
   - 所有文件列表（排除隐藏目录、venv、node_modules 等）
   - 文件类型统计（用于判断哪些类型需要 ignore）
   - 顶层目录结构

   **根据输出判断需要忽略的内容**，然后继续下一步。

3. **配置 repomix**

   创建 `repomix.config.json`（项目根目录）：
   ```json
   {
     "output": {
       "filePath": "PROJECT_INDEX/history/{项目名}_{datetime}.md",
       "style": "markdown",
       "showDirectoryStructure": true
     },
     "compression": {
       "enabled": true,
       "keepSignatures": true,
       "keepDocstrings": true,
       "keepInterfaces": true
     },
     "ignore": {
       "customPatterns": [
         "**/*.md",
         "**/*.json",
         "**/*.log",
         "PROJECT_INDEX/history/**",
         ".*/**"
       ],
       "useGitignore": true,
       "useDefaultIgnore": true
     }
   }
   ```

   **重要**：
   - `filePath` 中必须使用 `{datetime}` 占位符（不要写具体日期）
   - 将 `{项目名}` 替换为实际项目名（如 `KFsystem`、`MyProject`）
   - `{datetime}` 会在运行时自动替换为 `YYYY-MM-DD_HH-MM-SS` 格式

   **根据项目类型调整 `customPatterns`**：

   - **Python 项目**：添加 `"**/*.pyc"`, `"venv/**"`, `"__pycache__/**"`
   - **JavaScript/TypeScript**：添加 `"node_modules/**"`, `"dist/**"`, `"build/**"`
   - **Java**：添加 `"target/**"`, `"*.class"`, `"*.jar"`
   - **Go**：添加 `"vendor/**"`, `"bin/**"`, `"*.exe"`

   **忽略数据文件**：`"**/*.csv"`, `"**/*.xlsx"`, `"**/*.db"`, `"**/*.sqlite"`

   **忽略图片**：`"**/*.png"`, `"**/*.jpg"`, `"**/*.gif"`, `"**/*.svg"`

4. **正式生成索引**

   配置完成后，带 `--run` 参数运行：
   ```bash
   bash init_index.sh --run
   ```

   脚本会：
   - 将 `{datetime}` 替换为实际时间戳（格式：`YYYY-MM-DD_HH-MM-SS`）
   - 运行 repomix 生成索引
   - 恢复 `{datetime}` 占位符
   - 检查生成文件，提示遗漏的 ignore 规则

5. **检查并优化配置**

   根据 `init_index.sh --run` 的输出：
   - 如果仍有不该包含的文件类型，补充到 `ignore.customPatterns`
   - 再次运行 `bash update_index.sh` 生成优化后的索引

6. **创建架构文档**

   创建 `PROJECT_INDEX/architecture.md`，包含：
   ```markdown
   # 项目架构文档

   > 最后更新：YYYY-MM-DD

   ## 架构概览

   **技术栈**：
   - 后端：...
   - 前端：...
   - 数据库：...

   **核心设计模式**：
   - ...

   **数据流**：
   ```
   ...
   ```

   **扩展点**：
   1. ...

   **对外接口**：
   - ...

   ## 系统能力说明

   > 以下各项，根据实际代码探索结果填写存在的项，不存在的项直接删除，不要保留"无"或空行。

   **认证与授权**：（如有）
   - 认证方式：JWT / Session / OAuth / 无认证
   - 角色管理：有（列出角色）/ 无
   - 权限粒度：路由级 / 接口级 / 数据级

   **状态管理**：（前端）
   - 方案：Pinia / Vuex / Redux / Zustand / 无全局状态库（props + API 驱动）
   - 持久化：localStorage / sessionStorage / 无

   **缓存策略**：（如有）
   - 后端缓存：Redis / 内存缓存 / 无
   - 前端缓存：HTTP 缓存 / Service Worker / 无

   **任务队列 / 异步处理**：（如有）
   - 方案：Celery / RQ / 内置 asyncio / 无

   **多租户 / 数据隔离**：（如有）
   - 方案：按 tenant_id 隔离 / 独立数据库 / 无

   **国际化（i18n）**：（如有）
   - 方案：vue-i18n / react-intl / 硬编码中文 / 无

   **日志与监控**：（如有）
   - 日志：结构化日志（loguru/winston）/ 标准 logging / print 调试 / 无
   - 监控：Sentry / Prometheus / 无

   **WebSocket / 实时通信**：（如有）
   - 用途：...

   **文件存储**：（如有）
   - 方案：本地磁盘 / OSS / S3 / 无

   ## 模块说明

   ### /模块名/ - 模块职责
   **职责**：...
   **核心类**：...
   **扩展方式**：...

   ## 集成指南

   ### 如何添加新功能
   ...
   ```

   **何时更新**：新增模块、修改设计模式、变更数据流时手动编辑。

   > **模型说明**：填写"系统能力说明"时，先阅读 repomix 生成的代码签名索引，根据实际代码判断每项是否存在，删除不存在的项，不要凭空填写。

7. **创建数据库表结构文档**（有数据库的项目）

   创建 `PROJECT_INDEX/database_schema.md`，包含：
   ```markdown
   # 数据库表结构

   > 最后更新：YYYY-MM-DD

   ## 表：table_name

   | 字段 | 类型 | 说明 |
   |------|------|------|
   | id   | INTEGER | 主键，自增 |
   | name | TEXT | ... |

   **关系**：
   - `table_a.id` → `table_b.foreign_id`（一对多）
   ```

   **何时更新**：新增表、修改字段、变更关系时手动编辑。

8. **更新 CLAUDE.md**
   ```markdown
   ## 项目文档

   - **架构文档**：`PROJECT_INDEX/architecture.md`
   - **代码签名索引**：`PROJECT_INDEX/history/`（repomix 自动生成）

   ## 更新索引

   运行 `bash update_index.sh`
   ```

### 场景2：更新项目索引

**触发词**："更新项目索引"、"同步项目索引"

**步骤**：

1. **运行更新脚本**
   ```bash
   bash update_index.sh
   ```

   或直接运行 repomix：
   ```bash
   npx repomix@latest --config repomix.config.json --compress
   ```

2. **自动生成变更报告**

   `update_index.sh` 会自动比对最新的两个版本，生成 `PROJECT_INDEX/latest_changes.diff`。

3. **查看变更**
   ```bash
   cat PROJECT_INDEX/latest_changes.diff
   ```

   或使用分页查看：
   ```bash
   less PROJECT_INDEX/latest_changes.diff
   ```

4. **理解 diff 格式**

   - `+` 开头的行：新增的内容
   - `-` 开头的行：删除的内容
   - 无符号的行：上下文（未变化）

   **示例**：
   ```diff
   --- PROJECT_INDEX/history/KFsystem_2026-03-28.md
   +++ PROJECT_INDEX/history/KFsystem_2026-03-29.md
   @@ -10,6 +10,8 @@
    backend/app/api/config_api.py
    backend/app/api/export_api.py
   +backend/app/api/new_api.py        # 新增文件
    backend/app/models/database.py
   -backend/app/legacy/old_file.py    # 删除文件
   ```

5. **更新手动文档（如有变化）**

   - **架构变化**（新模块、设计模式、数据流）→ 编辑 `PROJECT_INDEX/architecture.md`
   - **数据库变化**（新表、新字段、关系变更）→ 编辑 `PROJECT_INDEX/database_schema.md`

### 场景3：查看项目变更

**触发词**："查看项目变更"、"比对项目索引"、"项目有什么变化"

**步骤**：

1. **创建比对脚本**（首次使用时）

   **`compare_index.sh`**：
   ```bash
   #!/bin/bash
   # 比对项目索引版本
   # 用法: bash compare_index.sh [旧版本] [新版本]

   if [ "$#" -eq 2 ]; then
       old="$1"
       new="$2"
   else
       latest=$(ls -t PROJECT_INDEX/history/*.md 2>/dev/null | head -1)
       previous=$(ls -t PROJECT_INDEX/history/*.md 2>/dev/null | head -2 | tail -1)

       if [ -z "$latest" ]; then
           echo "❌ 错误：未找到索引文件"
           exit 1
       fi

       if [ -z "$previous" ] || [ "$latest" = "$previous" ]; then
           echo "⚠️  只有一个版本，无法比对"
           echo "唯一版本: $latest"
           exit 0
       fi

       old="$previous"
       new="$latest"
   fi

   echo "比对: $old"
   echo "  vs: $new"
   echo ""
   diff -u "$old" "$new" | less
   ```

2. **比对最新的两个版本**
   ```bash
   bash compare_index.sh
   ```

   这会使用 `diff -u` 比对最新的两个历史文件，并用 `less` 分页显示。

2. **比对指定的两个版本**
   ```bash
   bash compare_index.sh PROJECT_INDEX/history/old.md PROJECT_INDEX/history/new.md
   ```

3. **查看已保存的变更**
   ```bash
   cat PROJECT_INDEX/latest_changes.diff
   ```

4. **分析变更**

   **新增文件**：查找 `+` 开头且包含文件路径的行
   ```bash
   grep "^+.*\.py$" PROJECT_INDEX/latest_changes.diff
   ```

   **删除文件**：查找 `-` 开头且包含文件路径的行
   ```bash
   grep "^-.*\.py$" PROJECT_INDEX/latest_changes.diff
   ```

   **新增函数/类**：查找 `+` 开头且包含 `def` 或 `class` 的行
   ```bash
   grep "^+.*def \|^+.*class " PROJECT_INDEX/latest_changes.diff
   ```

### 场景4：配置 repomix（适配新项目）

**触发词**："配置项目索引"、"适配项目索引"

**步骤**：

1. **识别项目类型**

   检查项目文件，判断类型：
   - 有 `requirements.txt` 或 `setup.py` → Python
   - 有 `package.json` → JavaScript/TypeScript
   - 有 `pom.xml` 或 `build.gradle` → Java
   - 有 `go.mod` → Go

2. **修改 repomix.config.json**

   根据项目类型调整 `ignore.customPatterns`：

   **Python 项目**：
   ```json
   "customPatterns": [
     "**/*.md",
     "**/*.json",
     "**/*.csv",
     "**/*.xlsx",
     "**/*.db",
     "**/*.png",
     "**/*.jpg",
     "**/*.log",
     "venv/**",
     "__pycache__/**",
     "**/*.pyc",
     "PROJECT_INDEX/history/**",
     ".*/**"
   ]
   ```

   **JavaScript/TypeScript 项目**：
   ```json
   "customPatterns": [
     "**/*.md",
     "**/*.json",
     "**/*.log",
     "node_modules/**",
     "dist/**",
     "build/**",
     "coverage/**",
     "PROJECT_INDEX/history/**",
     ".*/**"
   ]
   ```

   **Java 项目**：
   ```json
   "customPatterns": [
     "**/*.md",
     "**/*.json",
     "**/*.log",
     "target/**",
     "build/**",
     "*.class",
     "*.jar",
     "PROJECT_INDEX/history/**",
     ".*/**"
   ]
   ```

3. **修改输出文件名**

   将 `output.filePath` 中的 `{项目名}` 替换为实际项目名：
   ```json
   "filePath": "PROJECT_INDEX/history\\MyProject_{datetime}.md"
   ```

4. **测试配置**
   ```bash
   npx repomix@latest --config repomix.config.json --compress
   ```

   检查输出：
   - 是否包含预期的代码文件？
   - 是否正确忽略了不需要的文件？
   - 压缩模式是否保留了函数/类签名？

## 常见问题

### Q: repomix 输出文件太大怎么办？

A: 检查 `ignore.customPatterns`，确保忽略了：
- 依赖目录（`node_modules/`, `venv/`, `vendor/`）
- 编译产物（`dist/`, `build/`, `target/`）
- 数据文件（`*.csv`, `*.xlsx`, `*.db`）
- 图片文件（`*.png`, `*.jpg`, `*.gif`）

### Q: 如何只包含特定文件类型？

A: 使用 `include` 字段：
```json
"include": ["**/*.py", "**/*.js", "**/*.ts"]
```

### Q: 架构文档应该包含什么？

A: 建议包含：
- 技术栈
- 核心设计模式
- 数据流
- 模块说明（职责、核心类、扩展方式）
- 扩展点和对外接口
- 集成指南

### Q: 需要为每个文件写描述吗？

A: **不需要**。这是旧方案的问题。新方案中：
- repomix 自动生成文件列表和签名（足够了解项目结构）
- 架构文档只需要写模块级别的说明（不需要函数级别）

### Q: 如何查看历史变化？

A: 对比 `PROJECT_INDEX/history/` 中不同时间戳的文件：
```bash
diff PROJECT_INDEX/history/Project_2026-03-28.md PROJECT_INDEX/history/Project_2026-03-29.md
```

## 优势

✅ **简单可靠**：
- repomix 自动生成，准确无误
- 不需要复杂的脚本和模型指导

✅ **职责清晰**：
- 自动化：文件列表、函数签名
- 人工：架构说明、模块描述

✅ **零维护成本**：
- 不需要修复 bug
- 不需要担心路径匹配、描述保留等问题

✅ **灵活可控**：
- 架构文档随意编辑
- repomix 配置根据项目调整

## 参考资料

- repomix 官方文档：https://github.com/yamadashy/repomix
- 配置模板：`repomix.config.template.json`
- 配置指南：`PROJECT_INDEX/repomix_config_guide.md`
