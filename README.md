# public-holidays-blog-data（外部博客数据源）

**用途**：public-holidays.shop 的博客数据**不再编译进主站 build**（否则每天加帖触发全量
redeploy，把 2,208 个预渲染节假日页的 CDN 缓存全部打冷 → FOT 飙升，实测 30GB/月）。

本站博客页（`/[locale]/blog/*`）、sitemap、`CountryHolidayView` 的"相关博客"模块，运行时都从
本仓库的 `blog-posts.json` 读取（ISR，revalidate 60s + `revalidateTag('blog-posts')` 主动清）。

## 数据格式
`blog-posts.json` = `BlogPost[]`，字段同主站 `src/lib/types.ts` 的 `BlogPost`：
```json
{
  "id": 1,
  "title": "How to Calculate Holiday Pay in Germany",
  "slug": "how-to-calculate-holiday-pay-in-germany",
  "category": "finance",
  "author": "Michael Weber",
  "publishedDate": "2025-02-20T10:30:00Z",
  "lastModified": "2025-02-21T09:00:00Z",
  "imageUrl": "https://public-holidays.shop/images/blog/germany-holiday-pay.svg",
  "excerpt": "Understanding German holiday pay laws for employees.",
  "relatedCountries": ["DE"],
  "locale": "en",
  "content": "<p>...</p>",
  "faq": [{ "question": "...", "answer": "..." }]
}
```

## 主站读取地址
主站 `src/lib/blog-source.ts` 读 `process.env.BLOG_DATA_URL`（默认
`https://raw.githubusercontent.com/863683348/public-holidays-blog-data/main/blog-posts.json`）。
**部署前需在 Vercel 给 public-holidays 项目设环境变量 `BLOG_DATA_URL` 指向本仓库 raw 地址。**

## 发布新帖（方案 B 增量流程）
不要用老脚本写主站 `src/lib/blog-posts.ts`（已删除）。
改用主站 `scripts/sync-blog-data.mjs`：
```bash
node scripts/sync-blog-data.mjs --file SEO2026/dayN/ph-blog-dayN.json
# 或：cat newposts.json | node scripts/sync-blog-data.mjs
```
脚本会：① 合并新帖到本仓库 `blog-posts.json`（按 slug+locale 去重）→ ② `git push` 本仓库
（独立 deployment，主站缓存不受影响）→ ③ `POST /api/revalidate`（tags=['blog-posts']）让主站
秒级刷新。主站**完全不 redeploy**。

## 现状
- 初始全量导出：80 篇（en 45 / zh 35），见 `blog-posts.json`。
- 本仓库为公开 repo（`863683348/public-holidays-blog-data`，默认分支 `main`）。
