---
title: Confirm
description: Rename-Item -Confirm 參數深入。執行前出現確認提示，由你選 Y/N 決定。
sidebar:
  order: 7
---

## 定位

```
Server > PowerShell > basic-learn > rename > Confirm
```

## Summary

`-Confirm` 在命令真正執行前**停下來問你**，輸入 `Y` 才繼續、`N` 跳過。是把執行決定權暫時交回人手上的安全機制。

## 參數屬性

| 項目                        | 值                |
| --------------------------- | ----------------- |
| 名稱                        | `-Confirm`        |
| 型別                        | `SwitchParameter` |
| 別名                        | `cf`              |
| 預設值                      | `False`           |
| Parameter set               | `(All)`           |
| Position                    | `Named`           |
| Mandatory                   | `False`           |
| Pipeline (by value)         | `False`           |
| Pipeline (by property name) | `False`           |

## 典型用法

### 1. 基本確認改名

```powershell
Rename-Item -Path ".\confirm_report.txt" -NewName "confirm_report_done.txt" -Confirm
# Confirm
# Are you sure you want to perform this action?
# Performing the operation "Rename File" on target "Item: ...".
# [Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```

輸入 `Y` 才會真的改。

### 2. 取消執行

```powershell
Rename-Item -Path ".\confirm_log.txt" -NewName "confirm_log_done.txt" -Confirm
# ← 在提示輸入 N
Get-ChildItem .\confirm_log.txt    # 仍存在
```

### 3. 用別名 `-cf`

```powershell
Rename-Item -Path ".\confirm_report.txt" -NewName "confirm_report_done.txt" -cf
```

### 4. 在提示輸入 `?` 看選項說明

| 選項 | 意義 |
|------|------|
| `Y` | 執行這一次 |
| `A` | 之後通通執行 |
| `N` | 略過這一次 |
| `L` | 之後通通略過 |
| `S` | 暫停、開巢狀工作階段 |

### 5. 與 `-PassThru` 串

```powershell
Rename-Item -Path ".\confirm_output.txt" -NewName "confirm_output_done.txt" -Confirm -PassThru |
    Select-Object Name, FullName
```

`-Confirm` 負責「問你」，`-PassThru` 負責「給你結果」。職責互不重疊，可同時用。

## 與 `-WhatIf` 的角色對照

| | `-WhatIf` | `-Confirm` |
|---|---|---|
| 會執行嗎？ | ❌ 永遠不會 | ✅ 同意後會 |
| 需要人在場嗎？ | ❌ | ✅ |
| 適合用在 | 寫腳本前驗證 | 半自動操作、批次前再覆核一次 |

## 常見錯誤

- 在無人值守的腳本裡用 `-Confirm`，導致整個流程卡在提示。
- 看到提示直接按 Enter（預設 `Y`），失去了「停下來看一眼」的本意。
- 想預演卻用了 `-Confirm` — 預演要用 `-WhatIf`。

## 觀察

`-Confirm` 本身和 `$ConfirmPreference` 環境變數連動 — 它的真正語意是「**強制把此次操作的 ConfirmImpact 視為高**」，所以才會跳提示。理解這層後就會明白：在不同 cmdlet 上 `-Confirm` 的效果為何「有的會跳、有的不會跳」，那是 cmdlet 自己宣告的 impact 等級在影響。

對 `Rename-Item` 來說，這個參數的最佳使用情境不是日常單檔改名，而是**批次改名腳本的最後一道閘門**：把腳本算出的批次列表先預覽（`-WhatIf`），確認沒問題後拿掉 `-WhatIf`、改加 `-Confirm`，半自動跑過一輪。

## Related

- [rename skill 總覽](../)
- [WhatIf](./what-if/)
- [PassThru](./pass-thru/)

## External references

- [Rename-Item 官方文件 — Parameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item#parameters)
- [about_Preference_Variables — $ConfirmPreference](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_preference_variables)
