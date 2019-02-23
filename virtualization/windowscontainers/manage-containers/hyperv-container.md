---
title: Hyper-V による分離
description: コンテナーを分離する説明のプロセスから分離の HYPER-V の違いについて説明します。
keywords: Docker, コンテナー
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: db0f8c45c1cdb6617e4c347251284509e2a7d3bc
ms.sourcegitcommit: 914e0dd1168daf1d2b0f22bd011035016cc08baf
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/23/2019
ms.locfileid: "9099340"
---
# <a name="hyper-v-isolation"></a>Hyper-V による分離

Windows のコンテナー テクノロジには、2 つの異なるコンテナー、プロセス、HYPER-V 分離の分離レベルが含まれています。 両方の種類を作成、管理し、同じように機能します。 また、作成および使用するコンテナー イメージも同じです。 違うのは、コンテナー、ホスト オペレーティング システム、およびそのホストで実行されているその他のコンテナーとの間で作成される分離レベルです。

**プロセス分離**– 分離のホストにインスタンスを同時に実行できる複数のコンテナーを通じて提供名前空間、リソース管理、およびプロセスの分離テクノロジされます。  コンテナーでは、ホストと他のと同じカーネルを共有します。  これはほぼ同じで、コンテナーが Linux で実行する方法です。

**HYPER-V 分離**: のホスト上コンテナーの複数のインスタンスを同時に実行できる、ただし、各コンテナーは特別な仮想マシンの内部で実行します。 これは、各コンテナーとコンテナー ホスト間カーネル レベルの分離を提供します。

## <a name="hyper-v-isolation-examples"></a>HYPER-V 分離の例

### <a name="create-container"></a>コンテナーの作成

Docker に HYPER-V 分離コンテナーの管理は、Windows Server コンテナーの管理とほぼ同じです。 HYPER-V 分離コンテナーを作成するのには完全な Docker を使用して、`--isolation`パラメーターを設定する`--isolation=hyperv`します。

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>分離の説明

この例では、Windows Server と HYPER-V コンテナーの分離機能の違いを示します。 

ここでは、分離コンテナーが展開されていると、長時間 ping プロセスをホストするプロセスです。

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

コントラスト] には、この例は、ping プロセスも HYPER-V 分離コンテナーを起動します。 

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
