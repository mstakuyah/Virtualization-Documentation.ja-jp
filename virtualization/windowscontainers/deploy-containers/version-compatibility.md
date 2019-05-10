---
title: Windows コンテナーのバージョンの互換性
description: Windows の複数のバージョン間で、ビルドとコンテナーを実行する方法について説明します。
keywords: メタデータ, コンテナー, バージョン
author: taylorb-microsoft
ms.openlocfilehash: 64b6b400e12060b86594b90474fdedd73dfef45e
ms.sourcegitcommit: 561eaf94c0c0698d43228ebfcd316a7fcd835a59
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/09/2019
ms.locfileid: "9622787"
---
# <a name="windows-container-version-compatibility"></a>Windows コンテナーのバージョン互換性

Windows Server 2016 と Windows 10 の記念日の更新プログラム (14393 の両方のバージョン) を作成してコンテナーの Windows Server を実行する可能性のある最初の Windows リリースをでした。 これらのバージョンを使用してビルドされたコンテナーは、Windows Server Version 1709 などの新しいリリースで実行できますが、作業を開始する前にいくつかの注意点があります。

Windows コンテナーは、その機能を改善する過程で、互換性に影響を与える変更が行われています。 古いコンテナーでは、 [HYPER-V 分離](../manage-containers/hyperv-container.md)の新しいホスト上と同じで実行され、同じ (古い) カーネル バージョンが使用されます。 ただし、新しい Windows ビルドに基づいてコンテナーを実行する場合にのみで実行できます新しいホスト ビルドします。

|コンテナーの OS バージョン|ホストの OS バージョン|互換性|
|---|---|---|
|Windows Server 2016<br>ビルド: 14393.* |Windows Server 2016<br>ビルド: 14393.* |サポート`process`または`hyperv`分離|
|Windows Server 2016<br>ビルド: 14393.* |Windows Server Version 1709<br>ビルド: 16299.* |のみをサポートします`hyperv`分離|
|Windows Server 2016<br>ビルド: 14393.* |Windows 10 Fall Creators Update<br>ビルド: 16299.* |のみをサポートします`hyperv`分離|
|Windows Server 2016<br>ビルド: 14393.* |Windows Server バージョン 1803<br>ビルド 17134.* |のみをサポートします`hyperv`分離|
|Windows Server 2016<br>ビルド: 14393.* |Windows 10 バージョン 1803<br>ビルド 17134.* |のみをサポートします`hyperv`分離|
|Windows Server 2016<br>ビルド: 14393.* |Windows Server 2019<br>ビルド 17763.* |のみをサポートします`hyperv`分離|
|Windows Server 2016<br>ビルド: 14393.* |Windows 10、バージョン 1809<br>ビルド 17763.* |のみをサポートします`hyperv`分離|
|Windows Server バージョン 1709<br>ビルド: 16299.* |Windows Server 2016<br>ビルド: 14393.* |サポートされない|
|Windows Server バージョン 1709<br>ビルド: 16299.* |Windows Server バージョン 1709<br>ビルド: 16299.* |サポート`process`または`hyperv`分離|
|Windows Server バージョン 1709<br>ビルド: 16299.* |Windows 10 Fall Creators Update<br>ビルド: 16299.* |のみをサポートします`hyperv`分離|
|Windows Server バージョン 1709<br>ビルド: 16299.* |Windows Server、バージョン 1803<br>ビルド 17134.* |のみをサポートします`hyperv`分離|
|Windows Server バージョン 1709<br>ビルド: 16299.* |Windows 10 バージョン 1803<br>ビルド 17134.* |のみをサポートします`hyperv`分離|
|Windows Server バージョン 1709<br>ビルド: 16299.* |Windows Server 2019<br>ビルド 17763.* |のみをサポートします`hyperv`分離|
|Windows Server バージョン 1709<br>ビルド: 16299.* |Windows 10、バージョン 1809<br>ビルド 17763.* |のみをサポートします`hyperv`分離|
|Windows Server、バージョン 1803<br>ビルド 17134.* |Windows Server 2016<br>ビルド: 14393.* |サポートされない|
|Windows Server、バージョン 1803<br>ビルド 17134.* |Windows Server バージョン 1709<br>ビルド: 16299.* |サポートされていません|
|Windows Server、バージョン 1803<br>ビルド 17134.* |Windows 10 Fall Creators Update<br>ビルド: 16299.* |サポートされていません|
|Windows Server、バージョン 1803<br>ビルド 17134.* |Windows Server、バージョン 1803<br>ビルド 17134.* |サポート`process`または`hyperv`分離|
|Windows Server、バージョン 1803<br>ビルド 17134.* |Windows 10 バージョン 1803<br>ビルド 17134.* |のみをサポートします`hyperv`分離|
|Windows Server、バージョン 1803<br>ビルド 17134.* |Windows Server 2019<br>ビルド 17763.* |のみをサポートします`hyperv`分離|
|Windows Server、バージョン 1803<br>ビルド 17134.* |Windows 10、バージョン 1809<br>ビルド 17763.* |のみをサポートします`hyperv`分離|
|Windows Server 2019<br>ビルド 17763.* |Windows Server 2016<br>ビルド: 14393.* |サポートされない|
|Windows Server 2019<br>ビルド 17763.* |Windows Server バージョン 1709<br>ビルド: 16299.* |サポートされていません
|Windows Server 2019<br>ビルド 17763.* |Windows 10 Fall Creators Update<br>ビルド: 16299.* |サポートされていません|
|Windows Server 2019<br>ビルド 17763.* |Windows Server、バージョン 1803<br>ビルド 17134.* |サポートされない|
|Windows Server 2019<br>ビルド 17763.* |Windows 10 バージョン 1803<br>ビルド 17134.* |サポートされない|
|Windows Server 2019<br>ビルド 17763.* |Windows Server 2019<br>ビルド 17763.* |サポート`process`または`hyperv`分離|
|Windows Server 2019<br>ビルド 17763.* |Windows 10、バージョン 1809<br>ビルド 17763.* |のみをサポートします`hyperv`分離|

