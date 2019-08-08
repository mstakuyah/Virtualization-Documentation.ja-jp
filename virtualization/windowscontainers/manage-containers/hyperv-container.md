---
title: Hyper-V による分離
description: Hyper-v 分離のプロセス分離コンテナーとの違いについて Explaination します。
keywords: Docker, コンテナー
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 092312848173102bec5a791f2c48fe8166e70d5f
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998329"
---
# <a name="hyper-v-isolation"></a>Hyper-V による分離

Windows コンテナーテクノロジには、コンテナー、プロセス、Hyper-v 分離の2つの異なる分離レベルが含まれています。 どちらの種類も、同じように作成、管理、関数になります。 また、作成および使用するコンテナー イメージも同じです。 違うのは、コンテナー、ホスト オペレーティング システム、およびそのホストで実行されているその他のコンテナーとの間で作成される分離レベルです。

**プロセス分離**–複数のコンテナーインスタンスは、名前空間、リソースコントロール、プロセス分離技術を介して提供される分離機能を使って、ホストで同時に実行できます。  コンテナーは、同じカーネルをホストと共に共有します。  これは、Linux でのコンテナーの動作とほぼ同じです。

**Hyper-v 分離**–複数のコンテナーインスタンスをホストで同時に実行できます。ただし、各コンテナーは、特別な仮想マシンの内部で実行されます。 これにより、各コンテナーとコンテナーのホストとの間で、カーネルレベルの分離が提供されます。

## <a name="hyper-v-isolation-examples"></a>Hyper-v 分離の例

### <a name="create-container"></a>コンテナーの作成

Docker での Hyper-v 分離コンテナーの管理は、Windows Server コンテナーの管理とほぼ同じです。 Hyper-v 分離を備えたコンテナーを作成するには、 `--isolation`パラメーターを使用して`--isolation=hyperv`設定します。

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>分離の説明

次の例は、Windows Server と Hyper-v 分離の間の分離機能の違いを示しています。

ここでは、プロセス分離コンテナーが展開されており、実行時間の長い ping プロセスがホストされています。

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

対照的に、この例では、ping プロセスと共に Hyper-v の分離されたコンテナーを開始します。

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
