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
ms.openlocfilehash: 665e6186ff5f9530f12ba3d0a400d82bcac755a7
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999199"
---
# <a name="windows-containers-on-windows-server"></a>Windows Server の Windows コンテナー

この演習では、windows Server 2019 および Windows Server 2016 での Windows コンテナー機能の基本的な展開と使用について説明します。

このクイックスタートでは、次のことを実行します。

1. Windows Server でコンテナー機能を有効にする
2. Docker のインストール
3. 簡単な Windows コンテナーの実行

コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

このクイックスタートは、windows server 2019 および Windows Server 2016 の Windows Server コンテナーに固有のものです。 このページの左側の目次に、Windows 10 のコンテナーを含む追加のクイック スタート文書があります。

## <a name="prerequisites"></a>前提条件

次の要件を満たしていることを確認してください。
- Windows Server 2019 を実行している1台のコンピューターシステム (物理または仮想)。 Windows Server 2019 Insider Preview を使用している場合は、 [Window server 2019 の評価](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )に更新してください。

> Windows コンテナー機能を機能させるには、重要な更新プログラムが必要です。 すべての更新プログラムをインストールしてから、このチュートリアルを進めてください。

Azure に展開する場合は、こちらの[テンプレート](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)を使用すると展開が簡単です。

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>Docker のインストール

Docker をインストールするには、 [Oneget Provider PowerShell モジュール](https://github.com/oneget/oneget)(この例では、 [Microsoft dockerprovider](https://github.com/OneGet/MicrosoftDockerProvider)) を使ってインストールを実行します。 プロバイダーは、コンピューターでコンテナーの機能を有効にします。 また、再起動が必要になる Docker をインストールします。 Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。

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

> [!TIP]
> Docker を後で更新する場合は、次の操作を行います。
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

## <a name="deploy-your-first-container"></a>最初のコンテナーを展開する

この演習では、事前作成された .NET サンプル イメージを Docker Hub レジストリからダウンロードし、.NET Hello World アプリケーションを実行するシンプルなコンテナーを展開します。  

`docker run` を使用し、.Net コンテナーを展開します。 この操作でコンテナー イメージもダウンロードされるので、処理に数分かかる可能性があります。 使用している Windows Server のバージョンに応じて、次のコマンドを実行します。

#### <a name="windows-server-2019"></a>Windows Server 2019

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

#### <a name="windows-server-2016"></a>Windows Server 2016

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-sac2016
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

Docker Run コマンドの詳細については、Docker.com の「[Docker Run リファレンス](https://docs.docker.com/engine/reference/run/)」をご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [コンテナーの作成を自動化して画像を保存する方法について説明します。](./quick-start-images.md)
