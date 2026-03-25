# 流程圖（原始圖版）

> 這份檔案保留對話中產出的「圖」，以 Mermaid 形式可在 GitHub 直接渲染。

## 圖 1：03-14 主流程鏈結圖

```mermaid
flowchart LR
  SRC["原始來源\n中國信託.pdf / 聯邦銀行.pdf / 元大銀行.csv"] --> S1["step1_parse.py\n解析 + unmask_account"]
  ACC["accounts.txt"] --> S1
  S1 --> BANKCSV["輸出結果/各銀行/*_v3.csv"]

  ACC --> S2["step2_merge.py\nfill_missing_bank_code"]
  BANKCSV --> S2
  S2 --> MERGED["帳戶明細_v3.csv / 帳戶明細_v3.xlsx"]

  ACC --> S3["step3_test.py"]
  BANKCSV --> S3
  MERGED --> S3

  ACC --> FT["filter_transactions.py（新增）"]
  MERGED --> FT
  FT --> FILT["輸出結果/篩選結果/*.txt"]

  subgraph DIFF["3/04 → 3/14 差異鍊"]
    D1["accounts.txt 新增 Bitro(805)"]
    D2["step1: 未知銀行代碼帳號也載入"]
    D3["中信交易對象補全 +2 筆\n000176076**58447 → 0001760765058447"]
    D4["主表銀行代碼補全 +27 筆\n空白 → 805（皆手續費）"]
  end

  D1 --> D2 --> D3
  D1 --> D4
  D2 -.影響.-> S1
  D4 -.影響.-> S2
```

## 圖 2：03-04 基準流程圖

```mermaid
flowchart LR
  RA["run_all.py\n(一步跑 Step1→Step2→Step3)"] --> S1["step1_parse.py"]
  RA --> S2["step2_merge.py"]
  RA --> S3["step3_test.py"]

  SRC["原始來源\n中國信託.pdf / 元大銀行.csv / 聯邦銀行.pdf"] --> S1
  ACC["accounts.txt\n(主帳戶+貸款帳戶)\n未知銀行代碼會被跳過"] --> S1

  S1 --> BCSV["輸出結果/各銀行\n中國信託_v3.csv\n元大銀行_v3.csv\n聯邦銀行_v3.csv\n+ YYYY-MM 月份子資料夾"]

  ACC --> S2
  BCSV --> S2
  S2 --> MERGED["輸出結果\n帳戶明細_v3.csv\n帳戶明細_v3.xlsx\n帳戶餘額摘要.csv\n各月份月收支合計.csv"]

  ACC --> S3
  BCSV --> S3
  MERGED --> S3
  S3 --> RESULT["驗證結果\n錯誤/警告報告"]
```

## 圖 3：03-04 vs 03-14 差異節點對照圖

```mermaid
flowchart TB
  subgraph COMMON["共同主流程（兩版都有）"]
    A["原始來源\n中國信託.pdf / 元大.csv / 聯邦.pdf"] --> B["step1_parse.py"]
    C["accounts.txt"] --> B
    B --> D["輸出結果/各銀行/*_v3.csv"]
    C --> E["step2_merge.py"]
    D --> E
    E --> F["帳戶明細_v3.csv / .xlsx"]
    C --> G["step3_test.py"]
    D --> G
    F --> G
  end

  subgraph V304["2026-03-04 版"]
    X1["load_accounts 只收已知銀行代碼\n(822/806/803)"]
    X2["無 filter_transactions.py"]
  end

  subgraph V314["2026-03-14 版"]
    Y1["load_accounts 改為可載入未知銀行代碼帳號\n(供遮罩比對)"]
    Y2["新增 filter_transactions.py\n(查詢主表並輸出 txt)"]
    Y3["交易對象遮罩補全 +2 筆\n000176076**58447 → 0001760765058447"]
    Y4["銀行代碼空白補全 +27 筆\n(空白 → 805，皆中國信託手續費)"]
  end

  X1 -. 版本差異 .-> Y1
  X2 -. 版本差異 .-> Y2
  Y1 --> Y3
  C --> Y4
  E --> Y4
```

## 圖 4：2026-03-04 方法級流程圖（時間序）

