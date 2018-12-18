---
title: Windows Server の Windows コンテナー
description: コンテナー展開のクイック スタート
keywords: Docker, コンテナー
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: e27148873299543a89eaf92801b40732dd27b402
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973672"
---
# <a name="windows-containers-on-windows-server"></a>Windows Server の Windows コンテナー

この手順は、Windows Server 2019 で基本的な展開と Windows コンテナー機能の使用について説明します。

このクイック スタートでは、次の実行されます。

1. Windows Server でコンテナーの機能を有効にします。
2. Docker をインストールします。
3. 単純な Windows コンテナーの実行

コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

このクイック スタートでは、Windows Server 2019 の Windows Server コンテナーを特定します。 このページの左側の目次に、Windows 10 のコンテナーを含む追加のクイック スタート文書があります。

## <a name="prerequisites"></a>前提条件

次の要件を満たしていることを確認してください。
- 1 台のコンピューター システム (物理または仮想) には、Windows Server 2019 が実行されています。 Windows Server 2019 内部のプレビューを使用している場合は、[ウィンドウのサーバー 2019 評価](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019 )を更新してください。

> Windows コンテナー機能が動作するためには、重要な更新プログラムが必要です。 すべての更新プログラムをインストールしてから、このチュートリアルを進めてください。

Azure に展開する場合は、こちらの[テンプレート](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)を使用すると展開が簡単です。

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>Docker のインストール

Docker をインストールするのには、ここでは、インストールを実行するプロバイダーと[MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider)都合が良い[OneGet プロバイダー PowerShell モジュール](https://github.com/oneget/oneget)を使用します。 プロバイダーは、コンピューターでコンテナーの機能を有効にします。 また、再起動が必要になる Docker をインストールします。 Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。

管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

まず、[PowerShell ギャラリー](https://www.powershellgallery.com/packages/DockerMsftProvider)から Docker-Microsoft PackageManagement Provider をインストールします。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

次に、PackageManagement PowerShell モジュールを使って、最新バージョンの Docker をインストールします。

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell でパッケージ ソース "DockerDefault" を信頼するかどうかの確認を求められたら、「`A`」と入力してインストールを続行します。 インストールが完了したら、コンピューターを再起動します。

```powershell
Restart-Computer -Force
```

> ![ヒント]Docker を後で更新する場合。
>  - 次を実行して、インストールされているバージョンを確認します。 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 次を実行して、最新のバージョンを検索します。 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 準備ができたら、`Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` を実行してアップグレードした後、次を実行します。 `Start-Service Docker`

## <a name="install-windows-updates"></a>Windows の更新プログラムをインストールする

次を実行して、Windows Server システムが最新の状態であることを確認します。

```console
sconfig
```

これによりテキスト ベースの構成メニューが表示されます。オプション 6 "更新プログラムのダウンロードとインストール" を選択します。

```console
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

## <a name="deploy-your-first-container"></a>最初にコンテナーを展開します。

この演習では、事前作成された .NET サンプル イメージを Docker Hub レジストリからダウンロードし、.NET Hello World アプリケーションを実行するシンプルなコンテナーを展開します。  

`docker run` を使用し、.Net コンテナーを展開します。 この操作でコンテナー イメージもダウンロードされるので、処理に数分かかる可能性があります。

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

コンテナーが起動し、hello world メッセージが出力され、終了します。

```console
         Hello from .NET Core!
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
Platform: .NET Core
OS: Microsoft Windows 10.0.17763
```

Docker Run コマンドの詳細については、Docker.com の「[Docker Run リファレンス]( https://docs.docker.com/engine/reference/run/)」をご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [コンテナー ビルドを自動化してイメージを保存する方法を学習します。](./quick-start-images.md)