## <a name="matching-container-host-version-with-container-image-versions"></a>コンテナーとコンテナーの画像のバージョンのバージョンのホストに一致します。

### <a name="windows-server-containers"></a>Windows Server コンテナー

Windows Server コンテナーと、基になるホストは、単一のカーネルを共有するためのホストのコンテナーの基本イメージが一致する必要があります。 バージョンが異なる場合、コンテナーが起動しは完全機能的保証されません。 Windows オペレーティング システムには、バージョン管理の 4 つのレベル: メジャー、マイナー、ビルドとリビジョンします。 たとえば、10 のメジャー バージョン、0 のマイナー バージョン、103 の 14393、数、ビルドと、変更履歴、バージョン 10.0.14393.103 があります。 OS の新しいバージョン バージョン 1709、1803、秋作成者の更新] など、公開してときにのみ、ビルド番号が変更されます。 リビジョン番号は、Windows 更新プログラムが適用されると更新されます。

#### <a name="build-number-new-release-of-windows"></a>ビルド番号 (Windows の新しいリリース版)

起動しますコンテナー ホストとコンテナーのイメージのビルド番号が異なる場合は、Windows Server コンテナーがブロックされています。 たとえば、コンテナー ホストがバージョン 10.0.14393.* はとき (Windows Server 2016)、コンテナーのイメージはバージョン 10.0.16299.* (Windows Server バージョン 1709) しますコンテナーが起動せずします。  

#### <a name="revision-number-patching"></a>改訂数と (パッチ)

