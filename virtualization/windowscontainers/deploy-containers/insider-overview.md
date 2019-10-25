---
title: Windows Insider Program でコンテナーを使う
description: Windows Insider プログラムで windows コンテナーを使い始める方法について説明します。
keywords: docker、コンテナー、insider、windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257796"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Windows Insider プログラムでコンテナーを使う

この演習では、Windows Insider Preview プログラムで提供されている最新の insider ビルドの Windows Server に、Windows コンテナー機能を展開して使用する手順を説明します。 この演習では、コンテナーの役割をインストールし、基本 OS イメージのプレビュー版を展開します。 コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

## <a name="join-the-windows-insider-program"></a>Windows Insider Program に参加する

Windows のコンテナーの insider バージョンを実行するには、windows Server の最新ビルドを実行しているホストと、windows insider プログラムから windows 10 の最新のビルドを実行していることが必要です。 [Windows Insider プログラム](https://insider.windows.com/GettingStarted)に参加して使用条件を確認します。

> [!IMPORTANT]
> Windows server Insider Preview プログラムまたは windows 10 のビルドから windows Server のビルドを使用する必要があります。 windows Insider Preview プログラムでは、以下に示す基本イメージを使用します。 これらのビルドを使用せずに、対象の基本イメージを使用した場合、コンテナーを開始できません。

## <a name="install-docker"></a>Docker のインストール

Docker をまだインストールしていない場合は、「[はじめ](../quick-start/set-up-environment.md)に」のガイドに従って docker をインストールしてください。

## <a name="pull-an-insider-container-image"></a>Insider コンテナーの画像を取得する

Windows Insider プログラムの一部として、基本イメージの最新のビルドを使うことができます。

Nano Server Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

"Windows" と "IoTCore" の基本イメージには、プルできる insider バージョンも用意されています。 利用可能な insider ベースの画像の詳細については、[コンテナベース画像](../manage-containers/container-base-images.md)ドキュメントを参照してください。

> [!IMPORTANT]
> Windows コンテナ OS イメージ[EULA](../images-eula.md )と windows Insider プログラム[利用規約](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)をお読みください。
