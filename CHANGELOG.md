# Bright Data Python SDK 更新日志

## 版本 2.5.2 - 更安全的 CSV 导出

- **安全修复**：`export_csv()`（以及用于 `.csv` 文件的 `export()`）现在会自动转义可能被 Excel、Google Sheets 或 LibreOffice 等电子表格应用误判为公式的单元格值，因此打开导出文件时，抓取的数据不会意外触发链接或公式。
- **新增 `sanitize` 选项**：默认启用（`sanitize=True`）。如果需要恢复之前的原始导出行为，请将 `sanitize=False` 传递给 `export_csv()`。

## 版本 2.5.1 - 清理 lint

- **维护**：明确限定了 `ruff` lint 规则的作用域（`select`），并对整个仓库的导入进行了排序。没有行为变更。

## 版本 2.5.0 - CLI 凭据认证 + 抓取工具核心去重

- **CLI 凭据认证回退**：现在只需执行一次 `brightdata login`，也能为 SDK 完成认证。未提供令牌时，`BrightDataClient()` / `SyncBrightDataClient()` 会回退使用 Bright Data CLI 已保存的令牌。解析顺序：`token=` 参数 → `BRIGHTDATA_API_TOKEN` / `BRIGHTDATA_API_KEY` 环境变量 → CLI 登录 → 提示 `brightdata login` 且可据此采取操作的错误。
- **认证来源报告**：现在每个请求都会通过 `User-Agent` 标头（`brightdata-sdk-python/{version} (auth=param|env|cli_credentials)`）报告令牌的获取方式，但绝不会记录令牌本身。
- **内部：共享 `ScraperCore`**：7 个搜索爬虫工具（Amazon、ChatGPT、Instagram、LinkedIn、Pinterest、TikTok、YouTube）现在共用一个构建核心，不再使用手工复制的 `__init__`，从而修复了实现不一致的问题：Amazon/ChatGPT/LinkedIn 搜索缺少其他工具已有的 `BRIGHTDATA_API_TOKEN` 环境回退（减少了 174 行重复代码）。
- 测试：321 项通过（新增 57 项），覆盖所有凭据解析路径、全部 3 个平台路径（Linux/macOS/Windows）和优先级顺序。

## 版本 2.4.0 - 同步功能对齐、通用任务操作、数据集错误报告

- **同步客户端功能对齐**：`SyncBrightDataClient` 现在与异步接口保持一致。新增了 `client.datasets`（修复 `SyncBrightDataClient` `datasets` `AttributeError`）、5 个缺失的爬虫工具（`scrape.tiktok` / `youtube` / `reddit` / `perplexity` / `digikey`）、2 个缺失的垂直搜索（`search.tiktok` / `youtube`）、Pinterest 触发/状态/获取，以及 Instagram 搜索的 `profiles` / `reels_all`。
- **服务级任务操作（通用模式）**：现在每个抓取工具都提供通用的 `status` / `wait` / `fetch` / `to_result(snapshot_id)`（位于 `BaseWebScraper`），而 `DiscoverService` 新增了 `status` / `wait` / `fetch` / `to_result(task_id)`，因此像爬虫一样，仅凭 ID 即可操作已触发的任务。这些功能仅为新增；现有的 `job.fetch()` 等保持不变。
- **Discover 同步手动路径（新增）**：`SyncBrightDataClient` 新增了 `discover_status` / `discover_wait` / `discover_fetch` / `discover_to_result(task_id)`，以及通用的 `DiscoverSnapshot` 句柄。
- **契约变更**：同步 `discover_trigger()` 现在返回 `DiscoverSnapshot`（有类型且可操作的句柄），不再返回仅适用于异步的 `DiscoverJob`（无法在同步模式下使用）。迁移方式：通过 `client.discover_status(snap.task_id)` / `client.discover_fetch(snap.task_id)` 轮询。
- **修复**：数据集快照失败时，现在会显示 API 失败原因；如果没有识别到原因键，则回退显示原始快照状态响应以及 `snapshot_id`，而不是毫无帮助的 `DatasetError: Snapshot failed: None`。`SnapshotStatus` 现在会保留完整的 API 响应（`.raw`），并匹配更多原因键（`error` / `error_message` / `message` / `failure_reason`）。同步路径也继承了此修复。

