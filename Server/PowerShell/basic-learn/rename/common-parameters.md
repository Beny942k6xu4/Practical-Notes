---
title: CommonParameters
description: PowerShell 共通參數在 Rename-Item 上的可見效果整理 — 哪些有明顯輸出、哪些是概念辨識。
sidebar:
  order: 9
---

## 定位

```
Server > PowerShell > basic-learn > rename > CommonParameters
```

## Summary

**CommonParameters 不是 `Rename-Item` 獨有的**，而是所有支援 `[CmdletBinding()]` 的 cmdlet 都自動具備的一組通用參數。本頁聚焦它們在 `Rename-Item` 上**實際看得到效果**的部分，以及哪些只是「定義上存在、實際無感」的概念題。

## 全 12 個 CommonParameters

| 參數 | 別名 | 在 Rename-Item 上的可見性 |
|------|------|------|
| `-Verbose` | `vb` | ✅ 明顯 |
| `-ErrorAction` | `ea` | ✅ 明顯 |
| `-ErrorVariable` | `ev` | ✅ 明顯 |
| `-OutVariable` | `ov` | ✅ 需配 `-PassThru` |
| `-PipelineVariable` | `pv` | ✅ 明顯 |
| `-Debug` | `db` | ⚠️ 互動提示為主，無明顯 DEBUG 輸出 |
| `-InformationAction` | `infa` | ⚪ 概念題（cmdlet 本身不發 Information stream） |
| `-InformationVariable` | `iv` | ⚪ 同上 |
| `-OutBuffer` | `ob` | ⚪ 大量物件才看得出差別 |
| `-ProgressAction` | `proga` | ⚪ 本機小型改名不會出現進度列 |
| `-WarningAction` | `wa` | ⚪ Rename-Item 一般不發 warning |
| `-WarningVariable` | `wv` | ⚪ 同上 |

## 可見效果明顯的用法

### 1. `-Verbose` — 看 cmdlet 在做什麼

```powershell
Rename-Item -Path ".\verbose_source.txt" -NewName "verbose_done.txt" -Verbose
# VERBOSE: Performing the operation "Rename File" on target "...".
```

### 2. `-ErrorAction` — 控制錯誤行為

```powershell
Rename-Item -Path ".\missing.txt" -NewName "x.txt" -ErrorAction Continue          # 顯示錯誤、繼續
Rename-Item -Path ".\missing.txt" -NewName "x.txt" -ErrorAction SilentlyContinue  # 壓住輸出，錯誤仍發生
```

### 3. `-ErrorAction Stop` + `try/catch`

```powershell
try {
    Rename-Item -Path ".\missing.txt" -NewName "x.txt" -ErrorAction Stop
}
catch {
    $_.Exception.GetType().FullName
    # System.Management.Automation.PSInvalidOperationException
}
```

把原本「非終止錯誤」升級成「終止錯誤」，才能被 `try/catch` 接住 — 這是 PowerShell 例外處理的重點機制。

### 4. `-ErrorVariable` — 收集錯誤

```powershell
Rename-Item -Path ".\missing.txt" -NewName "x.txt" -ErrorVariable renameErr
$renameErr             # 這次的錯誤

Rename-Item -Path ".\missing.txt" -NewName "x.txt" -ErrorVariable +renameErr
$renameErr.Count       # 2，前綴 + 表示 append
```

### 5. `-OutVariable` — 必須配 `-PassThru`

```powershell
Rename-Item -Path ".\file.txt" -NewName "done.txt" -PassThru -OutVariable renameOut
$renameOut | Select-Object Name, FullName
```

`Rename-Item` 預設沒輸出，所以單獨用 `-OutVariable` 收不到東西。

### 6. `-PipelineVariable` — 後段仍能存取前段物件

```powershell
Get-ChildItem .\pipe_*.txt -PipelineVariable source |
    Rename-Item -NewName { "done_" + $_.Name } -PassThru |
    ForEach-Object {
        [pscustomobject]@{
            CurrentOutput        = $_.Name        # 改名後物件
            CurrentPipelineValue = $source.Name   # 原始來源物件
        }
    }
```

在多階段管線中保留來源物件參考。

## 觀察型 / 概念型參數

### `-Debug`

在 `Rename-Item` 上未產生像 `-Verbose` 那樣明顯的 `DEBUG:` 訊息；實際表現為「跳互動式提示」。結論：不是每個 CommonParameter 在每個 cmdlet 上都有同樣可見效果。

### `-InformationAction` / `-InformationVariable`

`Rename-Item` 本身不會走 `Write-Information`，所以這兩個參數在這個情境下純粹是辨識題。

### `-OutBuffer`

控制送往下一段管線前的物件批次大小。小檔小批的本機改名觀察不出差別。

### `-ProgressAction`

控制 `Write-Progress` 行為。`Rename-Item` 在本機小型操作不會自然產生進度列。

### `-WarningAction` / `-WarningVariable`

`Rename-Item` 一般不會產生 warning record；`$warnLog` 空白不是參數失效，是這次操作真的沒東西可收。

## 觀察

CommonParameters 的價值不是「每個都記住怎麼用」，而是建立一個判讀框架：

> 看到陌生 cmdlet 時，知道 `-Verbose` 可以開上帝視角、`-ErrorAction Stop` + `try/catch` 是統一的例外處理路徑、`-OutVariable` 是把成功結果留底的方法、`-WhatIf` / `-Confirm` 是安全閘。

更深的觀察：CommonParameters 的「**有沒有效**」其實是 cmdlet 與其 provider 共同決定的。同一個 `-Verbose`，在 `Rename-Item` 上吵、在某些 cmdlet 上完全靜默 — 這不是 bug，是 cmdlet 作者選擇了「不發 verbose record」。理解這點之後，「為什麼這個參數沒反應？」就不再是疑問，而是一個可診斷的問題。

## Related

- [rename skill 總覽](../)
- [PassThru](./pass-thru/)（與 `-OutVariable` 配合）
- [WhatIf](./what-if/)
- [Confirm](./confirm/)

## External references

- [about_CommonParameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_commonparameters)
- [about_Preference_Variables](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_preference_variables)
- [Rename-Item 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item)
