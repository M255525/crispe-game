# CLAUDE.md — crispe-game（CRISPE 卡牌配對練習遊戲）

依 CRISPE 提示詞框架設計的拖放卡牌配對遊戲，作為 AI 課程的課堂練習工具。規格來源：`C:\Users\mark_\SynologyDrive\工具\AI相關\canva\設計 CRISPE 角色扮演遊戲的寫作輔助工具.docx`。單檔前端，無建置步驟、無框架、零外部資源，直接開啟 `index.html`（`file://`）或以靜態伺服器託管即可。與 `Prompt/`（提示詞控制台）是姊妹專案——那邊是「產生」提示詞，這邊是「練習」分類。

## 遊戲規則（依 docx 規格）

- **本專案採 5 格 CRISPE**（刻意與 `Prompt/` 的 6 維度不同）：CR＝角色與能力（Capacity/Role 合為一格）、I＝背景洞察（Insight）、S＝具體任務、P＝語氣與風格、E＝限制與條件。docx 原文把首兩格標成 C 與 R，經使用者確認改為標準拆法 CR＋I。
- 每局從題庫隨機抽 5 個主題，每主題 5 張卡（共 25 張），依主題分 5 色。玩家把卡拖入正確提示格，每張 4 分、滿分 100。
- 放滿 5 格才能按「✔ 確認」判分；確認後格子亮綠（✓）／紅（✗）並自動彈出答案核對頁；「↻ 重新開始」＝重進當前主題（卡牌與本題計時重置）。主題可任選、可重挑戰，分數以最近一次確認為準。
- 全域三鍵：「▶ 開始」（開始／恢復計時，未開始前不能進主題）、「⏸ 暫停」（凍結計時＋覆蓋層鎖卡）、「🏁 完成」（停表進結算：各主題分數／用時＋總分＋總時間）。
- 五主題全數確認且總分 100 → 恭賀畫面＋彩帶＋完成音樂（`congratsShown` 旗標避免重複觸發）。

## 架構（單一 index.html）

- **資料**：`QUESTION_BANK`（13 組主題，每組 `{id, title, desc, cards:{CR,I,S,P,E}}`）、`SLOTS`（五格引導語，文字照 docx）、`THEME_COLORS`（5 色，依抽出順序指派）。題庫內容**全部虛構**、貼近台灣教學／中小企業情境；卡牌文字刻意不含「角色」「任務」等關鍵字提示，玩家須理解語意才能分類——**新增題目時維持此原則**。
- **狀態**：全域 `S = {drawn, results, totalMs, started, finished, congratsShown}`，存 localStorage（key: `crispeGameState`）；重新整理後回「待命」狀態（`running=false`），按「開始」續走，總時間與各主題成績保留。版面進行中的卡牌位置**不**持久化（重整回選單）。
- **計時**：200ms tick 累加 `S.totalMs` 與 `attemptMs`（本題時間；確認後 `boardLocked` 即停）。
- **拖曳**：Pointer Events 自製（`pointerdown` 在卡上、`pointermove/up` 掛 document），拖曳中卡牌 `position:fixed` ＋ `pointer-events:none`，用 `elementFromPoint().closest('.slot')` 判落點；格已有卡則舊卡回手牌。卡牌 `touch-action:none` 支援觸控。判分依據 `card.dataset.slot === slot.dataset.key`（答案在 DOM 可見，課堂用途可接受）。
- **音效**：WebAudio 合成（`sfx.place/good/bad/perfect/fanfare`），無音檔；首次點「開始」時 `resume()` 解鎖 AudioContext。
- **視覺**：牌桌主題——絨布綠桌面（radial 漸層）＋木紋頂欄＋奶油色紙牌（主題色上邊條）＋虛線牌位格＋計分板式 Consolas 計時晶片。恭賀彩帶為 JS 動態產生的 `.confetti-piece`。有 `prefers-reduced-motion` 減敏處理。

## 上線與跑馬燈／使用警語（2026-08-20）

已推公開 GitHub repo：<https://github.com/M255525/crispe-game>，並啟用 GitHub Pages（legacy branch-source，`master` 分支根目錄，比照 `fruit-ninja-cam` 的實際設定而非「一律用 Actions workflow」——因為本專案根目錄本來就是 `index.html`，legacy 模式直接可用）：<https://m255525.github.io/crispe-game/>。

