---
title: 分離モード
description: Hyper-v 分離がプロセス分離コンテナーとどのように異なるかについて説明します。
keywords: Docker, コンテナー
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161729"
---
# <a name="isolation-modes"></a>分離モード

Windows コンテナーには`process` 、実行時分離の2つ`Hyper-V`のモードと分離が用意されています。 両方の分離モードで実行されているコンテナーは、同じように作成、管理、関数になります。 また、作成および使用するコンテナー イメージも同じです。 分離モードの違いは、コンテナー、ホストオペレーティングシステム、およびそのホストで実行されている他のすべてのコンテナーの間に作成される分離レベルです。

## <a name="process-isolation"></a>プロセス分離

これはコンテナーの "従来の" 分離モードであり、 [Windows コンテナーの概要](../about/index.md)で説明されているものです。 プロセス分離を使うと、名前空間、リソースコントロール、プロセス分離技術を通じて分離された分離機能を使用して、複数のコンテナーインスタンスを指定したホストで同時に実行できます。 このモードで実行されている場合、コンテナーは同じカーネルをホストと共に共有します。  これは、Linux コンテナーの実行方法とほぼ同じです。

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V による分離
この分離モードでは、ホストとコンテナーのバージョンのセキュリティと互換性が強化されています。 Hyper-v 分離では、複数のコンテナーインスタンスがホストで同時に実行されます。ただし、各コンテナーは、高度に最適化された仮想マシンの内部で実行されるため、独自のカーネルを実質的に取得できます。 仮想マシンの存在は、コンテナーホストと共に、各コンテナーとの間でハードウェアレベルで分離されます。

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>分離の例

### <a name="create-container"></a>コンテナーの作成

Docker での Hyper-v の分離されたコンテナーの管理は、プロセス分離コンテナーの管理とほぼ同じです。 Hyper-v 分離を備えたコンテナーを作成するには、 `--isolation`パラメーターを使用して`--isolation=hyperv`設定します。

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

プロセス分離の完全な Docker でコンテナーを作成するに`--isolation`は、パラメーター `--isolation=process`を使用して設定します。

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows Server で実行されている windows コンテナーは、プロセス分離を使用して実行されます。 Windows 10 Pro で実行される windows コンテナーと、Hyper-v 分離を使って実行されるエンタープライズの既定 Windows 10 年 2018 10 月の更新プログラムから、Windows 10 Pro または Enterprise host を実行しているユーザーは、プロセス分離を使用して Windows コンテナーを実行できます。 ユーザーは、フラグを使ってプロセスの`--isolation=process`分離を直接要求する必要があります。

> [!WARNING]
> Windows 10 Pro と Enterprise でプロセス分離を実行することは、開発/テストを目的としています。 ホストで Windows 10 ビルド 17763 + が実行されている必要があります。また、エンジン18.09 以降を搭載した Docker バージョンが必要です。
> 
> 引き続き Windows Server を運用展開用のホストとして使用する必要があります。 Windows 10 Pro と Enterprise でこの機能を使用することによって、ホストとコンテナーのバージョンタグが一致していることを確認する必要があります。そうでないと、コンテナーが起動しないか、未定義の動作が発生する可能性があります。

### <a name="isolation-explanation"></a>分離の説明

次の例は、プロセスと Hyper-v 分離の間の分離機能の違いを示しています。

ここでは、プロセス分離されたコンテナーが展開されており、実行時間の長い ping プロセスがホストされています。

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

対照的に、この例では、ping プロセスと共に、Hyper-v と、それに対応するコンテナーを開始しています。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

同様に、`docker top` を使用して、コンテナーから実行中のプロセスを返すことができます。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

ただし、コンテナーホストでプロセスを検索すると、ping プロセスが見つからず、エラーがスローされます。

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
