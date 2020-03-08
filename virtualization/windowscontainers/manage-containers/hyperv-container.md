---
title: 分離モード
description: Hyper-v の分離とプロセス分離コンテナーとの違いについて説明します。
keywords: Docker, コンテナー
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 362805fa230f461414ccc53643644f6c1b3474a8
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853956"
---
# <a name="isolation-modes"></a>分離モード

Windows コンテナーには、`process` と `Hyper-V` 分離という2つの異なるランタイム分離モードが用意されています。 両方の分離モードで実行されているコンテナーは、同じように作成、管理、および機能します。 また、作成および使用するコンテナー イメージも同じです。 分離モードの違いは、コンテナー、ホストオペレーティングシステム、およびそのホストで実行されている他のすべてのコンテナーとの間に作成される分離の程度です。

## <a name="process-isolation"></a>プロセスの分離

これは、コンテナーの "従来の" 分離モードであり、「 [Windows コンテナーの概要](../about/index.md)」で説明されています。 プロセス分離を使用すると、名前空間、リソースコントロール、およびプロセス分離テクノロジによって提供される分離により、特定のホストで複数のコンテナーインスタンスを同時に実行できます。 このモードで実行すると、コンテナーは同じカーネルをホストと相互に共有します。  これは、Linux コンテナーの実行方法とほぼ同じです。

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V による分離
この分離モードでは、ホストとコンテナーのバージョン間でセキュリティが強化され、より広範な互換性が提供されます。 Hyper-v の分離を使用すると、ホスト上で複数のコンテナーインスタンスを同時に実行できます。ただし、各コンテナーは、高度に最適化された仮想マシンの内部で実行され、独自のカーネルを効果的に取得します。 仮想マシンの存在により、各コンテナーとコンテナーホストの間にハードウェアレベルの分離が提供されます。

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>分離の例

### <a name="create-container"></a>コンテナーの作成

Docker を使用した Hyper-v 分離コンテナーの管理は、プロセス分離コンテナーの管理とほぼ同じです。 Hyper-v の分離を完全に使用してコンテナーを作成するには、`--isolation` パラメーターを使用して `--isolation=hyperv`を設定します。

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Docker を介してプロセス分離を持つコンテナーを作成するには、`--isolation` パラメーターを使用して `--isolation=process`を設定します。

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows Server で実行されている windows コンテナーは、既定でプロセス分離を使用して実行されます。 Windows 10 Pro および Enterprise で実行されている windows コンテナーは、Hyper-v 分離で実行されるように既定で設定されています。 Windows 10 10 月2018更新プログラム以降では、Windows 10 Pro またはエンタープライズホストを実行するユーザーは、プロセス分離を使用して Windows コンテナーを実行できます。 ユーザーは、`--isolation=process` フラグを使用してプロセスの分離を直接要求する必要があります。

> [!WARNING]
> Windows 10 Pro および Enterprise でのプロセス分離を使用した実行は、開発/テストを目的としています。 ホストで Windows 10 build 17763 + が実行されている必要があります。また、エンジン18.09 以降の Docker バージョンが必要です。
> 
> 運用環境の展開では、引き続き Windows Server をホストとして使用する必要があります。 Windows 10 Pro および Enterprise でこの機能を使用することにより、ホストとコンテナーのバージョンタグが一致することを確認する必要があります。そうしないと、コンテナーの起動に失敗したり、未定義の動作が発生したりする可能性があります。

### <a name="isolation-explanation"></a>分離の説明

この例では、プロセスと Hyper-v 分離の分離機能の違いについて説明します。

ここでは、プロセス分離コンテナーが配置され、実行時間の長い ping プロセスがホストされます。

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
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

これに対して、この例では、ping プロセスも含めて、Hyper-v 対応のコンテナーを起動します。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

同様に、`docker top` を使用して、コンテナーから実行中のプロセスを返すことができます。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

ただし、コンテナーホストでプロセスを検索するときに、ping プロセスが見つからず、エラーがスローされます。

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
