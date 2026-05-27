---
title: rename
description: 「重新命名」這個能力。以 Rename-Item Cmdlet 為核心，逐參數拆解；同時與 Move-Item、Copy-Item 對照角色差異。
---

## 是什麼

`rename` skill 聚焦「**把已存在的項目改名**」這件事。在 PowerShell 中由 `Rename-Item` 實作，但「rename 能力」本身比單一 Cmdlet 大：包含與其他相鄰動作（move、copy、remove）的角色釐清。

## 何時用、何時不用

- ✅ 同位置改名（檔案、機碼、變數）
- ✅ 修改副檔名
- ✅ 配合 `Get-ChildItem` + `ForEach-Object` 批次命名
- ❌ 想覆蓋已存在的目標 → 改用 `Move-Item -Force`
- ❌ 想跨資料夾搬移 → 改用 `Move-Item`

## Concepts（依學習順序）

| 順序 | Concept | 對應參數 | 狀態 |
|---|---|---|---|
| 1 | **[Path](./path/)** | `-Path` | ✅ stable |
| 2 | **[NewName](./new-name/)** | `-NewName` | ✅ stable |
| 3 | **[LiteralPath](./literal-path/)** | `-LiteralPath` | ✅ stable |
| 4 | **[Force](./force/)** | `-Force` | ✅ stable |
| 5 | **[PassThru](./pass-thru/)** | `-PassThru` | ✅ stable |
| 6 | **[WhatIf](./what-if/)** | `-WhatIf` | ✅ stable |
| 7 | **[Confirm](./confirm/)** | `-Confirm` | ✅ stable |
| 8 | **[Credential](./credential/)** | `-Credential` | ✅ stable（概念辨識） |
| 9 | **[CommonParameters](./common-parameters/)** | `-Verbose` / `-ErrorAction` / … | ✅ stable |
| 10 | **[Examples](./examples/)** | _整合範例_ | ✅ stable |

> 第一輪 `rename` skill 已完成全部 concept 與範例彙整。下一輪將開新的 skill。

## 重點觀察

- `Rename-Item` 在目標名稱已存在時會失敗 — 設計上是**安全的**，但需要覆蓋時要改用 `Move-Item -Force`。
- 接收 pipeline 物件可省略 `-Path`，物件屬性會被 bind 過去。

## 外部參考

- [Rename-Item 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item)
- [about_CommonParameters](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_commonparameters)
