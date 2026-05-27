---
title: dotnet-api
description: PowerShell 背後的 .NET API 筆記。聚焦在 System.Management.Automation 命名空間中、PowerShell 腳本層會直接接觸到的型別與行為。
---

## 範圍

PowerShell 表面上是 shell，骨子裡是 .NET。要把「為什麼 cmdlet 這樣行為」、「為什麼物件這樣顯示」、「為什麼某個寫法靜默失敗」想透，最後一定會落到 `System.Management.Automation` 這幾個型別上。

本 topic 不是「.NET API 全攻略」，而是收錄**在腳本層日常會踩到、需要靠 API 視角才解釋得清楚**的概念。

## Skills

- **[psobject](./psobject/)** — `System.Management.Automation.PSObject`：PowerShell 物件視圖的本體；理解了它，等於理解 PowerShell 的物件管線、ETS、formatter、序列化為什麼是現在這個樣子。

> 之後預期擴充：`runspaces`、`cmdletbase`、`ets-update-typedata`、`provider` 等 skill。

## 為什麼獨立成 topic

`basic-learn` 收的是「Cmdlet 使用層」，每個 concept 對應一個參數或一個動作。
`dotnet-api` 收的是「API / runtime 層」，每個 concept 對應一個型別、一個方法群、或一個 runtime 機制。兩者學習目標不同：

| Topic         | 問的問題                                      | 例子                              |
| ------------- | --------------------------------------------- | --------------------------------- |
| `basic-learn` | 怎麼用、什麼時候用、有什麼參數                | `Rename-Item -Path / -NewName`    |
| `dotnet-api`  | runtime 怎麼解釋這段腳本？為什麼會有這個行為？ | `PSObject.BaseObject` auto-unwrap |

## 外部參考

- [System.Management.Automation 命名空間](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation)
- [about_Object_Creation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_object_creation)
