---
name: catl-pyproject
description: 用于构建兼容CATL企业生态的Python项目
license: Apache-2.0
metadata:
  author: byteman
  version: "2.0"
---

# catl-pyproject

该技能用于构建CATL企业生态兼容的Python项目，提供项目初始化、依赖管理、代码规范等功能。
该技能适用于新项目，如果是已有项目，可以使用该技能进行项目结构和代码规范的重构。
重构需要谨慎，避免破坏已有功能。
如果已有项目使用了自定义的ORM模型，重构时需要将其迁移到 `sqlmodel` 或 `sqlalchemy`，并确保数据迁移的正确性。

## when to use me

当CATL员工需要以下场景时使用该技能：
- 创建新的Python项目
- 规范现有项目的结构和代码风格
- 选择合适的技术栈和框架
- 配置开发工具链（linting、formatting、testing）

## 行为约束

使用本技能时，智能体必须遵守以下操作边界：
- **只编辑代码**，不要自行启动服务或运行长期运行的进程（如 `uvicorn`、`run.sh` 等）
- 服务日志需要用户在终端中自行查看，因此不要代替用户启动服务
- 如需验证，可通过编译检查、静态检查（如 `ruff`）等方式确认代码正确性，避免启动服务
- **依赖变更需克制**：仅在确有需要时（如为通过编译/静态检查安装缺失依赖）才执行 `uv add` 等操作；不要随意添加与任务无关的依赖，也不要擅自改动依赖清单中与本任务无关的部分
- **最小改动原则**：只改动实现目标所必需的代码，不顺手重构、重排或改写无关代码，避免超出需求范围
- **不覆盖用户已修改的文件**：对 `common/config.py`、`.env`、`app.toml` 等可能被用户手工调整的文件，修改前先确认
- **不留调试残留**：不遗留调试 `print`、临时 `TODO`、被注释掉的大段代码等无用内容
- **新增文件遵循现有结构**：需要新增文件时，按"项目结构"章节的约定放到位，不自创目录或打乱既有布局

## 项目初始化

创建新项目时，按以下顺序落地：

1. **创建项目骨架**（使用 uv）：
   ```bash
   uv init <project-name>           # 初始化项目，生成 pyproject.toml
   rm -f <project-name>/main.py     # 删除默认的 main.py 入口文件
   ```

2. **建立目录结构**：按"项目结构"章节创建 `project_name/` 下的 `common/`、`db/`、`cmd/` 及业务模块目录。

3. **配置 CATL pypi 源**：按"包管理"章节从系统 `pip config list` 获取 CATL 源，未配置则提示用户提供。

4. **添加基础依赖**（以 fastapi 项目为例）：
   ```bash
   uv add fastapi uvicorn loguru pydantic-settings sqlmodel httpx
   uv add --dev pytest ruff
   ```

5. **同步到 requirements.txt**：将生产依赖写入 `requirements.txt`（宽松版本），用于适配CATL流水线。

6. **配置开发工具**：在 `pyproject.toml` 中按"代码格式化"章节配置 `ruff`。

7. **搭建日志**：在 `project_name/common/log.py` 中按"日志"章节初始化 `loguru` 并拦截标准 `logging`。

8. **搭建配置**：在 `project_name/common/config.py` 中按"项目配置"章节定义配置类，并创建 `etc/app.toml`、`.env`、`.env.example`。

9. **搭建数据库**：在 `project_name/db/session.py` 建立会话，在 `project_name/db/models.py` 定义 ORM 模型，并按需创建 `migrations/` 迁移。

10. **配置测试**：在 `tests/` 下镜像源码目录层级，添加 `pytest` 测试。

11. **创建启动脚本**：编写 `run.sh` 适配CATL容器平台。

12. **提交 Git**：按"Git规范"书写提交信息。

## 项目结构

目录必须按照模块划分：

