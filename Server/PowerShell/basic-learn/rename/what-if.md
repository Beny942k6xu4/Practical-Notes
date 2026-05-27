---
title: WhatIf
description: Rename-Item -WhatIf 參數深入。模擬執行，只顯示「會發生什麼」，不會真的改檔。
sidebar:
  order: 6
---

## 定位

```
Server > PowerShell > basic-learn > rename > WhatIf
```

## Summary

`-WhatIf` 顯示「若命令真的執行，會發生什麼」，但**不會真的執行**。這是改名前的安全演練。

## 參數屬性

| 項目                        | 值                |
| --------------------------- | ----------------- |
| 名稱                        | `-WhatIf`         |
| 型別                        | `SwitchParameter` |
| 別名                        | `wi`              |
| 預設值                      | `False`           |
| Parameter set               | `(All)`           |
| Position                    | `Named`           |
| Mandatory                   | `False`           |
| Pipeline (by value)         | `False`           |
| Pipeline (by property name) | `False`           |

## 典型用法

### 1. 單一改名預演

```powershell
Rename-Item -Path ".\whatif_alpha.txt" -NewName "whatif_alpha_preview.txt" -WhatIf
# What if: Performing the operation "Rename File" on target
#          "Item: ...\whatif_alpha.txt Destination: ...\whatif_alpha_preview.txt".

Get-ChildItem .\whatif_alpha.txt   # ← 檔案仍叫原名
```

### 2. 用別名 `-wi`

```powershell
Rename-Item -Path ".\whatif_beta.txt" -NewName "whatif_beta_preview.txt" -wi
```

`-wi` 等同於 `-WhatIf`。互動時敲起來快。

### 3. 批次改名預演

```powershell
Get-ChildItem .\whatif_batch_*.txt |
    Rename-Item -NewName { $_.Name -replace 'batch','preview' } -WhatIf
```

會把每一筆的「來源 → 目標」全部印出來。**批次改名前一定先這樣跑一次**。

### 4. 演練後再真實執行

```powershell
Rename-Item -Path ".\whatif_alpha.txt" -NewName "whatif_alpha_done.txt" -WhatIf  # ← 看
Rename-Item -Path ".\whatif_alpha.txt" -NewName "whatif_alpha_done.txt"          # ← 做
```

## 與 `-Confirm` 的角色對照

```powershell
Rename-Item ... -WhatIf    # 只模擬，不問你
Rename-Item ... -Confirm   # 真要做，先問你 Y/N
```

| | `-WhatIf` | `-Confirm` |
|---|---|---|
| 會真的執行嗎？ | ❌ 從不 | ✅ 同意後會 |
| 需要互動嗎？ | ❌ | ✅ |
| 適合場景 | 看會不會打到錯的檔 | 真的要做、但想再確認一次 |

## 常見錯誤

- 看到 `What if:` 訊息就以為已經做完。
- 模擬後沒驗證「原檔還在」，誤以為改名失敗。
- 把 `-wi` 當成其他參數的別名（容易跟 `-vb` `-Verbose` 混）。

## 觀察

`-WhatIf` 是 PowerShell 內建的「演練模式」，所有支援 `SupportsShouldProcess` 的 cmdlet 都會自動有這個參數 — 不是 `Rename-Item` 特有的。

實務上更值得記住的是它的搭配：批次改名 + 含萬用字元的 `-Path` + 寫到一半的 scriptblock 表達式，全都應該先用 `-WhatIf` 證明命中的是你想動的那些檔，再拿掉它真做。少這一步，誤觸就是不可逆。

## Related

- [rename skill 總覽](../)
- [Confirm](./confirm/)
- [LiteralPath](./literal-path/)（用 `-WhatIf` 觀察萬用字元誤命中）

## External references

- [Rename-Item 官方文件 — Parameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item#parameters)
- [about_CommonParameters — WhatIf](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_commonparameters)
