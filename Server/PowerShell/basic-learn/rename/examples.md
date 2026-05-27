---
title: Examples
description: Rename-Item 實戰範例彙整 — 從基本改名到批次、覆蓋陷阱、副檔名轉換。
sidebar:
  order: 10
---

## 定位

```
Server > PowerShell > basic-learn > rename > Examples
```

## Summary

這頁是 `Rename-Item` 各 concept 整合後的小型實戰彙整。每個範例都是一段可直接使用的真實情境，附帶結論與會踩到的雷。

## 1. 同目錄基本改名

```powershell
Rename-Item -Path "daily_report.txt" -NewName "monday_report.txt"
```

- `Rename-Item` 在同一目錄修改檔名。
- `-NewName` 只給新名稱，不含路徑。
- 對應 concept：[Path](./path/)、[NewName](./new-name/)

## 2. 避免覆蓋既有目標

```powershell
# ❌ 失敗 — 目標已存在
Rename-Item -Path "log_new.txt" -NewName "log_current.txt"
# Cannot create a file when that file already exists.

# ✅ 真的要覆蓋
Move-Item -Path "log_new.txt" -Destination "log_current.txt" -Force
```

- `Rename-Item` 不會幫你安全覆蓋既有檔案 — 這是**官方保證的硬限制**。
- 即使加 `-Force` 也擋不掉，覆蓋是 `Move-Item` 的職責。
- 對應 concept：[Force](./force/)

## 3. 批次加前綴

```powershell
Get-ChildItem -Filter *.txt |
    ForEach-Object {
        Rename-Item -Path $_.FullName -NewName ("2026_" + $_.Name)
    }
```

結果：

- `log_current.txt` → `2026_log_current.txt`
- `monday_report.txt` → `2026_monday_report.txt`

要點：

- `Get-ChildItem` + `ForEach-Object` 是最基本的批次寫法。
- 用 `$_.FullName` 作 `-Path` 避免相對路徑歧義。
- 對應 concept：[Path](./path/)、[NewName](./new-name/)

### 更簡潔的等價寫法

```powershell
Get-ChildItem -Filter *.txt |
    Rename-Item -NewName { "2026_" + $_.Name }
```

`-NewName` 接 scriptblock 後可省掉 `ForEach-Object`，這是「pipeline-native」寫法。

## 4. 副檔名轉換

```powershell
Rename-Item -Path "draft_config.json" -NewName "draft_config.bak"
```

- 副檔名也只是名稱的一部分，沒有專門參數。
- 本質仍是「同位置改名」，不是搬移。

## 5. 批次改名前的安全程序

```powershell
# Step 1: 先模擬，看會打到哪些檔
Get-ChildItem .\*.txt |
    Rename-Item -NewName { $_.Name -replace 'batch','set' } -WhatIf

# Step 2: 確認沒打到不該打的，再真做
Get-ChildItem .\*.txt |
    Rename-Item -NewName { $_.Name -replace 'batch','set' }
```

對應 concept：[WhatIf](./what-if/)、[LiteralPath](./literal-path/)（避免萬用字元誤命中）

## 6. 改名後立刻檢查結果

```powershell
Rename-Item -Path ".\file.txt" -NewName "renamed.txt" -PassThru |
    Select-Object Name, FullName, LastWriteTime
```

- 預設 `Rename-Item` 沒輸出，加 `-PassThru` 才能繼續處理結果。
- 對應 concept：[PassThru](./pass-thru/)

## 重點整理

| 場景 | 工具 |
|------|------|
| 同位置改名 | `Rename-Item` |
| 換目錄（含覆蓋） | `Move-Item [-Force]` |
| 批次改名 | `Get-ChildItem | Rename-Item -NewName {…}` |
| 改前要先看 | `-WhatIf` |
| 改後要拿結果 | `-PassThru` |
| 含 `[]` `*` 的檔名 | `-LiteralPath` |

## Related

- [rename skill 總覽](../)
- [basic-learn topic](../../)

## External references

- [Rename-Item 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item)
- [Move-Item 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/move-item)