```
project-name
    ├─etc/                            # 配置文件
    │  └── app.toml                   #   TOML配置
    ├─docs/                           # 文档
    │  └── api.md                     #   接口文档
    ├─migrations/                     # 数据库迁移文件
    │  ├── 20260101_init.sql          #   初始化数据库（定义表接口等）
    │  └── 20260102_add_user.sql      #  添加用户表
    ├─project_name/                   # 源代码目录（可以被直接导入）
    │  ├── cmd/                       # 命令行工具
    │  │   ├── user.py                #   子命令定义
    │  │   └── main.py                #   命令行入口
    │  ├── common/                    # 通用模块
    │  │   ├── config.py              #   配置管理
    │  │   ├── log.py                 #   日志管理
    │  │   ├── exceptions.py           #   异常定义
    │  │   └── utils.py               #   通用工具
    │  ├── db/                        # 数据库相关
    │  │   ├── models.py              #   ORM模型定义
    │  │   └── session.py             #   数据库会话
    │  ├── app1/                      # 应用1（API服务）
    │  │    ├── asgi.py               #   应用1入口（ASGI应用）
    │  │    ├── api/                  #   接口定义
    │  │    │   ├── auth.py           #       认证接口
    │  │    │   └── user.py           #       用户接口
    │  │    └── manager.py            # 业务逻辑
    │  └── app2/                      # 应用2
    │       ├── asgi.py               #   应用2入口（ASGI应用）
    │       ├── api/                  #   接口定义
    │       │   └── task.py           #       任务接口
    │       └── manager.py            #   业务逻辑
    ├── tests/                        # 测试目录
    │   ├── app1/
    │   │   ├── api/
    │   │   │   ├── test_auth.py
    │   │   │   └── test_user.py
    │   │   └── test_manager.py
    │   └── app2/
    │       └── api/
    │           └── test_task.py
    ├── pyproject.toml                # 项目配置（必须）
    ├── README.md                     # 项目说明
    ├── .gitignore                    # Git忽略文件
    ├── .env                          # 开发环境变量
    ├── .env.example                  # 环境变量示例
    ├── requirements.txt              # 依赖文件（适配CATL流水线，必须有）
    └── run.sh                        # 服务启动脚本（适配CATL容器平台，必须有）
```

模块划分原则：
- `common/` - 通用功能（日志、配置、异常、工具函数）
- `db/` - 数据库相关（ORM模型、会话管理）
- `cmd/` - 命令行工具定义

`__init__.py` 要求（项目要求 Python >= 3.10）：
- Python 3.3+ 支持 namespace packages，大部分目录无需 `__init__.py` 即可作为包被导入
- **默认不创建 `__init__.py`**；以下情况可以创建：
  - 包在导入时必须执行初始化副作用、或需要严格控制包的 `__all__` 导出
  - 需要为模块编写详细的文档说明（具备实际价值的模块级文档）
- 判断是否创建的准则：`__init__.py` 中的**内容必须具有实际价值**；若内容只是可有可无的占位说明（如简单重复模块名的废话注释），则说明该文件不必要，不要创建

如果项目比较复杂，需要对接多个系统，可以创建与`project_name/` 同级的目录 `project_name_lib` 来管理第三方服务接口，
例如：s3, 云盘，UAA认证，redis。
第三方服务配置（例如：接口地址，认证信息）应当可通过环境变量或者配置文件进行配置，禁止在代码中硬编码。

## 包管理

必须使用 `uv` 管理项目和依赖，禁止使用 `pip install` 直接安装依赖到系统环境，禁止混用 poetry、pdm 等其他包管理器。

使用 `uv` 时，必须配置 CATL 私有 pypi 源，禁止使用公网默认源：
- 先从系统 `pip config list` 中获取已配置的 CATL pypi 源地址，若存在则直接沿用，并写入 `pyproject.toml` 的 `[[tool.uv.index]]` 中
- 若系统未配置 CATL 源，则提示用户提供私有源地址，再写入 `pyproject.toml`

