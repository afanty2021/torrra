[根目录](../../../CLAUDE.md) > [src/torrra](../CLAUDE.md) > **utils**

# Utils 工具模块

## 模块职责

Utils模块提供应用程序的通用工具函数和辅助功能，包括索引器工具、文件系统操作、通用辅助函数等，为其他模块提供基础支撑。

## 文件结构

```
utils/
├── __init__.py        # 模块初始化
├── fs.py             # 文件系统工具
├── helpers.py        # 通用辅助函数
└── indexer.py        # 索引器工具
```

## 文件系统工具 (fs.py)

### 资源路径管理
处理应用资源文件的路径解析，支持开发环境和打包环境的差异。

#### 核心函数
```python
def get_resource_path(relative_path: str) -> str:
    """
    获取资源文件的绝对路径，自动检测运行环境

    Args:
        relative_path: 相对于包根目录的资源路径

    Returns:
        资源文件的绝对路径
    """
```

#### 环境检测
- **开发环境**: 直接使用源码目录路径
- **打包环境** (PyInstaller): 使用`_MEIPASS`临时目录

#### 使用场景
```python
# 获取CSS样式文件
css_path = get_resource_path("app.css")

# 获取屏幕样式
welcome_css = get_resource_path("screens/welcome.css")
search_css = get_resource_path("screens/search.css")
```

## 通用辅助函数 (helpers.py)

### 文件大小格式化
```python
def human_readable_size(size_bytes: float) -> str:
    """
    将字节数转换为人类可读的文件大小格式

    Args:
        size_bytes: 文件大小（字节）

    Returns:
        格式化的文件大小字符串，如 "1.23 MB"

    Examples:
        >>> human_readable_size(1024)
        "1.00 KB"
        >>> human_readable_size(1048576)
        "1.00 MB"
    """
```

#### 支持的单位
- B (字节)
- KB (千字节)
- MB (兆字节)
- GB (吉字节)
- TB (太字节)
- PB (拍字节)

### 延迟导入工具
```python
def lazy_import(dotted_path: str):
    """
    延迟导入模块，支持动态加载

    Args:
        dotted_path: 模块的点分路径，如 "module.Class"

    Returns:
        导入的对象

    Raises:
        ImportError: 模块或对象不存在时抛出
    """
```

#### 使用场景
- **插件系统**: 动态加载索引器类
- **配置驱动**: 根据配置加载不同的实现
- **性能优化**: 延迟加载重型依赖

#### 使用示例
```python
# 动态加载索引器类
indexer_cls = lazy_import("torrra.indexers.jackett.JackettIndexer")
indexer = indexer_cls(url, api_key)

# 动态加载异常类
error_cls = lazy_import("torrra.core.exceptions.JackettConnectionError")
```

## 索引器工具 (indexer.py)

### 索引器命令处理
```python
def handle_indexer_command(
    name: IndexerName,
    indexer_cls_str: str,
    connection_error_cls_str: str,
    url: str | None,
    api_key: str | None,
    no_cache: bool,
):
    """
    处理索引器相关的CLI命令

    Args:
        name: 索引器名称 ("jackett" | "prowlarr")
        indexer_cls_str: 索引器类的完整路径
        connection_error_cls_str: 连接错误类的完整路径
        url: 索引器服务器URL
        api_key: API密钥
        no_cache: 是否禁用缓存
    """
```

#### 处理逻辑
1. **参数验证**: 检查URL和API密钥参数
2. **配置加载**: 从配置文件读取未提供的参数
3. **连接验证**: 验证索引器服务连接
4. **配置保存**: 将有效的连接信息保存到配置
5. **应用启动**: 启动TUI应用

### 自动检测功能
```python
def auto_detect_indexer_and_run(no_cache: bool):
    """
    自动检测默认索引器并启动应用

    Args:
        no_cache: 是否禁用缓存
    """
```

#### 检测流程
1. **读取配置**: 获取`indexers.default`配置
2. **加载参数**: 读取对应索引器的URL和API密钥
3. **启动应用**: 调用`handle_indexer_command`启动

## 使用示例

### 配置驱动的索引器加载
```python
from torrra.utils.indexer import handle_indexer_command

# 处理Jackett命令
handle_indexer_command(
    name="jackett",
    indexer_cls_str="torrra.indexers.jackett.JackettIndexer",
    connection_error_cls_str="torrra.core.exceptions.JackettConnectionError",
    url="http://localhost:9117",
    api_key="your_api_key",
    no_cache=False,
)
```

### 工具函数使用
```python
from torrra.utils.helpers import human_readable_size, lazy_import
from torrra.utils.fs import get_resource_path

# 格式化文件大小
size_str = human_readable_size(1073741824)  # "1.00 GB"

# 动态加载类
MyClass = lazy_import("some.module.MyClass")

# 获取资源路径
css_path = get_resource_path("styles/main.css")
```

## 设计特点

### 模块化设计
- **单一职责**: 每个文件专注特定功能域
- **松耦合**: 工具函数独立，可复用性强
- **类型安全**: 完整的类型注解

### 错误处理
- **友好提示**: 清晰的错误信息指导用户
- **配置检查**: 验证配置文件的完整性
- **连接诊断**: 详细的连接错误诊断

### 性能考虑
- **延迟加载**: 按需导入模块减少启动时间
- **缓存路径**: 资源路径解析结果可缓存
- **高效格式化**: 优化的文件大小格式化算法

## 扩展建议

### 新工具函数
1. **网络工具**: 网络连接检测、代理配置
2. **格式化工具**: 更多数据格式化函数
3. **验证工具**: URL、API密钥格式验证
4. **日志工具**: 统一的日志记录接口

### 现有功能增强
1. **智能缓存**: 资源路径解析结果缓存
2. **配置验证**: 更全面的配置文件验证
3. **性能监控**: 函数执行时间监控
4. **国际化**: 多语言支持的格式化函数

### 代码质量改进
1. **单元测试**: 为所有工具函数添加测试
2. **性能基准**: 建立性能基准测试
3. **文档完善**: 更详细的API文档和示例
4. **类型检查**: 增强类型注解覆盖

## 最佳实践

### 文件路径处理
- 使用`get_resource_path()`而不是硬编码路径
- 考虑跨平台兼容性（路径分隔符）
- 处理资源文件不存在的情况

### 动态导入
- 使用`lazy_import()`减少不必要的依赖
- 处理导入失败的情况
- 提供清晰的错误信息

### 配置管理
- 验证配置参数的有效性
- 提供默认值和配置示例
- 支持配置文件热重载

这个模块为整个Torrra应用提供了坚实的基础设施支持，确保代码的可维护性和可扩展性。