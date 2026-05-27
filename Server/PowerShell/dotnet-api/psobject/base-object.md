---
title: BaseObject
description: PSObject.BaseObject / ImmediateBaseObject / TypeNames。理解 PowerShell 的 auto-unwrap 為何讓直覺寫法靜默失敗。
sidebar:
  order: 2
---

## 定位

```
Server > PowerShell > dotnet-api > psobject > BaseObject
```

## Summary

`PSObject` 是視圖，`BaseObject` 是它包覆的真實 .NET 物件。**腳本層 99% 的「為什麼這裡 null？」都源於 PowerShell 在讀屬性時會先 auto-unwrap，導致 `.BaseObject` 找不到對應屬性回傳 null**。要拿到真正的 `BaseObject` 必須走隱藏成員 `.psobject.BaseObject`。

## 參數屬性 / 核心要點

| 名稱                  | 型別                  | 可寫 | 用途                                                                |
| --------------------- | --------------------- | ---- | ------------------------------------------------------------------- |
| `BaseObject`          | `object`              | 否   | 取得最底層的非 PSObject 物件（巢狀 PSObject 會被遞迴拆開）          |
| `ImmediateBaseObject` | `object`              | 否   | 只拆一層；用於需要看「中間那層」PSObject 的情境（C# 才看得出差別）  |
| `TypeNames`           | `Collection<string>`  | 是   | 影響 formatter / `Update-FormatData` / `Update-TypeData` 的查找順序 |

關鍵：純 PowerShell 環境下，`BaseObject` 和 `ImmediateBaseObject` 永遠相同——因為 PowerShell runtime 不會讓 `PSObject(Object)` 重複包 PSObject，傳入已是 PSObject 的物件會被直接還回去。差異只在 C# 端用反射手動建構巢狀 PSObject 時才看得到。

## 典型用法

### 1. 取得真實型別

```powershell
$dt = Get-Date

$dt.BaseObject                              # 空（null）
$null -eq $dt.BaseObject                    # True

$dt.psobject.BaseObject.GetType().FullName  # System.DateTime
```

直接寫 `.BaseObject` 會被 auto-unwrap 後在 `DateTime` 上找名為 `BaseObject` 的屬性，找不到 → null。走隱藏的 `.psobject` 才能拿到 PSObject 視圖本體，再從那讀 `BaseObject`。

### 2. 用 `PSTypeName` 改變 TypeNames

```powershell
$p = [pscustomobject]@{
    PSTypeName = 'MyApp.Person'
    Name       = 'Ben'
    Age        = 30
    Country    = 'Taiwan'
}
$p.psobject.TypeNames
# MyApp.Person
# System.Management.Automation.PSCustomObject
# System.Object

Update-TypeData -TypeName MyApp.Person -DefaultDisplayPropertySet Name, Age -Force
$p   # 預設只看到 Name 與 Age；Country 仍可讀，只是不顯示
```

`PSTypeName` 是 `[pscustomobject]@{}` 字面值的隱藏 key，會被插到 `TypeNames` 最前面，作為 ETS / formatter 查找的入口。

## 與 `Hashtable` 包裝的對照

```powershell
$ps = [psobject]@{ Name = 'Ben'; Age = 30 }
$ps | Add-Member -NotePropertyName Country -NotePropertyValue 'Taiwan'

$ps.Country                                # Taiwan（屬性確實存在）
$ps | Format-Table                          # 只看到 Name / Value 兩欄
$ps.psobject.BaseObject.GetType().FullName  # System.Collections.Hashtable
```

`BaseObject` 是 `Hashtable`，formatter 套用 Hashtable 的 `Name / Value` 兩欄樣式，**它不會去問 PSObject 額外掛的 NoteProperty**。屬性實際存在，只是預設顯示路徑跳過它。想看：

```powershell
$ps | Format-List Name, Age, Country
$ps | Select-Object Name, Age, Country
```

## 常見錯誤

| 寫法                           | 症狀                                                | 原因                                                       |
| ------------------------------ | --------------------------------------------------- | ---------------------------------------------------------- |
| `$x.BaseObject.GetType()`      | `You cannot call a method on a null-valued expression` | auto-unwrap 後在 BaseObject 上找不到同名屬性，回傳 null    |
| `[psobject]@{...}` 期待表格化  | 預設顯示變成 Name/Value 兩欄                         | BaseObject 是 Hashtable，formatter 跟著 Hashtable 走       |
| 改 `TypeNames` 後沒效果        | 顯示與預期不符                                       | 必須先 `Update-TypeData` 註冊；既存註冊要 `-Force` 才覆蓋  |

## 觀察

`BaseObject` 與 `ImmediateBaseObject` 在文件上像是兩個重要的、區分明確的方法。腳本層的真相是：因為 PowerShell runtime 主動防止 PSObject 巢狀化（`[psobject]::new(psobjectInstance)` 會回原物，不會多包一層），這兩個在純 PowerShell 操作下永遠相同。

實務只記一條：**任何看 BaseObject 的時候都寫 `.psobject.BaseObject`**。直覺寫法 `.BaseObject` 在 .NET 設計上是合理的，但 PowerShell 的 auto-unwrap 把它變成了一個會 silent-fail 的陷阱——這是腳本層與 API 層語意對不上、進而需要本筆記存在的最直接理由。

## Related

- [psobject skill 總覽](../)
- [Creation concept](../creation/)
- [Members concept](../members/)

## External references

- [PSObject.BaseObject Property](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.baseobject)
- [PSObject.ImmediateBaseObject Property](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.immediatebaseobject)
- [PSObject.TypeNames Property](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.typenames)
- [about_Types.ps1xml](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_types.ps1xml)