直接在 `pyproject.toml` 中声明依赖源（不通过环境变量配置）：

```toml
[[tool.uv.index]]
name = "catl"
url = "http://catl-pypi.example.com/simple"
default = true

[tool.uv]
index-strategy = "first-index"
```

> 说明：`index-strategy = "first-index"` 表示仅使用声明的 CATL 源，不额外回退公网源。

使用包管理器，必须将依赖包写入 requirements.txt 文件中，用于适配CATL流水线安装依赖。
requirements.txt 中依赖必须使用宽松版本（例如 `fastapi>=0.100`、`pydantic-settings>=2.0`），禁止锁定精确版本，以兼容流水线环境。

依赖必须分类管理：
```toml
[project]
name = "project-name"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "fastapi>=0.100",
    "uvicorn>=0.30",
    "loguru>=0.7",
    "pydantic-settings>=2.0",
    "sqlmodel>=0.0.16",
    "httpx>=0.27",
]

[dependency-groups]
dev = [
    "pytest>=7.0",
    "ruff>=0.1.0",
]
```

## 项目框架

根据项目类型，选择对应的成熟框架，禁止使用自定义框架，禁止用基本的库实现。

### Web应用

优先使用如下框架：
- **fastapi** - 异步高性能API框架（首选）
- **django** - 全功能Web框架（适合大型项目）
- **flask** - 轻量级Web框架（适合小型项目）

### 命令行

- **click** - 装饰器式的命令行接口
- **typer** - 基于click的现代命令行工具
- **argparse** - 标准库命令行解析

### 数据库访问

必须使用ORM框架访问数据库，禁止使用原生SQL语句（无法通过ORM获取数据的情况除外）：
特别注意：如果存量项目中定义的ORM没有uuid字段，重构时不要强制添加，避免破坏现有功能，接口中可以使用id作为参数。

- **sqlmodel** - SQLModel + Pydantic融合（推荐）
  - ORM模型必须继承自 `sqlmodel.SQLModel`
  - 禁止使用 `pydantic.BaseModel` 作为ORM模型
- **sqlalchemy** - 成熟的ORM框架
  - ORM模型必须继承自 `sqlalchemy.ext.declarative.declarative_base()`
- **peewee** - 轻量级ORM

ORM模型必须包含以下字段（仅针对新项目）：
- id主键
- uuid 字段
- created_at 创建时间(新增数据时自动更新)
- updated_at 更新时间（更新数据时自动更新该字段）

### 异步任务

- **apscheduler** - 基于Python的异步任务调度器
- **celery** - 分布式任务队列
- **arq** - 基于Redis的异步任务队列

### 数据处理

- **pandas** - 结构化数据处理
- **polars** - 高性能DataFrame库

### HTTP客户端

访问第三方服务HTTP接口时，必须使用 `httpx` 或 `requests` 库，禁止使用内置基础库（如 `urllib`）发起请求。
- **httpx** - 支持同步和异步（推荐，配合fastapi异步场景）
- **requests** - 成熟的同步HTTP客户端

使用 `httpx` 时必须显式设置超时时间，默认的5秒过短；超时时间必须可配置（通过配置文件或环境变量），不能硬编码。

```python
import httpx
from project_name.common.config import CONF

async with httpx.AsyncClient(timeout=CONF.http.timeout) as client:
    resp = await client.get("https://api.example.com/resource")
```

## 日志

必须使用 `loguru` 管理日志，禁止使用标准库 `logging` 直接输出日志。

配置要求：
- 默认日志输出到终端，不写入文件
- 拦截 `logging` 模块的日志输出，保证格式一致性
- 支持日志级别控制
- 默认不开启日志颜色输出
- 如需写入文件，必须按天切割，文件名包含日期

