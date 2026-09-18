# bnext AI 新聞摘要專案

數位時代「AI 與大數據」分類頁的文章索引，以互動式 HTML 捲動式單頁文件呈現。

---

## 檔案結構

```
D:\repo\bnext-ai-news\
├── CLAUDE.md                # 本文件
├── bnext-ai-news.md          # 原始資料（主要資料來源）
└── bnext-ai-news.html        # 互動式捲動頁面（從 MD 同步）
```

---

## 來源與抓取範圍

- 來源：`https://www.bnext.com.tw/categories/ai`（數位時代「AI 與大數據」大分類頁）
- 分頁格式：`?page=2`、`?page=3` ...
- 這個分類頁總頁數高達 450 頁以上，內容持續更新，不做全站同步
- 固定只追蹤最新 20 頁（第 1 到第 20 頁），每次更新都只在這個範圍內比對新增文章，不往後（第 21 頁以後）延伸抓取

---

## 更新指令

當使用者說「請更新」、「更新文章」或類似指令時，依序執行以下流程：

### 第一步：抓取最新文章

用 WebFetch 依序抓取以下 20 頁，每頁提取所有文章的標題、發布時間、連結 URL：

```
https://www.bnext.com.tw/categories/ai
https://www.bnext.com.tw/categories/ai?page=2
...
https://www.bnext.com.tw/categories/ai?page=20
```

固定只抓這 20 頁，不繼續往第 21 頁以後抓取。

### 第二步: 比對新舊, 識別新增文章

將抓到的文章 URL 與 `bnext-ai-news.md` 主題分類清單中的連結比對. 主題分類清單是唯一的完整文章清單, 每篇文章至少屬於一個分類.
URL 中的 article ID(如 `/article/91888/`)為唯一識別碼, 不在清單中的即為新文章, 依網站排列順序記錄標題, 發布時間, 連結備用.

### 第三步: 更新 MD 檔案

更新 `bnext-ai-news.md`:

1. 檔頭: 更新整理日期(今日), 文章總數
2. 更新摘要區塊: 更新總數, 新增數, 更新日期
3. 更新紀錄: 只保留最近 2 次更新
   - 刪除原本的「前次新增」區塊
   - 原本的「本次新增」標題改為「前次新增」
   - 在最上方新增「<今日日期> 本次新增 (N 篇)」區塊, 表格為 `| 標題 | 發布時間 | 分類 | 連結 |`, 分類欄以 ` / ` 分隔多個分類
   - 若本次無新文章, 不新增區塊, 也不輪替
4. 主題分類: 新文章以 `- [標題](URL)` 格式插入對應分類清單頂端; 若出現明顯新主題, 可新增分類

### 第四步: 更新 HTML 檔案

更新 `bnext-ai-news.html`, 與 MD 保持同步:

1. `<header>`: 更新整理日期與文章總數
2. 更新摘要 `<section>`: 更新四個 `.field` 卡片(總數, 新增數, 分類數, 日期)
3. 更新紀錄 `<section>`: 與 MD 相同的輪替方式, 刪除前次區塊, 原本次區塊的 `<h3>` 改為前次並移除該表格所有 `class="is-new"`, 再於 `.section-note` 之後插入新的本次區塊, 列加上 `class="is-new"`
4. 主題分類 `<section>`: 新文章插入對應 `.cat-panel` 內 `<ul>` 的頂端(`<li>` 必須有 `<a href="..." target="_blank">` 連結), 並同步更新對應 `.cat-tab` 上的 `<span class="count">` 篇數

### 第五步：Commit 與 Push (Auto Mode)

內容更新完成後, 若使用者是以 "Auto Mode" 下達 "請更新" 指令(例如同時對 bnext-ai-news 與 one_day_one_ai 兩個專案執行), 則不需再次詢問確認, 直接執行:

1. `git add bnext-ai-news.md bnext-ai-news.html`
2. `git commit -m "更新文章: <新增篇數> 篇 (<今日日期>)"`
3. `git push origin master`

若非 Auto Mode(例如使用者僅在本專案單獨對話中說 "請更新"), 則依一般流程, 在 commit/push 前先向使用者確認.

---

## HTML 設計規範

### 整體架構

單一可捲動頁面，不分投影片，結構固定為：

```html
<body>
  <button id="themeToggle">淺色模式</button>
  <header>...封面資訊...</header>
  <main>
    <section>更新摘要</section>
    <section>更新紀錄</section>
    <section>主題分類</section>
  </main>
</body>
```

`header` 置中呈現標題、副標題與來源／日期／篇數資訊；`main` 內每個 `section` 依序往下捲動，不使用任何分頁、投影片或全螢幕邏輯。

### 更新紀錄格式

`<section>` 內含一個 `<h2>更新紀錄</h2>`, 一段 `.section-note` 說明, 以及最多 2 個日期區塊(本次在上, 前次在下). 表格為 4 欄, 無編號欄:

