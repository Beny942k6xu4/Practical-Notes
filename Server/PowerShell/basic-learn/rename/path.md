---
title: Path
description: Rename-Item -Path 參數深入。指定要被重新命名的舊項目；rename skill 的核心輸入。
sidebar:
  order: 1
---

## 定位

```
Server > PowerShell > basic-learn > rename > Path
```

## Summary

`-Path` 指定**現在已經存在、要被改名的那個項目**。它是 `Rename-Item` 的核心輸入；改名要改誰，靠這個參數決定。

## 參數屬性

| 項目                        | 值       |
| --------------------------- | -------- |
| 名稱                        | `-Path`  |
| 型別                        | `String` |
| 預設值                      | _無_     |
| Parameter set               | `ByPath` |
| Position                    | `0`      |
| Mandatory                   | `True`   |
| Pipeline (by value)         | `True`   |
| Pipeline (by property name) | `True`   |
| Remaining args              | `False`  |

## 三種典型用法

### 1. 相對路徑

```powershell
Rename-Item -Path ".\path_alpha.txt" -NewName "path_alpha_done.txt"
```

以**目前工作目錄**為基準解析路徑。最常用、最簡潔，但會隨 `cd` 改變語意。

### 2. 絕對路徑

```powershell
Rename-Item -Path "C:\project\server\demo\path_beta.txt" -NewName "path_beta_done.txt"
```

不受目前所在位置影響，**腳本化首選**。

### 3. 由 pipeline 提供

```powershell
Get-ChildItem .\path_pipe.txt | Rename-Item -NewName "path_pipe_done.txt"
```

`Get-ChildItem` 輸出的 `FileInfo` 會被綁定到 `-Path`（by value + by property name 都支援）。
**這是批次改名的基礎寫法**。

## 與 `-NewName` 的角色對照

```powershell
Rename-Item -Path ".\old.txt" -NewName "new.txt"
#            ^^^^^^^^^^^^^^^   ^^^^^^^^^^^^^^^
#            舊（必須已存在）    新（同目錄、不含路徑）
```

`-NewName` **不能含路徑**；若需要搬移目錄，改用 `Move-Item`。

## 常見錯誤

- 把 `-Path` 和 `-NewName` 角色搞反。
- `-NewName` 誤填完整路徑（會失敗或得到意外名稱）。
- 在 pipeline 寫法裡多寫 `-Path` 又用管線（兩個來源衝突）。
- 用相對路徑寫腳本卻沒先 `Set-Location`，造成路徑解析不確定。

## 觀察

`-Path` 設計成 `Position 0` + 接 pipeline + by property name，目的是讓三種寫法都自然。
理解這個設計後，所有 `Get-ChildItem | Rename-Item …` 的批次寫法就只是它的特例。

## Related

- [rename skill 總覽](../)
- [basic-learn topic](../../)
- [PowerShell subdomain](../../../)

## External references

- [Rename-Item 官方文件 — Parameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item#parameters)
- [about_CommonParameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_commonparameters)
