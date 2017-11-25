---
title: "Windows Server の Windows コンテナー"
description: "コンテナー展開のクイック スタート"
keywords: "Docker, コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 73112b3d1b86b5c72cb6352faaaac3ae95b0957e
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/08/2017
---
# <a name="windows-containers-on-windows-server"></a>Windows Server の Windows コンテナー

この演習では、Windows Server 2016 の Windows コンテナー機能の基本的な展開と使用について段階的に確認します。 この演習では、コンテナーの役割をインストールし、単純な Windows Server コンテナーを展開します。 コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

このクイック スタートは Windows Server 2016 の Windows Server コンテナーのみに適用されます。 このページの左側の目次に、Windows 10 のコンテナーを含む追加のクイック スタート文書があります。

**前提条件:**

Windows Server 2016 を実行している 1 台のコンピューター システム (物理または仮想)。 Windows Server 2016 TP5 を使用している場合は、[Window Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ) に更新してください。

> Windows コンテナー機能が動作するためには、重要な更新プログラムが必要です。 すべての更新プログラムをインストールしてから、このチュートリアルを進めてください。

Azure に展開する場合は、こちらの[テンプレート](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)を使用すると展開が簡単です。<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="1-install-docker"></a>1.Docker のインストール

Docker をインストールするには、他のプロバイダー (ここでは [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider)) と連携してインストールを実行する [OneGet プロバイダー PowerShell モジュール](https://github.com/oneget/oneget)を使用します。 プロバイダーは、コンピューターでコンテナーの機能を有効にします。 また、再起動が必要になる Docker をインストールします。 Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。

管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

まず、[PowerShell ギャラリー](https://www.powershellgallery.com/packages/DockerMsftProvider)から Docker-Microsoft PackageManagement Provider をインストールします。

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

次に、PackageManagement PowerShell モジュールを使って、最新バージョンの Docker をインストールします。
```
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell でパッケージ ソース "DockerDefault" を信頼するかどうかの確認を求められたら、「`A`」と入力してインストールを続行します。 インストールが完了したら、コンピューターを再起動します。

```
Restart-Computer -Force
```

> ヒント: 後で Docker を更新する場合は、
>  - 次を実行して、インストールされているバージョンを確認します。 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 次を実行して、最新のバージョンを検索します。 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 準備ができたら、`Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` を実行してアップグレードした後、次を実行します。 `Start-Service Docker`

## <a name="2-install-windows-updates"></a>2. Windows の更新プログラムをインストールする

次を実行して、Windows Server システムが最新の状態であることを確認します。

```
sconfig
```

これによりテキスト ベースの構成メニューが表示されます。オプション 6 "更新プログラムのダウンロードとインストール" を選択します。

```
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

画面の指示に従い、オプション A を選択してすべての更新プログラムをダウンロードします。

## <a name="3-deploy-your-first-container"></a>3.最初のコンテナーの展開

この演習では、事前作成された .NET サンプル イメージを Docker Hub レジストリからダウンロードし、.NET Hello World アプリケーションを実行するシンプルなコンテナーを展開します。  

`docker run` を使用し、.Net コンテナーを展開します。 この操作でコンテナー イメージもダウンロードされるので、処理に数分かかる可能性があります。

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

コンテナーが起動し、hello world メッセージが出力され、終了します。

```console
         Dotnet-bot: Welcome to using .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....


**Environment**
Platform: .NET Core 1.0
OS: Microsoft Windows 10.0.14393
```

Docker Run コマンドの詳細については、Docker.com の「[Docker Run リファレンス]( https://docs.docker.com/engine/reference/run/)」をご覧ください。

## <a name="next-steps"></a>次の手順

[ビルドの自動化とイメージの保存](./quick-start-images.md)
