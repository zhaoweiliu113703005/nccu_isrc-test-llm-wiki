# AGENTS.md：這個 LLM Wiki 的規則（schema）

本知識庫依 Karpathy 的 LLM Wiki 模式運作：人負責挑選資料、提問；LLM 負責整理、交叉連結、維護一致性。

## 目錄
- `raw/`：原始資料，**唯讀**，永遠不要修改或刪除（網站上可直接下載）。
- `wiki/`：LLM 撰寫與維護的 Markdown 條目。
  - `sources/`：每份原始資料一頁摘要（「來源-」開頭）。
  - `concepts/`：主題條目（資格、期程、文件、金額……）。
  - `entities/`：機關、單位、組織。
  - `queries/`：值得保存的問答與檢查報告。
  - `index.md`：全部條目的目錄，每條一行說明。
  - `log.md`：只增不改的操作紀錄。

## 條目格式
每頁開頭放 frontmatter：
```
---
title: 條目名稱（與檔名相同）
type: source | concept | entity | query | meta
sources: [來源頁名稱, ...]
updated: YYYY-MM-DD
---
```
- 內文用 `[[條目名稱]]` 互相連結。
- 每個事實都要能追溯到 `sources` 中的來源；來源互相矛盾時，兩者都寫，並記到 [[待確認事項]]。
- 繁體中文、台灣用語；日期寫西元與民國並陳時以原文為準。

## 工作流程
**Ingest（加入新資料）**：把檔案放進 `raw/` → 寫一頁 `sources/來源-xxx.md` → 更新所有受影響的概念與實體條目 → 更新 `index.md` → 在 `log.md` 加一筆。
**Query（提問）**：先讀 `index.md` 找相關條目，再回答並標出依據的條目；答案有長期價值就存進 `queries/`。
**Lint（健檢）**：檢查矛盾、過期資訊、孤兒條目（沒有被連結）、缺來源的敘述，結果寫入 [[待確認事項]]。

## 發佈（Vercel）
- 整個資料夾就是網站根目錄，放在 GitHub，由 Vercel 自動部署。`index.html` 讀取 `wiki/pages.json` 列出的 Markdown 檔並顯示。
- **新增條目時，一定要把路徑加進 `wiki/pages.json`**，順序就是側欄的順序；AI 問答也只讀這份清單裡的條目。
- `raw/` 裡的檔名用英文小寫與連字號；在來源頁用 `[開啟 PDF](raw/檔名.pdf)` 連結。
- 圖片放 `assets/`，在 Markdown 裡用 `![說明](assets/檔名.jpg)`。
- `api/ask.js` 是 AI 問答函式，金鑰放在 Vercel 環境變數，**永遠不要寫進任何檔案**。
