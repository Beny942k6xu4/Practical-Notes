---
title: Force
description: Rename-Item -Force 參數深入。可處理唯讀／隱藏屬性，但仍不能覆蓋既有目標。
sidebar:
  order: 4
---

## 定位

```
Server > PowerShell > basic-learn > rename > Force
```

## Summary

`-Force` 嘗試處理一般情況下不容易變更的項目（例如唯讀、隱藏），但**官方保證的硬限制不變**：`Rename-Item -Force` 仍無法覆蓋已存在的目標名稱。實際效果依 provider 而異。

## 參數屬性

| 項目                        | 值                |
| --------------------------- | ----------------- |
| 名稱                        | `-Force`          |
| 型別                        | `SwitchParameter` |
| 預設值                      | `False`           |
| Parameter set               | `(All)`           |
| Position                    | `Named`           |
| Mandatory                   | `False`           |
| Pipeline (by value)         | `False`           |
| Pipeline (by property name) | `False`           |

## 典型用法

### 1. 觀察唯讀／隱藏屬性

```powershell
Get-Item .\force_readonly.txt | Select-Object Name, Attributes
Get-ChildItem -Force .\force_hidden.txt
```

判斷是否真的需要 `-Force` 前，先看屬性實際長什麼樣。

### 2. 對唯讀檔案改名

```powershell
Rename-Item -Path ".\force_readonly.txt" -NewName "force_readonly_done.txt" -Force
```

**注意**：在本機 FileSystem provider 下實測，不加 `-Force` 也常常能成功改唯讀檔的名字 — 這正是「實作依 provider 而異」的具體例子。

### 3. 想覆蓋既有目標（會失敗）

```powershell
Rename-Item -Path ".\force_source.txt" -NewName "force_existing.txt" -Force
# Cannot create a file when that file already exists.
```

即使加上 `-Force`，目標已存在仍然會擋下來。

### 4. 真的要覆蓋 → 改用 `Move-Item -Force`

```powershell
Move-Item -Path ".\force_source.txt" -Destination ".\force_existing.txt" -Force
```

這是覆蓋目標的「對的工具」。`Rename-Item -Force` 不是。

## 兩種「Force 行為」的對照

| 類別 | 行為 | 是否保證 |
|------|------|----------|
| 處理唯讀／隱藏屬性 | 可能放寬 | ❌ 依 provider |
| 覆蓋既有目標 | 一律不行 | ✅ 官方保證限制 |

## 常見錯誤

- 把 `-Force` 理解成「無條件成功」。
- 把實測過的 provider 行為當成 PowerShell 的絕對規則。
- 想覆蓋目標時硬加 `-Force`，誤以為遲早會成功。

## 觀察

`-Force` 的官方說明刻意分成兩段寫：「會嘗試…」與「但不能覆蓋安全限制／已存在的目標」。這個設計反映 PowerShell 對「強制」的態度 — 強制是放寬條件，不是繞過約束。

更實際的結論：看到 `-Force` 不要先假設它一定有差。先測「不加」會發生什麼，再決定「加」是否真的改變結果。如果兩者結果一樣，這份腳本可能根本不需要 `-Force`。

## Related

- [rename skill 總覽](../)
- [NewName](./new-name/)
- [WhatIf](./what-if/)

## External references

- [Rename-Item 官方文件 — Parameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item#parameters)
- [Move-Item 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/move-item)
