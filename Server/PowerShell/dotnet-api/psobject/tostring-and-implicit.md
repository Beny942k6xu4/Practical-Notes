---
title: ToString & Implicit
description: PSObject.ToString / IFormattable / $ofs 與 implicit operators。理解 $obj.ToString() 與 "$obj" 為何結果不同，以及覆寫 ToString 為何不影響 ConvertTo-Json。
sidebar:
  order: 5
---

## 定位

```
Server > PowerShell > dotnet-api > psobject > ToString & Implicit
```

## Summary

`PSObject.ToString()` 委派 BaseObject；若加了 `ScriptMethod ToString` 會優先呼叫它。**但 `"$obj"` 字串插值走的不是 `ToString`，是 PowerShell 自己的 `LanguagePrimitives.ConvertToString`**——這條捷徑會在 `PSCustomObject` 上 fallback 成 `@{key=value; ...}`。`Boolean / Double / Int32 / String / Hashtable` 到 `PSObject` 有 implicit operator，所以 `[psobject]$true` 等寫法合法。

## 參數屬性 / 核心要點

| 場景                                       | 實際呼叫                                       | 結果                                  |
| ------------------------------------------ | ---------------------------------------------- | ------------------------------------- |
| 沒覆寫 ToString 的 PSCustomObject          | `BaseObject.ToString()`                        | 空字串                                |
| `Add-Member ToString` ScriptMethod 後 呼叫 `.ToString()` | 走你的 ScriptMethod                  | 字串插值 `"$obj"` **也**會走它        |
| 沒覆寫，但用 `"$obj"`                      | `LanguagePrimitives.ConvertToString`           | `@{Name=Ben; Age=30}` 樣式            |
| BaseObject 是 IFormattable                 | `ToString(fmt, provider)` 委派下去             | 例如 DateTime、Decimal 才有意義       |
| 對陣列字串化                               | 元素以 `$ofs` 連接                             | 預設空白；可改 `$ofs = '\|'`          |
| `[psobject]$value`（基礎型別）             | 走 Implicit operator，等價 `PSObject(Object)`  | BaseObject 是原始型別                 |

## 典型用法

### 1. 預設 `ToString` 與字串插值的落差

```powershell
$p = [pscustomobject]@{ Name = 'Ben'; Age = 30 }
$p.ToString()      # ''               ← BaseObject (PSCustomObject) 沒實作有意義的 ToString
"$p"               # '@{Name=Ben; Age=30}'  ← LanguagePrimitives.ConvertToString 的 fallback 格式
```

**`$p.ToString()` 和 `"$p"` 不一樣**：前者直接呼叫方法，後者走 PowerShell 的字串化捷徑。覆寫 `ToString` 之後兩者才會一致。

### 2. 覆寫 ToString 後一致

```powershell
$p | Add-Member -MemberType ScriptMethod -Name ToString `
        -Value { "Person<$($this.Name)>" } -Force

$p.ToString()    # Person<Ben>
"$p"             # Person<Ben>   ← 字串插值也走自訂 ToString
```

注意要 `-Force`：`ToString` 本來就存在，不加 `-Force` 會丟 `a member with that name already exists`。

### 3. `$ofs` 控制陣列字串化分隔

```powershell
$arr = 1,2,3
"$arr"                 # "1 2 3"
$ofs = '|'
"$arr"                 # "1|2|3"
Remove-Variable ofs
"$arr"                 # "1 2 3"  ← 還原
```

`$ofs` 是 session 級自動變數，改了不還原會污染後續所有陣列字串化。函式內動 `$ofs` 用 `try / finally` 確保還原。

### 4. `IFormattable` 委派

```powershell
$dt = Get-Date
$ps = [psobject]::new($dt)
$ps.ToString('yyyy-MM-dd', $null)   # 2026-05-27
$dt.ToString('yyyy-MM-dd', $null)   # 一致
```

`PSObject.ToString(format, provider)` 委派到 BaseObject 的 `IFormattable`。`DateTime` / `Decimal` / `Double` 等有實作，所以可用；`PSCustomObject` 沒實作就會丟 `Cannot find an overload for "ToString" and the argument count: "2"`。

### 5. Implicit operators

```powershell
foreach ($v in @($true, 42, 3.14, 'hello', @{a=1})) {
    $ps = [psobject]$v
    $ps.psobject.BaseObject.GetType().FullName
}
# System.Boolean / System.Int32 / System.Double / System.String / System.Collections.Hashtable
```

`[psobject]$x` 對這些型別走 implicit operator，等價於 `PSObject(Object)`。

## 與 `ConvertTo-Json` 的角色對照

```powershell
$p = [pscustomobject]@{ Name = 'Ben'; Age = 30 }
$p | Add-Member -MemberType ScriptMethod -Name ToString `
        -Value { "Person<$($this.Name)>" } -Force

$p | ConvertTo-Json
# {
#   "Name": "Ben",
#   "Age":  30
# }
"$p" | ConvertTo-Json    # "Person<Ben>"
```

**`ConvertTo-Json` 不會呼叫 `ToString`**，它列舉 `.psobject.Properties`。想讓 JSON 變成單一字串要先自己 `"$p"` 再 pipe 進去。同理：`Export-Csv`、`Format-List`、`Format-Table` 都走屬性列舉，覆寫 `ToString` 對它們完全沒影響。

## 常見錯誤

| 寫法                                                | 症狀                                                | 原因                                                       |
| --------------------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------- |
| `Add-Member ... -Name ToString -Value {}` 沒加 `-Force` | `a member with that name already exists`            | ToString 本來就有，要 `-Force` 覆寫                        |
| PSCustomObject `.ToString('X', $null)`              | `Cannot find an overload ... argument count: "2"`   | BaseObject 不是 IFormattable                               |
| 期望 `ConvertTo-Json` 走自訂 ToString                | JSON 仍把屬性攤開                                    | JSON 序列化用成員列舉，不會呼叫 ToString                   |
| 改 `$ofs` 沒還原                                    | 之後所有陣列字串化都被影響                          | `$ofs` 是 session 級變數，記得 `Remove-Variable ofs`       |
| 期望 `$p.ToString()` 和 `"$p"` 結果一致              | 沒覆寫前不一致                                       | 字串插值走 `LanguagePrimitives.ConvertToString`，不是 ToString |

## 觀察

PowerShell 的「物件 → 字串」實際上有**至少三條獨立路徑**，它們**不互相 fallback**：

1. **`$obj.ToString()`**：呼叫 PSObject → 委派 BaseObject 的 `ToString`（除非有 `ScriptMethod` 覆寫）
2. **`"$obj"` 字串插值**：走 `LanguagePrimitives.ConvertToString`；對 PSCustomObject 有特殊 fallback `@{...}`；**有覆寫 ToString 時會走它**
3. **`ConvertTo-Json` / `Export-Csv` / `Format-*`**：列舉 `.psobject.Properties`，**完全不碰 `ToString`**

「我覆寫了 ToString，為什麼 JSON 沒變？」、「我 `$p.ToString()` 是空字串為什麼 `"$p"` 又有東西？」這些經典問題都源於把這三條路徑當成同一條。

實務原則：**要影響 JSON / CSV / Format → 改屬性結構；要影響字串插值與直接呼叫 → 覆寫 ToString（記得 `-Force`）**。兩件事不是同一回事。

## Related

- [psobject skill 總覽](../)
- [Members concept](../members/)
- [BaseObject concept](../base-object/)

## External references

- [PSObject.ToString Method](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.tostring)
- [PSObject Implicit Operators](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.op_implicit)
- [about_Automatic_Variables（含 `$OFS`）](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_automatic_variables)
