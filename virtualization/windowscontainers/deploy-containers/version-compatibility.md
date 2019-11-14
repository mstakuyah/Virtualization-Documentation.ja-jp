---
title: Windows コンテナーのバージョンの互換性
description: Windows の複数のバージョン間で、ビルドとコンテナーを実行する方法について説明します。
keywords: メタデータ, コンテナー, バージョン
author: taylorb-microsoft
ms.openlocfilehash: 1f068cd011b2172e75c240d566473ccab25d984a
ms.sourcegitcommit: 48ede8f27e089926b5b867037f31d14500af84ce
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2019
ms.locfileid: "10296032"
---
# <a name="windows-container-version-compatibility"></a>Windows コンテナーバージョンの互換性

Windows Server 2016 および Windows 10 記念日更新プログラム (両方ともバージョン 14393) は、Windows Server コンテナーを構築して実行できる最初の Windows のリリースです。 これらのバージョンを使用して作成されたコンテナーは、新しいリリースで実行できますが、作業を開始する前に知っておく必要があることがいくつかあります。

Windows コンテナーは、その機能を改善する過程で、互換性に影響を与える変更が行われています。 古いコンテナーは、 [hyper-v 分離](../manage-containers/hyperv-container.md)を使用して、新しいホストでも同じように動作し、同じ (古い) カーネルバージョンを使います。 ただし、新しい Windows ビルドに基づいてコンテナーを実行する場合は、新しいホストビルドでのみ実行できます。

## <a name="windows-server-host-os-compatibility"></a>Windows Server ホストの OS の互換性

<!-- start tab view -->
# [<a name="windows-server-version-1909"></a>Windows Server バージョン1909](#tab/windows-server-1909)

|コンテナーベースイメージ OS バージョン|Hyper-v 分離のサポート|プロセス分離のサポート|
|---|:---:|:---:|
|Windows Server バージョン1909|&#10004;|&#10004;|
|Windows Server バージョン1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# [<a name="windows-server-version-1903"></a>Windows Server バージョン1903](#tab/windows-server-1903)

|コンテナーベースイメージ OS バージョン|Hyper-v 分離のサポート|プロセス分離のサポート|
|---|:---:|:---:|
|Windows Server バージョン1909|&#10060;|&#10060;|
|Windows Server バージョン1903|&#10004;|&#10004;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# [<a name="windows-server-2019"></a>Windows Server 2019](#tab/windows-server-2019)

|コンテナーベースイメージ OS バージョン|Hyper-v 分離のサポート|プロセス分離のサポート|
|---|:---:|:---:|
|Windows Server バージョン1909|&#10060;|&#10060;|
|Windows Server バージョン1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10004;|
|Windows Server 2016|&#10004;|&#10060;|

# [<a name="windows-server-2016"></a>Windows Server 2016](#tab/windows-server-2016)

|コンテナーベースイメージ OS バージョン|Hyper-v 分離のサポート|プロセス分離のサポート|
|---|:---:|:---:|
|Windows Server バージョン1909|&#10060;|&#10060;|
|Windows Server バージョン1903|&#10060;|&#10060;|
|Windows Server 2019|&#10060;|&#10060;|
|Windows Server 2016|&#10004;|&#10004;|

---
<!-- stop tab view -->

## <a name="windows-10-host-os-compatibility"></a>Windows 10 ホストの OS の互換性

<!-- start tab view -->

# [<a name="windows-10-version-1909"></a>Windows 10 バージョン1909](#tab/windows-10-1909)

|コンテナーベースイメージ OS バージョン|Hyper-v 分離のサポート|プロセス分離のサポート|
|---|:---:|:---:|
|Windows Server バージョン1909|&#10004;|&#10060;|
|Windows Server バージョン1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# [<a name="windows-10-version-1903"></a>Windows 10 バージョン1903](#tab/windows-10-1903)

|コンテナーベースイメージ OS バージョン|Hyper-v 分離のサポート|プロセス分離のサポート|
|---|:---:|:---:|
|Windows Server バージョン1909|&#10060;|&#10060;|
|Windows Server バージョン1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

# [<a name="windows-10-version-1809"></a>Windows 10 バージョン1809](#tab/windows-10-1809)

|コンテナーベースイメージ OS バージョン|Hyper-v 分離のサポート|プロセス分離のサポート|
|---|:---:|:---:|
|Windows Server バージョン1909|&#10060;|&#10060;|
|Windows Server バージョン1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|Windows Server 2016|&#10004;|&#10060;|

---
<!-- stop tab view -->