```python
import sys
from loguru import logger
from project_name.common.config import CONF

# 移除默认处理器
logger.remove()

# 添加终端输出，颜色开关从配置读取（默认关闭）
logger.add(
    sys.stderr,
    format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level: <8}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>",
    level=CONF.log.level,
    colorize=CONF.log.colorize,
)

# 如需输出到文件，按天切割，文件名包含日期
# logger.add(
#     "logs/app_{time:YYYY-MM-DD}.log",
#     rotation="00:00",
#     retention="30 days",
#     level=CONF.log.level,
#     colorize=CONF.log.colorize,
# )

# 拦截标准logging
import logging

class InterceptHandler(logging.Handler):
    def emit(self, record):
        try:
            level = logger.level(record.levelname).name
        except ValueError:
            level = record.level
        logger.opt(depth=6, exception=record.exc_info).log(level, record.getMessage())

logging.basicConfig(handlers=[InterceptHandler()], level=0)
```

## 项目配置

必须使用配置文件管理配置，禁止在 `.py` 文件中硬编码配置值。

推荐使用 `pydantic-settings` 管理配置：
- 配置文件放在项目根目录下的 `etc` 目录
- 支持格式：`app.toml`、`app.yaml`、`app.json`
- 配置类必须继承自 `pydantic_settings.BaseSettings`
- 禁止使用 `pydantic.BaseModel` 作为顶层配置类
- 必须支持从环境变量读取配置
- 根据模块划分配置类，模块配置可继承 `pydantic.BaseModel`
- 配置不要全大写，应该使用小写字符
- 需要初始化全局配置对象，方便在项目中直接使用
- 使用环境变量覆盖配置时，嵌套字段必须以 `__`（双下划线）作为分隔符，禁止使用 `.`；例如设置 `LOG__LEVEL` 对应 `CONF.log.level`

```python
import os
from pathlib import Path
from urllib.parse import quote_plus

from pydantic import BaseModel, SecretStr
from pydantic_settings import (
    BaseSettings,
    PydanticBaseSettingsSource,
    SettingsConfigDict,
    TomlConfigSettingsSource,
)

class LogConfig(BaseModel):
    level: str = "INFO"
    colorize: bool = False
    file: str | None = None

class DBConfig(BaseModel):
    # 连接模板，占位符来自下方各字段；mysql 示例：
    # "mysql+pymysql://{user}:{password}@{host}:{port}/{database}?charset={charset}"
    connection: str = "sqlite:///data/app.db"
    host: str = "localhost"
    port: int = 3306
    user: str = "root"
    password: SecretStr = SecretStr("")
    database: str = "develop"
    charset: str = "utf8mb4"

    # Connection pool settings
    echo: bool = False
    pool_size: int = 10
    max_overflow: int = 20
    pool_timeout: int = 30
    pool_recycle: int = 3600

    @property
    def url(self):
        db_url = self.connection.format(
            host=self.host,
            port=self.port,
            user=quote_plus(self.user),
            password=quote_plus(self.password.get_secret_value()),
            database=self.database,
            charset=self.charset,
        )
        if db_url.startswith("sqlite:"):
            file = Path(db_url.replace("sqlite:///", ""))
            file.parent.mkdir(parents=True, exist_ok=True)
        return db_url


class HTTPConfig(BaseModel):
    timeout: float = 30.0


class AppConfig(BaseSettings):

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        env_nested_delimiter="__",
        extra="ignore",
        frozen=True,
        toml_file=os.getenv("CONF_FILE") or 'etc/app.toml',
    )

    log: LogConfig = LogConfig()
    db: DBConfig = DBConfig()
    http: HTTPConfig = HTTPConfig()

    @classmethod
    def settings_customise_sources(
        cls,
        settings_cls: type[BaseSettings],
        init_settings: PydanticBaseSettingsSource,
        env_settings: PydanticBaseSettingsSource,
        dotenv_settings: PydanticBaseSettingsSource,
        file_secret_settings: PydanticBaseSettingsSource,
    ):
        return (
            env_settings,
            dotenv_settings,
            file_secret_settings,
            TomlConfigSettingsSource(settings_cls),
            init_settings,
        )

CONF = AppConfig()
```