```html
<section>
  <h2>更新紀錄</h2>
  <p class="section-note">保留最近 2 次更新新增的文章, 發布時間為抓取當下的相對時間.</p>
  <h3>2026-09-18 本次新增 (3 篇)</h3>
  <div class="table-wrap">
    <table>
      <thead>
        <tr><th>標題</th><th>發布時間</th><th>分類</th><th>連結</th></tr>
      </thead>
      <tbody>
        <tr class="is-new"><td>文章標題</td><td>發布時間</td><td>分類A / 分類B</td><td><a href="..." target="_blank">閱讀</a></td></tr>
      </tbody>
    </table>
  </div>
  <h3>2026-09-14 前次新增 (2 篇)</h3>
  <div class="table-wrap">...</div>
</section>
```

分類欄使用 `.cat-tab` 上的分類短名稱.

### 色彩系統（CSS 變數）

採用 Anthropic / Claude 品牌配色，以 CSS 變數管理：

```css
/* Dark mode（預設） */
--bg:           #1C1917;   /* 暖棕黑背景 */
--bg-card:      #292524;   /* 卡片背景 */
--bg-hover:     #302D2A;
--bg-thead:     #242120;
--border:       #3D3833;
--border-row:   #2F2C29;
--text-primary: #E8E3DF;
--text-secondary:#A9A09A;
--text-muted:   #6B6560;
--accent:       #D97757;   /* Claude 銅橘色 */
--accent-hover: #E88A65;
--accent-dim:   rgba(217,119,87,0.18);

/* Light mode（body.light） */
--bg:           #FAF9F7;   /* 暖米白 */
--bg-card:      #F0EDE8;
--accent:       #C86A40;   /* 銅橘深色版（光背景對比用） */
```

禁止使用純黑（`#000`）、純紅（`#ff0000`）或冷調灰藍色系。

### 主題切換

右上角固定 `#themeToggle` 按鈕，純文字（不用 emoji），文字在「淺色模式」／「深色模式」間切換，狀態存於 `localStorage`，預設為 dark mode。

### 更新摘要卡片格式

四個 `.field` 卡片放在 `.api-fields` grid 內：

```html
<div class="api-fields">
  <div class="field">
    <span class="name">260</span>
    <div class="desc">篇文章</div>
  </div>
</div>
```

### 主題分類 nav 格式

主題分類採分類標籤 nav 加篩選：上方一排 `.cat-tab` 按鈕（`.cat-tabs` 內），下方對應 `.cat-panel` 面板（`.cat-panels` 內），同一時間只顯示一個作用中面板，透過 JS 依 `data-target` 與面板 `id` 對應切換：

```html
<div class="cat-tabs" role="tablist">
  <button type="button" class="cat-tab is-active" data-target="cat-product" role="tab" aria-selected="true">分類名稱<span class="count">19</span></button>
  <button type="button" class="cat-tab" data-target="cat-market" role="tab" aria-selected="false">分類名稱<span class="count">30</span></button>
</div>
<div class="cat-panels">
  <div class="cat-panel is-active" id="cat-product" role="tabpanel">
    <ul>
      <li><a href="https://www.bnext.com.tw/article/..." target="_blank">文章標題</a></li>
    </ul>
  </div>
  <div class="cat-panel" id="cat-market" role="tabpanel">
    <ul>...</ul>
  </div>
</div>
```

新增文章到既有分類時，於對應 `.cat-panel` 的 `<ul>` 內插入 `<li>`，並將該分類 `.cat-tab` 的 `<span class="count">` 數字加一；新增分類時需同時新增一個 `.cat-tab` 按鈕與對應 `.cat-panel`，`data-target` 與 `id` 需一致。

### 新文章標示

只有「本次新增」表格的列加 `class="is-new"`, 會顯示左側銅橘色邊框. 輪替成「前次新增」時必須移除:

```html
<tr class="is-new"><td>文章標題</td><td>發布時間</td><td>分類</td><td>...</td></tr>
```

---

## 主題分類（8 大類）

新文章依內容歸入對應分類，一篇文章可跨多個分類：

| 分類 | 關鍵判斷 |
|------|---------|
| 產品發布與更新 | 新模型、新功能、新訂閱方案上線 |
| 企業應用案例 | 特定公司／產業導入 AI 的實戰案例 |
| 產業動態與市場 | 股票、排行榜、使用率調查、市場趨勢 |
| 技術與安全 | 技術原理解析、資安事件、模型風險 |
| 職場與管理 | 管理心法、團隊協作、職場文化相關 |
| 教學與工具應用 | 具體操作教學（快捷鍵、功能盤點等） |
| 政策與法規 | 政府政策、法規、產業規範 |
| 其他 | 不屬於以上分類的文章 |

若有明顯新主題，可新增分類並更新主題分類 section。

---

## 注意事項

- MD 與 HTML 兩個檔案必須同步更新，不可只更新其中一個
- 不維護網站頁次表格, 完整文章清單以主題分類為準, 每篇文章至少歸入一個分類
- 更新紀錄只保留最近 2 次(本次與前次), 更早的紀錄直接刪除
- 發布時間保留抓取當下網站原始的相對時間(「3天前」, 「1個月前」), 不轉換為絕對日期, 只出現在更新紀錄中
- 每次更新後，更新摘要 section 的「新增數」顯示本次新增篇數（非累計）
- 僅維護最新 20 頁範圍，不做全站歷史回溯；第 21 頁以後的舊文章不在追蹤範圍內