## <a name="matching-container-host-version-with-container-image-versions"></a>コンテナーのイメージバージョンに対応するコンテナーホストのバージョン

### <a name="windows-server-containers"></a>Windows Server コンテナー

Windows Server コンテナーと基礎となるホストが1つのカーネルを共有するため、コンテナーの基本イメージ OS バージョンはホストと一致する必要があります。 バージョンが異なる場合は、コンテナーが起動する可能性がありますが、すべての機能が保証されるわけではありません。 Windows オペレーティングシステムのバージョン管理には、メジャー、マイナー、ビルド、リビジョンの4つのレベルがあります。 たとえば、バージョン10.0.14393.103 は、10のメジャーバージョン、マイナーバージョンが0、ビルド番号14393、および103のリビジョン番号です。 ビルド番号が変更されるのは、バージョン1709、1903などの新しいバージョンの OS が公開されている場合のみです。 リビジョン番号は、Windows 更新プログラムが適用されると更新されます。

#### <a name="build-number-new-release-of-windows"></a>ビルド番号 (Windows の新しいリリース)

コンテナーホストとコンテナーイメージの間のビルド番号が異なる場合、Windows Server コンテナーはブロックされます。 たとえば、コンテナーホストがバージョン10.0.14393 の場合 (Windows Server 2016) とコンテナーイメージがバージョン 10.0.16299. * (Windows Server バージョン 1709) の場合、コンテナーは開始されません。  

#### <a name="revision-number-patching"></a>リビジョン番号 (修正プログラム)

コンテナーホストとコンテナーイメージのリビジョン番号が異なる場合、Windows Server コンテナーはブロックされません。 たとえば、コンテナーホストのバージョンが 10.0.14393.1914 (Windows Server 2016 で KB4051033 が適用されている場合)、コンテナーイメージがバージョン 10.0.14393.1944 (Windows Server 2016 で KB4053579 が適用されている場合) でも、イメージは変更できます。数値が異なります。

Windows Server 2016 ベースのホストまたはイメージの場合、コンテナーイメージのリビジョンは、サポートされている構成であるホストと一致する必要があります。 ただし、Windows Server バージョン1709以降を使っているホストまたは画像の場合、この規則は適用されず、ホストとコンテナーの画像には一致するリビジョンが必要ありません。 最新の更新プログラムと更新プログラムを使って、システムを最新の状態に維持することをお勧めします。

#### <a name="practical-application"></a>実用的なアプリケーション

例 1: コンテナーホストが、KB4041691 を適用した Windows Server 2016 を実行している。 このホストに展開される Windows Server コンテナーは、バージョン 10.0.14393.1770 container の基本イメージに基づいている必要があります。 KB4053579 をホストコンテナーに適用する場合は、そのイメージも更新して、ホストコンテナーでサポートされていることを確認する必要があります。

例 2: コンテナーホストは、KB4043961 を適用した Windows Server バージョン1709を実行しています。 このホストに展開される Windows Server コンテナーは、Windows Server バージョン 1709 (10.0.16299) コンテナーベースのイメージに基づいている必要がありますが、host KB と一致する必要はありません。 KB4054517 がホストに適用されている場合でも、コンテナーのイメージはサポートされますが、潜在的なセキュリティ上の問題に対処するために、それらを更新することをお勧めします。

#### <a name="querying-version"></a>バージョンの照会

方法 1: バージョン1709で導入された cmd プロンプトと**ver**コマンドによって、リビジョンの詳細が返されるようになりました。

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

方法 2: 次のレジストリキーをクエリします。 HKEY_LOCAL_MACHINE \Software\Microsoft\Windows NT\CurrentVersion

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

