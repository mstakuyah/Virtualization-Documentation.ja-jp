---
title: HCS PowerSWhell
description: "HCS PowerShell コンテナーと Windows コンテナーを使用します。"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 45144ec5-f76a-4460-abd1-9b60e47506d6
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 413b9de08d182635908bc11ce3efc7623bb440e4

---

# 管理相互運用性

**この記事は暫定的な内容であり、変更される可能性があります。** 

## すべてのコンテナーの表示

コンテナーの一覧を取得するには、`Get-ComputeProcess` コマンドを使用します。

```none
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## コンテナーの停止

コンテナーが PowerShell または Docker のいずれを使用して作成されたかに関係なく、コンテナーを停止するには、`Stop-ComputeProcess` コマンドを使用します。

> この記事の執筆時点では、`Get-Container` コマンドの使用時にコンテナーを停止として表示する場合、VMMS サービスを再起動する必要があります。

```none
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```



<!--HONumber=Jun16_HO4-->


