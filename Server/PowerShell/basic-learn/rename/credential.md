---
title: Credential
description: Rename-Item -Credential 是「語法存在但目前 provider 不支援」的典型案例；換身分執行應改用 Invoke-Command。
sidebar:
  order: 8
---

## 定位

```
Server > PowerShell > basic-learn > rename > Credential
```

## Summary

`Rename-Item` 的 `-Credential` **在目前安裝的 providers（含 FileSystem）下不被支援**。需要以其他身分或在遠端執行時，改用 `Invoke-Command`。本頁是概念辨識題，不是實作題。

## 參數屬性

| 項目                        | 值              |
| --------------------------- | --------------- |
| 名稱                        | `-Credential`   |
| 型別                        | `PSCredential`  |
| 預設值                      | 目前使用者       |
| Parameter set               | `(All)`         |
| Position                    | `Named`         |
| Mandatory                   | `False`         |
| Pipeline (by value)         | `False`         |
| Pipeline (by property name) | `True`          |

## 為什麼這頁不放實作範例

官方文件對這個參數明寫了**不支援**。在本機 FileSystem 上硬跑：

```powershell
# 不要這樣做 — 這只會學到錯誤示範
$cred = Get-Credential
Rename-Item -Path ".\file.txt" -NewName "new.txt" -Credential $cred
```

要嘛被忽略，要嘛行為與你的預期不符。學「不可用的參數該怎麼用」沒有意義，學「碰到這種情境該換哪個工具」才有意義。

## 正確方向：用 `Invoke-Command`

### 1. 本機以另一個身分執行

```powershell
$cred = Get-Credential
Invoke-Command -ComputerName localhost -Credential $cred -ScriptBlock {
    Rename-Item -Path "C:\target\file.txt" -NewName "renamed.txt"
}
```

### 2. 遠端執行

```powershell
Invoke-Command -ComputerName fileserver01 -Credential $cred -ScriptBlock {
    Rename-Item -Path "D:\share\file.txt" -NewName "renamed.txt"
}
```

身分由「執行的工作階段」決定，而不是塞給單一 cmdlet。

## 從這題學到的判讀方法

碰到任何 cmdlet 的參數，**Help 列出來 ≠ 目前情境支援**。完整判讀至少要看：

1. 參數本身的描述
2. 該 cmdlet 的 **Notes** 區塊（很常藏 provider 限制）
3. 目前 provider（`Get-PSProvider`）是否在支援範圍

漏看 Notes 是這類錯誤最常見的源頭。

## 常見錯誤

- 看到 `-Credential` 在 Help 裡就假設可用。
- 把 unsupported parameter 硬做成實作題，浪費時間在錯誤示範上。
- 想換身分執行卻一直繞著 `Rename-Item` 本身打轉，忽略「換工具」這個選項。

## 觀察

這個參數的設計意義其實大於它的實用性：它**保留了介面**讓未來或第三方 provider 可以實作，但在內建 provider 上是空殼。這種「介面預留但無實作」的模式在 PowerShell 中其實不少 — 它強迫使用者去區分「cmdlet 簽章」與「provider 行為」這兩層。

進一步的觀念：當 PowerShell 的某個動作和「身分」沾上邊，幾乎一定要往 `Invoke-Command` / `Start-Process -Credential` / Remoting Session 方向想。直接塞 `-Credential` 到單一 cmdlet，在多數場景都不是正解。

## Related

- [rename skill 總覽](../)
- [Force](./force/)（同樣展示「provider 依賴行為」的概念）

## External references

- [Rename-Item 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.management/rename-item)
- [Invoke-Command 官方文件](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/invoke-command)
- [about_Remote](https://learn.microsoft.com/zh-tw/powershell/module/microsoft.powershell.core/about/about_remote)
