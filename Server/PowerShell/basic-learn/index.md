---
title: basic-learn
description: PowerShell 入門主題。從最高頻使用的 Cmdlet 起手，建立對 Cmdlet 命名、參數設計、pipeline 行為的直覺。
---

## 範圍

`basic-learn` 收錄**最常用、最值得早期理解**的 Cmdlet 與通用觀念。目標不是窮舉，而是建立可推廣的直覺：

- Cmdlet 命名規律（Verb-Noun）
- 參數的角色與位置（mandatory / positional / pipeline）
- 物件管線（不是字串管線）
- 路徑與 provider 抽象

## Skills

- **[rename](./rename/)** — 重新命名項目的能力。從 `Rename-Item` 起手。

> 之後預期擴充：`copy`、`move`、`remove`、`get`、`set` 等基礎動作對應的 skill。

## 學習順序

每個 skill 內部 concept 的順序由 `sidebar.order` 決定，依「最常用 → 進階」排列。新手從 `rename / -Path` 開始建立第一個完整概念。
