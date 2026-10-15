# 原寶：政大原住民族學生獎助學金小幫手（LLM Wiki）

政大原住民族學生的資訊百科，依 Karpathy 的 LLM Wiki 模式整理，部署在 Vercel：靜態網頁 + 一個 AI 問答函式。目前收錄原民會大專校院獎助學金的資料。

## 資料夾內容
```
index.html        網站本體（讀取 wiki/ 的 Markdown 並顯示、搜尋、AI 問答）
api/ask.js        AI 問答函式（Vercel Function，保管金鑰、呼叫 Agnes AI）
vercel.json       Vercel 設定（函式時間上限、讓函式讀得到 wiki/）
AGENTS.md         給 LLM 的維護規則（ingest / query / lint）
raw/              原始 PDF（唯讀，網站上可直接下載）
wiki/             LLM 維護的條目（Markdown）
  pages.json      條目清單，新增條目要加進來
assets/           圖片與 marked.min.js
```

## 部署（第一次，約 10 分鐘）

### 1. 把檔案放上 GitHub
1. 登入 GitHub → 右上角「+」→ New repository，取名（例如 `nccu-scholarship-wiki`），Public 或 Private 都可以 → Create。
2. 解壓縮後，在 repo 頁面點 **uploading an existing file**，把資料夾**裡面的**所有檔案和資料夾拖進去（`index.html` 要在最上層）→ Commit changes。

### 2. 在 Vercel 建立專案
1. 到 https://vercel.com 用 GitHub 帳號登入（Hobby 免費方案）。
2. Add New… → Project → 找到剛剛的 repo → Import。
3. 設定畫面：
   - Framework Preset：**Other**
   - Build Command、Output Directory：都留空
   - 展開 **Environment Variables**，新增：

| 名稱 | 內容 |
|---|---|
| `AI_API_KEY` | 你的 Agnes AI 金鑰（必填） |
| `AI_MODEL` | 選填。不填就用 `agnes-3.0-flash`（目前免費）；想換成 `agnes-2.5-flash` 再填 |

4. 按 Deploy，約 30 秒後會拿到網址，例如 `https://nccu-scholarship-wiki.vercel.app`。

### 3. 確認
- 點幾個條目、開一下 PDF。
- 在搜尋框問「成績 65 分可以申請嗎」，最上方出現「AI 回答」就成功了。

## 之後怎麼更新
1. 新資料（PDF）放進 `raw/`，檔名用英文。
2. 把 `AGENTS.md` 和相關條目交給 LLM，照 ingest 流程更新 `wiki/`、`wiki/pages.json`、`index.md`、`log.md`。
3. 把改好的檔案上傳到 GitHub 對應的資料夾（同名會覆蓋）。

GitHub 一更新，Vercel 會自動重新部署，約 30 秒後網站和 AI 都會用到新內容。

**改了環境變數（例如換模型或換金鑰）**：Vercel 不會自動套用，要到 Deployments → 最新一筆右邊的「⋯」→ Redeploy。

## 搜尋與 AI 問答
- 搜尋框可以輸入問句。AI 只回答**原住民族事務**與**政大校園生活**相關的問題，分三種情況：
  - wiki 有資料：依 wiki 回答並列出依據條目。
  - 範圍內但 wiki 沒寫（例如宿舍、族語認證）：回答「目前沒有這方面的資料」並附原資中心聯絡方式。
  - 範圍外（翻譯、寫作業、寫程式、閒聊、要它忽略規則等）：顯示固定的婉拒訊息，不提供任何內容。
- 服務範圍寫在 `api/ask.js` 的 `systemPrompt()`，婉拒文字是同一檔案的 `OFF_TOPIC`，要調整都改那裡。
- 下方同時列出關鍵字找到的相關條目；AI 出問題時會自動只顯示這部分。
- 首頁「試試看問」的範例問題和「最新消息」第一則的截止倒數，在 `index.html` 最上方的設定區（`EXAMPLES`、`DEADLINES`），換學期時記得一起改。其他最新消息寫在 `wiki/overview.md` 的「最新消息」小節。
- 每篇條目上方的「原始碼」分頁，可以直接看到 LLM 寫的 Markdown 原檔。
- 學生常用、但 wiki 沒用到的說法，可以加進 `index.html` 裡的 `SYN` 同義詞表，讓關鍵字搜尋也找得到。
- 想暫時關掉 AI：把 `index.html` 裡的 `const ASK_API = "/api/ask";` 改成 `const ASK_API = "";`。
- 提問次數不設限制；問題最長 200 字，只接受本站網頁發出的請求。
- 費用：`agnes-3.0-flash` 與 `agnes-2.5-flash` 目前限時免費。優惠結束後的刊例價為輸入 $0.05、輸出 $0.15（每百萬 token），以每題約一萬 token 估算，1,000 題約 0.5 美元。請留意 Agnes AI 的優惠公告與帳單。

## 查看答不出來的問題
Vercel 專案 → **Logs**：
- 搜尋 `[unanswered]`：範圍內但 wiki 沒寫的問題，拿來決定下一步要補哪些資料。
- 搜尋 `[off_topic]`：被婉拒的問題。如果發現有正當問題被誤擋，調整 `systemPrompt()` 裡的範圍說明。免費方案的紀錄只保留很短的時間，建議定期看。

## 疑難排解
- **AI 一直顯示「暫時無法使用」**：到 Vercel 的 Logs 看錯誤。`缺少環境變數 AI_API_KEY` → 補上後 Redeploy；`AI API 401` → 金鑰錯誤；`AI API 400/404` → 模型名稱錯誤。
- **網站顯示「讀不到 wiki 內容」**：檔案多包了一層資料夾，`index.html` 必須在 repo 最上層。
- **更新後看到舊內容**：按 Ctrl+F5 強制重新整理。
- **在自己電腦預覽**：直接雙擊 `index.html` 讀不到內容；可以在資料夾裡執行 `python -m http.server 8000`，再開 http://localhost:8000（這種方式沒有 AI 問答，只有關鍵字搜尋）。
