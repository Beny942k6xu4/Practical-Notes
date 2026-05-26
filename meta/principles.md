---
title: 知識組織原則
description: 本站採用的 Domain / Subdomain / Topic / Skill / Concept 五層邏輯模型與寫作慣例。
---

## 五層邏輯模型

```text
Domain  →  Subdomain  →  Topic  →  Skill  →  Concept
```

範例：

| 層 | 值 |
|---|---|
| Domain | `Server` |
| Subdomain | `PowerShell` |
| Topic | `basic-learn` |
| Skill | `rename` |
| Concept | `Path` |

對應位置：`/domains/server/powershell/basic-learn/rename/path/`

這個結構是**邏輯的，不是物理的**。資料夾路徑與 sidebar 是它的兩個表現，不是定義本身。

### Domain

廣義的實踐領域；只在範圍清楚、可重複擴充時建立。例：`Server`、`Web`、`Data`。

### Subdomain

Domain 下的主要分支或技術族。例：`Server` 下的 `PowerShell`、`Bash`、`Linux services`。

### Topic

學習單位。同一 subdomain 下幾個 skill 的合理集合。例：`basic-learn`（入門核心動作）、`scripting`（流程控制）、`remoting`（遠端管理）。

### Skill

一項能完整對外解釋的能力。對 PowerShell 來說，常見一個 Cmdlet 對應一個 skill（rename ↔ `Rename-Item`），但 skill 同時涵蓋與相鄰 Cmdlet 的角色關係。

### Concept

原子知識。一個 concept 對應一頁，常見對應到 Cmdlet 的一個參數或一個獨立觀念。

## 演化流程

1. **Capture** — 先寫，不糾結層級。可以從 concept 開始。
2. **Observe** — 看出兩三個 concept 形成 skill；幾個 skill 形成 topic。
3. **Promote** — 結構穩定後抽出 skill / topic 的 landing page。
4. **Refactor** — 拆過大的 topic、合併重複、補連結。

每篇 concept 的 `sidebar.order` 由整理者依「最常用 → 進階」決定。

## 寫作格式（每篇 concept 應包含）

- `title` / `description`（frontmatter）
- 「定位」一行（用麵包屑式 `A > B > C > D > E`）
- **Summary** — 一句話結論
- **參數屬性表 / 設定屬性表**（若適用）
- **典型用法**（可複製執行的範例）
- **與相鄰 concept 的對照**
- **常見錯誤**
- **觀察**（自己的洞見）
- **Related** — 內部連結
- **External references** — 官方文件

## 內部連結慣例

用相對 markdown 路徑，例如：

```md
[rename skill 總覽](../)
[Path concept](../path/)
```

Starlight 會自動解析、處理 base path、避免硬編碼 URL。

## Source of Truth

本站（Starlight）是**正式維護版**。[Practical-Collections](https://github.com/Beny942k6xu4/Practical-Collections) 是學習用副本。
新增或修改內容時，**先在 Starlight 完成**，再決定是否同步至 Collections。
