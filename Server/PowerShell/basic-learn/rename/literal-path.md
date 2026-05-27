---
title: LiteralPath
description: Rename-Item -LiteralPath 參數深入。把路徑當字面值處理，避開萬用字元誤判。
sidebar:
  order: 3
---

## 定位

```
Server > PowerShell > basic-learn > rename > LiteralPath
```

## Summary

`-LiteralPath` 把輸入路徑**完全當字面值**讀，不解讀 `*`、`?`、`[]` 等萬用字元。檔名含 `[]` 或其他特殊字元時，這是唯一安全的選擇。

## 參數屬性

| 項目                        | 值              |
| --------------------------- | --------------- |
| 名稱                        | `-LiteralPath`  |
| 型別                        | `String`        |
| 別名                        | `PSPath`        |
| 預設值                      | _無_            |
| Parameter set               | `ByLiteralPath` |
| Position                    | `Named`         |
| Mandatory                   | `True`          |
| Pipeline (by value)         | `False`         |
| Pipeline (by property name) | `True`          |

## 典型用法

### 1. 含 `[]` 的檔名

```powershell
Rename-Item -LiteralPath '.\literal[1].txt' -NewName 'literal-one.txt'
```

`literal[1].txt` 在 `-Path` 下會被當作「字元集 `1` 中的任一字」，但 `-LiteralPath` 把它讀成檔名本身。

### 2. 用別名 `-PSPath`

```powershell
Rename-Item -PSPath '.\report(2026)[final].txt' -NewName 'report-2026-final.txt'
```

`-PSPath` 是官方別名，功能等同。Pipeline 物件本身也有 `PSPath` 屬性，所以「by property name」綁定也走這條。

### 3. 與 `-Path` 的對照（觀察題）

同一個資料夾下同時放：

- `literal[2].txt`
- `literal2.txt`

```powershell
Rename-Item -Path '.\literal[2].txt' -NewName 'wrong_target.txt' -WhatIf
# What if: ... Item: ...\literal2.txt  ← 命中了錯的檔案

Rename-Item -LiteralPath '.\literal[2].txt' -NewName 'literal-two-bracket.txt'
# 才會真正命中 literal[2].txt
```

`-Path` 把 `[2]` 當成字元集 → 解析成「字元 `2`」→ 命中 `literal2.txt`。`-LiteralPath` 才是正確選擇。

## 與 `-Path` 的角色對照

```powershell
Rename-Item -Path        ".\*.log" -NewName "..."   # ← 萬用字元會展開
Rename-Item -LiteralPath ".\*.log" -NewName "..."   # ← 真的去找一個叫 *.log 的檔案
```

兩個參數在不同的 parameter set，**不能同時指定**。

## 為什麼建議用單引號

```powershell
"path[1].txt"   # 雙引號：仍會解讀變數展開等規則
'path[1].txt'   # 單引號：盡量原樣傳入
```

含特殊字元時用單引號，可避免 PowerShell 在字串階段就先動過內容，讓 `-LiteralPath` 的「照字面」更乾淨。

## 常見錯誤

- 看見 `[]`、`*`、`?` 還用 `-Path`，結果命中錯檔（或找不到檔）。
- 把 `-Path` 與 `-LiteralPath` 同時下，觸發 parameter set 衝突。
- 用雙引號包含 `$` 開頭的特殊路徑，被當變數展開。

## 觀察

`-LiteralPath` 的存在揭露了一件事：PowerShell 預設**所有路徑參數都會走萬用字元解析**。這在批次處理時是方便特性，但對「我就是要這個字面檔名」的場景反而是陷阱。

設計上把兩種行為拆成兩個 parameter，而不是用 switch 切換，就是要逼使用者在寫腳本時對「這個路徑是 pattern 還是 literal」做明確選擇 — 這個決策不能省。

## Related

- [rename skill 總覽](../)
- [Path](./path/)
- [NewName](./new-name/)

## External references

- [Rename-Item 官方文件 — Parameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item#parameters)
- [about_Wildcards](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_wildcards)
- [about_Quoting_Rules](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_quoting_rules)