---

## 版本 2.3.0 - 浏览器 API、爬虫工作室、175 个数据集

- **浏览器 API**：通过 CDP WebSocket 连接云端 Chrome。SDK 会构建 `wss://` URL，你可以使用 Playwright/Puppeteer 连接（`client.browser.get_connect_url()`）
- **爬虫工作室**：触发在 Bright Data IDE 中构建的自定义爬虫，并获取结果（`client.scraper_studio.run()`）
- **新增 75 个数据集**：Agoda、AutoZone、BBC、Best Buy、Bluesky、Booking、Costco、eBay、Etsy、GitHub、Google 新闻/Play/购物、Home Depot、Kroger、Lowe's、Macy's、Microcenter、Ozon、Quora、Realtor、Reddit、Snapchat、TikTok Shop、Tokopedia、Vimeo、Wayfair、Wikipedia、Wildberries、X/Twitter、Yahoo Finance、Zoopla 等，**总计 175 个**
- **代码库清理**：移除无用代码和旧版抽象，将 `datasets/client.py` 从 1635 行缩减至 285 行，修复 `ScrapeJob.to_result()` 崩溃问题，并清理未使用的协议、冗余配置层和过时的 API 模块
- **测试套件重写**：从头重建测试套件，包含 365 项单元测试，通过 `conftest.py` 共享 fixture，并专注于行为覆盖，关键模块的覆盖率现已达到 87–98%（客户端、爬虫工具、SERP、同步客户端、任务生命周期）

---

## 版本 2.2.1 - 100 个数据集 API

### ✨ 新功能

#### 扩展数据集覆盖范围
新增 92 个数据集集成，使总数达到 **100 个数据集**：

- **奢侈品牌**：Loewe、Berluti、Moynat、Hermes、Delvaux、Prada、Montblanc、YSL、Dior、Balenciaga、Bottega Veneta、Celine、Chanel、Fendi
- **电子商务**：Amazon（评论、卖家）、Walmart、Shopee、Lazada、Zalando、Sephora、Zara、Mango、Massimo Dutti、Asos、Shein、Ikea、H&M、Lego、Mouser、Digikey
- **社交媒体**：Instagram（个人资料、帖子）、TikTok、Pinterest（帖子、个人资料）、YouTube（个人资料、视频、评论）、Facebook 页面帖子
- **房地产**：Zillow、Airbnb、Australia Real Estate、Otodom Poland、Zonaprop Argentina、Metrocuadrado、Infocasas Uruguay、Properati、Toctoc、Inmuebles24 Mexico、Yapo Chile
- **商业数据**：Glassdoor（公司、评论、职位）、Indeed（公司、职位）、ZoomInfo、PitchBook、G2、Trustpilot、TrustRadius、Owler、Slintel、Manta、VentureRadar、增强型公司数据、增强型员工数据
- **其他**：全球邮政编码、美国律师、Google 地图评论、Yelp、Xing 个人资料、OLX Brazil、Webmotors Brasil、Chileautos、LinkedIn 职位

#### SERP 分页支持
新增顺序查询功能，可从 Google 获取超过 10 条搜索结果：

```python
async with BrightDataClient() as client:
    # Get up to 50 results with automatic pagination
    results = await client.search.google(
        query="python programming",
        num_results=50  # Fetches multiple pages sequentially
    )
```

---

## 版本 2.2.0 - 数据集 API

### ✨ 新功能

#### 数据集 API
访问 Bright Data 预采集的数据集，并使用筛选和导出功能。

