# WSJ 中文网标题 RSS (Google News)

这个项目会定时抓取 [华尔街日报中文网](https://cn.wsj.com) 在 Google News 上的最新标题，并生成 RSS 文件，方便 Feedly 或其他 RSS 阅读器订阅。

## RSS 地址
https://redondoc.github.io/wsj-zh-rss-proxy/rss.xml

> ⚠️ 注意：GitHub Pages 部署时，RSS 文件位于 `docs/rss.xml`，但访问 URL 去掉 `docs/` 才是最终可访问的 RSS 链接。  
> ⚠️ 注意：RSS 文件必须在 **Public** 仓库中，否则第三方订阅器（如 Feedly）无法访问。  
> ⚠️ 初次添加到 Feedly 或其他订阅器时，可能显示 “No entries”。请等待 **约 15 分钟** 后，工作流首次更新完成再查看。

## 功能说明

- **抓取来源**：Google News 搜索 `site:cn.wsj.com` 的新闻  
- **更新频率**：每 15 分钟自动更新（可在 GitHub Actions 中调整）  
- **内容**：仅标题和链接，方便快速浏览  
- **生成方式**：使用 Python `feedgen` 和 `feedparser` 自动生成 RSS  

## 使用方式

1. 打开 RSS 链接，确认能在浏览器直接访问  
2. 在 Feedly 或其他 RSS 阅读器中添加此 RSS 链接  
3. 更新间隔约 15 分钟，如果订阅后暂时为空，请稍等更新  

## 快速订阅（Feedly 示例）

1. 打开 [Feedly](https://feedly.com/)  
2. 点击左上角的 **+ Add Content**  
3. 粘贴 RSS 链接 `https://redondoc.github.io/wsj-zh-rss-proxy/rss.xml`  
4. 点击 **Follow** 即可

## GitHub Actions 工作流

工作流会自动抓取新闻并生成 `rss.xml`：

```yaml
name: Fetch WSJ via Google News

# 每 15 分钟抓一次，也可以手动触发
on:
  schedule:
    - cron: "*/15 * * * *"
  workflow_dispatch:

jobs:
  fetch:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Ensure docs directory exists
        run: mkdir -p docs

      - name: Fetch WSJ titles from Google News RSS
        run: |
          sudo apt-get update
          sudo apt-get install -y python3-pip
          pip3 install feedgen feedparser
          python3 - <<'EOF'
          from feedgen.feed import FeedGenerator
          import feedparser

          # RSS 源
          source_url = 'https://news.google.com/rss/search?q=site:cn.wsj.com&hl=zh-CN&gl=CN&ceid=CN:zh-Hans'
          d = feedparser.parse(source_url)

          fg = FeedGenerator()
          fg.title('WSJ 中文网标题 RSS (Google News)')
          fg.link(href='https://redondoc.github.io/wsj-zh-rss-proxy/rss.xml')
          fg.description('仅标题和链接，适合 Feedly 订阅')

          for entry in d.entries:
              fe = fg.add_entry()
              fe.title(entry.title)
              fe.link(href=entry.link)

          fg.rss_file('docs/rss.xml')
          EOF

      - name: Commit RSS if changed
        run: |
          git config --global user.name "redondoc"
          git config --global user.email "youremail@example.com" # 请替换为你的真实邮箱
          git add docs/rss.xml
          git diff --quiet && git diff --staged --quiet || git commit -m "Update RSS"
          git push origin main

## 注意事项

- 仓库必须是 **Public**  
- GitHub Pages 源必须部署自包含 RSS 文件的分支（例如 `main` 或 `docs/`）  
- 刚添加到 Feedly 时可能显示 “No entries”，请等待首次更新完成
