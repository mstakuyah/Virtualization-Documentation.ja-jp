---
title: Windows コンテナーのバージョンの互換性
description: Windows の複数のバージョン間で、ビルドとコンテナーを実行する方法について説明します。
keywords: メタデータ, コンテナー, バージョン
author: taylorb-microsoft
ms.openlocfilehash: c744da429ed8116363437d3117ae1432d7a94f8d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574953"
---
# <a name="windows-container-version-compatibility"></a>Windows コンテナーのバージョンの互換性

Windows Server コンテナーのビルドと実行に対応した最初の Windows リリースは、Windows Server 2016 と Windows 10 Anniversary Update (いずれもバージョン 14393) でした。 これらのバージョンを使用してビルドされたコンテナーは、Windows Server Version 1709 などの新しいリリースで実行できますが、作業を開始する前にいくつかの注意点があります。

Windows コンテナーは、その機能を改善する過程で、互換性に影響を与える変更が行われています。 従来のバージョンで作成したコンテナーは、[Hyper-V による分離](../manage-containers/hyperv-container.md)を使って新しいホスト上でそのまま実行でき、同じ (従来の) カーネル バージョンを引き続き使用できます。 ただし、新しい Windows ビルドで作成されたコンテナーを実行する場合は、新しいホスト ビルドで実行する必要があります。



<table>
    <tr>
    <th style="background-color:#BBDEFB">コンテナーの OS バージョン</th>
    <th span='6' style="background-color:#DCEDC8">ホストの OS バージョン</th>
    </tr>
    <tr>
        <td/>
        <td style="background-color:#F1F8E9"><b>Windows Server 2016</b><br/>ビルド: 14393.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 1609、1703</b><br/>ビルド: 14393.*、15063.*</td>
        <td style="background-color:#F1F8E9"><b>Windows Server Version 1709</b><br/>ビルド: 16299.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 Fall Creators Update</b><br/>ビルド: 16299.*</td>
        <td style="background-color:#F1F8E9"><b>Windows Server バージョン 1803</b><br/>ビルド 17134.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 版 1803</b><br/>ビルド 17134.*</td>
        <td style="background-color:#F1F8E9"><b>Windows Server 2019</b><br/>ビルド 17763.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 版 1809</b><br/>ビルド 17763.*</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>Windows Server 2016</b><br/>ビルド: 14393.*</td>
        <td>サポート対象: <br/> `process`  または `hyperv` による分離</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>Windows Server Version 1709</b><br/>ビルド: 16299.*</td>
        <td>サポートされていません。</td>
        <td>サポートされていません</td>
        <td>サポート対象: <br/> `process`  または `hyperv` による分離</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>Windows Server バージョン 1803</b><br/>ビルド 17134.*</td>
        <td>サポートされない</td>
        <td>サポートされていません</td>
        <td>サポートされていません</td>
        <td>サポートされていません</td>
        <td>サポート対象: <br/> `process`  または `hyperv` による分離</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>Windows Server 2019</b><br/>ビルド 17763.*</td>
        <td>サポートされない</td>
        <td>サポートされていません</td>
        <td>サポートされていません</td>
        <td>サポートされていません</td>
        <td>サポートされていません</td>
        <td>サポートされていません</td>
        <td>サポート対象: <br/> `process`  または `hyperv` による分離</td>
        <td>サポート対象: <br/> `hyperv` による分離のみ</td>
    </tr>
</table>               