```python
async with BrightDataClient() as client:
    # Filter dataset records
    snapshot_id = await client.datasets.amazon_products(
        filter={"name": "rating", "operator": ">=", "value": 4.5},
        records_limit=100
    )
    # Download results
    data = await client.datasets.amazon_products.download(snapshot_id)
```

<strong>8 个数据集：</strong>LinkedIn 个人资料、LinkedIn 公司、Amazon 商品、Crunchbase 公司、IMDB 电影、NBA 球员统计、Goodreads 图书、世界人口

**导出工具：**
```python
from brightdata.datasets import export_json, export_csv
export_json(data, "results.json")
export_csv(data, "results.csv")
```

### 📓 Notebook
- `notebooks/datasets/linkedin/linkedin.ipynb` - LinkedIn 数据集（个人资料和公司）
- `notebooks/datasets/amazon/amazon.ipynb` - Amazon 商品数据集
- `notebooks/datasets/crunchbase/crunchbase.ipynb` - Crunchbase 公司数据集

---

## 版本 2.1.2 - 网页爬虫与 Notebook

### 🐛 错误修复

#### LinkedIn 职位搜索
修复了 `client.search.linkedin.jobs()`，使其按关键词/地点搜索时使用正确的发现数据集。此前它错误地使用基于 URL 的职位爬虫数据集，而该数据集需要单个职位 URL，而不是搜索参数。

### 📓 Notebook

#### 新增 Notebook
- `notebooks/web_scrapers/linkedin.ipynb` - 针对所有端点的完整 LinkedIn 爬虫测试
- `notebooks/03_serp.ipynb` - Google 搜索 API 测试
- `notebooks/04_web_unlocker.ipynb` - 网络解锁器 HTML 抓取测试

#### 更新 Notebook
- `notebooks/02_pandas_integration.ipynb` - 使用 `asyncio.gather()` 模式进行高效批量抓取

---

## 版本 2.1.1 - Instagram 爬虫与版本集中管理

### ✨ 新功能

#### 完整实现 Instagram 爬虫工具

全面支持 Instagram 抓取，包括基于 URL 的提取和发现端点：

**基于 URL 的抓取（`client.scrape.instagram`）**
- `profiles(url)` - 从 Instagram 个人资料 URL 提取资料数据
- `posts(url)` - 从 Instagram 帖子 URL 提取帖子数据
- `comments(url)` - 从 Instagram 帖子 URL 提取评论
- `reels(url)` - 从 Instagram Reels URL 提取短视频数据

**发现/搜索（`client.search.instagram`）**
- `profiles(user_name)` - 通过精确用户名查找发现个人资料
- `posts(url, num_of_posts, start_date, end_date, post_type)` - 发现个人资料中的帖子
- `reels(url, num_of_posts, start_date, end_date)` - 发现个人资料中的 Reels
- `reels_all(url, num_of_posts, start_date, end_date)` - 发现个人资料中的所有 Reels

```python
async with BrightDataClient() as client:
    # URL-based scraping
    post = await client.scrape.instagram.posts("https://instagram.com/p/ABC123/")
    reel = await client.scrape.instagram.reels("https://instagram.com/reel/XYZ789/")

    # Discovery by username
    profile = await client.search.instagram.profiles(user_name="nasa")

    # Discover posts from profile with filters
    posts = await client.search.instagram.posts(
        url="https://instagram.com/nasa",
        num_of_posts=10,
        start_date="2024-01-01",
        end_date="2024-12-31"
    )
```

### 🔧 内部改进

#### 版本集中管理

现在从单一来源（`pyproject.toml`）管理版本。所有其他文件均通过 `importlib.metadata` 动态读取版本。

**之前（需要更新 5 个文件）：**
- `pyproject.toml`
- `src/brightdata/__init__.py`
- `src/brightdata/_version.py`
- `src/brightdata/core/engine.py`
- `src/brightdata/cli/main.py`

**现在（只需更新 1 个文件）：**
- `pyproject.toml` ← 单一可信来源

