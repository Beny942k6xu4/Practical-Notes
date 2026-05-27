---
title: NewName
description: Rename-Item -NewName 參數深入。只接收新名稱，不可填路徑；rename 與 move 的分水嶺。
sidebar:
  order: 2
---

## 定位

```
Server > PowerShell > basic-learn > rename > NewName
```

## Summary

`-NewName` 指定**改完之後要叫什麼名字**。它**只**接受名稱、不接受路徑；想換目錄要改用 `Move-Item`。

## 參數屬性

| 項目                        | 值       |
| --------------------------- | -------- |
| 名稱                        | `-NewName` |
| 型別                        | `String` |
| 預設值                      | _無_     |
| Parameter set               | `(All)`  |
| Position                    | `1`      |
| Mandatory                   | `True`   |
| Pipeline (by value)         | `False`  |
| Pipeline (by property name) | `True`   |

## 典型用法

### 1. 直接給新名稱

```powershell
Rename-Item -Path ".\newname_single.txt" -NewName "newname_done.txt"
```

最單純的用法：新名稱不含路徑、不含萬用字元。

### 2. 用 scriptblock 動態產生

```powershell
Get-ChildItem .\newname_batch_*.txt |
    Rename-Item -NewName { $_.Name -replace 'batch','set' }
```

`-NewName` 接受 scriptblock，對每個 pipeline 物件動態算出新名稱。**這是批次改名的主力寫法。**

### 3. 改副檔名

```powershell
Rename-Item -Path ".\draft_config.json" -NewName "draft_config.bak"
```

副檔名本質上也是名稱的一部分，不需要特殊參數。

## 與 `Move-Item` 的角色對照

```powershell
# ❌ 不合法 — -NewName 不接路徑
Rename-Item -Path ".\file.txt" -NewName "C:\Temp\file.txt"
# Cannot rename the specified target, because it represents a path or device name.

# ✅ 要改名 + 換位置
Move-Item -Path ".\file.txt" -Destination "C:\Temp\file.txt"
```

只要句子裡有「換到別的資料夾」這層意思，就不是 `Rename-Item` 的工作。

## 常見錯誤

- 把 `-NewName` 當成 `-Destination` 用，誤填完整路徑 → `Cannot rename ... represents a path or device name.`
- 想用 `-NewName` 同時搬移又改名。
- 忘記 scriptblock 寫法時 `$_` 是每筆 pipeline 物件，不是字串。

## 觀察

`-NewName` 設計成「只給名稱」其實是個明顯的職責劃分：rename 管命名、move 管位置。這個邊界讓兩個 cmdlet 的行為都很可預測 — 看到 `Rename-Item` 就知道不會發生跨資料夾搬移。

scriptblock 接 `-NewName` 是 PowerShell 的延遲求值特色，它讓「對每個物件算一個專屬新名稱」變成一行就寫得完的事。

## Related

- [rename skill 總覽](../)
- [Path](./path/)
- [LiteralPath](./literal-path/)

## External references

- [Rename-Item 官方文件 — Parameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item#parameters)
- [Move-Item 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/move-item)