Windows Server コンテナーはコンテナー ホストとコンテナーの画像の変更履歴数が異なるときに、開始禁止されています。 たとえば、コンテナー ホストされた 10.0.14393.1914 (適用 KB4051033 と Windows Server 2016)、コンテナーの画像は、バージョン 10.0.14393.1944 (適用 KB4053579 と Windows Server 2016) 場合は、[イメージは起動する場合でも、変更履歴数値は異なります。

ホストの Windows Server 2016 ベースまたは画像は、コンテナーの画像の変更履歴がサポートされている構成でホストと一致する必要があります。 ただし、ホストまたは Windows Server バージョン 1709 以降を使用して、画像、このルールが適用されません、ホストとコンテナーの画像が必要な変更履歴と一致するにありません。 システムの最新の更新プログラムと更新プログラムが最新の状態をお勧めします。

#### <a name="practical-application"></a>実用的なアプリケーション

例 1: しますコンテナー host が実行されて Windows Server 2016 適用 KB4041691 とします。 バージョン 10.0.14393.1770 コンテナーの基本イメージに基づいて、任意の Windows Server コンテナーがこのホストに配置する必要があります。 ホスト コンテナーに KB4053579 を適用する場合も、ホスト コンテナー サポートしているかどうかを確認する画像を更新する必要があります。

例 2: しますコンテナー host が実行されて 1709 のバージョンの Windows Server 適用 KB4043961 とします。 このホストに展開された任意の Windows Server コンテナーでは、Windows Server のバージョン (10.0.16299) 1709 コンテナー基本イメージ、に基づいて必要がありますが、サポート技術情報のホストに一致する必要はありません。 ホストに KB4054517 を適用する場合は、コンテナーの画像はサポートされますは、潜在的なセキュリティ上の問題に対処する更新することをお勧めします。

#### <a name="querying-version"></a>バージョンの照会

方法 1: は、バージョン 1709 の導入 cmd プロンプトと**バージョン**のコマンドはこれで、変更履歴の詳細を返します。

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

方法 2: クエリの次のレジストリ キー: を探して

次に例を示します。

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

どのようなバージョンを確認するには、基本イメージで使用するには、Docker ハブまたは画像の説明で提供されている画像ハッシュ テーブルにノート シールを確認します。 [Windows 10 の履歴を更新する](https://support.microsoft.com/help/12387/windows-10-update-history)] ページでは、各ビルドと変更履歴のリリース時に一覧表示します。

### <a name="hyper-v-isolation-for-containers"></a>コンテナーの HYPER-V 分離

HYPER-V 分離の有無には、Windows のコンテナーを実行できます。 Hyper-V による分離を使用すると、最適化された VM によって、コンテナーの周囲にセキュリティ保護された境界が形成されます。 コンテナーとホストのカーネルを共有する標準的な Windows コンテナーとは異なりは、各 HYPER-V 分離コンテナーは、Windows カーネルの独自のインスタンスを持ちます。 つまり、コンテナー ホストと画像にさまざまな OS のバージョンを持つことができます (詳細については、次の互換性マトリックスを参照してください)。  

Hyper-V による分離を使ってコンテナーを実行するには、docker run コマンドに `--isolation=hyperv` というタグを追加します。

## <a name="errors-from-mismatched-versions"></a>一致しないバージョンを使用した場合に発生するエラー

サポートを実行しようとすると、次のエラーが表示されます。

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

このエラーを解決する 3 つの方法があります。

- 適切なバージョンのに基づいて、コンテナーを再構築`mcr.microsoft.com/windows/nanoserver`または `mcr.microsoft.com/windows/servercore`
- 実行する場合は、新しいホスト場合は、 **docker 実行--分離 =... hyperv**
- Windows のバージョンが同じで別のホストにコンテナーを実行してください。

## <a name="choose-which-container-os-version-to-use"></a>使用するコンテナー OS のバージョンを選ぶ

>[!NOTE]
>2019 年 4 月 16日における「最新」ノート シールが不要になった発行または[Windows OS コンテナーの画像を基](https://hub.docker.com/_/microsoft-windows-base-os-images)に維持します。 特定のタグ、または次のリポジトリから画像を参照するときに宣言してください。

コンテナーを使用する必要があります。 どちらのバージョンを知る必要があります。 たとえば、1809 のバージョンの Windows Server を目的コンテナー OS としてしてに最新の更新プログラムを使用する場合は、タグを使用する必要があります`1809`を指定する基本 OS コンテナーの画像のバージョン、次のようにしている場合。

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

ただし、Windows Server 1809 のバージョンの特定の修正プログラムを実行する場合に、タグにサポート技術情報の数を指定できます。 たとえば、Windows Server のバージョンが 1809、KB4493509 の適用から Nano サーバー ベースの OS コンテナー イメージを取得する指定する場合次のようにします。

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

タグの OS のバージョンを指定して、以前に使用してスキーマをする必要がある正確な更新プログラムを指定することもできます。

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Windows Server 2019 と Windows Server 2016 に基づいて Server Core 基本イメージは、[長期的なサービス チャンネル (LTSC)](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc)を解放します。 LTSC を設定できる場合の Windows Server 2019 Server Core 画像のコンテナー OS としての最新の更新プログラムを使用する、次のようにリリースします。

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Docker Swarm を使ってバージョンを一致させる方法

Docker 群れには、同じバージョンのホストにコンテナーを使用している Windows のバージョンと一致する組み込みの方法が現在ありません。 新しいコンテナーを使用するサービスを更新する場合、正常に実行されます。

長期間の複数のバージョンの Windows を実行する必要がある場合は、必要な 2 つの方法: HYPER-V 分離を使用するか、ラベルの制約を使って常にするには、Windows ホストを構成するか。

### <a name="finding-a-service-that-wont-start"></a>起動しないサービスの検出方法

サービスが起動せず、表示されますが、`MODE`が`replicated`が`REPLICAS`は困った時は 0 にします。 OS のバージョンが問題を確認するには、次のコマンドを実行します。

サービス名を検索する**docker サービス ls**を実行します。

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

最新の試行の状態を取得する**docker サービス ps (サービス名)** を実行します。

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

表示する場合は`starting container failed: ...`、 **docker サービス ps--なしの trunc (コンテナー名)** を使って完全なエラーが表示されることができます。

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

これは、同じエラーとして`CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`します。

### <a name="fix---update-the-service-to-use-a-matching-version"></a>修正方法: 一致するバージョンが使用されるようにサービスを更新する

Docker Swarm を使う場合、2 つの考慮事項があります。 作成しているファイルを作成した画像を使用しているサービスがある場合は、それに応じて、参照を更新するされます。 次に例を示します。

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

その他の考慮事項が画像をポイントして、自分で作成した 1 つ (たとえば、contoso/myimage)。

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

この例では、docker-compose 行ではなく、その dockerfile を変更するのに[バージョンの不一致のエラー](#errors-from-mismatched-versions)で説明する方法を使用する必要があります。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>軽減策: Docker Swarm で Hyper-V による分離を使用する

コンテナーごとに HYPER-V 分離の使用をサポートするための提案ありますが、まだコードは行われません。 この提案の進行状況については、[GitHub](https://github.com/moby/moby/issues/31616) でご確認ください。 それが完成するまでの間、ホストは Hyper-V による分離を常時実行するように構成する必要があります。

これには、Docker サービスの構成を変更した後、Docker エンジンを再起動します。

1. 次のコマンドを編集します。 `C:\ProgramData\docker\config\daemon.json`
2. 次のコマンドを使って行を追加します。 `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >Daemon.json ファイルは、既定ではありません。 ディレクトリを調査して、このファイルが存在していない場合は、ファイルを作成する必要があります。 次のようにコピーするされます。

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. 終了し、ファイルを保存、再起動 docker エンジン PowerShell で次のコマンドレットを実行しています。

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. サービスを再起動した後、コンテナーを起動します。 を実行するいると、次のコマンドレットのコンテナーを検査してコンテナーの分離レベルを確認することができます。

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

このコマンドから、"Process" または "hyperv" が返されます。 上記のように変更し、daemon.json を設定した場合は、"hyperv" が表示されます。

### <a name="mitigation---use-labels-and-constraints"></a>軽減策: ラベルと制約を使用する

バージョンと一致するラベルと制約の使用方法を示します。

1. 各ノードにラベルを追加します。

    各ノードの 2 つのラベルを追加する:`OS`と`OsVersion`します。 ここではローカルでの実行を想定していますが、リモート ホスト上で変更を行ってこれらのラベルを設定することもできます。

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    その後、新しく追加されたラベルを表示する必要がある**docker ノードを検査する**コマンドを実行しているを確認することができます。

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. サービスの制約を追加します。

    これで、各ノードのラベルが付けられた、サービスの配置を決定する制約条件を更新できます。 次の例では、実際のサービスの名前を持つ"contoso_service"に置き換えます。

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    これにより、コードが実行できる場所が指定され、制限が課されます。

サービスの制約を使用する方法の詳細については、[参照を作成するサービス](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)をオンにします。

## <a name="matching-versions-using-kubernetes"></a>Kubernetes を使用したバージョンの一致

[Docker 選ばを使用して、一致するバージョン](#matching-versions-using-docker-swarm)に記載された同じの問題は、ポッドが Kubernetes でスケジュールされている場合に発生します。 ような方法では、この問題を回避できます。

- 開発、運用環境で同じ OS のバージョンに基づくコンテナーを再構築します。 学習する方法については、[どのコンテナーに使用する OS のバージョンの選択](#choose-which-container-os-version-to-use)を参照してください。
- ノードのラベルと nodeSelectors を使用して、Windows Server と Windows Server 2016 の両方のバージョン 1709 ノードの同じクラスターの場合、互換性のあるノードでポッドがスケジュールされていることを確認するには
- OS のバージョンに基づいて別のクラスターを使用する。

### <a name="finding-pods-failed-on-os-mismatch"></a>OS の不一致が原因で失敗したポッドの検索

この例では、展開にはと一致しない OS のバージョンでは、して HYPER-V 分離を有効になっていないノードでスケジュールしたポッドが含まれています。

同じエラーは、`kubectl describe pod <podname>` によって表示されたイベントにも存在します。 を数回試みた後ポッド状態は`CrashLoopBackOff`します。

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>リスク軽減のノードのラベルと nodeSelector を使用します。

すべてのノードの一覧を取得**kubectl ノードを取得する**を実行します。 その後、その他の情報に**kubectl 説明ノード (ノード名)** を実行できます。

次の例では、2 つの Windows ノードがさまざまなバージョンを実行しています。

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

みましょうこの例を使用して、バージョンと一致する方法を説明します。

1. 各ノード名の点に注意して`Kernel Version`システム情報] の [します。

    ここでは、次のような情報になります。

    名前         | バージョン
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. 各ノードに、`beta.kubernetes.io/osbuild` というラベルを追加します。 Windows Server 2016 には、HYPER-V 分離せずにサポートされる両方メジャー バージョンとマイナー バージョン (この例では 14393.1715) が必要があります。 Windows Server 1709 のバージョンには、一致するようにメジャー バージョン (この例では 16299) のみ必要があります。

    この例では次のようなラベルを追加するコマンドが表示されます。

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. **Kubectl 取得--ノードのラベルを表示する]** を実行して、ラベルがあることを確認します。

    この例では、次のような結果になります。

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. 展開するには、ノード セレクターを追加します。 この例の場合を追加する`nodeSelector`でコンテナー仕様に`beta.kubernetes.io/os`= windows と`beta.kubernetes.io/osbuild`= 14393.* または 16299 コンテナーで使用する基本 OS に一致するようにします。

    Windows Server 2016 向けにビルドされたコンテナーを実行するためのサンプル コード全体を次に示します。

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    これで、更新された展開を使ってポッドを起動できるようになりました。 ノード セレクターにも表示`kubectl describe pod <podname>`、追加されたことを確認するには、そのコマンドを実行できるようにします。

    この例の出力は、次のとおりです。

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
