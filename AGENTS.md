# AGENTS.md

給 AI coding agent（Claude Code、Codex、Gemini CLI 等）看的規則。動任何東西之前先讀完。

## 1. 這個專案是什麼

使用者的個人網站跟 blog，主要用來記錄讀過的 AI Safety 論文，用中文寫給其他人看。

- 網站：https://ddd-dong.github.io/blog/
- 技術：Hugo extended 0.155.2，加上自己寫的 terminal 風格模板，沒有用外部 theme
- 發文：使用者用 Decap CMS 後台（`/blog/admin/`）上架，CMS 會直接 commit 到 `main`
- 部署：`main` 有新 commit 時，GitHub Actions（`.github/workflows/hugo.yaml`）會自動 build 並發布到 GitHub Pages
- 讀者：懂基本 LLM 概念、但不一定做 security 的資工學生

這個 blog 最在意兩件事，下面所有規則都是為了它們：

1. **內容正確**：論文的數字、方法、結論都要對得到原文
2. **讀起來是使用者自己寫的**：觀點只能來自使用者，AI 不能代替他發言

## 2. 權限

- **不要 commit、不要 push、不要改 repo 裡的檔案**，除非使用者在這次對話明確說要
- 文章預設由使用者自己從 CMS 上架。AI 的工作是產出可以直接貼進 CMS 的內容（第 4 節）
- 使用者要求改模板或樣式時，只在本地改，改完給他看，由他決定要不要放

## 3. 檔案結構

```
AGENTS.md                     這份規則
CLAUDE.md                     只有一行 @AGENTS.md，讓 Claude Code 讀這份規則
TERMINAL_THEME_SPEC.md        視覺風格的完整規格（改樣式前要讀）
hugo.yaml                     站台設定
archetypes/default.md         hugo new 的預設範本
content/
  posts/                      文章，一篇一個 .md，由 CMS 建立
  talks/                      演講，一場一個 .md，加上 _index.md
  about.md                    關於我
data/publications.yaml        首頁 RECENT PUBLICATIONS 的資料
layouts/
  _default/baseof.html        全站骨架
  _default/list.html          文章列表
  _default/single.html        文章頁
  _default/terms.html         tags 總表
  _default/_markup/           圖片 render hook
  partials/t-*.html           header、側欄、footer、head
  index.html                  首頁
  404.html                    404 頁
static/css/terminal.css       全站唯一的 CSS
static/img/                   文章圖片（CMS 上傳到這裡）
static/admin/                 Decap CMS 設定
static/cv/                    CV PDF
```

不要動：`static/admin/`、`.github/workflows/`、`static/cv/`。`public/` 跟 `resources/` 是 build 產物，已經在 `.gitignore`。

## 4. 新增一篇論文筆記

使用者通常會說「幫我寫一篇 XXX 論文的筆記」，並附上他自己的筆記。照這個順序做：

1. **先拿到使用者的筆記**。沒有就停下來跟他要，不要自己開始寫。需要的內容見第 5 節
2. 讀論文原文（arXiv 或正式發表頁面）。只能用這篇論文本身，不要拿其他文章、新聞、blog 來補
3. 照第 6、7、9 節寫內文
4. 照第 10 節檢查一遍
5. 在回覆裡交出 CMS 的四個欄位，加上核對清單（第 8 節）

### 交出去的格式

CMS 的欄位只有這四個，照這個順序給：

| CMS 欄位 | 規則 |
| --- | --- |
| Title | 論文簡稱，例如 `TIP of the Iceberg` |
| Publish Date | 請使用者按「現在」，不用給值 |
| Description | 列表頁的摘要，60 字以內，書面語，例如 `Task-in-Prompt 攻擊：將敏感字編碼後要求模型解題，藉此繞過安全訓練與防護模型的偵測` |
| Body | 完整 Markdown，放在 code block 裡方便複製。外層用四個反引號，因為內文本身可能有 code block |

CMS 會自己產生檔名 `content/posts/YYYY-MM-DD-<slug>.md` 跟 front matter，AI 不用處理。CMS 沒有 tags 欄位，文章不放 tags。

