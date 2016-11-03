---
title: "Hyper-V コンテナー"
description: "Hyper-V コンテナーを展開します。"
keywords: "Docker, コンテナー"
author: enderb-ms
ms.date: 09/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 710391e72f42dbc43e9d3f093987277d3cd45429

---

# Hyper-V コンテナー

**この記事は暫定的な内容であり、変更される可能性があります。** 

Windows コンテナー テクノロジには、Windows Server コンテナーと Hyper-V コンテナーの 2 種類があります。 どちらのコンテナーの作成方法、管理方法、および機能も同じです。 また、作成および使用するコンテナー イメージも同じです。 違うのは、コンテナー、ホスト オペレーティング システム、およびそのホストで実行されているその他のコンテナーとの間で作成される分離レベルです。

**Windows Server コンテナー** – 名前空間、リソース コントロール、プロセスの分離テクノロジによって提供される分離により、ホスト上で複数のコンテナー インスタンスを同時に実行できます。  Windows Server コンテナーは、ホストとの間で、および互いの間で、同じカーネルを共有します。

**Hyper-V コンテナー** – ホスト上で複数のコンテナー インスタンスを同時に実行できます。ただし、各コンテナーは特殊な仮想マシン内で実行します。 これは、各 Hyper-V コンテナーとコンテナー ホスト間でのカーネル レベルの分離を提供します。

## Hyper-V コンテナー

### コンテナーの作成

Docker を使用した Hyper-V コンテナーの管理は、Windows Server コンテナーの管理の場合とほぼ同じです。 Docker を使用して Hyper-V コンテナーを作成する場合には、`--isolation=hyperv` パラメーターを使用します。

```none
docker run -it --isolation=hyperv nanoserver cmd
```

### 分離の説明

この例では、Windows Server コンテナーと Hyper-V コンテナーの分離機能を区別します。 

以下の場合、Windows Server コンテナーが展開されており、実行時間の長い ping プロセスがホストされます。

```none
docker run -d windowsservercore ping localhost -t
```

`docker top` コマンドを使用すると、以下のようにコンテナー内の ping プロセスが返されます。 この例のプロセスの ID は 3964 です。

```none
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

コンテナー ホストで `get-process` コマンドを使用して、ホストから実行中のすべての ping プロセスを返すことができます。 この例では、コンテナーの ID と一致するプロセス ID は 1 つです。 コンテナーとホストの両方で表示されるプロセスは同じです。

```none
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

一方、この例では Hyper-V コンテナーを起動して、ping プロセスも使用します。 

```none
docker run -d --isolation=hyperv nanoserver ping -t localhost
```

同様に、`docker top` を使用して、コンテナーから実行中のプロセスを返すことができます。

```none
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

ただし、コンテナー ホストでプロセスを検索しても、ping プロセスは検出されず、エラーはスローされます。

```none
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

最終的に、ホストで `vmwp` プロセスが表示されます。実行中の仮想マシンは、実行中のコンテナーをカプセル化し、ホストのオペレーティング システムから実行中のプロセスを保護します。

```none
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```



<!--HONumber=Oct16_HO4-->


