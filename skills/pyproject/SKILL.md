---
name: pyproject
description: A standardized Python project template using uv, FastAPI, and Pydantic.
---

# Python 项目标准模板

本技能用于创建一个结构清晰、规范统一的 Python 项目。

## 什么时候使用

当需要初始化一个新的 Python 后端项目时，特别是基于 FastAPI 的 Web 应用或命令行工具。

## 核心技术栈与约束

1. **包管理**: 强制使用 `uv` 进行依赖管理和环境隔离。
2. **配置管理**: 必须使用 `pydantic-settings`。支持 `.env` 文件和系统环境变量。配置类必须继承自 `BaseSettings`。
3. **数据模型**: 
   - API 交互模型（Request/Response）使用 `pydantic` V2。
   - 数据库 ORM 模型使用 `sqlalchemy` 2.0+ (异步模式优先)。
4. **Web 框架**: 默认使用 `fastapi`。
5. **CLI 工具**: 如需命令行入口，使用 `click` 或 `typer`。
6. **日志系统**: 统一使用 `loguru`。禁止直接使用 `print` 进行调试输出。
7. **测试框架**: 使用 `pytest` 配合 `pytest-asyncio`。
8. **代码质量**: 使用 `ruff` 进行 Linting 和 Formatting。

## 依赖管理规范

### 核心依赖
- `fastapi`
- `uvicorn[standard]`
- `pydantic`
- `pydantic-settings`
- `sqlalchemy`
- `pymysql` (或 `asyncmy` 用于异步 MySQL)
- `aiosqlite` (用于异步 SQLite)
- `loguru`
- `python-dotenv`

### 开发依赖
- `pytest`
- `pytest-asyncio`
- `httpx` (用于 API 测试)
- `ruff`
- `mypy`

## 项目目录结构规范

```text
.
├── .env                  # 本地环境变量配置（不提交到 Git）
├── .env.example          # 环境变量模板
├── pyproject.toml        # 项目元数据和依赖
├── README.md
├── src/
│   └── <package_name>/   # 主包名（连字符转下划线）
│       ├── __init__.py
│       ├── main.py       # 应用入口点
│       ├── common/       # 通用模块
│       │   ├── config.py     # 全局配置加载与 CONF 实例化
│       │   ├── logger.py # 日志初始化与配置
│       │   ├── database.py # 数据库会话管理
│       │   └── utils.py  # 纯工具函数
│       ├── core/         # 核心业务逻辑
│       │   ├── models.py # SQLAlchemy ORM 模型
│       │   └── schemas.py # Pydantic 数据校验模型
│       └── api/          # 路由层
│           ├── __init__.py
│           └── v1/       # API 版本控制
│               ├── endpoints/
│               └── router.py
├── tests/                # 测试用例
│   ├── conftest.py
│   └── test_api/
├── logs/                 # 运行时日志目录（不提交到 Git）
└── data/                 # 本地数据文件（不提交到 Git）
```

## 初始化标准流程

1. **项目创建**: 执行 `uv init <project-name>`。
1. 把以下文件路径添加到 `.gitignore` 文件中
  - uv.lock
  - .python-version
  - .venv
  - .vscode
  - .idea
  - .vscode-server
  - .vscode-server-insiders
  - .pytest_cache
  - .mypy_cache
  - .coverage
  - .mypy-cache
  - .pytest_cache
  - .mypy-cache
  - .pytest_cache
  - .coverage.xml
  - node_modeles
  - logs
  - data
  - tmp

1. 设置python源，默认使用阿里云的源
1. 添加项目默认依赖，命令: `uv add <包名>`
1. 添加开发依赖: `pytest`, `ruff`，命令：`uv add --dev <包名>`
1. 根据项目类型或者用户指定，选择对应的库，并添加其他依赖
1. 添加基础代码
1. 添加业务代码
1. 添加项目readme文档

2. **忽略文件**: 更新 `.gitignore`，包含：
   - `uv.lock`, `.python-version`, `.venv`
   - `*.pyc`, `__pycache__/`
   - `.env`, `logs/`, `data/`, `.pytest_cache/`
   - IDE 配置 (`.vscode/`, `.idea/`)
3. **依赖安装**:
   - 生产依赖: `uv add fastapi uvicorn[standard] pydantic-settings sqlalchemy loguru ...`
   - 开发依赖: `uv add --dev pytest ruff mypy httpx`