## <a name="matching-container-host-version-with-container-image-versions"></a>コンテナー ホストのバージョンとコンテナー イメージのバージョンを一致させる
### <a name="windows-server-containers"></a>Windows Server のコンテナー
Windows Server コンテナーとその基になっているホストは単一のカーネルを共有するため、コンテナーの基本イメージはホストの基本イメージと一致している必要があります。  バージョンが異なる場合、コンテナーは起動するかもしれませんが、完全に機能することは保証できません。 Windows オペレーティング システムは、メジャー、マイナー、ビルド、リビジョンの 4 つのレベルでバージョン管理されています (例: 10.0.14393.103)。 ビルド番号(14393 など) は、新しいバージョンの OS (Version 1709、1803、Fall Creators Update など) が公開された場合にのみ変更されます。リビジョン番号 (103 など) は、Windows 更新プログラムが適用されると更新されます。
#### <a name="build-number-new-release-of-windows"></a>ビルド番号 (Windows の新しいリリース)
Windows Server コンテナーは、コンテナー ホストとコンテナー イメージの間でビルド番号が異なると起動をブロックされます (例: 10.0.14393.* (Windows Server 2016) と 10.0.16299.* (Windows Server Version 1709))。  
#### <a name="revision-number-patching"></a>リビジョン番号 (修正プログラムの適用)
Windows Server コンテナーは、コンテナー ホストとコンテナー イメージの間でリビジョン番号が異なっても起動をブロックされません__ (例: 10.0.14393.1914 (KB4051033 が適用された Windows Server 2016) と 10.0.14393.1944 (KB4053579 が適用された Windows Server 2016))。  
Windows Server 2016 ベースのホスト/イメージの場合 – コンテナー イメージのリビジョンは、サポートされる構成でホストと一致する必要があります。  Windows Server Version 1709 以降、これは当てはまりません。ホストとコンテナー イメージのリビジョンが一致する必要はありません。  通常どおり、システムに最新のパッチおよび更新プログラムを適用しておくことをお勧めします。
#### <a name="practical-application"></a>実際の適用例
例 1: KB4041691 が適用された Windows Server 2016 をコンテナー ホストが実行している場合。  このホストに展開されている Windows Server コンテナーはすべて、10.0.14393.1770 コンテナー基本イメージに基づいている必要があります。  ホストに KB4053579 を適用した場合、サポートされた状態を維持するには、同時にコンテナー イメージも更新する必要があります。
例 2: KB4043961 が適用された Windows Server Version 1709 をコンテナー ホストが実行している場合。  このホストに展開されている Windows Server コンテナーはすべて、Windows Server Version 1709 (10.0.16299) コンテナー基本イメージに基づいている必要がありますが、ホストの KB に一致する必要はありません。  ホストに KB4054517 が適用されていれば、コンテナー イメージを更新する必要はありませんが、セキュリティの問題に十分に対応するには更新を行ってください。
#### <a name="querying-version"></a>バージョンの照会
方法 1: Version 1709 以降、cmd プロンプトと `ver` コマンドでリビジョン詳細が返されるようになりました。
```
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125] 
```
方法 2: 次のレジストリ キーを照会します: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion 例:
```
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```
または
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

基本イメージが使用しているバージョンは、Docker ハブのタグまたはイメージの説明で提供されるイメージのハッシュ テーブルで確認できます。  「[Windows 10 の更新履歴](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)」のページでは、各ビルドとリビジョンがリリースされた日付がわかります。

### <a name="hyper-v-isolation-for-containers"></a>Hyper-V によるコンテナーの分離
Windows コンテナーは、Hyper-V による分離を使用して実行することも、使用せずに実行することもできます。  Hyper-V による分離を使用すると、最適化された VM によって、コンテナーの周囲にセキュリティ保護された境界が形成されます。  標準の Windows コンテナーが、コンテナーやホストとの間でカーネルを共有するのに対し、Hyper-V によって分離されたコンテナーは、専用の Windows カーネル インスタンスを利用します。  このため、コンテナー ホストとコンテナー イメージで異なるバージョンの OS を使用できます (下の互換性マトリクスを参照)。  

Hyper-V による分離を使ってコンテナーを実行するには、docker run コマンドに `--isolation=hyperv` というタグを追加します。

## <a name="errors-from-mismatched-versions"></a>一致しないバージョンを使用した場合に発生するエラー

サポートされていない組み合わせを実行しようとすると、次のエラーが発生します。

