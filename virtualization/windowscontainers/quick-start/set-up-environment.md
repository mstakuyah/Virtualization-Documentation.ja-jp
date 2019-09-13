---
title: Windows 10 の windows と Linux のコンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129375"
---
# <a name="get-started-configure-your-environment-for-containers"></a>はじめに: コンテナーの環境の構成

このクイックスタートでは、次の方法について説明します。

> [!div class="checklist"]
> * コンテナーの環境をセットアップする
> * 最初のコンテナーイメージを実行する
> * Containerize のシンプルな .NET コアアプリケーション

## <a name="prerequisites"></a>前提条件

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

次の要件を満たしていることを確認してください。

- Windows Server 2016 以降を実行している1台のコンピューターシステム (物理または仮想)。

> [!NOTE]
> Windows Server 2019 Insider Preview を使用している場合は、 [Window server 2019 の評価](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )に更新してください。

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional と Enterprise](#tab/Windows-10-Client)

次の要件を満たしていることを確認してください。

- Windows 10 Professional または Enterprise for 記念日更新プログラム (バージョン 1607) 以降を実行している1つの物理コンピューターシステム。
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)を有効にする必要があります。

> [!NOTE]
>  Windows 10 年10月の更新プログラム2018以降では、開発者/テスト用の windows 10 Enterprise または Professional でのプロセス分離モードでの Windows コンテナーの実行は、ユーザーによって許可されなくなりました。 詳細については、[よく寄せ](../about/faq.md)られる質問を参照してください。 
> 
> Windows Server コンテナーでは、Windows 10 の既定の Hyper-v 分離が使用されているため、開発者は、実稼働環境で使用するのと同じカーネルバージョンと構成を提供します。 Hyper-v の分離の詳細については、「ドキュメントの[概念](../manage-containers/hyperv-container.md)」を参照してください。

---
<!-- stop tab view -->

## <a name="install-docker"></a>Docker のインストール

Docker は、Windows コンテナーを操作するための最終的なツールチェーンです。 Docker は、ユーザーが特定のホストのコンテナーを管理するための CLI を提供します。コンテナーの作成、コンテナーの削除などを行います。 Docker の詳細については、ドキュメントの[コンセプト](../manage-containers/configure-docker-daemon.md)のページをご覧ください。

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Windows Server では、Docker は、Microsoft によって[Dockermicrosoft](https://github.com/OneGet/MicrosoftDockerProvider)によって公開された[Onekerprovider PowerShell モジュール](https://github.com/oneget/oneget)によってインストールされます。 プロバイダー:

- コンピューターでコンテナー機能を有効にします
- コンピューターに Docker エンジンとクライアントをインストールします。

Docker をインストールするには、管理者特権の PowerShell セッションを開き、 [Powershell ギャラリー](https://www.powershellgallery.com/packages/DockerMsftProvider)から Docker-Microsoft PackageManagement プロバイダーをインストールします。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

次に、PackageManagement PowerShell モジュールを使用して、最新バージョンの Docker をインストールします。

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell でパッケージ ソース "DockerDefault" を信頼するかどうかの確認を求められたら、「`A`」と入力してインストールを続行します。 インストールが完了したら、コンピューターを再起動する必要があります。

```powershell
Restart-Computer -Force
```

> [!TIP]
> Docker を後で更新する場合は、次の操作を行います。
>  - 次を実行して、インストールされているバージョンを確認します。 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 次を実行して、最新のバージョンを検索します。 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 準備ができたら、`Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` を実行してアップグレードした後、次を実行します。 `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional と Enterprise](#tab/Windows-10-Client)

Windows 10 Professional と Enterprise では、従来のインストーラーを使用して Docker がインストールされます。 [Docker デスクトップ](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードして、インストーラーを実行します。 ログインする必要があります。 まだアカウントを持っていない場合は、アカウントを作成します。 詳細なインストール手順については、「 [Docker ドキュメント](https://docs.docker.com/docker-for-windows/install)」を参照してください。

インストール後、Docker デスクトップでは、既定で Linux コンテナーが実行されています。 Docker トレイメニューを使用するか、PowerShell プロンプトで次のコマンドを実行して、Windows コンテナーに切り替えます。

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>次のステップ

これで環境が正しく構成されました。次に、このリンクを使って、コンテナーを取得して実行する方法を学習します。

> [!div class="nextstepaction"]
> [最初のコンテナーを実行する](./run-your-first-container.md)
