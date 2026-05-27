---
title: Creation
description: PSObject 的四個建構子，以及 [pscustomobject]@{} 字面值寫法。釐清 [psobject]@{} 與 [pscustomobject]@{} 的差別。
sidebar:
  order: 1
---

## 定位

```
Server > PowerShell > dotnet-api > psobject > Creation
```

## Summary

`PSObject` 有四個 constructor，但腳本層日常只會碰到兩個：無參數 `PSObject()` 和接物件的 `PSObject(Object)`。對應在 PowerShell 端最常見的捷徑是 `[pscustomobject]@{...}` —— 它是「打開 PowerShell 自訂物件」的標準寫法；其他寫法是邊角案例或舊式相容。

## 參數屬性 / 核心要點

| 寫法                            | 實際呼叫的 ctor          | BaseObject 型別                                | 顯示行為                | 建議用途           |
| ------------------------------- | ------------------------ | ---------------------------------------------- | ----------------------- | ------------------ |
| `[psobject]::new()`             | `PSObject()`             | `PSCustomObject`                               | 空物件，加 member 後 OK | 需要逐步加成員時   |
| `[psobject]::new(10)`           | `PSObject(Int32)`        | `PSCustomObject`                               | 同上                    | 效能微優化（罕用） |
| `[psobject]::new($x)`           | `PSObject(Object)`       | `$x` 的實際型別                                | 套用 `$x` 的 formatter  | 想擴充既有物件     |
| `[psobject]$x`                  | 等同 `PSObject(Object)`  | `$x` 的實際型別                                | 同上                    | 同上               |
| `[pscustomobject]@{...}`        | 內部走 `PSObject(Object)` | `PSCustomObject`（用 hashtable 內容初始化）    | 標準表格化              | **日常推薦**       |
| `New-Object psobject -Property` | 同上                     | `PSCustomObject`                               | 標準表格化              | 舊腳本相容         |

> 要看 BaseObject 真正的型別，必須走 `.psobject.BaseObject.GetType()`，不能 `.BaseObject.GetType()`。理由屬於 [BaseObject](../base-object/) concept。

## 典型用法

### 1. 一次建好（首選）

```powershell
$p = [pscustomobject]@{
    Name    = 'Ben'
    Age     = 30
    Country = 'Taiwan'
}
$p.psobject.BaseObject.GetType().FullName
# System.Management.Automation.PSCustomObject
```

### 2. 包裝既有 .NET 物件

```powershell
$dt = Get-Date
$wrapped = [psobject]::new($dt)
$wrapped | Add-Member -NotePropertyName Tag -NotePropertyValue 'now'
$wrapped.psobject.BaseObject.GetType().FullName   # System.DateTime
$wrapped.Tag                                       # now
$wrapped                                           # 仍以 DateTime 的 formatter 顯示
```

包裝模式保留 `BaseObject` 的能力（屬性、方法、formatter），同時可以掛 ETS 成員。

### 3. 逐步加成員

```powershell
$a = [psobject]::new()
$a | Add-Member -NotePropertyName Name -NotePropertyValue 'Ben'
$a | Add-Member -MemberType ScriptProperty -Name IsAdult -Value { $true }
```

語意上等價於 `[pscustomobject]@{Name='Ben'}` 之後再 `Add-Member` 一個 ScriptProperty，但表達順序不同。

## 與 `[psobject]@{}` 的角色對照

```powershell
$e1 = [psobject]@{       Name = 'Ben'; Age = 30 }
$e2 = [pscustomobject]@{ Name = 'Ben'; Age = 30 }

$e1 | Add-Member -NotePropertyName Country -NotePropertyValue 'Taiwan'
$e2 | Add-Member -NotePropertyName Country -NotePropertyValue 'Taiwan'

$e1.GetType().FullName                       # System.Collections.Hashtable
$e2.GetType().FullName                       # System.Management.Automation.PSCustomObject

$e1 | Format-Table     # 只看到 Name / Value 兩欄，Country 被吃掉
$e2 | Format-Table     # Name Age Country 三欄
```

差別：
- `[psobject]@{}` 把 Hashtable 包成 PSObject 視圖，**BaseObject 仍是 Hashtable**，formatter 跟著 Hashtable 走。
- `[pscustomobject]@{}` 把 Hashtable 的 key 攤平成 PSCustomObject 的 NoteProperty，**BaseObject 是 PSCustomObject**，每個 key 都是真的屬性。

兩者都「能加 NoteProperty」，差別在「預設 formatter 看不看得見」。

## 常見錯誤

- `[psobject]:new()`（單冒號）→ `ParserError: Unexpected token ':new'`。靜態呼叫運算子是 `::`。
- 用 `[psobject]@{}` 包 Hashtable 卻期待表格輸出 → 解法是改用 `[pscustomobject]@{}`，或 `Select-Object` 投影。
- 用 `$obj.BaseObject` 看真正型別 → 拿到 `null`。改用 `$obj.psobject.BaseObject`。

## 觀察

`[pscustomobject]@{}` 之所以成為現代 PowerShell 的標準寫法，不是因為它「比較短」，而是因為它**讓 BaseObject 直接就是 PSCustomObject**，於是整條「屬性可加、可寫、可被 formatter 看見、可被 `ConvertTo-Json` 列舉」的鏈路一次到位。其他寫法都是這條鏈路上的某種「降級」：要嘛 BaseObject 不對（`[psobject]@{}`），要嘛繞遠路（`New-Object`），要嘛太低階（手動 `Add-Member`）。

把這條鏈路想清楚之後，就能反推「為什麼這個物件顯示不正常」這類問題：99% 是 BaseObject 沒選對。

## Related

- [psobject skill 總覽](../)
- [BaseObject concept](../base-object/)
- [Members concept](../members/)

## External references

- [PSObject Constructors — .NET API](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.-ctor)
- [about_Object_Creation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_object_creation)
