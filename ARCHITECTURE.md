# 系統架構書 — Moscito 叫序資料庫（HCP 版）

> 架構照搬 [bridge-relay-bidding](https://github.com/Bonki1268/bridge-relay-bidding)
> （北歐接力精確體系網站）。本文件只記錄與它不同的地方；共用的演算法細節
> （表頭偵測、角色分類、文字渲染管線、已知地雷）請看那邊的 ARCHITECTURE.md。

## 1. 專案是什麼

把 Bo-Yin Yang《Terrorist's Moscito》（2000）整理成的七份 Excel 叫序表，
轉成純靜態、有 RWD 的網站，部署在 GitHub Pages。

原文以 rp（Reese / Relay Points：A=3、K=2、Q=1，單張大牌扣 1）計算強度。
本站的 xlsx 已全部換算成大牌點，寫成「約 X 點（原文 rp）」——原文 rp 留在
括號內，因為接力回答的級數是以 rp 一級一級示出的。換算方式見每份 xlsx
「說明」分頁最後的「rp → 大牌點換算」。

```
原始資料/*.xlsx  ──  python3 build.py  ──▶  網站/*.html  ──  push  ──▶  GitHub Pages
```

`網站/` 底下的 `.html` 全部是產出物，不要手改；`網站/assets/style.css`、
`site.js` 是手動維護（從 bridge-relay-bidding 複製後小改），build 不會覆寫。

## 2. 目錄結構

```
moscito-relay-bidding/
├── build.py                     ← 網站產生器
├── ARCHITECTURE.md
├── .github/workflows/pages.yml   ← 與 bridge-relay-bidding 相同，部署 網站/
├── 原始資料/
│   ├── 一梅花開叫.xlsx  一方塊開叫.xlsx  高花開叫.xlsx  一無王開叫.xlsx
│   └── 阻擊開叫.xlsx    接力方案.xlsx    競叫.xlsx
└── 網站/
    ├── index.html
    ├── assets/style.css, site.js
    └── 叫序資料/一梅花開叫.html … 競叫.html
```

沒有「原始書籍檔案」：這個專案只有叫序表，沒有逐章還原的 Markdown。

## 3. 與 bridge-relay-bidding 的 build.py 差異

| 項目 | bridge-relay-bidding | 本專案 |
|---|---|---|
| 頁面清單 | `NAV_ITEMS` 與 `main()` 各寫一次 | 只寫在 `PAGES`（xlsx、標題、首頁角標、簡介），導覽列、首頁卡片、`main()` 都由它產生 |
| 首頁 | 手寫卡片 HTML | `build_index()` 依 `PAGES` 產生，工作表數自動計算 |
| 書籍章節 | `BOOK_DOCS` + markdown | 移除 |
| 接力工具連結 | `linkify_tools()` 連到速查頁 | 改成 `linkify_pages()`：「見接力方案檔」「見競叫檔」等文字連到對應頁面（`PAGE_REF_ALIASES`） |
| 表號引用 | `REF_RE` 認「表N」 | 加上 `(?![\dA-Za-z])`：原書表號「表 6C」「表 9A」不會被誤當成第 6、9 分頁 |
| 註記列 | 只有表尾的單欄列才收成註腳 | 表中段「＊」開頭、只有第一欄的列畫成 `tr.note-row`（style.css 有對應樣式） |
| 巢狀接力 | 只看第一欄的 `▸`／全形空白 | 第一欄是「序列」時，改看第二欄（叫品欄）；沒有子列的 `▸` 也會去掉標記 |

## 4. xlsx 的格式約定（給 build.py 解析用）

- **表頭列**：粗體 + 深藍底 `1F3864`（與 bridge-relay-bidding 的 `HEADER_NAVY` 相同）。
  表頭之前的列 = 標題與情境說明（前導區塊）。
- **段落小標**：同樣是粗體 + `1F3864`，會變成表中的 `subhead` 列。
- **「說明」分頁**：表頭為「項目／內容」，走 `render_info()`。
- **巢狀接力**：上層列以 `▸ ` 開頭；子接力列以全形空白 `　` 開頭。
- **分頁跳轉**：寫「見表 3」「見表 6、7」；分頁標題要以「數字.」開頭（如 `3.1C-1D-1S`）。
- **註記**：「＊」開頭、只寫第一欄（可合併儲存格）。在表尾 = 頁尾附註；在表中 = 註記列。

在 Excel 改內容時，**不要改表頭與段落小標的底色**，否則 `analyze_sheet()`
找不到表頭，整頁會跑版（同 bridge-relay-bidding 第 4.1 節的警告）。

## 5. 維護 SOP

```bash
# 1. 改 原始資料/*.xlsx
# 2. 重新產生網站
python3 build.py
# 3. 用瀏覽器打開 網站/index.html 目測
# 4. commit + push（GitHub Actions 自動部署）
```

新增一份叫序表：把 xlsx 放進 `原始資料/`，在 `build.py` 的 `PAGES` 加一筆即可。

首次部署：在 GitHub 建 public repo、push 後到 Settings → Pages 把 Source 設成
「GitHub Actions」（或 `gh api -X POST repos/<owner>/<repo>/pages -f build_type=workflow`）。
推送 workflow 檔需要 token 有 `workflow` scope。
