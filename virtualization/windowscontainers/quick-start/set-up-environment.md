---
title: Windows 10 の windows コンテナーと Linux コンテナー
description: コンテナーに対して Windows 10 または Windows Server をセットアップしてから、最初のコンテナーイメージの実行に進みます。
keywords: docker、コンテナー、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909562"
---
# <a name="get-started-prep-windows-for-containers"></a>はじめに: コンテナーのウィンドウの準備

このチュートリアルでは、次の方法について説明します。

- コンテナーに対して Windows 10 または Windows Server をセットアップする
- 最初のコンテナーイメージを実行する
- 単純な .NET core アプリケーションのコンテナー化

## <a name="prerequisites"></a>前提条件

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server でコンテナーを実行するには、windows Server (半期チャネル)、Windows Server 2019、または Windows Server 2016 を実行している物理サーバーまたは仮想マシンが必要です。

テストでは、 [Window server 2019 評価版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )または[Windows server Insider Preview](https://insider.windows.com/for-business-getting-started-server/)のコピーをダウンロードできます。

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Windows 10 でコンテナーを実行するには、次のものが必要です。

- 記念日更新プログラム (バージョン 1607) 以降を実行している、Windows 10 Professional または Enterprise を実行している1台の物理コンピューターシステム。
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)を有効にする必要があります。

> [!NOTE]
>  Windows 10 10 月の更新プログラム2018以降、開発/テストのために Windows 10 Enterprise または Professional で windows コンテナーをプロセス分離モードで実行することをユーザーに許可しなくなりました。 詳細については、 [FAQ](../about/faq.md)を参照してください。 
> 
> Windows Server のコンテナーでは、運用環境で使用されるものと同じカーネルバージョンおよび構成を開発者に提供するために、Windows 10 で既定で Hyper-v 分離を使用します。 Hyper-v の分離の詳細については、ドキュメントの[概念](../manage-containers/hyperv-container.md)領域を参照してください。

---
<!-- stop tab view -->

## <a name="install-docker"></a>Docker のインストール

最初の手順では、Windows コンテナーを操作するために必要な Docker をインストールします。 Docker は、共通の API とコマンドラインインターフェイス (CLI) を使用して、コンテナーの標準のランタイム環境を提供します。

構成の詳細については、「 [Windows 上の Docker エンジン](../manage-docker/configure-docker-daemon.md)」を参照してください。

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server に Docker をインストールするには、Microsoft によって発行された[Oneget Provider PowerShell モジュール](https://github.com/oneget/oneget)を[dockermicrosoft プロバイダー](https://github.com/OneGet/MicrosoftDockerProvider)と呼びます。 このプロバイダーにより、Windows のコンテナー機能が有効になり、Docker エンジンとクライアントがインストールされます。 以下にその方法を示します。

1. 管理者特権の PowerShell セッションを開き、 [PowerShell ギャラリー](https://www.powershellgallery.com/packages/DockerMsftProvider)から Docker-Microsoft PackageManagement プロバイダーをインストールします。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   NuGet プロバイダーのインストールを求めるメッセージが表示されたら、「`Y`」と入力してインストールします。

2. PackageManagement PowerShell モジュールを使用して、最新バージョンの Docker をインストールします。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   PowerShell でパッケージ ソース "DockerDefault" を信頼するかどうかの確認を求められたら、「`A`」と入力してインストールを続行します。
3. インストールが完了したら、コンピューターを再起動します。

   ```powershell
   Restart-Computer -Force
   ```

後で Docker を更新する場合は、次のようにします。

- インストールされているバージョンを確認し `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- `Find-Package -Name Docker -ProviderName DockerMsftProvider` で現在のバージョンを検索する
- 準備ができたら、`Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`でアップグレードし、その後にを `Start-Service Docker`

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

次の手順を使用して、Windows 10 Professional および Enterprise エディションに Docker をインストールできます。 

1. [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードしてインストールします。まだお持ちでない場合は、無料の docker アカウントを作成します。 詳細については、 [Docker のドキュメント](https://docs.docker.com/docker-for-windows/install)を参照してください。

2. インストール中に、既定のコンテナーの種類を Windows コンテナーに設定します。 インストールの完了後に切り替えるには、次に示すように、Windows システムトレイの Docker 項目を使用するか、PowerShell プロンプトで次のコマンドを使用します。

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![[Windows コンテナーに切り替え] コマンドを示す Docker システムトレイメニュー。](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>次のステップ

環境が正しく構成されたので、次のリンクを使用してコンテナーを実行する方法を確認してください。

> [!div class="nextstepaction"]
> [最初のコンテナーを実行する](./run-your-first-container.md)
