---
title: Members
description: PSObject.Members / Properties / Methods 與 Add-Member 的 MemberType 對照。理解 NoteProperty、ScriptProperty、AliasProperty、ScriptMethod 的差別。
sidebar:
  order: 3
---

## 定位

```
Server > PowerShell > dotnet-api > psobject > Members
```

## Summary

`PSObject` 上有三個成員集合：`Members`（總和）、`Properties`、`Methods`，全部透過 `.psobject.Members` / `.psobject.Properties` / `.psobject.Methods` 取得。配合 `Add-Member` 的各種 `-MemberType`，可以在不修改 .NET 類別的前提下為任何物件加上屬性、方法、別名與計算欄位——這套就是 PowerShell 的 ETS（Extended Type System）。

## 參數屬性 / 核心要點

| MemberType                    | 像什麼            | 值的來源                              | 可寫                              | 典型用途                    |
| ----------------------------- | ----------------- | ------------------------------------- | --------------------------------- | --------------------------- |
| `NoteProperty`                | 靜態欄位          | `-NotePropertyValue`                  | 是                                | 攤平資料、傳值              |
| `ScriptProperty`              | computed property | 一段 scriptblock                      | 取決於 `-SecondValue` 是否提供 set | 衍生值，例如 `IsAdult`     |
| `AliasProperty`               | 別名              | 指向同物件的另一屬性                  | 跟原屬性走                        | 改名、維持相容              |
| `ScriptMethod`                | 方法              | scriptblock，內用 `$this`             | n/a                               | 物件導向風格的行為          |
| `CodeProperty` / `CodeMethod` | 來自 .NET 靜態方法 | type + method name                    | 視實作                            | 把 C# 方法掛在 PS 物件上    |

關鍵約束：`Add-Member` 對任何物件都生效，但**只有當 BaseObject 是 PSCustomObject 時，預設 formatter 才會看見新成員**。對 Hashtable 加 NoteProperty 不會錯，但 `Format-Table` 看不到——理由屬於 [BaseObject](../base-object/) concept。

## 典型用法

### 1. 列舉屬性 metadata

```powershell
$p = [pscustomobject]@{ Name = 'Ben'; Age = 30 }
$p.psobject.Properties |
    Select-Object Name, MemberType, IsGettable, IsSettable
# Name MemberType   IsGettable IsSettable
# ---- ----------   ---------- ----------
# Name NoteProperty       True       True
# Age  NoteProperty       True       True
```

比 `Get-Member` 直接，可以在腳本裡做進一步過濾。

### 2. NoteProperty vs ScriptProperty 的差別

```powershell
$p = [pscustomobject]@{ Name = 'Ben' }
$p | Add-Member -NotePropertyName BirthYear -NotePropertyValue 1995
$p | Add-Member -MemberType ScriptProperty -Name Age2 -Value {
    (Get-Date).Year - $this.BirthYear
}

$p.BirthYear; $p.Age2     # 1995, (current year - 1995)
$p.BirthYear = 2000
$p.BirthYear; $p.Age2     # 2000, Age2 也跟著變
```

`NoteProperty` 是儲存值；`ScriptProperty` 每次讀都會重新跑 scriptblock。

### 3. ScriptMethod 與 `$this`

```powershell
$p = [pscustomobject]@{ Name = 'Ben' }
$p | Add-Member -MemberType ScriptMethod -Name Greet -Value {
    param([string]$lang)
    switch ($lang) {
        'en'    { "Hello, $($this.Name)!" }
        'zh'    { "哈囉，$($this.Name)！" }
        default { "Hi, $($this.Name)" }
    }
}
$p.Greet('en')
$p.Greet('zh')
```

`$this` 在 ScriptMethod 內代表呼叫它的物件；在 scriptblock 外為 null。

### 4. AliasProperty

```powershell
$p = [pscustomobject]@{ FirstName = 'Ben' }
$p | Add-Member -MemberType AliasProperty -Name Name -Value FirstName
$p.Name = 'Eve'
$p.FirstName    # Eve（兩個是同一個儲存槽）
```

## 與 Hashtable + Add-Member 的對照

```powershell
$h = @{ Name = 'Ben' }
$h | Add-Member -MemberType ScriptMethod -Name Greet -Value { "Hi $($this.Name)" }
$h.Greet()                    # "Hi Ben"  ← 能呼叫
$h.GetType().FullName         # System.Collections.Hashtable  ← 仍是 Hashtable
$h | Format-Table             # Name / Value 兩欄，看不到 Greet
```

`Add-Member` 能讓 Hashtable 「擁有」方法，但顯示與序列化體驗很糟。要完整體驗用 `[pscustomobject]@{}` 起手。

## 常見錯誤

| 寫法                              | 訊息 / 症狀                                  | 原因                                              |
| --------------------------------- | -------------------------------------------- | ------------------------------------------------- |
| 重複 `Add-Member` 同名屬性        | `a member with that name already exists`     | 預設不覆寫；要 `-Force`                           |
| ScriptProperty 引用不存在欄位     | 靜默回 `False` / `$null`                     | PowerShell 對 null 成員不丟錯，`$null -ge 18` 也是 False |
| Add-Member 後 Hashtable 仍顯示兩欄 | 預設顯示沒看到新成員                         | BaseObject 是 Hashtable，formatter 不走 PSObject  |
| `$this` 在 ScriptMethod 外為 null | `$this is null`                              | `$this` 只在被當成 method 呼叫時才有值            |

## 觀察

`ScriptProperty` 的「對缺失欄位靜默回 `$null` / `False`」是腳本層最隱蔽的 bug 來源。例：

```powershell
$p = [pscustomobject]@{ Name = 'Ben' }   # 沒寫 Age
$p | Add-Member -MemberType ScriptProperty -Name IsAdult -Value { $this.Age -ge 18 }
$p.IsAdult     # False
```

呼叫端看到 `False` 會以為「明確未成年」，實際上是「資料缺失」。寫 `ScriptProperty` 的時候，**對所有依賴欄位都要先做 null 檢查**，必要時改丟例外，而不是讓沉默的 `False` 漏出去：

```powershell
$p | Add-Member -MemberType ScriptProperty -Name IsAdult -Value {
    if ($null -eq $this.Age) { throw 'Age is missing' }
    $this.Age -ge 18
} -Force
```

這條原則延伸到所有透過 ETS 衍生的計算欄位：「靜默的預設值」遠比「炸開的 exception」難 debug。

## Related

- [psobject skill 總覽](../)
- [BaseObject concept](../base-object/)
- [Creation concept](../creation/)

## External references

- [PSObject.Members Property](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.members)
- [Add-Member 官方文件](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-member)
- [about_Types.ps1xml](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_types.ps1xml)
