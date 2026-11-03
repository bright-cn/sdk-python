# Bright Data Python SDK

[Bright Data](https://www.bright.cn/) API 的官方 Python SDK。抓取任何网站、获取 SERP 结果、绕过反爬虫检测和验证码，并访问 100 多个现成数据集。

[![Python](https://img.shields.io/badge/python-3.9%2B-blue)](https://www.python.org/) [![许可证](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## 安装

```bash
pip install brightdata-sdk
```

## 配置

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取 API 令牌：

```bash
export BRIGHTDATA_API_TOKEN="your_api_token_here"
```

**已通过 CLI 登录？** SDK 无需配置即可使用，它会自动回退至 `brightdata login` 存储的凭据。

## 快速开始

此 SDK **原生支持异步**。同时也提供同步客户端（参阅[同步客户端](#同步客户端)）。

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        result = await client.scrape_url("https://example.com")
        print(result.data)

asyncio.run(main())
```

## 使用示例

### 网页爬虫

```python
async with BrightDataClient() as client:
    result = await client.scrape_url("https://example.com")
    print(result.data)
```

#### 网页爬虫异步模式

如需进行非阻塞网页爬虫，请使用 `mode="async"`。这会触发请求并返回 `response_id`，SDK 将自动轮询，直至结果就绪：

```python
async with BrightDataClient() as client:
    # Triggers request → gets response_id → polls until ready
    result = await client.scrape_url(
        url="https://example.com",
        mode="async",
        poll_interval=5,    # Check every 5 seconds
        poll_timeout=180    # Web Unlocker async can take ~2 minutes
    )
    print(result.data)

    # Batch scraping multiple URLs concurrently
    urls = ["https://example.com", "https://example.org", "https://example.net"]
    results = await client.scrape_url(url=urls, mode="async", poll_timeout=180)
```

**工作方式：**
1. 向 `/unblocker/req` 发送请求 → 立即返回 `response_id`
2. 轮询 `/unblocker/get_result?response_id=...`，直至就绪或超时
3. 返回抓取的数据

**何时使用异步模式：**
- 批量抓取大量 URL
- 继续其他工作的同时在后台处理

<strong>性能说明：</strong>网络解锁器异步模式通常大约需要 2 分钟才能完成。如需更快地获取单个 URL 的结果，请使用默认同步模式（不传递 `mode` 参数）。

### 搜索引擎（SERP）

跨 Google、Bing 和 Yandex 搜索。Google 和 Bing 返回解析后的自然搜索结果（标题/URL/描述/排名）；Yandex 没有上游解析器，因此返回原始 HTML。

```python
async with BrightDataClient() as client:
    # Google — parsed results
    result = await client.search.google(query="python scraping", num_results=10)
    for item in result.data:
        print(item["title"], item["url"])

    # Bing — parsed results (same shape as Google)
    result = await client.search.bing(query="python scraping", num_results=10)
    for item in result.data:
        print(item["title"], item["url"])

    # Yandex — raw HTML only (parse it yourself with BeautifulSoup etc.)
    result = await client.search.yandex(query="python scraping", num_results=10)
    print(f"{len(result.raw_html)} chars of HTML")
```

批量查询：传递字符串列表而不是单个字符串，每项查询都会并发运行，并返回一个 `List[SearchResult]`。

#### SERP 异步模式

如需非阻塞 SERP 请求，请使用 `mode="async"`：

```python
async with BrightDataClient() as client:
    # Non-blocking - polls for results
    result = await client.search.google(
        query="python programming",
        mode="async",
        poll_interval=2,   # Check every 2 seconds
        poll_timeout=30    # Give up after 30 seconds
    )

    for item in result.data:
        print(item['title'], item['link'])
```

**何时使用异步模式：**
- 包含大量查询的批量操作
- 继续其他工作的同时在后台处理
- 抓取可能比平时耗时更长时

<strong>注意：</strong>异步模式使用相同的区域，并返回与同步模式相同的数据结构，无需额外配置！

### 网页爬虫工具 API

SDK 包含适用于热门网站的现成爬虫工具：Amazon、LinkedIn、Instagram、Facebook 等。

**模式：**`client.scrape.<platform>.<method>(url)`

**示例：Amazon**
```python
async with BrightDataClient() as client:
    # Product details
    result = await client.scrape.amazon.products(url="https://amazon.com/dp/B0CRMZHDG8")

    # Reviews
    result = await client.scrape.amazon.reviews(url="https://amazon.com/dp/B0CRMZHDG8")

    # Sellers
    result = await client.scrape.amazon.sellers(url="https://amazon.com/dp/B0CRMZHDG8")
```

**可用爬虫工具：**
- `client.scrape.amazon`：商品、评论、卖家
- `client.scrape.linkedin`：个人资料、公司、职位、帖子
- `client.scrape.instagram`：个人资料、帖子、评论、Reels
- `client.scrape.facebook`：帖子、评论、Reels、页面、Marketplace、活动
- `client.scrape.tiktok`：个人资料、按关键词/个人资料/URL 获取帖子
- `client.scrape.youtube`：视频、频道、评论
- `client.scrape.reddit`：帖子、评论、按关键词获取帖子
- `client.scrape.pinterest`：帖子、个人资料、按关键词/个人资料获取帖子
- `client.scrape.chatgpt`：发送提示词、获取响应
- `client.scrape.perplexity`：返回 AI 排名结果的查询
- `client.scrape.digikey`：电子元器件商品

### 浏览器 API

通过 Chrome DevTools Protocol（CDP）访问云托管的 Chrome 实例。SDK 会构建连接 URL，你可以使用 Playwright、Puppeteer 或 Selenium 操作浏览器。

```python
from brightdata import BrightDataClient
from playwright.async_api import async_playwright

client = BrightDataClient(
    browser_username="brd-customer-<id>-zone-<zone>",
    browser_password="<password>",
)

url = client.browser.get_connect_url(country="us")  # country is optional

async with async_playwright() as pw:
    browser = await pw.chromium.connect_over_cdp(url)
    page = await browser.new_page()
    await page.goto("https://example.com")
    html = await page.content()
    await browser.close()
```

<strong>适用场景：</strong>需要完整浏览器自动化的网站，JS 渲染、登录流程、交互式点击。对于普通 HTML 获取，建议使用 `client.scrape_url()`。

### Discover API

AI 排名的网页搜索。与 SERP（按搜索引擎顺序返回结果）不同，Discover 接收一个 `query` 和一个 `intent` 短语，并按相关性重新排名。还可选择将完整页面内容提取为 Markdown。

```python
async with BrightDataClient() as client:
    result = await client.discover(
        query="artificial intelligence trends 2026",
        intent="latest AI technology developments",
        country="us",
        num_results=10,
    )
    for item in result.data:
        print(f"[{item['relevance_score']:.2f}] {item['title']}  {item['link']}")
```

对于长时间运行的发现任务，请分别触发和轮询：

```python
job = await client.discover_trigger(query="...", intent="...")
result = await job.wait_and_fetch(timeout=60)
```

<strong>何时使用 Discover 而不是 SERP：</strong>如果需要根据自然语言意图进行*实体级*相关性排名（例如“查找专注于可持续发展的 AI 公司”），请使用 Discover。如果需要原始搜索引擎结果，请使用 SERP。

### 爬虫工作室

运行在 [Bright Data 爬虫工作室](https://www.bright.cn/cp/scrapers)中构建的自定义采集器。一次调用即可触发任务、轮询直至就绪并返回记录：

```python
async with BrightDataClient() as client:
    data = await client.scraper_studio.run(
        collector="c_abc123",                       # your collector ID
        input={"url": "https://example.com"},       # collector-specific input
        timeout=180,
    )
    print(f"Got {len(data)} records")
```

如需手动控制生命周期，请使用 `client.scraper_studio.trigger()` → `.status()` → `.fetch()`。

## 数据集 API

访问 Bright Data 提供的 100 多个现成数据集：来自热门平台的预采集结构化数据。

```python
async with BrightDataClient() as client:
    # Filter a dataset — returns a snapshot_id
    snapshot_id = await client.datasets.imdb_movies(
        filter={"name": "title", "operator": "includes", "value": "black"},
        records_limit=5
    )

    # Download when ready (polls until snapshot is complete)
    data = await client.datasets.imdb_movies.download(snapshot_id)
    print(f"Got {len(data)} records")

    # Quick sample: .sample() auto-discovers fields, no filter needed
    # Works on any dataset
    snapshot_id = await client.datasets.imdb_movies.sample(records_limit=5)
```

**将结果导出到文件：**

```python
from brightdata.datasets import export

export(data, "results.json")   # JSON
export(data, "results.csv")    # CSV
export(data, "results.jsonl")  # JSONL
```

**可用数据集类别：**
- <strong>电子商务：</strong>Amazon、Walmart、Shopee、Lazada、Zalando、Zara、H&M、Shein、IKEA、Sephora 等
- <strong>商业情报：</strong>ZoomInfo、PitchBook、Owler、Slintel、VentureRadar、Manta
- <strong>职位与人力资源：</strong>Glassdoor（公司、评论、职位）、Indeed（公司、职位）、Xing
- <strong>评论：</strong>Google 地图、Yelp、G2、Trustpilot、TrustRadius
- <strong>社交媒体：</strong>Pinterest（帖子、个人资料）、Facebook 页面
- <strong>房地产：</strong>Zillow、Airbnb 和 8 个以上区域平台
- <strong>奢侈品牌：</strong>Chanel、Dior、Prada、Balenciaga、Hermes、YSL 等
- <strong>娱乐：</strong>IMDB、NBA、Goodreads

**查看可用字段：**

```python
metadata = await client.datasets.imdb_movies.get_metadata()
for name, field in metadata.fields.items():
    print(f"{name}: {field.type}")
```

## 异步用法

并发运行多个请求：

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        urls = ["https://example.com/page1", "https://example.com/page2", "https://example.com/page3"]
        tasks = [client.scrape_url(url) for url in urls]
        results = await asyncio.gather(*tasks)

asyncio.run(main())
```

### 手动触发/轮询/获取

对于长时间运行的抓取任务：

```python
async with BrightDataClient() as client:
    # Trigger
    job = await client.scrape.amazon.products_trigger(url="https://amazon.com/dp/B123")

    # Wait for completion
    await job.wait(timeout=180)

    # Fetch results
    data = await job.fetch()
```

## 同步客户端

对于较简单的使用场景，请使用 `SyncBrightDataClient`：

```python
from brightdata import SyncBrightDataClient

with SyncBrightDataClient() as client:
    result = client.scrape_url("https://example.com")
    print(result.data)

    # All methods work the same
    result = client.scrape.amazon.products(url="https://amazon.com/dp/B123")
    result = client.search.google(query="python")
```

详情请参阅 [docs/sync_client.md](docs/sync_client.md)。

## 账户与区域

检查账户、列出区域并清理未使用的区域：

```python
async with BrightDataClient() as client:
    # Verify token + connectivity (never raises — returns False on any error)
    if await client.test_connection():
        print("Connected")

    # Account status, usage stats, credit balance
    info = await client.get_account_info()
    print(info)

    # All zones on this account, with type and usage
    zones = await client.list_zones()
    for z in zones:
        print(z["name"], z["type"])

    # Delete a zone — destructive, confirm before calling
    await client.delete_zone("zone_to_remove")
```

## 故障排除

**RuntimeError：无法在异步上下文中使用 SyncBrightDataClient**
```python
# Wrong - using sync client in async function
async def main():
    with SyncBrightDataClient() as client:  # Error!
        ...

# Correct - use async client
async def main():
    async with BrightDataClient() as client:
        result = await client.scrape_url("https://example.com")
```

**RuntimeError：BrightDataClient 未初始化**
```python
# Wrong - forgot context manager
client = BrightDataClient()
result = await client.scrape_url("...")  # Error!

# Correct - use context manager
async with BrightDataClient() as client:
    result = await client.scrape_url("...")
```

## 许可证

MIT 许可证
