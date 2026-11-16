---
title: "Harvest Season for SLUB: From io_uring Vulnerability to Novel Sheaf-Based Exploitation Techniques"
date: 2026-11-16
description: "OffensiveCon Tokyo 2026，與 Hao-Yu Yang（naup96321）共同演講。兩個 io_uring 0-day、SLUB sheaf / barn 機制的分析，以及三個以 sheaf 為基礎的新利用技巧。"
tags: ["talk", "offensivecon", "kernel", "io_uring", "exploitation"]
---

與 [Hao-Yu Yang（naup96321 / 堇姬 Naup）](https://www.offensivecon.jp/speakers/2026/hao-yu-yang-and-yu-ting-lin.html)共同演講。

## 內容

講的是 io_uring 與 Linux kernel 新的 SLUB allocator sheaf / barn 機制。

兩個 io_uring 0-day：一個是 CQ / SQ ring resize 與 deferred task work 之間的 race，另一個是 provided-buffer bundle 錯誤路徑上的 UAF。以此組出在多項 kernel 保護機制開啟的情況下仍可運作的本地提權。

sheaf / barn 機制會讓 cross-cache attack 這類常見技巧失效，所以我們對它做了深入分析，並指出 SLUB freelist mitigation 與 sheaf 之間的落差。接著提出三個以 sheaf 為基礎的新利用技巧，其中一個讓 cross-cache attack 不再依賴 buddy system —— 這個技巧也被用來建構第二個漏洞的利用鏈。

## 連結

- [議程頁面](https://www.offensivecon.jp/speakers/2026/hao-yu-yang-and-yu-ting-lin.html)
