---
title: "重點整理：PhantomSkill —— Agent Skill 生態裡的惡意程式碼注入"
date: 2026-09-08T20:15:00
description: "把明目張膽的惡意腳本改寫成「看起來只是寫得爛」的漏洞程式碼，藏進 skill 的 scripts/ 目錄。ASR 58.8%、warning 只有 11.4%、功能保留 96.6%。"
tags: ["paper", "agent-security", "supply-chain", "coding-agent"]
---

arXiv:[2606.19191](https://arxiv.org/abs/2606.19191)，Yu-Ting Lin、Chia-Mu Yu，2026-06-17。

Agent skill 讓 coding agent 可以從第三方套件取得領域能力，但這同時開了一個新的供應鏈攻擊面。這篇的核心觀察是：**大家在看 SKILL.md，沒人在看 `scripts/`**。

## 威脅模型

被攻擊的對象是能讀檔、能改專案、能執行 shell 的 LLM coding agent。攻擊者把惡意的 skill 變體發佈到公開倉庫，外觀和功能都跟正常的 skill 沒兩樣。

一個 skill 有四個部分：

- `name`
- `description`
- `SKILL.md` 的詳細指示
- **auxiliary resources**（輔助資源，通常是 `scripts/` 底下的腳本）

攻擊藏在第四項。這裡有個結構性的理由 —— **progressive disclosure**：詳細資源只在需要時才載入，所以審查的注意力天然集中在前三項。而且大家對「文字誘導」有戒心（prompt injection 已經被講爛了），對「一個可執行的腳本」反而覺得理所當然。

論文評估四種攻擊目標：憑證外洩、指令執行、agent 操縱、破壞。

## VulMask：把後門偽裝成漏洞

這是這篇最漂亮的一步。與其把惡意行為藏起來，不如把它**改寫成一個看起來像是不小心寫出來的漏洞**。

1. **選擇**：挑一個和預期惡意行為相符的漏洞類別（CWE）
2. **改寫**：用 LLM 把明確的 payload 轉成 vulnerability-shaped code
3. **嵌入**：把改寫後的 payload 整合進原本的輔助資源，同時保留原功能
4. **驗證**：在隔離環境測試，確認功能沒壞、觸發條件有效

結果是一段在攻擊者指定的條件被觸發前都保持休眠的程式碼。靜態掃描看到的不是 malware，是一個 CWE；LLM reviewer 看到的不是後門，是「這裡寫得不太安全」。而後者在絕大多數 review 流程裡都只會被標成低優先級。

## 實驗設定

- **Agent**：Cursor，後端分別接 GLM-4.7、Qwen3、GPT-5.5、Opus-4.7
- **生成模型**（用來做改寫）：GPT-5.5、GLM-4.7-Flash、Qwen3-Coder-30B
- **Skill 語料**：公開倉庫的代表性集合，涵蓋檔案工具、Git 自動化、coding 輔助、資料處理
- **自動審查器**：Mondoo、Cisco 靜態分析、Cisco LLM（GPT-4o）、Snyk agent scan

## 數字

GPT-5.5 上的整體結果：

| 指標 | VulMask | 明目張膽的惡意腳本 | Prompt injection |
|---|---|---|---|
| Attack Success Rate | **58.8%** | 46.3% | 0% |
| Warning Rate | **11.4%** | 25.4% | — |
| Utility Preservation | 96.6% | — | — |

Prompt injection 那個 0% 很說明問題：純文字誘導在現代 coding agent 上已經基本失效，但把同樣的意圖包裝成可執行資源就過了。

分攻擊目標來看（GPT-5.5）：

| 攻擊目標 | ASR | 偵測率 |
|---|---|---|
| 憑證外洩 | 52.9% | 11.8% |
| 指令執行 | 57.1% | 17.1% |
| Agent 操縱 | 60.0% | 2.9% |
| 破壞 | **79.2%** | 4.2% |

破壞類的 ASR 最高而偵測率幾乎最低，這其實合理 —— 破壞性操作（刪檔、覆寫）本來就長得像正常的檔案工具會做的事。

其他兩個數字：

- Cisco LLM（GPT-4o）對 malware 等級的偵測從 **100% 降到 76.3%**
- Agent 自己的偵測（GPT-5.5）從 25.4% 降到 11.4%
- 跨生成模型的遷移性：ASR 維持在 **41.0%–58.3%**

最後這點意味著攻擊者不需要用最強的模型來產生 payload。

## 提出的防禦

1. **Resource-level review**：marketplace 要掃的是腳本、相依套件和產生出來的成品，不只是描述文字
2. **執行期圍堵**：最小權限的檔案存取、網路限制、對敏感操作要有明確政策
3. **重新分類**：把 skill 裡「可觸發的漏洞」當成潛在的惡意 payload 處理，而不是低優先級的 issue

第三點是整篇的關鍵主張，也是最難落地的一點。現有的漏洞分級假設漏洞是無意的；一旦漏洞可能是刻意種的，「這只是個 CWE-78」這種判斷就不再安全。

## 限制

- 只在代表性的環境上評估，未來的框架可能不同
- 依賴預先定義的 CWE 類別
- 沒有涵蓋新興 agent 平台的完整多樣性
- 實驗都在受控的 sandbox 裡進行
