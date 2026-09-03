---
name: nacos-mcp-generator
description: 快速创建对接 Nacos 平台的 MCP 项目模板，基于 uv、fastmcp、pydantic 和 nacos-mcp-wrapper-python 构建。
---

# Nacos MCP 项目生成器

本技能用于创建一个结构清晰、规范统一的对接Nacos 平台的MCP项目。

## 什么时候使用

当需要初始化一个新的对接Nacos 平台的MCP项目时使用该技能。

## 核心技术栈与约束

1. **包管理**: 强制使用 `uv` 进行依赖管理和环境隔离。
2. **配置管理**: 必须使用 `pydantic-settings`。支持 `.env` 文件和系统环境变量。配置类必须继承自 `BaseSettings`。
3. **数据模型**: tools 交互模型使用 `pydantic` V2。
4. **MCP 框架**: 使用 `fastmcp`。
5. **日志系统**: 统一使用 `loguru`。禁止直接使用 `print` 进行调试输出。
7. **测试框架**: 使用 `pytest` 配合 `pytest-asyncio`。
8. **代码质量**: 使用 `ruff` 进行 Linting 和 Formatting。

## 依赖管理规范

### 核心依赖
- `nacos-mcp-wrapper-python==1.0.9`
- `nacos-sdk-python==2.0.`
- `nacos-maintainer-sdk-python==0.1.2`
- `fastmcp>=2.8.0`
- `pydantic-settings`

### 开发依赖
- `pytest`
- `ruff`


## 项目目录结构规范

```text
├── <package_name>/   # 主包名（连字符转下划线）
│   └── common/       # 通用模块
│       ├── config.py     # 全局配置加载与 CONF 实例化
│       └── utils.py  # 纯工具函数
├── mcp.py            # 纯工具函数
├── tests/                # 测试用例
├── .env                  # 本地环境变量配置（不提交到 Git）
├── .env.example          # 环境变量模板
├── pyproject.toml        # 项目元数据和依赖
├── README.md
├── main.py               # 项目启动入口
└── logs/                 # 运行时日志目录（不提交到 Git）
```

## 初始化标准流程

1. **项目创建**: 执行 `uv init <project-name>`
2. **设置内部Pypi源**:
    ```
    [[tool.uv.index]]
    url = 'http://jfrogreader:AP7SoAxHBehQfx7oGp1VSeCqzm6@jfrog.catlbattery.com/artifactory/api/pypi/pypi-group/simple'
    default = true
    ```
3. **添加项目核心依赖**: 使用 `uv add <包名>`
    - nacos-sdk-python==2.0.9
    - nacos-maintainer-sdk-python==0.1.2
    - nacos-mcp-wrapper-python==1.0.9
    - fastmcp>=2.8.0
    - loguru
    - pydantic-settings

4. **添加开发依赖**: 使用 `uv add --dev <包名>`
    - pytest
    - ruff

5. 把以下文件路径添加到 `.gitignore` 文件中
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

6.  **忽略文件**: 更新 `.gitignore`，包含：
   - `uv.lock`, `.python-version`, `.venv`
   - `*.pyc`, `__pycache__/`
   - `.env`, `logs/`, `data/`, `.pytest_cache/`
   - IDE 配置 (`.vscode/`, `.idea/`)

7. 添加基础代码
8. 初始化环境变量


### 项目目录结构
```text
.
├── .env                    # 本地环境变量配置（不提交到 Git）
├── pyproject.toml          # 项目元数据和依赖
├── README.md
├── <package_name>/         # 主包名（连字符转下划线）
│     ├── common/           # 通用模块
│     │   ├── config.py     # 全局配置加载与 CONF 实例化
│     │   └── utils.py      # 纯工具函数
│     └── mcp.py            # MCP 应用和tools
├── main.py                 # 应用入口点
└── tests/                  # 测试用例
```

### 基础代码

### 添加配置

在 `<package_name>/common/config.py` 中添加nacos和mcp 配置:
```python

from pydantic import BaseModel, SecretStr
from pydantic_settings import BaseSettings, SettingsConfigDict


class NacosSettings(BaseModel):
    server_addr: str = "rosefinch-nacos-sit.project-273:8848"
    username: str = "nacos"
    password: SecretStr = SecretStr("")

    namespace: str = "public"


class MCPSettings(BaseModel):
    name: str = "python-nacos-mcp-demo"
    instructions: str = "This is a simple MCP server demo"
    version: str = "1.0.0"
    transport: str = 'streamable-http'
    host: str = "0.0.0.0"
    port: int = 8081



class AppConfig(BaseSettings):
    """Application configuration."""

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        env_nested_delimiter=".",
        case_sensitive=False,
        extra="ignore",
    )

    nacos: NacosSettings = NacosSettings()
    mcp: MCPSettings = MCPSettings()


CONF = AppConfig()

```

### 添加tool示例

在 `<package_name>/mcp.py` 中创建 MCP应用，以及tool示例:

**注意: MCP应用必须使用 nacos_mcp_wrapper 的 NacosMCP类， 而不是fastmcp的类。**

示例:

```python
from nacos_mcp_wrapper.server.nacos_mcp import NacosMCP, NacosSettings

from nacos_mcp_demo.common.config import CONF

APP = NacosMCP(
    CONF.mcp.name,
    nacos_settings=NacosSettings(
        SERVER_ADDR=CONF.nacos.server_addr,
        USERNAME=CONF.nacos.username,
        PASSWORD=CONF.nacos.password.get_secret_value(),
    ),
    instructions=CONF.mcp.instructions,
    version=CONF.mcp.version,
    host=CONF.mcp.host,
    port=CONF.mcp.port,
)


@APP.tool()
def add_numbers(a: float, b: float) -> float:
    """两个数字相加"""
    return a + b

```

### 添加启动代码

修改入口文件 `main.py`，代码如下:

```python
from nacos_mcp_demo.common.config import CONF
from nacos_mcp_demo.mcp import APP


def main():
    APP.run(transport=CONF.mcp.transport)


if __name__ == "__main__":
    main()

```
### 初始化开发环境

1. 执行 `uv sync`

在 .env 文件中添加如下环境变量

```env
// 设置你的MCP服务名
MCP.NAME = "your-mcp-server-name"

// 设置NACOS 服务密码
NACOS.PASSWOR = Nacos@135790
```

## 在 README.md 文件中添加项目文档

至少包含如下内容：

1. 项目说明
2. 配置说明
3. 启动命令 `python main.py`
