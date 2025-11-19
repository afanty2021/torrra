[根目录](../../../CLAUDE.md) > [src/torrra](../CLAUDE.md) > **screens**

## 变更记录 (Changelog)

**2025-11-19 15:30:45** - 完成CSS样式文件分析
- 新增分析了3个CSS文件的详细内容
- 补充了样式系统的设计文档
- 完善了TUI界面组件的样式说明

# Screens TUI界面模块

## 模块职责

Screens模块实现Torrra的终端用户界面（TUI），基于Textual框架构建，提供美观的交互式界面用于种子搜索和下载管理。

## 界面架构

采用屏幕（Screen）模式组织界面，每个屏幕负责特定的功能区域。

```
TUI应用流程:
TorrraApp -> WelcomeScreen -> SearchScreen
```

## 文件结构

```
screens/
├── __init__.py        # 模块初始化
├── welcome.py         # 欢迎屏幕
├── welcome.css        # 欢迎屏幕样式
├── search.py          # 搜索屏幕
└── search.css         # 搜索屏幕样式

../app.css             # 全局应用样式
```

## 样式系统详解

### 全局样式 (app.css)

**文件位置**: `src/torrra/app.css`

**功能**: 定义全局控件样式和工具类

```css
/* 全局输入框样式 */
Input {
    padding: 0 1;
    margin: 0;
    background: transparent;
    border: solid $secondary-muted;
    &:focus {
        border: solid $secondary;
    }
}

/* 全局工具类 */
.hidden {
    display: none;
}
```

**设计特点**:
- **统一输入框样式**: 所有输入框具有一致的视觉效果
- **响应式焦点指示**: 使用颜色变化表示焦点状态
- **工具类支持**: 提供隐藏元素的通用类

### 欢迎屏幕样式 (welcome.css)

**文件位置**: `src/torrra/screens/welcome.css`

**功能**: 美化欢迎界面的布局和视觉效果

```css
WelcomeScreen {
    align: center middle;

    /* 主容器布局 */
    Container#welcome_container {
        max-width: 50;
        height: auto;

        /* ASCII横幅样式 */
        Static#banner {
            text-align: center;
        }

        /* 副标题样式 */
        Static#subtitle {
            color: $text-secondary;
            text-align: center;
        }

        /* 版本信息样式 */
        Static#version {
            color: $foreground-muted;
            text-align: center;
        }

        /* 命令列表容器 */
        Container#commands_container {
            margin-top: 1;
            align-horizontal: center;
            height: auto;

            /* 命令网格布局 */
            Grid#commands_grid {
                width: 75%;
                height: 3;
                grid-size: 2;
                grid-columns: 1fr auto;

                /* 命令文本样式 */
                Static {
                    text-align: left;
                    color: $foreground-muted;

                    /* 主标题样式 */
                    &#title {
                        text-align: center;
                        column-span: 2;
                        color: $text-secondary;
                    }

                    /* 快捷键高亮 */
                    &.key {
                        text-align: right;
                        color: $text-secondary;
                    }
                }
            }
        }
    }
}
```

**设计特点**:
- **居中布局**: 整体界面居中对齐
- **响应式宽度**: 最大宽度限制，保持美观
- **颜色层次**: 使用不同颜色区分信息重要性
- **网格布局**: 命令列表使用2列网格显示
- **视觉层次**: 通过颜色和对齐方式创建清晰的信息层次

### 搜索屏幕样式 (search.css)

**文件位置**: `src/torrra/screens/search.css`

**功能**: 定义搜索界面的复杂布局和交互样式