**变更：**
- `__init__.py` 现在使用 `importlib.metadata.version("brightdata-sdk")`
- 已删除 `_version.py`（不再需要）
- `engine.py` 导入 `__version__` 以用于 User-Agent 标头
- `cli/main.py` 导入 `__version__` 以用于 `--version` 标志

---

## 版本 2.1.0 - 异步模式、API 简化与错误修复

### ✨ 新功能

#### SERP 异步模式

使用 Bright Data 解锁端点为搜索引擎 API 新增非阻塞异步模式：

```python
from brightdata import BrightDataClient

async with BrightDataClient() as client:
    # Non-blocking - polls for results
    result = await client.search.google(
        query="python programming",
        mode="async",        # Enable async mode
        poll_interval=2,     # Check every 2 seconds
        poll_timeout=30      # Give up after 30 seconds
    )
```

<strong>支持的搜索引擎：</strong>Google、Bing、Yandex

<strong>性能：</strong>SERP 异步模式通常约 3 秒即可完成。

#### 网络解锁器异步模式

为网络解锁器 API 新增非阻塞异步模式：

```python
async with BrightDataClient() as client:
    result = await client.scrape_url(
        url="https://example.com",
        mode="async",
        poll_interval=5,     # Check every 5 seconds
        poll_timeout=180     # Web Unlocker async takes ~2 minutes
    )

    # Batch scraping multiple URLs
    urls = ["https://example.com", "https://example.org"]
    results = await client.scrape_url(url=urls, mode="async", poll_timeout=180)
```

<strong>性能警告：</strong>网络解锁器异步模式大约需要 2 分钟才能完成。如需更快地抓取单个 URL，请使用默认同步模式。

**异步模式的工作方式：**
1. 向 `/unblocker/req` 触发请求（立即返回）
2. 轮询 `/unblocker/get_result`，直至就绪或超时
3. 返回与同步模式相同的数据结构

**主要优势：**
- ✅ 非阻塞请求：抓取期间可继续其他工作
- ✅ 批量优化：触发多个 URL，稍后统一收集
- ✅ 与同步模式相同的数据结构
- ✅ **无需额外配置**：可与现有区域配合使用
- ✅ **无需 customer_id**：从 API 令牌派生

**参阅：**[异步模式指南](docs/async_mode_guide.md)了解详细用法

### 🐛 错误修复

- **修复 SyncBrightDataClient**：移除了错误传递给 `BrightDataClient` 的未使用 `customer_id` 参数
- **修复网络解锁器异步超时**：将默认 `poll_timeout` 从 30 秒改为 180 秒（网络解锁器异步模式大约需要 145 秒）

### 🚨 破坏性变更

#### 移除 GenericScraper
```python
# OLD (v2.0.0)
result = await client.scrape.generic.url("https://example.com")

# NEW (v2.1.0) - Use scrape_url() directly
result = await client.scrape_url("https://example.com")
```

#### 异步方法命名约定
已移除 `_async` 后缀。现在 `method()` 默认为异步版本，`method_sync()` 为同步版本。

```python
# OLD (v2.0.0)
result = await scraper.products_async(url)
await job.wait_async()
data = await job.fetch_async()

# NEW (v2.1.0)
result = await scraper.products(url)
await job.wait()
data = await job.fetch()
```

#### CLI 命令变更
```bash
# OLD
brightdata scrape generic --url https://example.com

# NEW
brightdata scrape url --url https://example.com
```

### ✨ 新功能

#### 完善 SyncBrightDataClient
新增全面的 `sync_client.py`，完整覆盖所有爬虫工具：

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient() as client:
    # All methods work synchronously
    result = client.scrape.amazon.products(url)
    result = client.scrape.linkedin.profiles(url)
    result = client.search.google("query")