基本イメージで使用されるバージョンを確認するには、イメージの説明で示されている Docker hub または画像ハッシュテーブルのタグを確認します。 各ビルドとリビジョンがリリースされた場合は、 [Windows 10 の更新履歴](https://support.microsoft.com/help/12387/windows-10-update-history)ページに一覧表示されます。

### <a name="hyper-v-isolation-for-containers"></a>コンテナーの hyper-v 分離

Windows コンテナーは、または Hyper-v 分離を使用して、または使用せずに実行できます。 Hyper-V による分離を使用すると、最適化された VM によって、コンテナーの周囲にセキュリティ保護された境界が形成されます。 コンテナーとホストの間でカーネルを共有する標準の Windows コンテナーとは異なり、各 Hyper-v 分離コンテナーには、Windows カーネルの独自のインスタンスがあります。 つまり、コンテナーホストとイメージに異なる OS バージョンを含めることができます (詳しくは、次の互換性マトリックスをご覧ください)。  

Hyper-V による分離を使ってコンテナーを実行するには、docker run コマンドに `--isolation=hyperv` というタグを追加します。

## <a name="errors-from-mismatched-versions"></a>一致しないバージョンを使用した場合に発生するエラー

サポートされていない組み合わせを実行しようとすると、次のエラーが表示されます。

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

このエラーを解決するには、次の3つの方法があります。

- 適切なバージョンの`mcr.microsoft.com/windows/nanoserver`またはに基づいてコンテナーを再構築します。 `mcr.microsoft.com/windows/servercore`
- ホストが新しい場合は、 **docker run--分離 = hyperv...**
- 同じ Windows バージョンの別のホストでコンテナーを実行してみてください。

## <a name="choose-which-container-os-version-to-use"></a>使用するコンテナー OS のバージョンを選択する

>[!NOTE]
>2019年4月16日時点で、 [Windows ベース OS コンテナーイメージ](https://hub.docker.com/_/microsoft-windows-base-os-images)の "最新" タグは発行または維持されなくなりました。 これらの repos から画像を取得または参照するときに、特定のタグを宣言してください。

コンテナーにどのバージョンを使用する必要があるかを確認する必要があります。 たとえば、Windows Server バージョン1809をコンテナー OS として使用し、最新の更新プログラムを適用する必要がある場合は、次`1809`のように、必要な基本 OS コンテナーイメージのバージョンを指定するときに、タグを使用する必要があります。

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

ただし、Windows Server バージョン1809の特定の修正プログラムが必要な場合は、タグで KB 番号を指定できます。 たとえば、KB4493509 が適用された Windows Server バージョン1809から Nano Server ベース OS コンテナーイメージを取得するには、次のように指定します。

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

また、以前に使用したスキーマで必要とされる正確なパッチを指定することもできます。これには、タグで OS のバージョンを指定します。

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Windows Server 2019 および Windows Server 2016 に基づく Server Core ベースの画像は、[長期サービスチャネル (LTSC)](https://docs.microsoft.com/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc)のリリースです。 サーバーのコアイメージのコンテナー OS として Windows Server 2019 が必要な場合に、最新の更新プログラムを適用するには、次のように LTSC release を指定します。

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Docker Swarm を使ってバージョンを一致させる方法

現在、Docker 群れには、コンテナーが同じバージョンのホストに使用している Windows のバージョンに対応するための組み込みの方法がありません。 新しいコンテナーを使用するようにサービスを更新すると、正常に実行されます。

複数のバージョンの Windows を長期間実行する必要がある場合は、次の2つの方法があります。 Windows ホストが常に Hyper-v 分離を使用するように構成するか、ラベル制約を使うことができます。

### <a name="finding-a-service-that-wont-start"></a>起動しないサービスの検出方法

サービスが開始されない場合は、が`MODE` `replicated` `REPLICAS`表示されますが、0になることがあります。 OS のバージョンが問題であるかどうかを確認するには、次のコマンドを実行します。

**Docker service ls**を実行してサービス名を検索します。

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

状態と最新の試行を取得するには、 **docker サービス ps (サービス名)** を実行します。

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

表示`starting container failed: ...`された場合は、 **docker service ps--trunc (コンテナ名)** の完全なエラーが表示されます。

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

このエラーはと`CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`同じです。

### <a name="fix---update-the-service-to-use-a-matching-version"></a>修正方法: 一致するバージョンが使用されるようにサービスを更新する

Docker Swarm を使う場合、2 つの考慮事項があります。 作成していない画像を使用するサービスが含まれている作成ファイルがある場合は、それに応じて参照情報を更新する必要があります。 次に例を示します。

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

もう1つの考慮事項は、ポイントしている画像が自分で作成したものであるかどうか (たとえば、contoso/myimage) です。

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

この場合、「[不一致バージョンのエラー](#errors-from-mismatched-versions) 」で説明されている方法を使用して、docker-を作成する行ではなく、dockerfile を変更する必要があります。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>軽減策: Docker Swarm で Hyper-V による分離を使用する

Hyper-v 分離をコンテナーごとに使用することをサポートする提案がありますが、コードはまだ完了していません。 この提案の進行状況については、[GitHub](https://github.com/moby/moby/issues/31616) でご確認ください。 それが完成するまでの間、ホストは Hyper-V による分離を常時実行するように構成する必要があります。

これには、Docker サービスの構成を変更した後、Docker エンジンを再起動します。

1. 次のコマンドを編集します。 `C:\ProgramData\docker\config\daemon.json`
2. 次のコマンドを使って行を追加します。 `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >既定では、daemon.log ファイルは存在しません。 ディレクトリを調査して、このファイルが存在していない場合は、ファイルを作成する必要があります。 次のようにコピーします。

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. ファイルを閉じて保存してから、PowerShell で次のコマンドレットを実行して、docker エンジンを再起動します。

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. サービスを再起動したら、コンテナーを起動します。 動作したら、次のコマンドレットを使用してコンテナーを調べることで、コンテナーの分離レベルを確認できます。

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

このコマンドから、"Process" または "hyperv" が返されます。 上記のように変更し、daemon.json を設定した場合は、"hyperv" が表示されます。

### <a name="mitigation---use-labels-and-constraints"></a>軽減策: ラベルと制約を使用する

次に、バージョンと一致するようにラベルと制約を使用する方法について説明します。

1. 各ノードにラベルを追加します。

    各ノードで、 `OS`と`OsVersion`の2つのラベルを追加します。 ここではローカルでの実行を想定していますが、リモート ホスト上で変更を行ってこれらのラベルを設定することもできます。

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    次に、[ **docker ノード検査**] コマンドを実行して、新しく追加されたラベルを表示する必要があります。

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

    各ノードのラベルが付けられたので、サービスの配置を決定する制約を更新できます。 次の例では、"contoso_service" を実際のサービス名に置き換えます。

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    これにより、コードが実行できる場所が指定され、制限が課されます。

サービスの制約の使用方法の詳細については、「[サービスの作成](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)」を参照してください。

## <a name="matching-versions-using-kubernetes"></a>Kubernetes を使用したバージョンの一致

[Docker の群れを使っ](#matching-versions-using-docker-swarm)た同じバージョンで、Kubernetes で pod がスケジュールされている場合に同じ問題が発生する可能性があります。 この問題は、次のような戦略で回避することができます。

- 開発と運用の同じ OS バージョンに基づいてコンテナーを再構築します。 方法については、「[使用するコンテナー OS のバージョンを選択](#choose-which-container-os-version-to-use)する」を参照してください。
- Windows Server 2016 と Windows Server バージョン1709ノードの両方が同じクラスター内にある場合は、ノードラベルと nodeSelectors を使って、互換性のあるノードで pod がスケジュールされていることを確認します。
- OS のバージョンに基づいて別のクラスターを使用する。

### <a name="finding-pods-failed-on-os-mismatch"></a>OS の不一致が原因で失敗したポッドの検索

この場合、展開には、OS のバージョンが不一致で、Hyper-v の分離が有効になっていないノードでスケジュールされた pod が含まれています。

同じエラーは、`kubectl describe pod <podname>` によって表示されたイベントにも存在します。 数回試行した後、pod の状態`CrashLoopBackOff`はおそらくのようになります。

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

### <a name="mitigation---using-node-labels-and-nodeselector"></a>軽減-ノードラベルと nodeSelector の使用

すべてのノードの一覧を取得するには、 **kubectl get ノード**を実行します。 その後、 **kubectl 説明ノード (ノード名)** を実行して、詳細を取得できます。

次の例では、2つの Windows ノードで異なるバージョンが実行されています。

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

次の例を使用して、バージョンを一致させる方法を示します。

1. 各ノード名と`Kernel Version`システム情報からメモを取ってください。

    この例では、次のような情報が表示されます。

    名前         | バージョン
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. 各ノードに、`beta.kubernetes.io/osbuild` というラベルを追加します。 Windows Server 2016 は、メジャーバージョンとマイナーバージョン (この例では 14393.1715) の両方が、Hyper-v 分離なしでサポートされるようにする必要があります。 Windows Server バージョン1709では、一致するメジャーバージョン (この例では 16299) のみが必要です。

    この例では、ラベルを追加するコマンドは次のようになります。

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. **Kubectl**を実行して、ラベルがあるかどうかを確認します。

    この例では、次のような出力が表示されます。

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. 展開にノードセレクターを追加します。 この例では`nodeSelector` 、= windows と`beta.kubernetes.io/os` `beta.kubernetes.io/osbuild` = 14393. * または16299を持つコンテナー仕様にを追加して、コンテナーで使用されているベース OS と一致させます。

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

    これで、更新された展開を使ってポッドを起動できるようになりました。 ノードセレクターも表示されるの`kubectl describe pod <podname>`で、そのコマンドを実行して追加されたことを確認できます。

    この例の出力は次のようになります。

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
