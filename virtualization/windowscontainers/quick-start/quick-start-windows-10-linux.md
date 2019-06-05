---
title: Windows 10 の windows と Linux のコンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 926e5cd64053b5ea795bb2c75a231700aed443ca
ms.sourcegitcommit: f6457ee0635864e8e8bb07da43a6f76388ee3cd1
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/05/2019
ms.locfileid: "9734656"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 の Linux コンテナー

> [!div class="op_single_selector"]
> - [Linux Containers on Windows](quick-start-windows-10-linux.md)
> - [Windows の windows コンテナー](quick-start-windows-10.md)

この練習では、Windows 10 での Linux コンテナーの作成と実行について説明します。

このクイックスタートでは、次のことを実行します。

1. Docker デスクトップのインストール
2. Windows 上の Linux コンテナーを使って簡単な Linux コンテナーを実行する (LCOW)

このクイック スタートは、Windows 10 に固有です。 その他のクイックスタートドキュメントは、このページの左側にある目次に記載されています。

## <a name="prerequisites"></a>前提条件

次の要件を満たしていることを確認してください。
- Windows 10 Professional、Windows 10 Enterprise、または Windows Server 2019 バージョン1809以降を実行している1台の物理コンピューターシステム
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)が有効になっていることを確認します。

***Hyper-v 分離:*** Windows 上の linux コンテナーでは、開発者が適切な Linux カーネルを使ってコンテナーを実行するために、Windows 10 の Hyper-v 分離が必要です。 Hyper-v 分離の詳細については、「 [Windows コンテナーについて](../about/index.md)」のページをご覧ください。

## <a name="install-docker-desktop"></a>Docker デスクトップをインストールする

[Docker デスクトップ](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードしてインストーラーを実行します (ログインする必要があります)。 まだアカウントを持っていない場合は、アカウントを作成します)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

> 既に Docker をインストールしている場合は、LCOW をサポートするバージョン18.02 またはそれ以降をご利用ください。 Docker の実行`docker -v`または** チェックを確認します。

> *Docker Settings > デーモン*の [実験的な機能] オプションは、lcow コンテナーを実行するためにアクティブ化する必要があります。

## <a name="run-your-first-lcow-container"></a>最初の LCOW コンテナーを実行する

この例では、BusyBox コンテナーが配置されます。 まず、"Hello World" BusyBox のイメージを実行してみます。

```console
docker run --rm busybox echo hello_world
```

これは、Docker がイメージをプルしようとしたときにエラーが返されることに注意してください。 これは、Dockers が、イメージとホスト`--platform`オペレーティングシステムが適切に一致していることを確認するために、フラグによってディレクティブが必要であるために発生します。 Windows コンテナーモードの既定のプラットフォームは Windows であるため、 `--platform linux`コンテナーを取得して実行するためのフラグを追加します。

```console
docker run --rm --platform linux busybox echo hello_world
```

指定したプラットフォームでイメージがプルされると、 `--platform`フラグは不要になります。 これをテストするには、このコマンドを使わずにコマンドを実行します。

```console
docker run --rm busybox echo hello_world
```

実行`docker images`して、インストールされている画像の一覧を返します。 この場合、Windows と Linux の両方のイメージ。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> ボーナス: 実行時に、Docker の[ブログ投稿](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)をご覧ください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルアプリを作成する方法について説明します。](./building-sample-app.md)
