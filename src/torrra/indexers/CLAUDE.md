[根目录](../../../CLAUDE.md) > [src/torrra](../CLAUDE.md) > **indexers**

# Indexers 索引器模块

## 模块职责

Indexers模块提供与种子索引器服务的集成适配器，支持Jackett和Prowlarr两个主要的索引器服务，实现搜索功能的统一接口。

## 支持的索引器

- **Jackett**: 流行的种子聚合器，支持大量公共和私有 trackers
- **Prowlarr**: Sonarr/Radarr团队开发的索引器管理工具

## 文件结构

```
indexers/
├── __init__.py      # 模块初始化
├── jackett.py       # Jackett API适配器
└── prowlarr.py      # Prowlarr API适配器
```

## 统一接口设计

两个索引器都实现相同的接口模式：

### 核心方法
- `__init__(url: str, api_key: str, timeout: int = 10)`: 初始化索引器
- `search(query: str, use_cache: bool = True) -> list[Torrent]`: 搜索种子
- `validate() -> bool`: 验证索引器连接
- `_normalize_result(r: dict[str, Any]) -> Torrent`: 标准化搜索结果

## Jackett适配器 (jackett.py)

### API端点
- **搜索**: `{url}/api/v2.0/indexers/all/results`
- **验证**: `{url}/api/v2.0/indexers/nonexistent_indexer/results`

### 请求参数
```python
params = {
    "apikey": api_key,
    "query": query
}
```

### 响应字段映射
```python
def _normalize_result(self, r: dict[str, Any]) -> Torrent:
    return Torrent(
        title=r.get("Title", "unknown"),
        size=r.get("Size", 0),
        seeders=r.get("Seeders", 0),
        leechers=r.get("Peers", 0),
        source=r.get("Tracker", "unknown"),
        magnet_uri=r.get("MagnetUri") or r.get("Link"),
    )
```

### 连接验证
通过请求不存在的索引器来验证API密钥的有效性，HTTP 500状态码且包含"nonexistent_indexer"表示连接正常。

## Prowlarr适配器 (prowlarr.py)

### API端点
- **搜索**: `{url}/api/v1/search`
- **验证**: `{url}/api/v1/health`

### 请求参数
```python
params = {
    "apikey": api_key,
    "query": query
}
```

### 响应字段映射
```python
def _normalize_result(self, r: dict[str, Any]) -> Torrent:
    return Torrent(
        title=r.get("title", "unknown"),
        size=r.get("size", 0),
        seeders=r.get("seeders", 0),
        leechers=r.get("leechers", 0),
        source=r.get("indexer", "unknown"),
        magnet_uri=r.get("magnetUrl") or r.get("downloadUrl"),
    )
```

### 连接验证
通过健康检查端点验证连接，直接返回API响应状态。

## 缓存集成

两个索引器都集成了缓存系统：

```python
async def search(self, query: str, use_cache: bool = True) -> list[Torrent]:
    key = make_cache_key(f"{self.__class__.__name__.lower()}", query)

    # 检查缓存
    if use_cache and has_cache(key):
        raw_data = cast(list[TorrentDict], get_cache(key))
        return [Torrent.from_dict(d) for d in raw_data]

    # 执行搜索
    results = [...]

    # 保存到缓存
    if use_cache and results:
        set_cache(key, [t.to_dict() for t in results])

    return results
```

## 错误处理

### 连接错误
- `JackettConnectionError`: Jackett连接失败
- `ProwlarrConnectionError`: Prowlarr连接失败

### 错误类型
1. **网络错误**: 服务器不可达
2. **认证错误**: API密钥无效 (HTTP 401)
3. **服务器错误**: 意外的HTTP状态码

## 使用示例

### 直接使用
```python
from torrra.indexers.jackett import JackettIndexer
from torrra.indexers.prowlarr import ProwlarrIndexer

# Jackett
jackett = JackettIndexer("http://localhost:9117", "api_key")
results = await jackett.search("movie title")

# Prowlarr
prowlarr = ProwlarrIndexer("http://localhost:9696", "api_key")
results = await prowlarr.search("tv series")
```

### 验证连接
```python
if await jackett.validate():
    print("Jackett连接正常")
else:
    print("Jackett连接失败")
```

## 配置示例

### Jackett配置
```toml
[indexers.jackett]
url = "http://localhost:9117"
api_key = "your_jackett_api_key"
```

### Prowlarr配置
```toml
[indexers.prowlarr]
url = "http://localhost:9696"
api_key = "your_prowlarr_api_key"
```

## 设计特点

1. **统一接口**: 两个索引器提供相同的API接口
2. **异步支持**: 所有网络操作都是异步的
3. **缓存集成**: 自动缓存搜索结果提高性能
4. **错误处理**: 详细的错误信息和异常类型
5. **结果标准化**: 不同索引器的响应统一转换为Torrent对象

## 扩展新索引器

要添加新的索引器支持：

1. 在`indexers/`目录下创建新的适配器文件
2. 实现标准的接口方法
3. 在`_types.py`中添加新的索引器名称类型
4. 在`__main__.py`中添加对应的CLI命令
5. 更新相关的工具函数

### 新索引器模板
```python
class NewIndexer:
    def __init__(self, url: str, api_key: str, timeout: int = 10):
        self.url = url.rstrip("/")
        self.api_key = api_key
        self.timeout = timeout

    async def search(self, query: str, use_cache: bool = True) -> list[Torrent]:
        # 实现搜索逻辑
        pass

    async def validate(self) -> bool:
        # 实现验证逻辑
        pass

    def _normalize_result(self, r: dict[str, Any]) -> Torrent:
        # 实现结果标准化
        pass
```

## 性能考虑

1. **超时设置**: 默认10秒超时，可根据网络环境调整
2. **缓存策略**: 5分钟TTL，减少重复API调用
3. **连接复用**: 使用httpx的AsyncClient进行连接复用
4. **结果去重**: 在搜索结果中移除重复的磁力链接