使用环境变量覆盖配置的完整示例见"环境变量"章节，所有环境变量对应 `CONF` 的字段，嵌套字段使用 `__` 分隔（如 `LOG__LEVEL` 对应 `CONF.log.level`）。

## 代码规范

### API接口

使用RESTful风格的API接口，禁止使用RPC风格的接口。
访问资源使用 uuid 作为唯一标识，禁止使用自增ID作为接口参数。

API 路由必须以 `/api` 开头，并最好加上版本号，例如 `/api/v1/users`：
- 版本号使用 `v1`、`v2` 形式，便于向后兼容与多版本并存
- 健康检查等运维探活接口除外，可放在 `/api` 前缀之外（如 `/healthz`）

若使用 fastapi，通过 router 统一设置前缀：

```python
# project_name/app1/api/user.py
from fastapi import APIRouter

router = APIRouter(prefix="/api/v1/users", tags=["user"])

@router.get("/{user_id}")
async def get_user(user_id: str):
    ...
```

所有对外提供的 Web 应用必须添加健康检查接口（如 `/healthz`），用于容器探活与巡检：
- 该接口**不需要认证**
- 接口默认直接返回空的 JSON（如 `{}`）
- 如需返回状态，可包含简单状态字段，但不得暴露内部敏感信息

```python
# health check 接口示例（不需要认证）
@app.get("/healthz")
async def healthz():
    return {}
```

API必须添加全局异常处理，记录异常堆栈（推荐使用 `logger`），避免未知异常无日志记录。

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from loguru import logger
from project_name.common.exceptions import AppError

app = FastAPI()

@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    logger.warning("应用异常 path={}, error={}", request.url.path, exc)
    return JSONResponse(status_code=400, content={"code": exc.code, "message": str(exc)})

@app.exception_handler(Exception)
async def unhandled_exception_handler(request: Request, exc: Exception):
    logger.exception("未捕获异常 path={}", request.url.path)
    return JSONResponse(status_code=500, content={"code": 500, "message": "服务器内部错误"})
```

业务模块采用分层结构，目录结构示例：
```
app1/
├── api/            # 接口定义层：定义路由、请求/响应模型，负责参数校验
│   ├── auth.py     #   认证接口
│   └── user.py     #   用户接口
└── manager.py      # 业务逻辑层：实现具体业务规则，被接口层调用
```

分层约束：
- `api/` 层只负责HTTP层的解析与响应，不包含业务逻辑
- `manager.py`（业务逻辑）不直接依赖`api/`层，便于复用和测试
- `api/` 层调用 `manager.py` 中的业务方法，并将结果转换为响应

若使用 fastapi，业务模块的注册方式（位于各应用模块的 ASGI 入口文件 `asgi.py`）：
```python
# project_name/app1/asgi.py
from fastapi import FastAPI
from project_name.app1.api import auth, user

app = FastAPI()
app.include_router(auth.router)
app.include_router(user.router)
```

各应用入口文件的命名约定：
- **`asgi.py`** - 异步 Web 框架（如 fastapi）的应用入口，暴露 ASGI 应用对象
- **`wsgi.py`** - 同步 Web 框架（如 flask、django）的应用入口，暴露 WSGI 应用对象
- **`cmd/main.py`** - 命令行工具入口
- **其他名字** - 非 Web、非命令行应用（如 celery 任务、后台定时任务）不使用上述约定名，入口按实际用途命名（如 `celery_app.py`、`worker.py`）

> 每个应用独立一个入口文件，可以由不同名称区分，避免出现多个同名 `main.py` 造成混淆。

### 类型注解

必须使用完整的类型注解，禁止省略类型：

```python
# 错误
def get_user(id):
    return db.query(User).filter(User.id == id).first()

# 正确
def get_user(user_id: int) -> User | None:
    return db.query(User).filter(User.id == user_id).first()