```

**支持的同步封装：**
- `SyncAmazonScraper` - 商品、评论、卖家（以及触发/状态/获取）
- `SyncLinkedInScraper` - 个人资料、职位、公司、帖子
- `SyncInstagramScraper` - 个人资料、帖子、评论、Reels
- `SyncFacebookScraper` - 按个人资料获取帖子、按群组获取帖子、评论、Reels
- `SyncChatGPTScraper` - 单个提示词、多个提示词
- `SyncSearchService` - Google、Bing、Yandex
- `SyncCrawlerService` - 爬取、抓取

#### 强制使用上下文管理器
客户端方法现在要求正确初始化上下文管理器：

```python
# Correct usage
async with BrightDataClient() as client:
    result = await client.scrape_url(url)

# Will raise RuntimeError
client = BrightDataClient()
result = await client.scrape_url(url)  # Error: not initialized
```

### 🔄 迁移指南

#### 方法重命名
| 旧版（v2.0.0） | 新版（v2.1.0） |
|--------------|--------------|
| `products_async()` | `products()` |
| `reviews_async()` | `reviews()` |
| `profiles_async()` | `profiles()` |
| `jobs_async()` | `jobs()` |
| `wait_async()` | `wait()` |
| `fetch_async()` | `fetch()` |
| `to_result_async()` | `to_result()` |
| `status_async()` | `status()` |
| `scrape.generic.url()` | `scrape_url()` |

#### 快速迁移
```bash
# Find and replace in your codebase:
_async() → ()
scrape.generic.url → scrape_url
```

### 📚 文档
- 新增[异步模式指南](docs/async_mode_guide.md)：全面介绍异步模式
- 简化 README，并提供更清晰的示例
- 更新所有示例和测试以使用新的命名约定

### 🧪 测试
- 为 `AsyncUnblockerClient` 新增单元测试
- 为 SERP 和网络解锁器异步模式新增集成测试
- 验证向后兼容性（现有代码无需修改即可运行）

---

## 版本 2.0.0 - 架构全面重写

### 🚨 破坏性变更

#### 客户端初始化
```python
# OLD (v1.1.3)
from brightdata import bdclient
client = bdclient(api_token="your_token")

# NEW (v2.0.0)
from brightdata import BrightDataClient
client = BrightDataClient(token="your_token")
```

#### API 结构变更
- **旧版**：扁平化 API，方法直接位于客户端上（`client.scrape()`、`client.search()`）
- **新版**：基于服务的分层 API（`client.scrape.amazon.products()`、`client.search.google()`）

#### 方法命名约定
```python
# OLD
client.scrape_linkedin.profiles(url)
client.search_linkedin.jobs()

# NEW
client.scrape.linkedin.profiles(url)
client.search.linkedin.jobs()
```

#### 返回类型
- **旧版**：原始字典和字符串
- **新版**：结构化的 `ScrapeResult` 和 `SearchResult` 对象，包含元数据和耗时指标

#### Python 版本要求
- **旧版**：Python 3.8+
- **新版**：Python 3.9+（不再支持 Python 3.8）

### 🎯 主要架构变更

#### 1. 异步优先架构
**旧版**：同步执行，并使用 `ThreadPoolExecutor` 实现并发
```python
# Old approach - thread-based parallelism
with ThreadPoolExecutor(max_workers=10) as executor:
    results = executor.map(self.scrape, urls)
```

**新版**：全面采用原生 async/await，并提供同步封装
```python
# New approach - native async (method() is async by default)
async def products(self, url):
    async with self.engine:
        return await self._execute_workflow(...)

# Sync client uses persistent event loop
with SyncBrightDataClient() as client:
    result = client.scrape.amazon.products(url)