```
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

これは次の方法で解決できます。

- 以下のいずれかのバージョンに基づいてコンテナを再構築する: `microsoft/nanoserver` または  `microsoft/windowsservercore`
- ホストがコンテナーよりも新しいバージョンの場合は、次のコマンドを使用する:  `docker run --isolation=hyperv ...`
- 同じ Windows バージョンがインストールされた別のホスト上で実行する。

## <a name="choosing-container-os-versions"></a>コンテナーの OS バージョンの選択

> 注: "最新" タグは、Windows Server 2016 の現在の [LTSC 製品](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview) で更新されます。 Windows Server Version 1709 リリースに一致するコンテナー イメージが必要な場合は、以下を参照してください。

コンテナーの OS バージョンを選択するには、目的に適したバージョンを確認する必要があります。 Windows Server Version 1709 を使っていて、それに対応する最新の修正プログラムが必要な場合は、必要なベース OS コンテナー イメージのバージョンを指定するときに、次のように ”1709” タグを使用します。

``` Dockerfile
FROM microsoft/windowsservercore:1709
...
```

ただし、Windows Server Version 1709 の特定の修正プログラムが必要な場合は、タグで KB 番号を指定できます。 たとえば、Windows Server Version 1709 の Nano Server ベース OS コンテナー イメージに KB4043961 を適用する必要がある場合は、次のように指定します。

``` Dockerfile
FROM microsoft/nanoserver:1709_KB4043961
...
```

Windows Server 2016 の Nano Server ベース OS のコンテナー イメージが必要な場合も、"最新" タグを使ってそれらのベース OS のコンテナー イメージの最新バージョンを取得できます。

``` Dockerfile
FROM microsoft/nanoserver
...
```
また上記のスキーマを使い、タグで OS バージョンを指定して、必要な修正プログラムを正確に指定することもできます。

``` Dockerfile
FROM microsoft/nanoserver:10.0.14393.1770
...
```

## <a name="matching-versions-using-docker-swarm"></a>Docker Swarm を使ってバージョンを一致させる方法

現在、Docker Swarm には、コンテナーで使われている Windows のバージョンを判断して、同じバージョンを使うホストと組み合わせる方法は組み込まれていません。 新しいコンテナーを使うようにサービスが更新された場合、それに従って正常に実行されます。

ある程度の時間にわたって、複数バージョンの Windows を実行する必要がある場合、2 つのアプローチが使用できます。  1 つは Windows ホスト上で Hyper-V による分離が常時使用されるように構成する方法、もう 1 つはラベルの制約を使う方法です。

### <a name="finding-a-service-that-wont-start"></a>起動しないサービスの検出方法

サービスが起動しない場合、`MODE` には `replicated` と表示されますが、`REPLICAS` が 0 のままになります。 問題が OS のバージョンにあるかどうかを確認するには、次のコマンドを使用します。

 `docker service ls` : サービス名を検索します。

```
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

`docker service ps <name>` : 状態と最新の試行回数を取得します。

```
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

"starting container failed" (コンテナーの起動に失敗しました) というエラーが表示された場合、次のコマンドを使ってエラーの全文を表示できます。 `docker service ps --no-trunc <container name>`


```
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

これにより、同じエラーが表示されます。 `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`


### <a name="fix---update-the-service-to-use-a-matching-version"></a>修正方法: 一致するバージョンが使用されるようにサービスを更新する

Docker Swarm を使う場合、2 つの考慮事項があります。 まず、使用する Compose ファイルに含まれているサービスが、ユーザーの作成でないイメージを使用する場合、それに従って参照を変更する必要があります。 以下に例を示します。

``` docker-compose
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

もう 1 つの考慮事項として、ポイント先のイメージがユーザーの作成したイメージである場合 (たとえば、contoso/myimage) は、次のように指定します。
``` docker-compose
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```
この場合、docker-compose 行ではなく、前のセクションで説明した方法を使って Dockerfile を変更することをお勧めします。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>軽減策: Docker Swarm で Hyper-V による分離を使用する

コンテナーごとに Hyper-V による分離をサポートするように提案が行われていますが、コードはまだ完成していません。 この提案の進行状況については、[GitHub](https://github.com/moby/moby/issues/31616) でご確認ください。 それが完成するまでの間、ホストは Hyper-V による分離を常時実行するように構成する必要があります。

これには、Docker サービスの構成を変更した後、Docker エンジンを再起動します。

1. 次のコマンドを編集します。 `C:\ProgramData\docker\config\daemon.json`
2. 次のコマンドを使って行を追加します。 `"exec-opts":["isolation=hyperv"]`

注: daemon.json ファイルは、既定では存在しません。 ディレクトリを調査して、このファイルが存在していない場合は、ファイルを作成する必要があります。 その後、次の内容をコピーします。

```JSON
{
    "exec-opts":["isolation=hyperv"]
}
```
ファイルを 閉じて保存します。 その後、Docker エンジンを再起動します。

```PowerShell
Stop-Service docker
Start-Service docker
```

サービスを再起動した後、コンテナーを起動します。 コンテナーを実行した後は、コンテナーを調べて、コンテナーの分離レベルを確認できます。

```PowerShell
docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
```

このコマンドから、"Process" または "hyperv" が返されます。 上記のように変更し、daemon.json を設定した場合は、"hyperv" が表示されます。

### <a name="mitigation---use-labels-and-constraints"></a>軽減策: ラベルと制約を使用する

**手順 1: ラベルを各ノードに追加する**

各ノードで、`OS` と `OsVersion` の 2 つのラベルを追加します。 ここではローカルでの実行を想定していますが、リモート ホスト上で変更を行ってこれらのラベルを設定することもできます。

```powershell
docker node update --label-add OS="windows" $ENV:COMPUTERNAME
docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
```

その後、`docker node inspect` を使って各ノードを調べると、追加された新しいラベルが表示されます

```
        "Spec": {
            "Labels": {
                "OS": "windows",
                "OsVersion": "10.0.16296"
            },
            "Role": "manager",
            "Availability": "active"
        }