```

### 代码格式化

使用 `ruff` 进行代码检查和格式化（推荐），或 `black` + `flake8`：

```toml
# pyproject.toml
[tool.ruff]
line-length = 88
target-version = "py310"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "N", "UP", "S", "B", "A", "C4", "SIM"]
ignore = ["E501"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

### 错误处理

- 使用自定义异常类，禁止捕获所有异常
- 异常信息必须包含足够的上下文
- 使用 `raise ... from err` 保留异常链
- 禁止在异常处理中直接写 `pass` 吞掉异常；如果确实无需处理，也必须记录警告日志

```python
class AppError(Exception):
    """应用基础异常"""
    pass

class ValidationError(AppError):
    """数据验证错误"""
    pass

try:
    result = process_data(data)
except ValueError as err:
    raise ValidationError(f"数据处理失败: {data}") from err
```

### 日志记录

核心操作必须记录日志，例如：创建、删除、修改资源，下载、上传等。
日志内容应包含操作人、操作对象及操作结果，便于问题排查和审计。

日志消息禁止使用 f-string，必须使用标准格式化处理（如 `{}` 占位符 + `logger.info("... {} ...", arg)` 的方式）。

```python
from loguru import logger

def create_user(user_id: str, operator: str) -> User:
    user = User(...)
    db.add(user)
    db.commit()
    logger.info("创建用户成功 user_id={}, operator={}", user_id, operator)
    return user
```

### 文档字符串

使用 Google 风格的文档字符串：

```python
def calculate_total(items: list[Item], tax_rate: float = 0.1) -> float:
    """计算订单总价。

    Args:
        items: 商品列表
        tax_rate: 税率，默认10%

    Returns:
        包含税费的总价

    Raises:
        ValueError: 当商品列表为空时
    """
    if not items:
        raise ValueError("商品列表不能为空")
    return sum(item.price for item in items) * (1 + tax_rate)
```

## 测试

使用 `pytest` 作为测试框架，测试文件必须命名为 `test_*.py`：

```python
# tests/app1/api/test_user.py
import pytest
from project_name.app1.manager import create_user

def test_create_user_success():
    user = create_user(name="test", email="test@example.com")
    assert user.name == "test"
    assert user.email == "test@example.com"

def test_create_user_invalid_email():
    with pytest.raises(ValidationError):
        create_user(name="test", email="invalid")
```

测试目录结构镜像源代码目录层级：
```
tests/
├── app1/
│   ├── api/
│   │   ├── test_auth.py
│   │   └── test_user.py
│   └── test_manager.py
└── app2/
    └── api/
        └── test_task.py
```

## 环境变量

敏感配置和环境相关配置必须使用环境变量，禁止硬编码在代码中。
环境变量命名与配置模型对应，嵌套字段使用 `__`（双下划线）分隔（见"项目配置"章节）。

```bash
# .env.example
LOG__LEVEL=INFO
LOG__COLORIZE=false
DB__HOST=localhost
DB__PORT=3306
DB__DATABASE=develop
DB__PASSWORD=your-password-here
HTTP__TIMEOUT=30.0
```

## Git规范

### 提交信息

使用 Conventional Commits 规范：

```
feat: 添加用户注册功能
fix: 修复登录验证错误
docs: 更新API文档
style: 格式化代码
refactor: 重构用户服务
test: 添加单元测试
chore: 更新依赖
```

## 安全要求

- 禁止在代码中硬编码密钥、密码等敏感信息
- 禁止将 `.env` 文件提交到版本控制
- 使用 `.gitignore` 排除敏感文件
- 对外暴露的API接口必须进行身份验证和权限检查，内部接口可根据实际暴露情况决定是否需要认证
- 敏感操作（删除、修改他人数据等）必须进行权限校验，不能仅依赖前端隐藏

## 性能优化

- 使用异步 io 处理高并发场景
- 合理使用缓存（Redis、内存缓存）
- 数据库查询必须使用索引
- 避免N+1查询问题
- 使用连接池管理数据库连接
