---
title: Windows Insider Program でコンテナーを使用する
description: Windows Insider program で windows コンテナーの使用を開始する方法について説明します。
keywords: docker、コンテナー、insider、windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909892"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Windows Insider program でコンテナーを使用する

この演習では、Windows Insider Preview プログラムで提供されている最新の insider ビルドの Windows Server に、Windows コンテナー機能を展開して使用する手順を説明します。 この演習では、コンテナーの役割をインストールし、基本 OS イメージのプレビュー版を展開します。 コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

## <a name="join-the-windows-insider-program"></a>Windows Insider Program に参加する

Windows コンテナーの insider バージョンを実行するには、windows insider プログラムから windows Server の最新のビルドを実行しているホスト、または windows Insider program からの Windows 10 の最新のビルドを実行している必要があります。 [Windows Insider プログラム](https://insider.windows.com/GettingStarted)に参加して、使用条件を確認します。

> [!IMPORTANT]
> 以下で説明する基本イメージを使用するには、windows Server Insider Preview プログラムから windows Server のビルドを使用するか、windows Insider Preview プログラムから windows 10 のビルドを使用する必要があります。 これらのビルドを使用せずに、対象の基本イメージを使用した場合、コンテナーを開始できません。

## <a name="install-docker"></a>Docker のインストール

Docker をまだインストールしていない場合は、「[はじめ](../quick-start/set-up-environment.md)に」ガイドに従って docker をインストールしてください。

## <a name="pull-an-insider-container-image"></a>Insider コンテナーイメージをプルする

Windows Insider program の一部であるため、基本イメージには最新のビルドを使用できます。

Nano Server Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

"Windows" と "IoTCore" の基本イメージには、プルに使用できる insider バージョンもあります。 利用可能な insider base イメージの詳細については、[コンテナーの基本イメージ](../manage-containers/container-base-images.md)に関するドキュメントを参照してください。

> [!IMPORTANT]
> Windows コンテナー OS イメージの[EULA](../images-eula.md )と windows Insider Program[の使用条件](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)をお読みください。
