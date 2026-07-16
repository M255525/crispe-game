# CLAUDE.md — crispe-game（CRISPE 卡牌配對練習遊戲）

依 CRISPE 提示詞框架設計的拖放卡牌配對遊戲，作為 AI 課程的課堂練習工具。規格來源：`C:\Users\mark_\SynologyDrive\工具\AI相關\canva\設計 CRISPE 角色扮演遊戲的寫作輔助工具.docx`。單檔前端，無建置步驟、無框架、零外部資源，直接開啟 `index.html`（`file://`）或以靜態伺服器託管即可。與 `Prompt/`（提示詞控制台）是姊妹專案——那邊是「產生」提示詞，這邊是「練習」分類。

## 遊戲規則（依 docx 規格）

- **本專案採 5 格 CRISPE**（刻意與 `Prompt/` 的 6 維度不同，照 docx）：C＝角色與能力、R＝背景洞察（Insight）、S＝具體任務、P＝語氣與風格、E＝限制與條件。
- 每局從題庫隨機抽 5 個主題，每主題 5 張卡（共 25 張），依主題分 5 色。玩家把卡拖入正確提示格，每張 4 分、滿分 100。
- 放滿 5 格才能按「✔ 確認」判分；確認後格子亮綠（✓）／紅（✗）並自動彈出答案核對頁；「↻ 重新開始」＝重進當前主題（卡牌與本題計時重置）。主題可任選、可重挑戰，分數以最近一次確認為準。
- 全域三鍵：「▶ 開始」（開始／恢復計時，未開始前不能進主題）、「⏸ 暫停」（凍結計時＋覆蓋層鎖卡）、「🏁 完成」（停表進結算：各主題分數／用時＋總分＋總時間）。
- 五主題全數確認且總分 100 → 恭賀畫面＋彩帶＋完成音樂（`congratsShown` 旗標避免重複觸發）。

## 架構（單一 index.html）

- **資料**：`QUESTION_BANK`（13 組主題，每組 `{id, title, desc, cards:{C,R,S,P,E}}`）、`SLOTS`（五格引導語，文字照 docx）、`THEME_COLORS`（5 色，依抽出順序指派）。題庫內容**全部虛構**、貼近台灣教學／中小企業情境；卡牌文字刻意不含「角色」「任務」等關鍵字提示，玩家須理解語意才能分類——**新增題目時維持此原則**。
- **狀態**：全域 `S = {drawn, results, totalMs, started, finished, congratsShown}`，存 localStorage（key: `crispeGameState`）；重新整理後回「待命」狀態（`running=false`），按「開始」續走，總時間與各主題成績保留。版面進行中的卡牌位置**不**持久化（重整回選單）。
- **計時**：200ms tick 累加 `S.totalMs` 與 `attemptMs`（本題時間；確認後 `boardLocked` 即停）。
- **拖曳**：Pointer Events 自製（`pointerdown` 在卡上、`pointermove/up` 掛 document），拖曳中卡牌 `position:fixed` ＋ `pointer-events:none`，用 `elementFromPoint().closest('.slot')` 判落點；格已有卡則舊卡回手牌。卡牌 `touch-action:none` 支援觸控。判分依據 `card.dataset.slot === slot.dataset.key`（答案在 DOM 可見，課堂用途可接受）。
- **音效**：WebAudio 合成（`sfx.place/good/bad/perfect/fanfare`），無音檔；首次點「開始」時 `resume()` 解鎖 AudioContext。
- **視覺**：牌桌主題——絨布綠桌面（radial 漸層）＋木紋頂欄＋奶油色紙牌（主題色上邊條）＋虛線牌位格＋計分板式 Consolas 計時晶片。恭賀彩帶為 JS 動態產生的 `.confetti-piece`。有 `prefers-reduced-motion` 減敏處理。

## 指令

無建置／測試指令。修改後直接開瀏覽器驗證，或 `python -m http.server <port> --directory crispe-game` 暫起伺服器測完關閉（工作區埠號 8765–8777 已被其他專案佔用，測試時用 8779 之類）。自動化驗證以 Playwright `browser_evaluate` 派發 PointerEvent 模擬拖曳最可靠（本工作區截圖偶發逾時）；注意 MCP 瀏覽器視窗可見，真人同時操作會干擾跨呼叫的狀態斷言——關鍵流程放在單一 evaluate 內完成。
