# 流程圖版本紀錄（對話後半段）

> 範圍：從「可以把執行程序化成一張鏈結圖嗎」開始，到「轉成 React 可複製版本」為止。
> 目的：保留流程圖演進與可回溯版本。

## 版本時間線

| 版本 | 主題 | 輸出型態 | 主要變更 |
|---|---|---|---|
| V1 | 03-14 主流程鏈結圖 | Mermaid | 初版流程鏈結（Step1->Step2->Step3 + 差異節點） |
| V2 | 03-04 基準流程圖 | Mermaid | 補上 03-04 當時流程作為比較基準 |
| V3 | 03-04 vs 03-14 差異對照圖 | Mermaid | 新增差異子圖（帳戶載入策略、新增 filter 支線） |
| V4 | 方法級 + 時間序列圖（03-04） | Mermaid | 從檔案步驟改為方法節點，按執行順序由上到下 |
| V5 | 方法級 + 時間序列圖（03-14） | Mermaid | 同上並加入新增支線 `filter_transactions.py` |
| V6 | React 可複製版（雙圖） | React + Mermaid | 提供 `@mermaid-js/react` 元件可直接貼入 |
| V7 | React 左右並排差異高亮版 | React + Mermaid | 加入 classDef，標示 `changed` / `added` 節點 |

## 方法級主路徑（最終口徑）

### 03-04
`run_all.main()`
-> `step1.main()`
-> `load_accounts()`（未知銀行代碼 skip）
-> `build_sources()/auto_detect_sources()/match_account()`
-> `parse_ctbc()/parse_yuanta()/parse_lianbang()`
-> `save_bank_csv()`
-> `step2.main()`
-> `fill_missing_bank_code()`
-> `write_xlsx()/write_balance_summaries()`
-> `step3.main()`
-> `check_bank_csv()/check_merged()`

### 03-14
主路徑同 03-04，但兩點變化：
1. `step1_parse.load_accounts()` 改為未知銀行代碼也載入（供遮罩補全比對）。
2. 新增支線：`filter_transactions.main()`
   -> `load_account_map()`
   -> `parse_query()`
   -> `match_row()`
   -> `format_output()`
   -> `輸出結果/篩選結果/*.txt`

## 差異追蹤指標（固定欄位）

1. 主表總筆數（舊->新）
2. 新增/刪除交易筆數
3. 遮罩補全筆數
4. 銀行代碼空白補全筆數（含代碼分布）
5. `step3_test` 錯誤/警告數

## 本次納入版本控管的檔案

- `docs/chat-logs/2026-03-26.md`（摘要對話）
- `docs/version-logs/flowchart-history-2026-03-26.md`（本檔）