```mermaid
flowchart TB
  R0["run_all.main()<br/>file: run_all.py"] --> R1["run(script)<br/>file: run_all.py"]

  R1 --> S10["step1.main()<br/>file: step1_parse.py"]
  A0["accounts.txt<br/>file: accounts.txt"] --> S11["load_accounts()<br/>未知銀行代碼: skip<br/>file: step1_parse.py"]
  I0["原始來源(pdf/csv)"] --> S13["extract_file_text()<br/>file: step1_parse.py"]

  S10 --> S11 --> S12["build_sources()<br/>file: step1_parse.py"]
  S12 --> S14["auto_detect_sources()<br/>file: step1_parse.py"]
  S13 --> S15["match_account()<br/>file: step1_parse.py"]
  S14 --> S15 --> S16["parse_ctbc()/parse_yuanta()/parse_lianbang()<br/>file: step1_parse.py"]
  S16 --> S17["save_bank_csv()<br/>file: step1_parse.py"]
  S17 --> O1["輸出：各銀行 *_v3.csv + YYYY-MM 子資料夾"]

  O1 --> S20["step2.main()<br/>file: step2_merge.py"]
  A0 --> S21["load_accounts()<br/>file: step2_merge.py"]
  S20 --> S21 --> S22["fill_missing_bank_code()<br/>file: step2_merge.py"]
  S22 --> S23["write_xlsx()<br/>file: step2_merge.py"]
  S22 --> S24["write_balance_summaries()<br/>file: step2_merge.py"]
  S23 --> O2["輸出：帳戶明細_v3.csv / 帳戶明細_v3.xlsx"]
  S24 --> O3["輸出：帳戶餘額摘要.csv / 月收支合計.csv"]

  O1 --> S30["step3.main()<br/>file: step3_test.py"]
  O2 --> S30
  S30 --> S31["check_bank_csv()<br/>file: step3_test.py"]
  S31 --> S32["check_merged()<br/>file: step3_test.py"]
  S32 --> O4["輸出：驗證結果（錯誤/警告/通過）"]
```

## 圖 5：2026-03-14 方法級流程圖（時間序）

```mermaid
flowchart TB
  R0["run_all.main()<br/>file: run_all.py"] --> R1["run(script)<br/>file: run_all.py"]

  R1 --> S10["step1.main()<br/>file: step1_parse.py"]
  A0["accounts.txt（含 Bitro 805）<br/>file: accounts.txt"] --> S11["load_accounts()<br/>未知銀行代碼: include<br/>file: step1_parse.py"]
  I0["原始來源(pdf/csv)"] --> S13["extract_file_text()<br/>file: step1_parse.py"]

  S10 --> S11 --> S12["build_sources()<br/>file: step1_parse.py"]
  S12 --> S14["auto_detect_sources()<br/>file: step1_parse.py"]
  S13 --> S15["match_account()<br/>file: step1_parse.py"]
  S14 --> S15 --> S16["parse_ctbc()/parse_yuanta()/parse_lianbang()<br/>file: step1_parse.py"]
  S16 --> S17["save_bank_csv()<br/>file: step1_parse.py"]
  S17 --> O1["輸出：各銀行 *_v3.csv + YYYY-MM 子資料夾"]

  O1 --> S20["step2.main()<br/>file: step2_merge.py"]
  A0 --> S21["load_accounts()<br/>file: step2_merge.py"]
  S20 --> S21 --> S22["fill_missing_bank_code()<br/>file: step2_merge.py"]
  S22 --> S23["write_xlsx()<br/>file: step2_merge.py"]
  S22 --> S24["write_balance_summaries()<br/>file: step2_merge.py"]
  S23 --> O2["輸出：帳戶明細_v3.csv / 帳戶明細_v3.xlsx"]
  S24 --> O3["輸出：帳戶餘額摘要.csv / 月收支合計.csv"]

  O1 --> S30["step3.main()<br/>file: step3_test.py"]
  O2 --> S30
  S30 --> S31["check_bank_csv()<br/>file: step3_test.py"]
  S31 --> S32["check_merged()<br/>file: step3_test.py"]
  S32 --> O4["輸出：驗證結果（錯誤/警告/通過）"]

  O2 --> F0["filter_transactions.main()（新增支線）<br/>file: filter_transactions.py"]
  A0 --> F1["load_account_map()<br/>file: filter_transactions.py"]
  F0 --> F1 --> F2["parse_query()<br/>file: filter_transactions.py"]
  F2 --> F3["match_row()<br/>file: filter_transactions.py"]
  F3 --> F4["format_output()<br/>file: filter_transactions.py"]
  F4 --> O5["輸出：輸出結果/篩選結果/*.txt"]
```

