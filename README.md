# bnext AI 新聞摘要

追蹤數位時代「AI 與大數據」分類頁（[bnext.com.tw/categories/ai](https://www.bnext.com.tw/categories/ai)）的文章索引，以互動式 HTML 捲動式單頁文件呈現。

## 檔案結構

```
├── CLAUDE.md          # 專案規則文件（來源、抓取範圍、更新流程、HTML 設計規範）
├── bnext-ai-news.md    # 文章資料（主要資料來源）
└── bnext-ai-news.html  # 互動式捲動頁面（從 MD 同步）
```

## 使用方式

直接用瀏覽器開啟 `bnext-ai-news.html` 即可瀏覽，支援深色／淺色主題切換與主題分類篩選。

## 更新文章

在 Claude Code 中對本專案說「請更新」，會依 `CLAUDE.md` 中定義的流程，重新抓取來源頁最新 20 頁範圍內的文章，比對新增後同步更新 `bnext-ai-news.md` 與 `bnext-ai-news.html`。

## 追蹤範圍

- 來源固定為 `https://www.bnext.com.tw/categories/ai`
- 只維護最新 20 頁範圍，不做全站歷史回溯
- 主題分類（8 大類）：產品發布與更新、企業應用案例、產業動態與市場、技術與安全、職場與管理、教學與工具應用、政策與法規、其他

詳細規則請見 `CLAUDE.md`。
