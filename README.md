<div align="center">
    <img width="160" height="160" alt="antyr" src="https://mrprfrm.github.io/antyr/assets/antyr-logo.png" />
    <h1>antyr</h1>
    <p><strong>Pipeline-like web crawling</strong></p>
    <p>A minimalistic web crawling toolkit for Python</p>
    <p>
        <a href="https://pypi.org/project/antyr"><img src="https://img.shields.io/pypi/v/antyr" alt="PyPI"></a>
        <a href="https://pypi.org/project/antyr"><img src="https://img.shields.io/pypi/pyversions/antyr" alt="Python"></a>
        <a href="https://github.com/mrprfrm/antyr/blob/master/LICENSE"><img src="https://img.shields.io/pypi/l/antyr" alt="License"></a>
        <a href="https://mrprfrm.github.io/antyr"><img src="https://img.shields.io/badge/docs-mkdocs-blue" alt="Docs"></a>
    </p>
</div>

---

This project focuses on **core crawling primitives**: making HTTP requests, consuming responses as streams, and persisting streamed content with explicit, cancellation-safe lifetimes.

Unlike full-featured frameworks, `antyr` does not implement an end-to-end scraping pipeline. Parsing, extraction logic, data modeling, retries, scheduling, and storage are left to the caller.

> If you want a batteries-included scraping framework, consider Scrapy.

## Installation

Install via pip:

```bash
pip install antyr
```

Or using [uv](https://github.com/astral-sh/uv):

```bash
uv add antyr
```

Please note that the following packages will be installed alongside `antyr`:

- `trio` – structured concurrency runtime
- `httpx[socks]` – HTTP client with SOCKS proxy support
- `stem` – Tor control port integration

## Quickstart

The examples below show how to fetch a resource and either process its contents as a stream or persist it to disk.

### Fetch and process a response as a stream

Instead of buffering the entire response in memory, the response can be processed incrementally as it is received.

```python
import trio
from antyr import HttpCrawler

async def main() -> None:
    async with HttpCrawler("https://httpbin.org") as crawler:
        stream = await crawler.fetch("/json").content_stream()

        async for chunk in stream:
            # process chunk

trio.run(main)
```

If the response body is an archive, it can be extracted before processing by calling `extract()`. The extracted content is exposed through the same streaming interface.

```python
import trio
from antyr import HttpCrawler

async def main() -> None:
    async with HttpCrawler("https://example.com") as crawler:
        stream = await crawler.fetch("/archive.zip").extract()

        async for chunk in stream:
            # process chunk

trio.run(main)
```

### Stream to disk

Stream the response body directly to disk.

```python
import trio
from antyr import HttpCrawler

async def main() -> None:
    async with HttpCrawler("https://httpbin.org") as crawler:
        await crawler.fetch("/image/png").save("downloads")

trio.run(main)
```

The target filename is derived from the response headers or URL and normalized for filesystem safety.