```css
SearchScreen Vertical {
    /* 搜索输入框 */
    Input#search {
        dock: top;
    }

    /* 结果数据表格 */
    DataTable#results_table {
        background: transparent;
        border: solid $secondary-muted;
        padding: 0 1;
        scrollbar-size: 0 0;
        height: 1fr;
        &:focus {
            border: solid $secondary;
        }
        &:disabled {
            border: solid $secondary-muted;
        }

        /* 表格头部样式 */
        & > .datatable--header,
        & > .datatable--header-hover {
            background: transparent;
        }
    }

    /* 加载指示器和状态文本 */
    Vertical#loader {
        align: center middle;
        Static#status {
            content-align-horizontal: center;
        }
        LoadingIndicator#indicator {
            height: auto;
            content-align-horizontal: center;
        }
    }

    /* 下载区域容器 */
    Container#downloads_container {
        dock: bottom;
        height: auto;
        border: solid $secondary-muted;
        padding: 0 1;
        &:focus {
            border: solid $secondary;
        }

        /* 底部进度条和操作按钮 */
        Horizontal#progressbar-and-actions {
            height: auto;
            ProgressBar#progressbar {
                width: 1fr;
                #bar {
                    width: 90%;
                }
            }
            Static#actions {
                content-align-horizontal: right;
                width: auto;
            }
        }
    }
}
```

**设计特点**:
- **分区布局**: 清晰的功能区域划分
- **响应式表格**: 表格占据主要空间，自适应高度
- **焦点管理**: 明确的焦点指示和状态反馈
- **进度可视化**: 下载进度的直观显示
- **加载动画**: 搜索时的加载指示器

## 欢迎屏幕 (welcome.py)

### 功能特性
- 显示应用横幅和版本信息
- 提供搜索输入框
- 展示键盘快捷键指南
- 支持主题切换（暗黑/明亮）

### 界面组件
```python
class WelcomeScreen(Screen[str]):
    # 横幅区域
    Static(BANNER, id="banner")

    # 副标题
    Static("Find and download torrents...", id="subtitle")

    # 搜索输入框
    Input(placeholder="Search...", id="search")

    # 版本信息
    Static(f"v{__version__}{provider_name}", id="version")

    # 快捷键指南
    Container(id="commands_container")
```

### 交互处理
- **搜索提交**: `on(Input.Submitted, "#search")`
- **返回值**: 返回用户输入的搜索查询字符串

### 样式设计
- 使用ASCII艺术字显示应用名称
- 简洁的键盘快捷键提示
- 响应式布局适配不同终端尺寸

## 搜索屏幕 (search.py)

### 功能特性
- 显示搜索结果表格
- 实时下载进度监控
- 支持暂停/恢复下载
- 动态表格列宽调整
- 下载状态实时更新

### 界面布局
```python
class SearchScreen(Screen[None]):
    # 搜索输入区域
    Input(placeholder="Search...", id="search")

    # 加载指示器
    Vertical(id="loader"):
        Static(id="status")
        LoadingIndicator(id="indicator")

    # 搜索结果表格
    DataTable(
        id="results_table",
        columns=["No.", "Title", "Size", "Seed", "Leech", "Source"]
    )

    # 下载管理区域
    Container(id="downloads_container"):
        Static(id="status")
        ProgressBar(id="progressbar")
        Static(id="actions")
```

### 表格列配置
- **No.**: 序号，固定宽度3字符
- **Title**: 种子标题，动态宽度，最小25字符
- **Size**: 文件大小，固定宽度10字符
- **Seed**: 做种数，固定宽度4字符
- **Leech**: 下载数，固定宽度5字符
- **Source**: 来源，固定宽度6字符

### 交互功能

#### 键盘快捷键
- **s**: 聚焦搜索框
- **p**: 暂停下载
- **r**: 恢复下载
- **Enter**: 选择种子开始下载
- **q**: 退出应用

#### 下载流程
1. **选择种子**: 用户在表格中选择一行
2. **获取元数据**: 解析磁力链接或种子文件
3. **创建下载会话**: 使用libtorrent开始下载
4. **进度监控**: 实时显示下载状态和进度
5. **完成后做种**: 下载完成后继续做种