如果使用者要 AI 直接寫成檔案，格式照 CMS 產出的樣子：

```yaml
---
title: TIP of the Iceberg
date: 2026-11-16T20:35:00
description: Task-in-Prompt 攻擊：將敏感字編碼後要求模型解題，藉此繞過安全訓練與防護模型的偵測
---
```

`date` 不能晚於現在，Hugo 不會 build 未來時間的文章。

## 5. 使用者要提供什麼

AI 查得到論文內容，但查不到使用者怎麼看。開始寫之前要有使用者的筆記，條列、口語就好：

```
- 為什麼讀：在找 jailbreak 相關的 baseline，看到標題雙關覺得好笑就點進去
- 跟我的關係：我在做 agent 安全，agent 讀到的 tool 回傳值如果是編碼過的，會不會一樣被「解題」騙過去
- 認同：ArtPrompt 會成功不是因為 ASCII art 本身，而是「解碼任務」這件事
- 懷疑：judge 是 LLaMA-3.1-70B，它自己也是被測模型之一，有點球員兼裁判
- 還沒想通：如果把輸出端也加 classifier，TIP 還剩多少成功率？論文沒做
```

缺哪一條，對應的內容就少寫，不要自己補。

## 6. 論文筆記的固定格式

Body 的結構固定如下，段落名稱跟順序不能改：

**開頭：論文資訊**（三行 blockquote）

```
> **論文完整英文標題**
> 作者 ・ 發表處 ・ [arXiv 編號](https://arxiv.org/abs/編號)
> 一句話：30 字以內的重點
```

作者、發表處、arXiv 編號照論文首頁或正式發表頁面抄，查不到就寫 `（原文未提供）`，不能猜。

**內文段落**

| 段落 | 篇幅 | 內容 |
| --- | --- | --- |
| `## 這篇在做什麼` | 150 到 250 字 | 論文的問題、提出什麼、測了什麼 |
| `## 攻擊怎麼運作` | 200 到 300 字 | 機制，**一定要附一個無害示意**。不是攻擊論文就改成 `## 方法怎麼運作` |
| `## 實驗結果` | 150 到 250 字 | 最多 4 個數字，每個都標出處，例如（Table 2） |
| `## 我怎麼看` | 150 到 250 字 | 只能改寫使用者的筆記 |
| `## 還沒想通的地方` | 1 到 3 點條列 | 筆記裡有「還沒想通」才寫這節，沒有就整節省略 |

- 內文段落合計 800 到 1200 字（中文一字算 1，英文單字或數字一個算 1，標點跟 code block 不算）
- 不寫 H1，標題由 Title 欄位產生；段落裡要再分可以用 H3
- 圖片用 CMS 上傳，路徑會是 `/blog/img/檔名`

## 7. 硬性規則

違反任何一條都要重寫。

**正確性**

1. 論文的方法、數字、模型、資料集、結論只能來自論文原文。原文沒寫的不能寫成論文說的
2. 不能改變原文的確定程度。論文說「可能」就不能寫成「證明了」
3. 數字必須跟原文完全一致，並標出處（Table / Figure / Section）
4. 論文作者自己的主張，跟論文引述別人的主張要分開寫，引述別人時寫出是誰

**不替使用者發言**

5. 「我怎麼看」跟「還沒想通的地方」的每一句都要對得到使用者筆記裡的某一條。只能調整語序跟用詞，不能延伸出筆記沒寫的結論
6. 每一條筆記只用一次，不要把「懷疑」改寫成問題拿去塞「還沒想通」
7. 前三節不能出現 AI 自己的評價、推論或比較（例如「這顯示了」「所以應該」）。想補充的東西放核對清單

**安全**

8. 不寫任何可以直接拿去用的有害 prompt。攻擊示意只能用無害的內容，例如把 `HELLO` 用 Caesar 位移 3 變成 `KHOOR`
9. 論文的攻擊目標只寫類別名稱，不寫具體內容

## 8. 核對清單（給使用者，不發布）

交內容時在回覆裡附一張表，使用者會照著對原文：