4. **源配置**: 在 `pyproject.toml` 中配置阿里云 PyPI 镜像以加速下载。
5. **基础架构搭建**: 按照“基础代码要求”生成核心文件。
6. **代码目录结构**: 不需要每个文件夹都添加 `__init__.py`
7. **模块导入**: 模块导入应该以绝对路径导入，如 `from myproject.common.config import CONF`
7. **文档生成**: 按照项目业务逻辑生成Readme文档。

## 基础代码实现约束

### 1. 配置模块 (`common/config.py`)
- 必须定义一个全局单例 `CONF = Settings()`。
- 配置项必须按功能分组（如 `DBSettings`, `LogSettings`）。
- 项目涉及的常量必须可配置（如：超时时间，周期任务间隔，数据库连接信息，外部系统接口等）
- 敏感信息（密码、密钥）必须在 `.env.example` 中给出占位符，严禁硬编码。

### 2. 日志规范 (`common/logger.py`)
- 格式: `YYYY-MM-DD HH:mm:ss.SSS | {level} | {name}:{function}:{line} - {message}`。
- 必须集成 `contextvars` 以在日志中自动注入 `trace_id`。
- 日志文件应按天切割并保留最近 7 天的记录。

### 3. Web 应用中间件 (`main.py`)
- **Trace ID**: 必须实现中间件，为每个请求生成唯一的 `X-Trace-ID`，并存入 `contextvars`。
- **异常处理**: 统一捕获异常并返回标准化的 JSON 错误响应。
- **CORS**: 根据配置动态启用跨域支持。
- **接口规范**: 所有接口必须遵循 RESTful 规范

### 4. 数据库规范
- 必须使用异步引擎 (`create_async_engine`)。
- 必须实现依赖注入 `get_db()` 供路由使用，确保会话正确关闭。
- 模型定义遵循 SQLAlchemy 2.0 风格（使用 `Mapped` 和 `mapped_column`）。

## 代码审查与交付标准

1. **静态检查**: 运行 `ruff check . --fix` 和 `ruff format .` 确保无警告且格式统一。
2. **类型检查**: 运行 `mypy src/` 确保类型注解完整（除非有明确理由忽略）。
3. **单元测试**: 核心业务逻辑必须覆盖至少 80% 的代码行。
4. **启动验证**: 确保 `PYTHONPATH=src uv run python -m <package_name>.main` 能成功启动服务。

## 常见陷阱规避

- **禁止**在循环中进行数据库查询（N+1 问题）。应使用 `selectinload` 或 `joinedload` 进行预加载。
- **禁止**在 API 响应中直接暴露 ORM 对象，必须通过 Pydantic Schema 转换，以防止敏感字段泄露和序列化错误。
- **禁止**在配置文件中使用明文密码，生产环境应通过环境变量注入。
- **禁止**在异步函数中使用同步阻塞操作（如同步的 `requests` 库或文件 I/O），这会阻塞事件循环。应使用 `httpx` 或 `aiofiles` 等异步库。
- **禁止**全局共享可变状态（如全局列表或字典）来存储请求级数据，必须使用 `contextvars` 或 FastAPI 的 `Depends` 机制。
- **禁止**忽略异步资源的关闭。数据库会话、HTTP 客户端等必须使用 `async with` 或在依赖项中通过 `yield` 正确管理生命周期。
- **禁止**在生产环境中开启 `debug=True` 或 `reload=True`，这会带来严重的安全隐患和性能损耗。
- **禁止**在 Pydantic V2 模型中混用 V1 的配置类（如 `class Config`），应使用 `model_config = SettingsConfigDict(...)`。
- **禁止**在路由函数中直接处理复杂的业务逻辑，应将核心逻辑抽取到 `core/` 或 `service/` 层，保持路由层轻量化。
- **禁止**使用裸字符串作为数据库表名或字段名，必须通过 ORM 模型的属性访问，以防止 SQL 注入并提高重构安全性。
- **禁止**在前端请求中信任任何客户端传入的数据，所有输入必须在 Pydantic Schema 层进行严格校验（类型、长度、格式）。
- **禁止**在日志中记录敏感信息（如密码、Token、身份证号），应对敏感字段进行脱敏处理或过滤。
- **禁止**忽略 `asyncio` 的警告信息，应定期运行 `python -W all` 检查潜在的异步编程错误（如未 await 的协程）。
- **禁止**在 `pyproject.toml` 中使用过于宽泛的版本约束（如 `*` 或 `>=0`），应指定明确的次版本号范围以确保构建可复现性。
- **禁止**在 API 响应中返回 `None` 值，应在 Pydantic Schema 中明确标注字段为可选（`Optional`）或在序列化时过滤掉空值。
- **禁止**使用全局异常处理器捕获所有异常而不记录堆栈信息，必须确保关键错误能输出完整的 Traceback 以便排查。
- **禁止**在单元测试中直接连接真实数据库，必须使用 `pytest` 的 `fixture` 配合内存数据库（如 SQLite :memory:）或 Mock 对象。
- **禁止**在代码中使用硬编码的 URL、IP 地址或端口号，所有网络配置必须通过环境变量或配置文件注入。
- **禁止**忽略 FastAPI 的 `Depends` 依赖项执行顺序，复杂的依赖链应通过显式声明而非隐式调用来管理。
- **禁止**在异步任务中启动后台线程而不进行同步管理，应优先使用 `asyncio.create_task` 并妥善处理任务取消逻辑。
- **禁止**在模型定义中使用 mutable default arguments（如 `list` 或 `dict`），Pydantic V2 中应使用 `field(default_factory=list)`。

