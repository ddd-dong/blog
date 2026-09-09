# Terminal 主題規範（給 Claude Code 的任務說明）

目標：把 `ddd-dong/blog`（Hugo + PaperMod + GitHub Pages，baseURL 含 `/blog/`）整站換成「終端機 / 作戰指揮介面」風格。首頁已完成（`layouts/index.html` + `static/css/terminal.css`），其餘頁面要跟首頁一致。

## 0. 硬性限制

- 不動 `themes/PaperMod/` 任何檔案，只在專案根目錄 `layouts/`、`assets/`、`static/` 覆寫。
- 所有內部連結用 `relURL` / `.RelPermalink`，不要手寫 `/blog/`。
- 不放任何遊戲的商標、名稱、角色、圖片；只借用風格。
- 不引入 build step 以外的東西（不要 npm、Tailwind、framework）。純 Hugo 模板 + CSS + 少量原生 JS。
- 亮暗切換用 localStorage key `pref-theme`，值 `light` / `dark`，`<html data-theme="...">`，全站共用。
- `prefers-reduced-motion: reduce` 時關掉所有非使用者觸發的動畫。
- 每次改完 `hugo --gc --minify` 要能無 warning 通過；用 `hugo server` 目視檢查桌機（1280）與手機（390）。
- 完成後 `git commit` 分成合理的小步（tokens → base layout → list → single → 其他），不要一坨。

## 1. 設計 tokens（唯一真相在 `static/css/terminal.css` 的 `:root` / `[data-theme="light"]`）

| token | dark | light | 用途 |
|---|---|---|---|
| `--bg` | `#101214` | `#eef0f2` | 頁面底 |
| `--bg2` | `#181b1f` | `#e3e6e9` | hover / 次級底 |
| `--panel` | `#1f2327` | `#f7f8f9` | 面板底 |
| `--line` | `#3a4148` | `#c9cfd5` | 一般框線、分隔 |
| `--line-strong` | `#8a939c` | `#6b747c` | 切角描邊、次要文字 |
| `--fg` | `#e6e9ec` | `#15181b` | 主文字 |
| `--fg-dim` | `#9aa3ac` | `#5c656e` | 次文字 |
| `--accent` | `#f4a21b` | `#e08a00` | 唯一強調色：選中條、進度、斜紋、連結 hover |
| `--grid` | `rgba(255,255,255,.035)` | `rgba(0,0,0,.05)` | 背景 64px 格線 |
| `--wm` | `rgba(255,255,255,.07)` | `rgba(0,0,0,.08)` | 浮水印描邊 |

規則：
- 顏色只准用 token，禁止在其他 CSS 硬編 hex。要新色先加 token。
- 強調色只有一個。不要再加第二個顏色。
- 字體：標題 `Barlow Condensed`（500/700/800），內文 `Barlow`（400/500/600），中文 fallback `Noto Sans TC`。程式碼 `ui-monospace, "JetBrains Mono", monospace`。
- 內文最大寬度 `68ch`，行高 1.7；標題 letter-spacing 正值（`.02em` 到 `.2em`），內文不要 letter-spacing。

## 2. 共用元件（已在 `terminal.css`，其他頁直接用）

- `.cut`：切角面板。`clip-path` 左上 / 右下各切 `--cut`（14px），加 `::after` 內描邊、`::before` 角落短線。所有卡片、側欄面板、文章 meta 區都用它。
- `.hazard`：6px 橘色斜紋條。只用在「區塊分界」，一頁最多 2 條。
- `.tgl`：亮暗切換按鈕（小切角、hover 反白）。
- `.tags span`：小切角標籤，用於 tags / categories。
- `header`：56px sticky 頂列，左品牌、中導覽、右時鐘 + 切換鈕。
- `aside`：200px 左側模組列表，選中項左邊 3px 橘條。手機時變成頂部橫向 chips。
- `.wm`：右下描邊大字浮水印（首頁是 `DDD`，其他頁見下表）。
- `#boot`：載入畫面。**只在首頁**，其他頁不要。

需要新增的元件（寫進 `terminal.css` 底部，維持同一套 token）：
- `.post-list li`：一行一篇，`grid: date | title | tags`，hover 整行底色 `--bg2`，左邊出現 3px 橘條（跟側欄一致）。
- `.meta`：文章 meta 條，Barlow Condensed 13px letter-spacing `.16em`，欄位用 ` / ` 分隔，不要用 `·`。
- `.prose`：文章內文容器（見第 4 節）。
- `.toc`：目錄面板，`.cut` 包住，sticky 在右欄。
- `.pager`：上一篇 / 下一篇，兩個 `.cut` 並排。

## 3. 檔案結構（要產出的覆寫）

```
layouts/
  _default/
    baseof.html        # 自己的骨架：head、header、aside、main、footer。不要 include PaperMod 的 head/header partial
    list.html          # posts/、tags/xxx/、categories/xxx/ 共用
    single.html        # 文章頁
    terms.html         # tags/ 總表
  partials/
    t-head.html        # meta、字體、terminal.css、theme 初始化 script（讀 localStorage，避免閃爍，放在 head 最前）
    t-header.html      # 頂列（從 index.html 抽出來共用，index.html 也改成 include）
    t-aside.html       # 側欄；用 .Section / .IsHome 判斷哪項加 .on
    t-footer.html      # 一行：BUILD {{ now.Format "2006.01" }} / HUGO {{ hugo.Version }} / 版權
  index.html           # 現有首頁，改為 include 上述 partial
  404.html             # 見第 5 節
static/css/terminal.css
```

