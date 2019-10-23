---
title: Windows Insider Program でコンテナーを使う
description: Windows Insider プログラムで windows コンテナーを使い始める方法について説明します。
keywords: docker、コンテナー、insider、windows
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254406"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Windows Insider プログラムでコンテナーを使う

この演習では、Windows Insider Preview プログラムで提供されている最新の insider ビルドの Windows Server に、Windows コンテナー機能を展開して使用する手順を説明します。 この演習では、コンテナーの役割をインストールし、基本 OS イメージのプレビュー版を展開します。 コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

> [!NOTE]
> このコンテンツは、Windows Server Insider Preview プログラムの Windows Server コンテナーに固有のものです。 Windows コンテナーを使用するために insider 以外の手順を探している場合は、 [「はじめに」のガイドを](../quick-start/set-up-environment.md)参照してください。

## <a name="join-the-windows-insider-program"></a>Windows Insider Program に参加する

Windows のコンテナーの insider バージョンを実行するには、windows Server の最新ビルドを実行しているホストと、windows insider プログラムから windows 10 の最新のビルドを実行していることが必要です。 [Windows Insider プログラム](https://insider.windows.com/GettingStarted)に参加して使用条件を確認します。

> [!IMPORTANT]
> Windows server Insider Preview プログラムまたは windows 10 のビルドから windows Server のビルドを使用する必要があります。 windows Insider Preview プログラムでは、以下に示す基本イメージを使用します。 これらのビルドを使用せずに、対象の基本イメージを使用した場合、コンテナーを開始できません。

## <a name="install-docker"></a>Docker のインストール

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Windows Server Insider](#tab/Windows-Server-Insider)

Docker EE をインストールするには、OneGet プロバイダー PowerShell モジュールを使用します。 このプロバイダーは、コンピューターでコンテナー機能を有効にして、Docker EE をインストールします。これには、再起動が必要です。 管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

> [!NOTE]
> Windows Server Insider ビルドでの Docker EE のインストールには、Insider 以外のビルドに使用されるものとは異なる OneGet provider が必要です。 Docker EE と DockerMsftProvider OneGet プロバイダーが既にインストールされている場合は、続行する前に削除します。

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Windows Insider ビルドで使用するための OneGet PowerShell モジュールをインストールします。

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

OneGet を使用して最新バージョンの Docker EE Preview をインストールします。

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

インストールが完了したら、コンピューターを再起動します。

```powershell
Restart-Computer -Force
```

# [<a name="windows-10-insider"></a>Windows 10 Insider](#tab/Windows-10-Insider)

Windows 10 Insider の場合、Docker Edge は、Docker Desktop stable と同じインストーラーを使ってインストールされます。 [Docker デスクトップ](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードして、インストーラーを実行します。 ログインする必要があります。 まだアカウントを持っていない場合は、アカウントを作成します。 詳細なインストール手順については、「 [Docker ドキュメント](https://docs.docker.com/docker-for-windows/install)」を参照してください。

インストール後、Docker の [設定] を開き、"Edge" チャネルに切り替えます。

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>Insider コンテナーの画像を取得する

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 Windows Insider プログラムの一部として、基本イメージの最新のビルドを使うことができます。 [コンテナーベースのイメージ](../manage-containers/container-base-images.md)ドキュメントで利用可能な基本イメージの詳細については、こちらを参照してください。

Nano Server Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Windows コンテナ OS イメージ[EULA](../images-eula.md )と windows Insider プログラム[利用規約](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)をお読みください。