## 推荐实践 (Best Practices)

### 1. 文件 IO 操作
- **推荐使用 `aiofiles`**：在任何异步上下文中进行文件读写时，必须使用 `aiofiles` 库。它提供了与内置 `open()` 兼容的异步接口，能有效防止阻塞事件循环。
- **示例**：
  ```python
  import aiofiles
  async with aiofiles.open('file.txt', mode='r') as f:
      contents = await f.read()
  ```

### 2. 依赖注入管理
- **推荐使用 `Annotated` 语法**：在 FastAPI 路由参数中使用 `Annotated` 来声明依赖项，这能使代码更简洁且易于维护。
- **示例**：
  ```python
  from typing import Annotated
  from fastapi import Depends
  
  def get_current_user(token: str = Depends(oauth2_scheme)):
      return verify_token(token)
  
  @app.get("/users/me")
  def read_users_me(current_user: Annotated[User, Depends(get_current_user)]):
      return current_user
  ```

### 3. 数据库查询优化
- **推荐使用 `selectinload`**：在处理一对多或多对多关系时，优先使用 `selectinload` 进行预加载，以避免 N+1 查询问题并减少数据库往返次数。

### 4. 环境变量管理
- **推荐使用 `.env.example` 模板**：在项目根目录提供一份不包含真实敏感信息的 `.env.example` 文件，列出所有必需的环境变量及其说明，方便新成员快速配置环境。

### 5. 异步任务处理
- **推荐使用 `BackgroundTasks`**：对于不需要立即返回结果且耗时较短的操作（如发送邮件、记录审计日志），使用 FastAPI 内置的 `BackgroundTasks`；对于耗时较长或需要可靠执行的任务，建议集成 Celery 或 Arq。

### 6. 文档与注释
- **推荐使用 Google Style Docstrings**：为所有公共函数和类编写文档字符串，配合 `mypy` 和 `ruff` 可以生成高质量的 API 文档并提高代码可读性。

### 7. API 文档规范 (Swagger/Redoc)
- **推荐启用双文档支持**：在 `FastAPI` 实例化时，同时配置 `docs_url` (Swagger UI) 和 `redoc_url` (ReDoc)，以满足不同开发者的查阅习惯。
- **生产环境安全**：在生产环境中，建议通过配置关闭自动文档接口（设置 `docs_url=None` 和 `redoc_url=None`），或通过中间件增加访问权限校验，防止接口细节泄露。
- **规范化标签与摘要**：在每个路由装饰器中明确指定 `tags`、`summary` 和 `description`，确保生成的 OpenAPI 文档结构清晰、语义明确。
- **示例数据填充**：在 Pydantic Schema 中使用 `Field(..., examples=[...])` 提供请求和响应的示例数据，提升前端对接效率。
- **文件路径**：访问文件路径，请使用`Path`库，而不是字符串拼接（除非特殊情况不得不用字符串）