| 正文中的說法 | 論文出處 | 類型 | 需要確認嗎 |
| --- | --- | --- | --- |
| GPT-4o plain text ASR 0.03、TIP Python 0.86 | Table 2 | 論文明寫 | 是 |
| judge 是被測模型之一 | Section 4 | 我的筆記 | 否 |

- 類型只有三種：`論文明寫`、`我的筆記`、`AI 推論`
- 正文裡每個數字、每個「論文指出」都要列
- 一列只能有一種類型；同一句同時有論文事實跟使用者的評價就拆成兩列
- 最多 15 列，同一出處可以合併

## 9. 寫作風格

- 繁體中文（台灣用語），**不能有簡體字**。專有名詞保留英文，英文跟中文之間空一格
- 論文介紹的段落（前三節、Description、一句話）用書面語，平實、直接、句子短
- **不要過度口語**，例如「擋不太住」「這招」「都行」「一筆一筆看」「就順便越獄了」
- 「我怎麼看」可以保留使用者筆記原本的語氣
- 不浮誇，不要開場寒暄，不要結尾總結段
- 不用破折號（—）、箭頭（→）、emoji
- 禁用：「值得注意的是」「總而言之」「綜上所述」「結語」「啟示」「提醒我們」「不僅…更…」「讓我們」「深入探討」「希望對你有幫助」「歡迎指正」
- 少用粗體跟表格，一篇最多一張表

## 10. 交內容之前的檢查

- [ ] 拿到使用者的筆記才開始寫
- [ ] 段落名稱、順序、字數符合第 6 節
- [ ] 每個數字都對過原文，出處標對，實驗結果不超過 4 個數字
- [ ] 「我怎麼看」每一句都對得到筆記
- [ ] 沒有禁用詞、破折號、箭頭、emoji、簡體字、過度口語
- [ ] 攻擊示意是無害的
- [ ] 四個 CMS 欄位跟核對清單都附在回覆裡

## 11. CMS 已知問題

- 貼 Body 之前要先按編輯器右上角的 M↓ 切到 Markdown 模式，不然 code block 跟清單會跑掉
- CMS 有時會把連結的 `]` 跳脫成 `\]`，連結就會壞掉。上架後請使用者確認論文資訊那行的 arXiv 連結點得開
- CMS 會在清單項目之間、blockquote 每行之間插空行，顯示上沒有影響，不用修

## 12. 改樣式或模板

只在使用者要求時做，而且只在本地改（第 2 節）。完整規格在 `TERMINAL_THEME_SPEC.md`，重點：

- 顏色只能用 `terminal.css` 裡 `:root` / `[data-theme="light"]` 的 token，不能在別處寫死 hex
- 強調色只有 `--accent`（橘色）一個
- `.cut` 切角面板、`.hazard` 斜紋條（一頁最多 2 條）、`.tags` 這些現有元件優先重用
- 模板裡的內部連結一律用 `relURL` / `.RelPermalink`，不要寫死 `/blog/`
- 不用 npm、Tailwind、任何 framework。純 Hugo 模板加 CSS，少量原生 JS
- 不能有捲動動畫、hover 位移、打字機效果；`prefers-reduced-motion` 要關掉動畫
- 桌機 1280 跟手機 390 寬、亮暗兩個主題都要看過
- `TERMINAL_THEME_SPEC.md` 提到的 PaperMod 已經拿掉了，相關段落可以忽略
- 改完跑 `hugo --gc --minify`，要零 warning

## 13. 其他頁面

- **演講**：`content/talks/<YYYY>-<活動名>.md`，一場一個檔案。front matter 有 `title`（議題英文原名）、`date`、`description`、`tags`（英文小寫）。內文分 `## 內容` 跟 `## 連結`，共同講者要寫出來並附連結
- **發表論文**：`data/publications.yaml`，新的放最上面，首頁只顯示前 4 筆。欄位：`title`、`authors`、`venue`、`date`（`YYYY-MM`）、`url`。只放使用者確認過的發表
- **關於我**：`content/about.md`，信箱維持 `[at]` 寫法

## 14. 指令

```
hugo server -D           本機預覽（含草稿），http://localhost:1313/blog/
hugo --gc --minify       正式 build，要零 warning
```
