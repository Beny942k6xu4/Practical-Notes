---
title: AsPSObject & Copy
description: AsPSObject / Copy / Equals / GetHashCode / CompareTo。理解委派模式、淺複製，以及為何 PSCustomObject 的相等比對總是 reference equality。
sidebar:
  order: 4
---

## 定位

```
Server > PowerShell > dotnet-api > psobject > AsPSObject & Copy
```

## Summary

`PSObject` 的 `AsPSObject` 用來統一 API 邊界、`Copy()` 是淺複製、`Equals` / `GetHashCode` / `CompareTo` 全部委派給 `BaseObject`。對 `[pscustomobject]@{}` 而言：兩個內容相同的物件 `Equals` 一定 `False`、`CompareTo` 一定丟錯——要做內容比對請用 `Compare-Object -Property`。

## 參數屬性 / 核心要點

| 方法          | 行為                                                                                  | 注意                                             |
| ------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `AsPSObject`  | 已是 PSObject → 原物還回；否則新建包裝                                                | 用於 API 邊界統一型別                            |
| `Copy()`      | **淺複製**：BaseObject 是值型別會複製、`ICloneable` 會呼叫 `Clone()`；其餘按 reference 共用 | 內部巢狀的陣列 / list / 物件 ref 仍與原物件共用  |
| `Equals`      | 委派 BaseObject                                                                       | 兩個獨立的 PSCustomObject 即使內容相同也 `False` |
| `GetHashCode` | 委派 BaseObject                                                                       | 兩個獨立 PSCustomObject 雜湊不同                 |
| `CompareTo`   | 委派 BaseObject 的 `IComparable`                                                       | BaseObject 不是 IComparable → 丟 `MethodNotFound` |

## 典型用法

### 1. AsPSObject：API 邊界統一

```powershell
$p  = [pscustomobject]@{ X = 1 }
$p2 = [psobject]::AsPSObject($p)
[object]::ReferenceEquals($p, $p2)   # True：已是 PSObject 直接還原物

$s  = 'hello'
$ps = [psobject]::AsPSObject($s)
$ps.psobject.BaseObject.GetType().FullName       # System.String

# PowerShell 在 [object] 參數綁定時會 auto-unwrap PSObject，
# 所以 ReferenceEquals 看起來「同個 reference」，純 PowerShell 看不出新包裝：
[object]::ReferenceEquals($ps, $s)               # True（被 auto-unwrap 騙到）
```

實務只記一條：`AsPSObject` 對已是 PSObject 的回原物、對其他型別保證回傳一個 PSObject 視圖，後續就能掛 ETS 成員。要在純 PowerShell 端證實「PSObject 本體不同 reference」需要走 C#。

### 2. Copy：淺複製的真實含義

```powershell
$a = [pscustomobject]@{ Name = 'Ben'; Age = 30 }
$b = $a.psobject.Copy()
$b.Name = 'Eve'
$a.Name; $b.Name   # Ben, Eve  ← NoteProperty 互不影響
```

但只要值是 reference type：

```powershell
$a = [pscustomobject]@{ Tags = @('x','y') }
$b = $a.psobject.Copy()
$b.Tags[0] = 'Z'
$a.Tags[0]                                       # Z  ← 也被改了
[object]::ReferenceEquals($a.Tags, $b.Tags)      # True

# 真正要獨立：
$b.Tags = @($a.Tags.Clone())
```

### 3. 內容比對請用 `Compare-Object`

```powershell
$a = [pscustomobject]@{ Name = 'Ben'; Age = 30 }
$b = [pscustomobject]@{ Name = 'Ben'; Age = 30 }

$a.Equals($b)   # False
$a -eq $b       # False
$null -eq (Compare-Object $a $b -Property Name, Age)   # True：完全相同
```

## 與 `CompareTo` 的角色對照

```powershell
$a = [pscustomobject]@{ Name = 'Ben' }
$b = [pscustomobject]@{ Name = 'Ben' }
$a.CompareTo($b)
# MethodInvocationException: does not contain a method named 'CompareTo'.

$s1 = [psobject]::AsPSObject('apple')
$s2 = [psobject]::AsPSObject('banana')
$s1.CompareTo($s2)   # -1：String 是 IComparable，委派成功
```

能不能 `CompareTo`，**看 BaseObject 是否實作 `IComparable`**，不是 PSObject 本身。

## 常見錯誤

| 寫法                                                        | 症狀             | 原因                                             |
| ----------------------------------------------------------- | ---------------- | ------------------------------------------------ |
| `$a.Equals($b)` 期待內容比對                                | 永遠 `False`     | 委派 BaseObject，PSCustomObject 沒覆寫 Equals    |
| `$a.CompareTo($b)`（兩個 PSCustomObject）                   | `MethodNotFound` | BaseObject 不是 IComparable                      |
| `$b = $a.psobject.Copy(); $b.Tags[0] = ...` 期望不影響 `$a` | `$a.Tags` 跟著變 | `Copy()` 是淺複製，陣列 reference 共用           |
| `Compare-Object $a $b` 沒給 `-Property`                     | 結果常常不直觀   | 沒指定屬性時用 `ToString`，PSCustomObject 多半是空 |

## 觀察

`Equals` / `GetHashCode` / `CompareTo` 的「全部委派 BaseObject」設計，是 PSObject 作為**視圖**身份的一致表現——它本身不持有相等語意，所有關於「值」的判斷都丟回給真實物件。

這個設計造成兩個結果：

1. 對 `[pscustomobject]@{}` 而言，因為 `PSCustomObject` 本身沒覆寫 `Equals`，相等比對退化成 reference equality。**永遠不可能用 `Equals` 比兩個 PSCustomObject 是否「內容相同」**。
2. `Copy()` 的「淺」是雙重的：PSObject 視圖層做了淺複製、BaseObject 對引用型別欄位也只是 reference 共用。「我 Copy 完之後改 `$b.Tags` 結果 `$a` 也被改」這個經典 bug 不是 bug，是 API 寫得很清楚的合約：要深複製請自己做（典型捷徑：`ConvertTo-Json -Depth 10 | ConvertFrom-Json`，代價是型別會掉、`DateTime` 會變字串）。

實務原則：**比較內容用 `Compare-Object -Property`、要獨立的副本要自己 deep clone**。`Equals` / `Copy()` 不要寄望它們符合直覺。

## Related

- [psobject skill 總覽](../)
- [Members concept](../members/)
- [BaseObject concept](../base-object/)

## External references

- [PSObject.AsPSObject Method](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.aspsobject)
- [PSObject.Copy Method](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.psobject.copy)
- [Compare-Object 官方文件](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/compare-object)
