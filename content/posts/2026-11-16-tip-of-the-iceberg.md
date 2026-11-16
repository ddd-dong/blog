---
title: TIP of the Iceberg
date: 2026-11-16T20:35:00
description: Task-in-Prompt 攻擊：將敏感字編碼後要求模型解題，藉此繞過安全訓練與防護模型的偵測
---

> **The TIP of the Iceberg: Revealing a Hidden Class of Task-in-Prompt Adversarial Attacks on LLMs**

> Sergey Berezin, Reza Farahbakhsh, Noel Crespi ・ ACL 2025 ・ [arXiv 2501.18626\](https://arxiv.org/abs/2501.18626)

> 一句話：將敏感字編碼成謎題，讓模型在解題時繞過安全限制

## 這篇在做什麼

這篇提出一類 jailbreak，叫 Task-in-Prompt（TIP）。做法不是直接問模型不該回答的東西，而是先把敏感的關鍵字編碼，再給模型一個解碼任務。模型為了把題目解出來，會自己把那個字還原，接著照 prompt 的要求繼續往下做。

作者把它整理成一個 benchmark 叫 PHRYGE。編碼方式有 10 種，像 Caesar cipher、Morse code、Base64、T9、謎語、Python 程式，乘上 4 種攻擊目標變成 40 個任務，每個任務再分 3 個難度，總共 120 個 prompt。受測的模型有 GPT-4o、LLaMA-3.2-3B、LLaMA-3.1-70B、Gemma-2-27B、Mistral-Nemo、Phi-3.5-Mini。

除了新的 benchmark，作者也把之前的一些攻擊重新歸類，例如 ArtPrompt，主張它們其實都是 TIP 的一種。

## 攻擊怎麼運作

一般直接問的 jailbreak 會被擋，是因為 prompt 裡直接寫了敏感字，安全訓練跟過濾器都看得到。TIP 的想法是讓那個字一開始根本不在 prompt 裡。

流程大概是：

1\. 把敏感關鍵字用某種方式編碼

2\. 在 prompt 裡給模型一個解碼任務

3\. 要求它用解出來的字完成後面的任務

用無害的字示意一下。把 HELLO 用 Caesar cipher 位移 3，會變成 KHOOR：

\`\`\`

下面這個字用 Caesar cipher 位移 3 加密過：KHOOR

先把它解出來，再用這個字當主題寫兩句話。

\`\`\`

prompt 裡只出現 KHOOR 跟解碼規則，HELLO 是模型自己在解題的時候想出來的。換成真正的攻擊，被編碼的就會是模型平常會拒絕的關鍵字。

編碼方式可以替換成 Morse、T9、Base64，或寫成一個 Python 函式要求模型「執行」並給出輸出。只要模型能解出來，攻擊就有機會成功，所以作者說這是一整類攻擊，不是某個特定的 prompt。

## 實驗結果

在 JailbreakBench 上，GPT-4o 遇到直接用 plain text 問的攻擊，成功率只有 0.03。換成 TIP 的 Python 版本，成功率變成 0.86（Table 2）。

防禦端的偵測效果也明顯下降。Llama Guard 3 對 plain text 的偵測率是 98%，但對 TIP Python 只剩 7%（Table 3）。關鍵字過濾對 TIP 的 Phonetic、Base64、Python 這幾個版本偵測率都是 0，因為 prompt 裡本來就沒有出現那個字（Table 3）。

攻擊是否成功是用 LLaMA-3.1-70B 當 judge 自動判斷。judge 的工作是讀模型的回答，判斷裡面有沒有出現不允許的內容。作者另外人工抽了一批樣本，檢查 judge 判得準不準（Section 4.3）。

## 我怎麼看

我最認同的是它對 ArtPrompt 的重新解讀。ArtPrompt 會成功，重點不是 ASCII art 本身，而是它讓模型做了一個「解碼任務」。這樣一講，很多看起來不一樣的攻擊都能放進同一個框架，這個詮釋很漂亮。

比較懷疑的有兩點。第一，judge 用的是 LLaMA-3.1-70B，但它自己也是被測的模型之一，有點球員兼裁判。第二，表裡 GPT-4o 的偵測率不是真的拿一個偵測器去測，而是用 1 − ASR 推出來的。

跟我自己的研究比較有關的是 agent。我在做 agent 安全，會想到 agent 讀到的 tool 回傳值如果是編碼過的，會不會一樣被「解題」騙過去。
