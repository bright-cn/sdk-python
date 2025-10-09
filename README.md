<img width="1300" height="200" alt="sdk-banner(1)" src="https://github.com/user-attachments/assets/c4a7857e-10dd-420b-947a-ed2ea5825cb8" />

<h3 align="center">Bright Data 提供的 Python SDK，易用且可扩展的网页搜索与抓取方法</h3>
<p></p>

## 安装
在终端中安装此软件包：

```python
pip install brightdata-sdk
```
> 如果在 macOS 上使用，请先为你的项目打开虚拟环境

## 快速开始

创建一个 [Bright Data](https://www.bright.cn/cp/setting/) 账户并复制你的 API Key

### 初始化客户端

```python
from brightdata import bdclient

client = bdclient(api_token="your_api_token_here") # 也可在 .env 文件中设置为 BRIGHTDATA_API_TOKEN
```

### 发起第一个请求
在你的代码中加入一个 serp 函数
```python
results = client.search("best selling shoes")

print(client.parse_content(results))
```

<img width="4774" height="2149" alt="final-banner" src="https://github.com/user-attachments/assets/1ef4f6ad-b5f2-469f-a260-36d1eeaf8dba" />

## 功能

| 功能                        | 函数                   | 描述
|--------------------------|-----------------------------|-------------------------------------
| **抓取任意网站** | `scrape`                    | 使用 Bright 的抓取与反爬/反机器人检测能力抓取任意网站
| **网页搜索**           | `search`                    | 通过查询搜索 Google 及其他搜索引擎（支持批量搜索）
| **网站爬取**         | `crawl`                     | 以高级过滤与深度控制发现并抓取网站中的多页面
| **AI 驱动抽取** | `extract`                  | 使用自然语言查询与 OpenAI 从网站抽取特定信息
| **内容解析**      | `parse_content`             | 从 API 响应（JSON 或 HTML）中提取文本、链接、图片与结构化数据
| **浏览器自动化**   | `connect_browser`           | 获取用于与 Bright Data 抓取浏览器集成的 Playwright/Selenium WebSocket 端点
| **搜索 chatGPT**       | `search_chatGPT`            | 向 chatGPT 提示并抓取其回答，支持多输入与追问
| **搜索 LinkedIn**      | `search_linkedin.posts()`, `search_linkedin.jobs()`, `search_linkedin.profiles()` | 通过特定查询搜索 LinkedIn，并获得结构化数据
| **抓取 LinkedIn**      | `scrape_linkedin.posts()`, `scrape_linkedin.jobs()`, `scrape_linkedin.profiles()`, `scrape_linkedin.companies()` | 抓取 LinkedIn 并获得结构化数据
| **下载函数**   | `download_snapshot`, `download_content`  | 下载同步与异步请求的内容
| **Client 类**         | `bdclient`         | 处理认证、自动创建 Zone，并提供稳健的错误处理选项
| **并行处理**  | **全部函数**  | 所有函数都支持对多个 URL 或查询进行并发处理，并支持多种输出格式

### 试用其中一个函数

#### `Search()`
```python
# 简单的单查询搜索
result = client.search("pizza restaurants")

# 尝试使用多个查询（并行处理），并设置自定义配置
queries = ["pizza", "restaurants", "delivery"]
results = client.search(
    queries,
    search_engine="bing",
    country="gb",
    format="raw"
)
```
#### `scrape()`
```python
# 简单的单 URL 抓取
result = client.scrape("https://example.com")

# 多个 URL（并行处理）与自定义选项
urls = ["https://example1.com", "https://example2.com", "https://example3.com"]
results = client.scrape(
    "urls",
    format="raw",
    country="gb",
    data_format="screenshot"
)
```
#### `search_chatGPT()`
```python
result = client.search_chatGPT(
    prompt="what day is it today?"
    # prompt=["What are the top 3 programming languages in 2024?", "Best hotels in New York", "Explain quantum computing"],
    # additional_prompt=["Can you explain why?", "Are you sure?", ""]  
)

client.download_content(result) # 如果遇到超时错误，会显示 snapshot_id，你可以使用 download_snapshot() 下载
```

#### `search_linkedin.`
可用函数：
client.**`search_linkedin.posts()`**，client.**`search_linkedin.jobs()`**，client.**`search_linkedin.profiles()`**
```python
# 通过姓名搜索 LinkedIn 个人档案
first_names = ["James", "Idan"]
last_names = ["Smith", "Vilenski"]

result = client.search_linkedin.profiles(first_names, last_names) # 也可切换为异步
# 将打印 snapshot_id，可通过 download_snapshot() 函数下载
```

#### `scrape_linkedin.`
可用函数

client.**`scrape_linkedin.posts()`**，client.**`scrape_linkedin.jobs()`**，client.**`scrape_linkedin.profiles()`**，client.**`scrape_linkedin.companies()`**
```python
post_urls = [
    "https://www.linkedin.com/posts/orlenchner_scrapecon-activity-7180537307521769472-oSYN?trk=public_profile",
    "https://www.linkedin.com/pulse/getting-value-out-sunburst-guillaume-de-b%C3%A9naz%C3%A9?trk=public_profile_article_view"
]

results = client.scrape_linkedin.posts(post_urls) # 也可切换为异步

print(results) # 将打印 snapshot_id，可通过 download_snapshot() 函数下载
```

#### `crawl()`
```python
# 带过滤器的单 URL 爬取
result = client.crawl(
    url="https://example.com/",
    depth=2,
    filter="/product/",           # 仅爬取包含 "/product/" 的 URL
    exclude_filter="/ads/",       # 排除包含 "/ads/" 的 URL
    custom_output_fields=["markdown", "url", "page_title"]
)
print(f"Crawl initiated. Snapshot ID: {result['snapshot_id']}")

# 下载爬取结果
data = client.download_snapshot(result['snapshot_id'])
```

#### `parse_content()`
```python
# 解析抓取结果
scraped_data = client.scrape("https://example.com")
parsed = client.parse_content(
    scraped_data, 
    extract_text=True, 
    extract_links=True, 
    extract_images=True
)
print(f"Title: {parsed['title']}")
print(f"Text length: {len(parsed['text'])}")
print(f"Found {len(parsed['links'])} links")
```

#### `extract()`
```python
# 基础抽取（在 query 中包含 URL）
result = client.extract("Extract news headlines from CNN.com")
print(result)

# 使用 URL 参数并返回结构化输出
schema = {
    "type": "object",
    "properties": {
        "headlines": {
            "type": "array",
            "items": {"type": "string"}
        }
    },
    "required": ["headlines"]
}

result = client.extract(
    query="Extract main headlines",
    url="https://cnn.com",
    output_scheme=schema
)
print(result)  # 返回与 schema 匹配的结构化 JSON
```

#### `connect_browser()`
```python
# 适用于 Playwright（默认 browser_type）
from playwright.sync_api import sync_playwright

client = bdclient(
    api_token="your_api_token",
    browser_username="username-zone-browser_zone1",
    browser_password="your_password"
)

with sync_playwright() as playwright:
    browser = playwright.chromium.connect_over_cdp(client.connect_browser())
    page = browser.new_page()
    page.goto("https://example.com")
    print(f"Title: {page.title()}")
    browser.close()
```

**`download_content`**（用于同步请求）
```python
data = client.scrape("https://example.com")
client.download_content(data) 
```
**`download_snapshot`**（用于异步请求）
```python
# 将此函数保存到单独文件
client.download_snapshot("") # 填入你的 snapshot_id
```

> [!TIP]
> 将鼠标悬停在包中的 "search" 或任一函数上，即可查看其全部可用参数。

![Hover-Over1](https://github.com/user-attachments/assets/51324485-5769-48d5-8f13-0b534385142e)

## 函数参数
<details>
    <summary>🔍 <strong>Search(...)</strong></summary>
    
使用 SERP API 进行搜索。接受与 scrape() 相同的参数，另外包括：

```python
- `query`: 搜索查询字符串或查询列表
- `search_engine`: "google", "bing", 或 "yandex"
- 其他参数与 scrape() 相同
```
    
</details>
<details>
    <summary>🔗 <strong>scrape(...)</strong></summary>

使用 Web Unlocker 抓取单个或多个 URL。

```python
- `url`: 单个 URL 字符串或 URL 列表
- `zone`: Zone 标识（若为空将自动配置）
- `format`: "json" 或 "raw"
- `method`: HTTP 方法
- `country`: 两位国家代码
- `data_format`: "markdown", "screenshot" 等
- `async_request`: 启用异步处理
- `max_workers`: 最大并发工作数（默认：10）
- `timeout`: 请求超时（秒，默认：30）
```

</details>
<details>
    <summary>🕷️ <strong>crawl(...)</strong></summary>

以高级过滤发现并抓取网站的多页面。

```python
- `url`: 需要爬取的单个或多个 URL（必填）
- `ignore_sitemap`: 爬取时忽略 sitemap（可选）
- `depth`: 相对输入 URL 的最大爬取深度（可选）
- `filter`: 仅包含某些 URL 的正则（如 "/product/"）
- `exclude_filter`: 排除某些 URL 的正则（如 "/ads/"）
- `custom_output_fields`: 需要包含的输出字段列表（可选）
- `include_errors`: 响应中包含错误（默认：True）
```

</details>
<details>
    <summary>🔍 <strong>parse_content(...)</strong></summary>

从 API 响应中提取与解析有效信息。

```python
- `data`: 来自 scrape()、search() 或 crawl() 的响应数据
- `extract_text`: 提取干净文本内容（默认：True）
- `extract_links`: 从内容中提取所有链接（默认：False）
- `extract_images`: 从内容中提取图片 URL（默认：False）
```

</details>
<details>
    <summary>🤖 <strong>extract(...)</strong></summary>

使用 OpenAI 的 AI 驱动自然语言处理从网站抽取特定信息。

```python
- `query`: 描述需要抽取内容的自然语言查询（必填）
- `url`: 需要抽取的单个或多个 URL（可选——若不提供，将从 query 中解析 URL）
- `output_scheme`: OpenAI 结构化输出的 JSON Schema（可选——启用可靠 JSON 响应）
- `llm_key`: OpenAI API Key（可选——若不提供则使用环境变量 OPENAI_API_KEY）

# 返回：ExtractResult 对象（类字符串，带元数据属性）
# 可用属性：.url, .query, .source_title, .token_usage, .content_length
```

</details>
<details>
    <summary>🌐 <strong>connect_browser(...)</strong></summary>

获取用于与 Bright Data 抓取浏览器进行浏览器自动化的 WebSocket 端点。

```python
# 必需的 client 参数：
- `browser_username`: 浏览器 API 用户名（格式："username-zone-{zone_name}"）
- `browser_password`: 浏览器 API 认证密码
- `browser_type`: "playwright", "puppeteer", 或 "selenium"（默认："playwright"）

# 返回：WebSocket 端点 URL 字符串
```

</details>
<details>
    <summary>💾 <strong>Download_Content(...)</strong></summary>

将内容保存到本地文件。

```python
- `content`: 待保存的内容
- `filename`: 输出文件名（若为 None 将自动生成）
- `format`: 文件格式（"json"、"csv"、"txt" 等）
```

</details>
<details>
    <summary>⚙️ <strong>配置常量</strong></summary>

<p></p>

| 常量               | 默认值 | 描述                     |
| ---------------------- | ------- | ------------------------------- |
| `DEFAULT_MAX_WORKERS`  | `10`    | 最大并行任务数              |
| `DEFAULT_TIMEOUT`      | `30`    | 请求超时时间（秒）    |
| `CONNECTION_POOL_SIZE` | `20`    | 最大并发 HTTP 连接数 |
| `MAX_RETRIES`          | `3`     | 失败重试次数       |
| `RETRY_BACKOFF_FACTOR` | `1.5`   | 指数退避倍数  |

</details>

##  高级配置

<details>
    <summary>🔧 <strong>环境变量</strong></summary>

在项目根目录创建一个 `.env` 文件：

```env
BRIGHTDATA_API_TOKEN=your_bright_data_api_token
WEB_UNLOCKER_ZONE=your_web_unlocker_zone        # 可选
SERP_ZONE=your_serp_zone                        # 可选
BROWSER_ZONE=your_browser_zone                  # 可选
BRIGHTDATA_BROWSER_USERNAME=username-zone-name  # 用于浏览器自动化
BRIGHTDATA_BROWSER_PASSWORD=your_browser_password  # 用于浏览器自动化
OPENAI_API_KEY=your_openai_api_key              # 用于 extract() 函数
```

</details>
<details>
    <summary>🌐 <strong>管理 Zones</strong></summary>

列出所有活跃的 Zones

```python
# 列出所有活跃的 Zones
zones = client.list_zones()
print(f"Found {len(zones)} zones")
```

配置自定义 Zone 名称

```python
client = bdclient(
    api_token="your_token",
    auto_create_zones=False,          # 否则将自动创建 Zone
    web_unlocker_zone="custom_zone",
    serp_zone="custom_serp_zone"
)

```

</details>
<details>
    <summary>👥 <strong>Client 管理</strong></summary>
    
bdclient 类 - 完整参数列表
    
```python
bdclient(
    api_token: str = None,                    # 你的 Bright Data API token（必填）
    auto_create_zones: bool = True,           # 若不存在则自动创建 zones
    web_unlocker_zone: str = None,            # 自定义 Web Unlocker zone 名称
    serp_zone: str = None,                    # 自定义 SERP zone 名称
    browser_zone: str = None,                 # 自定义浏览器 zone 名称
    browser_username: str = None,             # 浏览器 API 用户名（格式："username-zone-{zone_name}"）
    browser_password: str = None,             # 浏览器 API 密码
    browser_type: str = "playwright",         # 浏览器自动化工具："playwright"、"puppeteer"、"selenium"
    log_level: str = "INFO",                  # 日志级别："DEBUG"、"INFO"、"WARNING"、"ERROR"、"CRITICAL"
    structured_logging: bool = True,          # 使用结构化 JSON 日志
    verbose: bool = None                      # 启用详细日志（若为 True 则覆盖 log_level）
)
```
    
</details>
<details>
    <summary>⚠️ <strong>错误处理</strong></summary>
    
bdclient 类
    
SDK 内置输入校验与失败重试逻辑。

如遇到 Zone 相关问题，可使用 **list_zones()** 函数检查你的活跃 Zones，并在[**账户设置**](https://www.bright.cn/cp/setting/users)中确认你的 API Key 具有 **admin permissions**（管理员权限）。
    
</details>

## 支持

如遇到任何问题，请联系 [Bright Data 支持](https://www.bright.cn/contact)，或在本仓库提交 issue。