### 消息处理
```python
class SearchResults(Message):
    def __init__(self, results: list[Torrent], query: str):
        self.results = results
        self.query = query

# 搜索结果处理
@on(SearchResults)
def _show_search_results(self, message: SearchResults):
    # 显示搜索结果在表格中
```

### 下载管理

#### libtorrent集成
```python
# 创建下载会话
self.lt_session = lt.session()
self.lt_session.listen_on(6881, 6891)

# 添加种子下载
params = {
    "save_path": config.get("general.download_path"),
    "storage_mode": lt.storage_mode_t.storage_mode_sparse,
}
self.lt_handle = lt.add_magnet_uri(self.lt_session, magnet_uri, params)
```

#### 进度更新
- **下载状态**: 显示当前状态（下载中/暂停/做种）
- **进度条**: 可视化下载进度
- **统计信息**: 显示种子数、连接数、上传量等

### 响应式设计
```python
def on_resize(self) -> None:
    # 动态调整标题列宽度
    available_width = self.size.width - self.other_cols_total - padding
    title_col_width = max(self.title_col_minimum, available_width)
    table.columns[ColumnKey("title_col")].width = title_col_width
```

## CSS设计原则

### 变量系统
- **主题变量**: `$secondary`, `$secondary-muted`, `$text-secondary`, `$foreground-muted`
- **自适应主题**: 支持暗黑/明亮模式自动切换
- **一致性**: 全统使用统一的颜色变量

### 布局策略
- **Dock布局**: 使用dock属性定义区域位置
- **Flex布局**: 使用1fr实现弹性空间分配
- **网格系统**: 复杂布局使用Grid组件
- **响应式**: 根据终端尺寸动态调整

### 交互状态
- **焦点状态**: 清晰的焦点指示
- **禁用状态**: 适当的视觉反馈
- **加载状态**: 动画指示器
- **hover效果**: 增强交互体验

## 设计模式

### Screen模式
每个功能模块作为独立的Screen，便于维护和扩展。

### 异步工作器
```python
@work(exclusive=True, thread=True)
async def _perform_search(self, query: str):
    # 后台执行搜索，不阻塞UI
```

### 消息传递
使用Textual的消息系统进行组件间通信。

## 用户体验优化

1. **即时反馈**: 加载指示器和状态提示
2. **键盘导航**: 完整的键盘快捷键支持
3. **响应式**: 自适应不同终端尺寸
4. **视觉层次**: 清晰的信息层次和焦点指示
5. **状态保持**: 记住用户界面状态

## 扩展建议

### 新屏幕类型
- **设置屏幕**: 配置管理界面
- **历史屏幕**: 下载历史记录
- **统计屏幕**: 下载统计信息

### 功能增强
- **批量下载**: 支持多选下载
- **分类筛选**: 按类型筛选搜索结果
- **收藏功能**: 收藏感兴趣的种子
- **预览功能**: 预览种子文件列表

### 交互改进
- **右键菜单**: 上下文菜单操作
- **拖拽排序**: 自定义表格列顺序
- **快捷键自定义**: 用户自定义快捷键
- **主题系统**: 更多主题选项

### 样式扩展
- **动画效果**: 添加过渡动画
- **图标支持**: 使用Unicode图标增强视觉效果
- **自定义主题**: 允许用户自定义颜色方案
- **响应式断点**: 更精细的响应式控制

## 性能优化

1. **虚拟滚动**: 大量结果时的性能优化
2. **延迟加载**: 按需加载搜索结果
3. **缓存策略**: 缓存界面状态和样式
4. **异步渲染**: 避免阻塞UI线程
5. **CSS优化**: 减少样式重绘和重排

## 相关文件清单

### Python文件
- `welcome.py` - 欢迎屏幕实现
- `search.py` - 搜索屏幕实现
- `__init__.py` - 模块初始化

### CSS文件
- `app.css` - 全局应用样式
- `welcome.css` - 欢迎屏幕样式
- `search.css` - 搜索屏幕样式