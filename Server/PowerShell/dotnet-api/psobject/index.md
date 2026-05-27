---
title: psobject
description: System.Management.Automation.PSObject — PowerShell 物件視圖的本體。涵蓋建立、BaseObject、Members、AsPSObject/Copy、ToString 與隱式轉換。
---

## 是什麼

`PSObject` 是 PowerShell 包在每個物件外面的**視圖**。所有從 cmdlet 流出來、所有透過管線傳遞、所有被 `Format-*` 顯示的物件，本質上都是「某個 .NET 物件 + 一層 PSObject 視圖」。理解 PSObject 就能解釋：

- 為什麼 `$obj.BaseObject` 常常是 `null`
- 為什麼 `[psobject]@{}` 和 `[pscustomobject]@{}` 顯示不一樣
- 為什麼 `Add-Member` 對 Hashtable 有效但 Format-Table 看不到
- 為什麼兩個內容相同的 `[pscustomobject]` `Equals` 回 `False`
- 為什麼 `ConvertTo-Json` 不會跟著你覆寫的 `ToString` 走

## 何時用、何時不用

- ✅ 寫自訂物件回傳給管線 → 用 `[pscustomobject]@{}`
- ✅ 想為既有 .NET 物件臨時加屬性 → `[psobject]::new($x)` + `Add-Member`
- ✅ 要寫穩定的物件型別 + formatter → `PSTypeName` + `Update-TypeData`
- ❌ 想拿到「真實」型別判斷 → 別只看 `GetType()`；要看 `.psobject.BaseObject.GetType()`
- ❌ 想做內容相等比對 → 別用 `Equals`，改用 `Compare-Object -Property`

## Concepts（依學習順序）

| 順序 | Concept | 對應 API | 狀態 |
|---|---|---|---|
| 1 | **[Creation](./creation/)** | `PSObject()` / `PSObject(Object)` / `[pscustomobject]@{}` | ✅ stable |
| 2 | **[BaseObject](./base-object/)** | `BaseObject` / `ImmediateBaseObject` / `TypeNames` | ✅ stable |
| 3 | **[Members](./members/)** | `Members` / `Properties` / `Methods` + `Add-Member` | ✅ stable |
| 4 | **[AsPSObject & Copy](./as-psobject-and-copy/)** | `AsPSObject` / `Copy` / `Equals` / `CompareTo` | ✅ stable |
| 5 | **[ToString & Implicit](./tostring-and-implicit/)** | `ToString` / `IFormattable` / Implicit operators / `$ofs` | ✅ stable |

> 第一輪 `psobject` skill 已完成 5 個核心 concept。下一輪可能擴 `Serialization`、`ETS DefaultDisplayPropertySet`、與 C# 端的 PSObject 互動。

## 重點觀察

- **PSObject 是視圖，不是值**：腳本層幾乎所有「直覺反例」都來自 PowerShell runtime 在你不注意時偷偷做的 auto-unwrap / auto-wrap。
- **顯示 ≠ 屬性**：屬性存不存在跟看不看得見是兩件事；formatter 看的是 BaseObject 的型別，不是 PSObject 上加的 NoteProperty。
- **委派模式**：`Equals` / `GetHashCode` / `CompareTo` / `ToString(fmt, provider)` 全是「PSObject 不做事，委派給 BaseObject」。BaseObject 沒實作對應介面就會丟錯。

## 外部參考

- [PSObject Class — .NET API 文件](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject)
- [about_Object_Creation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_object_creation)
- [about_Types.ps1xml](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_types.ps1xml)