```

#### 2. 基于服务的架构
**旧版**：包含所有方法的单体 `bdclient` 类 **新版**：包含专用服务的分层架构
```
BrightDataClient
├── scrape (ScrapeService)
│   ├── amazon (AmazonScraper)
│   ├── linkedin (LinkedInScraper)
│   └── instagram (InstagramScraper)
├── search (SearchService)
│   ├── google
│   ├── bing
│   └── yandex
└── crawler (CrawlService)
```

#### 3. 工作流模式实现
**旧版**：直接发送 HTTP 请求并立即获得响应 **新版**：针对长时间运行的操作采用触发/轮询/获取工作流
```python
# New workflow pattern
snapshot_id = await trigger(payload)     # Start job
status = await poll_until_ready(snapshot_id)  # Check progress
data = await fetch_results(snapshot_id)  # Get results
```

### ✨ 新功能

#### 1. 全面的平台支持
| 平台 | 旧版 SDK | 新版 SDK | 新增功能 |
|----------|---------|---------|------------------|
| Amazon | ❌ | ✅ | 商品、评论、卖家（独立数据集） |
| LinkedIn | ✅ 基础支持 | ✅ 完整支持 | 增强的抓取和搜索方法 |
| Instagram | ❌ | ✅ | 个人资料、帖子、评论、Reels |
| Facebook | ❌ | ✅ | 帖子、评论、小组 |
| ChatGPT | ✅ 基础支持 | ✅ 增强 | 改进提示词交互 |
| Google 搜索 | ✅ | ✅ 增强 | 结构更完善的专用服务 |
| Bing/Yandex | ✅ | ✅ 增强 | 独立的服务方法 |

#### 2. 手动任务控制
```python
# New capability - fine-grained control over scraping jobs
job = await scraper.products_trigger(url)
# Do other work...
status = await job.status()
if status == "ready":
    data = await job.fetch()
```

#### 3. 类型安全的载荷（Dataclass）
```python
# New - structured payloads with validation
from brightdata import AmazonProductPayload
payload = AmazonProductPayload(
    url="https://amazon.com/dp/B123",
    reviews_count=100
)

# Old - untyped dictionaries
payload = {"url": "...", "reviews_count": 100}
```

#### 4. CLI 工具
```bash
# New - command-line interface
brightdata scrape amazon products --url https://amazon.com/dp/B123
brightdata search google --query "python sdk"
brightdata crawler discover --url https://example.com --depth 3

# Old - no CLI support
```

#### 5. 爬虫工具注册表模式
```python
# New - self-registering scrapers
@register("amazon")
class AmazonScraper(BaseWebScraper):
    DATASET_ID = "gd_l7q7dkf244hwxbl93"
```

#### 6. 高级遥测
- 通过堆栈检查跟踪 SDK 函数
- 所有操作均使用微秒精度时间戳
- 全面跟踪各平台成本
- 在结果中提供详细的耗时指标

### 🚀 性能改进

#### 连接管理
- **旧版**：每个请求建立新连接，仅提供基础会话管理
- **新版**：使用 keep-alive 的高级连接池（总计 100 个，每台主机 30 个）

#### 并发模型
- **旧版**：基于线程，受 GIL 限制
- **新版**：基于事件循环，实现真正的异步并发

#### 资源管理
- **旧版**：使用 requests 库进行基础清理
- **新版**：采用上下文管理器和幂等操作的三层清理策略

#### 速率限制
- **旧版**：无内置速率限制
- **新版**：可选的 `AsyncLimiter` 集成（默认为每秒 10 个请求）

### 📦 依赖项变更

#### 移除的依赖项
- `beautifulsoup4` - 解析已移至服务器端
- `openai` - ChatGPT 抓取不再需要

#### 新增依赖项
- `tldextract` - 用于注册表的域名提取
- `pydantic` - 数据验证（可选）
- `aiolimiter` - 速率限制支持
- `click` - CLI 框架

#### 更新的依赖项
- `aiohttp>=3.8.0` - 核心异步 HTTP 客户端（此前同步模式使用 requests）

### 🔧 配置变更

#### 环境变量
```bash
# Supported in both old and new versions:
BRIGHTDATA_API_TOKEN=token
WEB_UNLOCKER_ZONE=zone
SERP_ZONE=zone
BROWSER_ZONE=zone
BRIGHTDATA_BROWSER_USERNAME=username
BRIGHTDATA_BROWSER_PASSWORD=password

