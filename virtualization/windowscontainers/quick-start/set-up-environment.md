---
title: Windows 10 の windows と Linux のコンテナー
description: コンテナー用に Windows 10 または Windows Server をセットアップした後、最初のコンテナーイメージの実行に進みます。
keywords: docker、コンテナー、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288120"
---
# <a name="get-started-prep-windows-for-containers"></a>はじめに: コンテナーのための Windows の準備

このチュートリアルでは、次の方法について説明します。

- コンテナー用に Windows 10 または Windows Server をセットアップする
- 最初のコンテナーイメージを実行する
- Containerize のシンプルな .NET コアアプリケーション

## <a name="prerequisites"></a>前提条件

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Windows Server でコンテナーを実行するには、Windows Server (半期チャネル)、Windows Server 2019、または Windows Server 2016 を実行している物理サーバーまたは仮想マシンが必要です。

テストのために、 [Window server 2019 評価版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )または[Windows server Insider Preview](https://insider.windows.com/for-business-getting-started-server/)のコピーをダウンロードできます。

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

Windows 10 でコンテナーを実行するには、次のものが必要です。

- Windows 10 Professional または Enterprise for 記念日更新プログラム (バージョン 1607) 以降を実行している1つの物理コンピューターシステム。
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)を有効にする必要があります。

> [!NOTE]
>  Windows 10 年10月の更新プログラム2018以降では、開発者/テスト用の windows 10 Enterprise または Professional でのプロセス分離モードでの Windows コンテナーの実行は、ユーザーによって許可されなくなりました。 詳細については、[よく寄せ](../about/faq.md)られる質問を参照してください。 
> 
> Windows Server コンテナーでは、Windows 10 の既定の Hyper-v 分離が使用されているため、開発者は、実稼働環境で使用するのと同じカーネルバージョンと構成を提供します。 Hyper-v の分離の詳細については、「ドキュメントの[概念](../manage-containers/hyperv-container.md)」を参照してください。

---
<!-- stop tab view -->

## <a name="install-docker"></a>Docker のインストール

最初の手順は、Windows コンテナーを操作するために必要な Docker をインストールすることです。 Docker は、共通の API とコマンドラインインターフェイス (CLI) を使って、コンテナーの標準のランタイム環境を提供します。

構成の詳細については、「 [Windows の Docker Engine](../manage-docker/configure-docker-daemon.md)」を参照してください。

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Windows Server に Docker をインストールするには、Microsoft によって公開された[Onekerprovider PowerShell モジュール](https://github.com/oneget/oneget)を使用します。 [](https://github.com/OneGet/MicrosoftDockerProvider) このプロバイダーは、Windows のコンテナー機能を有効にし、Docker エンジンとクライアントをインストールします。 その方法は次のとおりです。

1. 管理者の PowerShell セッションを開き、 [Powershell ギャラリー](https://www.powershellgallery.com/packages/DockerMsftProvider)から Docker-Microsoft PackageManagement プロバイダーをインストールします。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   NuGet プロバイダーをインストールするかどうかを確認する`Y`メッセージが表示されたら、入力してインストールします。

2. PackageManagement PowerShell モジュールを使用して、最新バージョンの Docker をインストールします。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   PowerShell でパッケージ ソース "DockerDefault" を信頼するかどうかの確認を求められたら、「`A`」と入力してインストールを続行します。
3. インストールが完了したら、コンピューターを再起動します。

   ```powershell
   Restart-Computer -Force
   ```

Docker を後で更新する場合は、次の操作を行います。

- 次を実行して、インストールされているバージョンを確認します。 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- 次を実行して、最新のバージョンを検索します。 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- 準備ができたら、`Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` を実行してアップグレードした後、次を実行します。 `Start-Service Docker`

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

次の手順を使用して、Windows 10 Professional と Enterprise エディションに Docker をインストールできます。 

1. まだインストールしていない場合は、 [Docker デスクトップ](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードしてインストールし、無料の docker アカウントを作成します。 詳細については、「 [Docker ドキュメント](https://docs.docker.com/docker-for-windows/install)」を参照してください。

2. インストール時に、既定のコンテナーの種類を [Windows コンテナー] に設定します。 インストールが完了した後で切り替えるには、Windows システムトレイの Docker アイテム (以下に示すように) を使用するか、PowerShell プロンプトで次のコマンドを使用します。

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![[Windows コンテナーに切り替える] コマンドが表示された Docker システムトレイメニュー](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>次のステップ

環境が正しく構成されたので、次のリンクを参照して、コンテナーの実行方法を確認してください。

> [!div class="nextstepaction"]
> [最初のコンテナーを実行する](./run-your-first-container.md)
