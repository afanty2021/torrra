[根目录](../../../CLAUDE.md) > [src/torrra](../CLAUDE.md) > **core**

# Core 核心功能模块

## 模块职责

Core模块提供Torrra应用的基础设施服务，包括配置管理、缓存系统、异常处理、常量定义和共享上下文。

## 文件结构

```
core/
├── __init__.py      # 模块初始化
├── cache.py         # 缓存系统实现
├── config.py        # 配置管理
├── constants.py     # 常量定义
├── context.py       # 共享上下文
└── exceptions.py    # 自定义异常
```

## 配置管理 (config.py)

### Config类
负责管理用户配置的读写，使用TOML格式存储配置文件。

#### 主要方法
- `get(key_path: str) -> str`: 获取配置值
- `set(key_path: str, value: str) -> None`: 设置配置值
- `list() -> list[str]`: 列出所有配置项

#### 配置文件位置
- **路径**: `~/.config/torrra/config.toml`
- **自动创建**: 如果不存在会创建默认配置

#### 默认配置结构
```toml
[general]
download_path = "~/Downloads"
remember_last_path = true
```

## 缓存系统 (cache.py)

基于diskcache实现的搜索结果缓存系统。

#### 缓存函数
- `has_cache(key: str) -> bool`: 检查缓存是否存在
- `get_cache(key: str) -> Any`: 获取缓存内容
- `set_cache(key: str, value: Any, expire: int) -> None`: 设置缓存
- `delete_cache(key: str) -> None`: 删除缓存

#### 缓存配置
- **缓存目录**: `~/.cache/torrra/`
- **默认TTL**: 300秒 (5分钟)
- **缓存键格式**: `{prefix}:{sha256_hash}`

## 异常处理 (exceptions.py)

定义应用专用的异常类型。

#### 异常类型
- `ConfigError`: 配置相关错误
- `JackettConnectionError`: Jackett连接错误
- `ProwlarrConnectionError`: Prowlarr连接错误

## 共享上下文 (context.py)

提供全局共享的配置实例。

```python
from torrra.core.config import Config
config = Config()  # 全局单例
```

## 常量定义 (constants.py)

定义应用中使用的常量。

```python
CACHE_TTL = 300  # 缓存过期时间（秒）
```

## 使用示例

### 配置操作
```python
from torrra.core.context import config

# 获取配置
download_path = config.get("general.download_path")

# 设置配置
config.set("indexers.jackett.url", "http://localhost:9117")

# 列出所有配置
all_configs = config.list()
```

### 缓存操作
```python
from torrra.core.cache import make_cache_key, has_cache, get_cache, set_cache

# 创建缓存键
key = make_cache_key("jackett", "query_string")

# 检查缓存
if has_cache(key):
    results = get_cache(key)

# 设置缓存
set_cache(key, search_results)
```

## 设计特点

1. **类型安全**: 使用类型注解确保代码安全
2. **自动创建**: 配置文件和缓存目录自动创建
3. **跨平台**: 使用platformdirs确保跨平台兼容性
4. **异常安全**: 自定义异常提供清晰的错误信息
5. **单例模式**: 配置实例采用单例模式共享

## 扩展建议

1. **配置验证**: 添加配置项的类型和范围验证
2. **缓存策略**: 实现更复杂的缓存过期和清理策略
3. **配置迁移**: 支持配置文件版本升级和迁移
4. **监控日志**: 添加配置访问和缓存操作的日志记录