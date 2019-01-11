---
title: Hyper-V コンテナー
description: HYPER-V コンテナーがプロセス コンテナーを異なる方法の説明します。
keywords: Docker, コンテナー
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: caaf4186f43c69dfbc35d04dd8909876ed082906
ms.sourcegitcommit: 4336d7617c30d26a987ad3450b048e17404c365d
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001001"
---
# <a name="hyper-v-containers"></a>Hyper-V コンテナー

**この記事は暫定的な内容であり、変更される可能性があります。** 

Windows のコンテナー テクノロジには、2 種類コンテナー、Windows Server コンテナー (プロセス コンテナー) および HYPER-V コンテナーにはが含まれています。 どちらのコンテナーの作成方法、管理方法、および機能も同じです。 また、作成および使用するコンテナー イメージも同じです。 違うのは、コンテナー、ホスト オペレーティング システム、およびそのホストで実行されているその他のコンテナーとの間で作成される分離レベルです。

**Windows Server コンテナー** – 名前空間、リソース コントロール、プロセスの分離テクノロジによって提供される分離により、ホスト上で複数のコンテナー インスタンスを同時に実行できます。  Windows Server コンテナーは、ホストとの間で、および互いの間で、同じカーネルを共有します。  これはほぼ同じで、コンテナーが Linux で実行する方法です。

**HYPER-V コンテナー** – のホスト上コンテナーの複数のインスタンスを同時に実行できる、ただし、各コンテナーは特別な仮想マシンの内部で実行します。 これは、各 Hyper-V コンテナーとコンテナー ホスト間でのカーネル レベルの分離を提供します。

## <a name="hyper-v-container-examples"></a>HYPER-V コンテナーの例

### <a name="create-container"></a>コンテナーの作成

Docker に HYPER-V コンテナーの管理は、Windows Server コンテナーの管理とほぼ同じです。 Docker に HYPER-V コンテナーを作成するには、`--isolation`パラメーターを設定する`--isolation=hyperv`します。

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>分離の説明

この例では、Windows Server と HYPER-V コンテナーの分離機能の違いを示します。 

以下の場合、Windows Server コンテナーが展開されており、実行時間の長い ping プロセスがホストされます。

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
```

`docker top` コマンドを使用すると、以下のようにコンテナー内の ping プロセスが返されます。 この例のプロセスの ID は 3964 です。

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

コンテナー ホストで `get-process` コマンドを使用して、ホストから実行中のすべての ping プロセスを返すことができます。 この例では、コンテナーの ID と一致するプロセス ID は 1 つです。 コンテナーとホストの両方で表示されるプロセスは同じです。

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

一方、この例では Hyper-V コンテナーを起動して、ping プロセスも使用します。 

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
```

同様に、`docker top` を使用して、コンテナーから実行中のプロセスを返すことができます。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

ただし、コンテナー ホストでプロセスを検索しても、ping プロセスは検出されず、エラーはスローされます。

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

最終的に、ホストで `vmwp` プロセスが表示されます。実行中の仮想マシンは、実行中のコンテナーをカプセル化し、ホストのオペレーティング システムから実行中のプロセスを保護します。

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