**頂部跑馬燈**：`#marqueeBar` 抓取工作區共用的 Google Sheet 公告內容（同一個授權伺服器 Apps Script 網址，沒有序號登入機制所以直接帶空序號打，只取 `marquee` 欄位），做法與 `Rummikub`／`fruit-ninja-cam` 一致，`localStorage` key 為 `crispeGameMarquee`。**版面整合方式依本專案既有版面模型客製**（跟 `shared-widget-rollout` skill 的判斷表對應）：`.topbar` 是 `position:sticky;top:0`，所以顯示跑馬燈時要同時做兩件事——`body.has-marquee{padding-top:26px}`（把初始文件流往下推，否則 topbar 初始位置會被固定的跑馬燈蓋住）＋`body.has-marquee .topbar{top:26px}`（sticky 吸頂偏移量也加 26px，捲動後 topbar 才會吸在跑馬燈下方而不是貼齊視窗頂端）。已用 Playwright `browser_evaluate` 驗證初始渲染與捲動後兩種狀態皆無疊圖。

**使用警語＋創作者資訊**：`.footnote`（「⚠ 本工具僅供個人娛樂與教學示範使用，禁止未經授權公開發布、販售或商業化使用。」＋「創作者：蔡豐全（Mark Tsai）」，文字與其餘工具逐字相同）放在 `#menuView` 的 `.menu-hint` 下方——這個遊戲沒有獨立「開始畫面」，`#menuView`（主題選單）是唯一常駐、不需要進入挑戰就看得到的畫面，所以放這裡，比照 `fruit-ninja-cam` 的判斷邏輯；不放進 `#boardView`（挑戰進行中畫面已經很滿，加常駐文字會擠壓卡牌區）。

**改完 index.html 記得重新產生 Google Sites 嵌入版**（見下方指令）——這次跑馬燈與 footnote 的改動已同步重新產生過。

**操作手冊 `manual.html`**（2026-08-20 新增）：獨立頁面，內含玩法說明、CRISPE 五格對照、常見狀況、創作者資料（與 `Prompt`／`SN-maker` 等工具逐字相同的完整版本——姓名、信箱、專長、證照、經歷）、授權限制；`.topbar` 的 `.controls` 區塊新增「📖 操作手冊」連結（`target="_blank"`，避免跳走弄丟進行中的計時／拖曳狀態），CSS 新增 `a.ctl-btn` 補上 `<a>` 標籤預設沒有的 `display:inline-flex`／`text-decoration:none`／`cursor:pointer`（`.ctl-btn` 原本只針對 `<button>` 設計）。

## Google Sites 嵌入版（platform/）

`platform/CRISPE卡牌配對-GoogleSites嵌入用.html` 是供 Google 協作平台「插入 → 嵌入 → 嵌入程式碼」貼上的變體（做法比照 `Rummikub/`）：即 index.html 去掉 `<!DOCTYPE>`／`<html>`／`<head>`／`<body>` 外殼、只留 `<meta charset>`＋`<style>`＋內容＋`<script>` 的片段；同資料夾的 `一鍵複製-貼到GoogleSites.bat` 會把嵌入碼複製到剪貼簿（嵌入框建議拉高至少 900px）。**修改 index.html 後必須重新產生嵌入版**（於本專案根目錄執行）：

```bash
python -c "import re,io;src=io.open('index.html',encoding='utf-8').read();style=re.search(r'<style>.*?</style>',src,re.S).group(0);body=re.search(r'<body>\n(.*)\n</body>',src,re.S).group(1);io.open('platform/CRISPE卡牌配對-GoogleSites嵌入用.html','w',encoding='utf-8').write('<meta charset=\"UTF-8\">\n'+style+'\n\n'+body+'\n')"
```

嵌入片段開頭的 `<meta charset="UTF-8">` 是刻意加的：片段被獨立開啟（本機測試）時瀏覽器編碼嗅探會失敗導致 JS 語法錯誤，加了才能雙擊直測；貼進 Google Sites 亦無害。Sites 的沙箱 iframe 可能禁 localStorage——程式內 save/load 已包 try/catch，屆時只是不記分數、遊戲照玩。

## 指令

無建置／測試指令。修改後直接開瀏覽器驗證，或 `python -m http.server <port> --directory crispe-game` 暫起伺服器測完關閉（工作區埠號 8765–8777 已被其他專案佔用，測試時用 8779 之類）。自動化驗證以 Playwright `browser_evaluate` 派發 PointerEvent 模擬拖曳最可靠（本工作區截圖偶發逾時）；注意 MCP 瀏覽器視窗可見，真人同時操作會干擾跨呼叫的狀態斷言——關鍵流程放在單一 evaluate 內完成。
