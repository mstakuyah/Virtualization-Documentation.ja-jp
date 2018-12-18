---
title: Windows、Windows 10 の Linux コンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 036e4f80eaa6e7ce2c151d7732e670c0492bc61f
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973815"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 の Linux コンテナー

> [!div class="op_single_selector"]
> - [Linux Containers on Windows](quick-start-windows-10-linux.md)
> - [Windows 版の Windows コンテナー](quick-start-windows-10.md)

練習を作成および Linux コンテナーを実行して、Windows 10 で説明します。

このクイック スタートでは、次の実行されます。

1. Windows 版の Docker がインストールされています。
2. Windows (LCOW) で Linux コンテナーを使用して単純な Linux コンテナーを実行します。

このクイック スタートは、Windows 10 に固有です。 その他のクイック スタート ドキュメントは、このページの左側の目次に記載されています。

## <a name="prerequisites"></a>前提条件

次の要件を満たしていることを確認してください。
- Windows 10 Professional または Enterprise 秋作成者の更新プログラム (バージョン 1709) またはそれ以降を実行して、1 つの物理コンピューター システム
- [HYPER-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)が有効になっていることを確認します。

***HYPER-V 分離:*** Windows の Linux コンテナーを開発者に提供するために Windows 10 で HYPER-V 分離を必要としますコンテナーを実行する適切な linux します。 HYPER-V に関する分離あります [[コンテナーのバージョン情報](../about/index.md)] ページ。

## <a name="install-docker-for-windows"></a>Windows 版の Docker をインストールします。

[Windows 版の Docker](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードして (にログインする必要があるインストーラーを実行します。 作成アカウントを既に持っていない場合)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

> インストールされている Docker がある場合 LCOW をサポートするには、18.02 以降のバージョンがあることを確認します。 チェックを実行して`docker -v`や*に関する Docker*をチェックします。

> '実験機能] のオプションで*Docker 設定 > デーモン*LCOW コンテナーを実行するアクティブ化する必要があります。

## <a name="run-your-first-lcow-container"></a>最初の LCOW コンテナーを実行します。

この例では、BusyBox コンテナーが配置されます。 まず、' Hello World' BusyBox のイメージを実行します。

```console
docker run --rm busybox echo hello_world
```

このエラーを返します Docker イメージを取得しようとしたときに注意してください。 これは、Dockers 経由でディレクティブを必要とするため、`--platform`画像とホスト オペレーティング システムが適切に一致することを確認するフラグを設定します。 Windows コンテナー モードで既定のプラットフォームが Windows であるため、追加する`--platform linux`フラグを抽出し、コンテナーを実行します。

```console
docker run --rm --platform linux busybox echo hello_world
```

画像が表示された、プラットフォームのとして別途された後、`--platform`フラグが不要になった。 これをテストすることがなくコマンドを実行します。

```console
docker run --rm busybox echo hello_world
```

実行`docker images`にインストールされている画像のリストを返します。 この例では、Windows および Linux の両方の画像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> ボーナス: は、LCOW を実行して Docker の対応する[ブログ投稿記事](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)を参照してください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルのアプリを作成する方法をについてください。](./building-sample-app.md)