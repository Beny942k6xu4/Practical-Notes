---
title: PassThru
description: Rename-Item -PassThru 參數深入。讓預設無輸出的 Rename-Item 回傳改名後物件，能接續管線。
sidebar:
  order: 5
---

## 定位

```
Server > PowerShell > basic-learn > rename > PassThru
```

## Summary

`Rename-Item` 預設**不輸出任何物件**。加上 `-PassThru` 後，它會把改名後的項目當輸出送回管線 — 才能後續 `Select-Object`、存進變數或繼續處理。

## 參數屬性

| 項目                        | 值                |
| --------------------------- | ----------------- |
| 名稱                        | `-PassThru`       |
| 型別                        | `SwitchParameter` |
| 預設值                      | `False`           |
| Parameter set               | `(All)`           |
| Position                    | `Named`           |
| Mandatory                   | `False`           |
| Pipeline (by value)         | `False`           |
| Pipeline (by property name) | `False`           |

## 典型用法

### 1. 證明預設沒有輸出

```powershell
$result = Rename-Item -Path ".\passthru_alpha.txt" -NewName "passthru_alpha_done.txt"
$result    # ← $null
```

不加 `-PassThru` 時，賦值的左邊變數會是 `$null`。

### 2. 加上 `-PassThru` 取得改名後物件

```powershell
Rename-Item -Path ".\passthru_beta.txt" -NewName "passthru_beta_done.txt" -PassThru |
    Select-Object Name, Length, LastWriteTime
```

回傳的是**改名後**的 `FileInfo`，所以 `Name` 看到的會是新名稱。

### 3. 收進變數

```powershell
$item = Rename-Item -Path ".\passthru_pipe.txt" -NewName "passthru_pipe_done.txt" -PassThru
$item | Format-List Name, FullName
```

### 4. 批次改名同時收集每一筆結果

```powershell
Get-ChildItem .\passthru_*.txt |
    Rename-Item -NewName { "done_" + $_.Name } -PassThru |
    Select-Object Name
```

每改成功一筆，就回傳一筆 — 適合做改名後對帳。

## 與 `-Verbose` 的角色對照

```powershell
Rename-Item ... -Verbose     # 顯示「我做了什麼」，不會回傳物件，不能 pipeline
Rename-Item ... -PassThru    # 回傳物件，可繼續 pipeline，不會印 VERBOSE 訊息
```

`-Verbose` 是給人看的；`-PassThru` 是給程式用的。

## 常見錯誤

- 誤以為 `Rename-Item` 預設就會輸出物件 → 後面接 `Select-Object` 沒反應。
- 把 `-PassThru` 跟 `-Verbose` 混淆。
- 配合 `-OutVariable` 用卻沒加 `-PassThru` → `$ov` 永遠是空的。

## 觀察

`-PassThru` 的存在揭示了 PowerShell 對「副作用 cmdlet」的設計哲學：預設**靜默**，需要結果時再明確要。這跟 Unix `mv` 的「成功就什麼都不說」是同源思路，但 PowerShell 多給了一個對稱選項 — 你可以選擇要「物件流」而不只是「結束狀態」。

實務上的判準很簡單：這次改名後**還需不需要對結果做事**？需要就加 `-PassThru`，不需要就省。

## Related

- [rename skill 總覽](../)
- [CommonParameters](./common-parameters/)（含 `-OutVariable` 用法）

## External references

- [Rename-Item 官方文件 — Parameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item#parameters)