```

**手順 2: サービスの制約を追加する**

各ノードにラベルを設定した後、サービスの配置を決定する制約を更新することができます。 以下の例で、"contoso_service" を実際のサービス名と置き換えてください。

```
docker service update \
    --constraint-add "node.labels.OS == windows" \
    --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
    contoso_service
```

これにより、コードが実行できる場所が指定され、制限が課されます。

サービスの制約の使用方法について詳しくは、サービスの作成に関する[リファレンス](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)をご覧ください。


## <a name="matching-versions-using-kubernetes"></a>Kubernetes を使用したバージョンの一致

同じ問題は、Kubernetes でポッドがスケジュールされているときにも発生することがあります。 これも同様の方法を使って回避できます。

- 開発環境と運用環境で同じ OS バージョンに基づいてコンテナーを再構築する。詳しくは、上記の「**コンテナーの OS バージョンの選択**」をご覧ください。
- Windows Server 2016 と Windows Server Version 1709 の両方のノードが同じクラスター上にある場合は、ラベルとノード セレクターを使って、互換性のあるノードにポッドがスケジュールされていることを確認する。
- OS のバージョンに基づいて別のクラスターを使用する。


### <a name="finding-pods-failed-on-os-mismatch"></a>OS の不一致が原因で失敗したポッドの検索

次の例の展開では、OS バージョンが一致していないノード上でポッドがスケジュールされ、しかも Hyper-V による分離が無効です。 同じエラーは、`kubectl describe pod <podname>` によって表示されたイベントにも存在します。 複数回の試行後、多くの場合、ポッドの状態が次のようになります。 `CrashLoopBackOff`

```
$ kubectl -n plang describe po fabrikamfiber.web-789699744-rqv6p

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


### <a name="mitigation---using-node-labels--nodeselector"></a>軽減策: ノード ラベルとノード セレクターを使用する

`kubectl get node` を使うと、すべてのノードの一覧が表示されます。詳細を表示するには、`kubectl describe node <nodename>` を使います。 

この例では、2 つの Windows ノードが存在し、それぞれ異なるバージョンが実行されています。

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


1. システム情報から各ノード名と `Kernel Version` (カーネル バージョン) をメモします。

名前         | バージョン
-------------|--------------------------------------------------------
38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354



2. 各ノードに、`beta.kubernetes.io/osbuild` というラベルを追加します。 Hyper-V による分離を指定しない場合、Windows Server 2016 では、メジャー バージョンとマイナー バージョンの両方 (14393.1715) がサポートされている必要があります。 Windows Server Version 1709 の場合は、メジャー バージョン (16299) のみが一致すれば問題ありません。

上のサンプル コードの場合は、次のように指定します。

```
$ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


node "38519acs9010" labeled
$ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

node "38519acs9011" labeled

```

3. 次のコマンドを使ってラベルが追加されていることを確認します。 `kubectl get nodes --show-labels`

```
$ kubectl get nodes --show-labels

NAME                        STATUS                     AGE       VERSION                    LABELS
38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
```

4. 展開にノード セレクターを追加します。

`beta.kubernetes.io/os`= windows と `beta.kubernetes.io/osbuild`= 14393.* または 16299 を使って、`nodeSelector` をコンテナー仕様に追加し、コンテナーで使用されているベース OS に一致させます。

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

これで、更新された展開を使ってポッドを起動できるようになりました。 ノード セレクターも `kubectl describe pod <podname>` に表示され、追加されていることが確認できます。

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
