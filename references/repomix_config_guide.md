# repomix 配置说明

## 当前项目配置

当前 `repomix.config.json` 针对本项目（Python + Vue）配置。

### 关键配置项

**1. 输出路径**
```json
"filePath": "PROJECT_INDEX/history\\KFsystem_{datetime}.md"
```
- `{datetime}` 会被替换为时间戳
- 建议格式：`PROJECT_INDEX/history\\{项目名}_{datetime}.md`

**2. 压缩模式**
```json
"compression": {
  "enabled": true,
  "keepSignatures": true,    // 保留函数/类签名
  "keepDocstrings": true,    // 保留文档字符串
  "keepInterfaces": true     // 保留接口定义
}
```

**3. 忽略文件**
```json
"ignore": {
  "customPatterns": [
    "**/*.md",           // 忽略文档
    "**/*.json",         // 忽略配置
    "**/*.xlsx",         // 忽略数据文件
    "**/*.png",          // 忽略图片
    "frontend/node_modules/**",  // 忽略依赖
    "PROJECT_INDEX/history/**",  // 忽略历史索引
    "Datas/**",          // 忽略数据目录
    ".*/**"              // 忽略隐藏目录
  ],
  "useGitignore": true,
  "useDefaultIgnore": true
}
```

## 适配其他项目

### Python 项目
保持当前配置，主要关注：
- `.py` 文件（默认包含）
- 忽略 `venv/`、`__pycache__/`、`*.pyc`（默认忽略）

### JavaScript/TypeScript 项目
修改 `customPatterns`：
```json
"customPatterns": [
  "**/*.md",
  "node_modules/**",
  "dist/**",
  "build/**",
  "coverage/**",
  "**/*.test.js",
  "**/*.spec.ts"
]
```

### Java 项目
修改 `customPatterns`：
```json
"customPatterns": [
  "**/*.md",
  "target/**",
  "build/**",
  "*.class",
  "*.jar"
]
```

### Go 项目
修改 `customPatterns`：
```json
"customPatterns": [
  "**/*.md",
  "vendor/**",
  "bin/**",
  "*.exe"
]
```

## 配置建议

### 1. 项目特定的忽略规则
根据项目类型添加：
- 数据文件：`*.csv`, `*.xlsx`, `*.db`
- 编译产物：`dist/`, `build/`, `target/`
- 依赖目录：`node_modules/`, `vendor/`, `venv/`
- 测试文件：`**/*.test.*`, `**/*.spec.*`（可选）

### 2. 包含特定文件
如果需要包含特定文件类型：
```json
"include": ["**/*.py", "**/*.js", "**/*.ts"]
```

### 3. 输出文件命名
建议使用项目名作为前缀：
```json
"filePath": "PROJECT_INDEX/history\\{项目名}_{datetime}.md"
```

## 快速配置模板

创建 `repomix.config.template.json`，复制到新项目后修改：
1. 修改 `output.filePath` 中的项目名
2. 修改 `ignore.customPatterns` 适配项目类型
3. 运行 `npx repomix@latest --config repomix.config.json --compress`

## 验证配置

运行后检查：
1. 输出文件是否包含预期的代码文件
2. 是否正确忽略了不需要的文件（图片、数据、依赖）
3. 压缩模式是否保留了函数/类签名