側欄「選中」對照：

| 路徑 | aside .on | 浮水印字 |
|---|---|---|
| `/` | HOME | DDD |
| `/posts/`, `/posts/xxx/` | WRITING | LOG |
| `/tags/research/` 與 tag 為 research 的文章 | RESEARCH | RES |
| `/talks/` | TALKS | TLK |
| `/about/` | CONTACT | ID |
| 其他 tags / categories | WRITING | TAG |

## 4. 各頁規格

### 4.1 baseof.html
- `<html lang="zh-Hant" data-theme="dark">`，head 最前放 inline script：`document.documentElement.dataset.theme = localStorage.getItem('pref-theme') || (matchMedia('(prefers-color-scheme: light)').matches ? 'light' : 'dark')`，避免首屏閃色。
- 結構：`header` → `main`（`aside` + `section.stage`）→ `footer`。跟 index.html 的 grid 一致（`200px 1fr`，900px 以下單欄）。
- `section.stage` 內 `{{ block "main" . }}{{ end }}`。

### 4.2 list.html（文章列表）
- 頂部：`h1` 用 Barlow Condensed 800，大寫，內容 = section / term 名稱；下方一行 `.meta`：`{{ len .Pages }} ENTRIES / SORTED BY DATE`。
- 一條 `.hazard`。
- `.post-list`：`<ol>`，每項 `date | title | tags`，date 用 `2006-01-02` tabular-nums，title 是連結，tags 最多顯示 3 個 `.tags span`。
- 有 `.Summary` 時在 title 下方顯示一行，`--fg-dim`，超過兩行截斷。
- 分頁：`{{ template "_internal/pagination.html" . }}` 不要用，自己寫 `.pager`：`PREV ◂ / PAGE 02 ▸ NEXT`。
- 空列表：顯示 `NO ENTRIES` 加一句「還沒有文章」，不要空白。

### 4.3 single.html（文章頁）
- 兩欄：左內文（最大 `68ch`）、右 `.toc`（`{{ .TableOfContents }}`，只在 `.Params.toc != false` 且標題數 ≥ 3 時顯示；1100px 以下收到內文上方）。
- 標題區：`h1`（Barlow Condensed 800，clamp 36–64px），下方 `.meta`：`{{ .Date.Format "2006-01-02" }} / {{ .ReadingTime }} MIN / {{ .WordCount }} 字`，tags 用 `.tags`。
- 內文 `.prose` 樣式：
  - `h2` 前面加 `--accent` 短橫（`::before`，24px x 2px），`h3` 用 `--line-strong` 左邊 2px。
  - `a` 底線 1px `--line-strong`，hover 換 `--accent`。
  - `blockquote` 用 `.cut` 包，左邊 3px `--accent`。
  - `pre` 用 `.cut`，背景 `--bg`，字 13px；`code` 行內用 `--bg2` 底。
  - `table` 全寬，表頭 Barlow Condensed letter-spacing `.12em`，列分隔 `--line` 虛線。
  - `img` 全寬，`.cut` 切角。
- 底部：`.pager`（`.PrevInSection` / `.NextInSection`），沒有的一側顯示 `END OF LOG`。
- 保留 PaperMod 原本的 `hugo.yaml` params 相容：`ShowReadingTime`、`ShowToc`、`ShowBreadCrumbs` 若為 false 要尊重。
- Shortcodes 不要動，PaperMod 內建的照常運作。

### 4.4 terms.html（tags 總表）
- `h1` = TAGS，下方用 `.tags` 排所有 term，每個標籤後面小字顯示數量 `(12)`。

### 4.5 talks / about
- `content/talks/_index.md`、`content/about.md` 若不存在就建立最小內容（front matter + 一句 placeholder），用 `single.html` 渲染即可，不用另做模板。

### 4.6 404.html
- 沿用 baseof，中央一個 `.cut` 面板：`ERR 404 / TARGET NOT FOUND`，下方一個 `.tgl` 樣式的連結回首頁。浮水印 `404`。

## 5. 動態規則

- 允許：頁面載入時 **首頁** boot 條；hover / focus 顏色變化（`.15s`）；亮暗切換 `.25s`。
- 禁止：捲動觸發的 fade-in、卡片 hover 位移、視差、打字機效果、閃爍游標。
- 時鐘只在 header，`Asia/Taipei`，`en-GB` 24 小時。

## 6. 無障礙與品質底線

- 所有互動元素要有可見 `:focus-visible`（1px `--accent` outline，offset 2px）。
- 文字對比：`--fg-dim` 在兩個主題下對 `--panel` 都要 ≥ 4.5:1，改色前檢查。
- 圖片要 `alt`；裝飾性元素（浮水印、斜紋、boot）加 `aria-hidden="true"`。
- 不使用 `<div>` 當按鈕；切換鈕是 `<button>`。

## 7. 完成定義

- [ ] `hugo --gc --minify` 無 warning
- [ ] 首頁、`/posts/`、任一文章、`/tags/`、任一 tag 頁、`/404.html` 在 1280 / 390 寬各截一張圖確認
- [ ] 亮暗切換在首頁按下後，進文章頁維持
- [ ] 沒有任何 hex 顏色出現在 `terminal.css` 的 token 區塊以外（`grep -nE "#[0-9a-fA-F]{3,6}" static/css/terminal.css` 只命中 `:root` 與 `[data-theme="light"]`）
- [ ] `git log` 是分步 commit，訊息用英文祈使句（`Add terminal baseof layout`）