# Note: Rate limiting is NOT configured via environment variable
# It must be set programmatically when creating the client
```

#### 客户端参数
```python
# Old (v1.1.3)
client = bdclient(
    api_token="token",                  # Required parameter name
    auto_create_zones=True,              # Default: True
    web_unlocker_zone="sdk_unlocker",   # Default from env or 'sdk_unlocker'
    serp_zone="sdk_serp",               # Default from env or 'sdk_serp'
    browser_zone="sdk_browser",         # Default from env or 'sdk_browser'
    browser_username="username",
    browser_password="password",
    browser_type="playwright",
    log_level="INFO",
    structured_logging=True,
    verbose=False
)

# New (v2.0.0)
client = BrightDataClient(
    token="token",                       # Changed parameter name (was api_token)
    customer_id="id",                    # New parameter (optional)
    timeout=30,                          # New parameter (default: 30)
    auto_create_zones=False,             # Changed default: now False (was True)
    web_unlocker_zone="web_unlocker1",  # Changed default name
    serp_zone="serp_api1",              # Changed default name
    browser_zone="browser_api1",        # Changed default name
    validate_token=False,                # New parameter
    rate_limit=10,                      # New parameter (optional)
    rate_period=1.0                     # New parameter (default: 1.0)
)
# Note: browser credentials and logging config removed from client init
```

### 🔄 迁移指南

#### 基础抓取
```python
# Old
result = client.scrape(url, zone="my_zone", response_format="json")

# New (minimal change)
result = client.scrape_url(url, zone="my_zone", response_format="json")

# New (recommended - platform-specific)
result = client.scrape.amazon.products(url)
```

#### LinkedIn 操作
```python
# Old
profiles = client.scrape_linkedin.profiles(url)
jobs = client.search_linkedin.jobs(location="Paris")

# New
profiles = client.scrape.linkedin.profiles(url)
jobs = client.search.linkedin.jobs(location="Paris")
```

#### 搜索操作
```python
# Old
results = client.search(query, search_engine="google")

# New
results = client.search.google(query)
```

#### 异步迁移
```python
# Old (sync only)
result = client.scrape(url)

# New (async-first)
async def main():
    async with BrightDataClient(token="...") as client:
        result = await client.scrape_url(url)

# Or use sync client
with SyncBrightDataClient(token="...") as client:
    result = client.scrape_url(url)
```


### 🎯 总结

版本 2.0.0 是对 Bright Data Python SDK 的**全面重写**，而不是增量更新。新架构优先考虑：

1. **现代 Python 模式**：异步优先，并提供完善的资源管理
2. **开发者体验**：分层 API、类型安全、CLI 工具
3. **生产环境可靠性**：全面的错误处理和遥测
4. **平台覆盖范围**：为所有主要平台提供专用爬虫工具
5. **灵活性**：三个控制级别（简单、工作流、手动）

这是一个需要修改代码的**破坏性版本**。迁移工作将带来以下收益：
- 并发操作处理能力提升 10 倍
- 新增 50 多个平台专用方法
- 为现代应用提供完善的异步支持
- 全面跟踪耗时和成本
- 面向未来新平台的架构

### 📝 升级检查清单

- [ ] 将 Python 更新至 3.9+
- [ ] 将 import 语句从 `bdclient` 更新为 `BrightDataClient`
- [ ] 迁移至分层 API 结构
- [ ] 更新方法调用以使用新的命名约定
- [ ] 处理新的 `ScrapeResult`/`SearchResult` 返回类型
- [ ] 考虑使用异步优先方式以获得更高性能
- [ ] 针对新的异常类型检查并更新错误处理
- [ ] 按需测试速率限制配置
- [ ] 验证各平台爬虫工具的迁移
