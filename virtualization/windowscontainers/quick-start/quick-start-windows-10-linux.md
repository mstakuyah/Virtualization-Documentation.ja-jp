---
title: Windows 10 の windows コンテナーと Linux コンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909582"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 の Linux コンテナー

この演習では、Windows 10 での Linux コンテナーの作成と実行について説明します。

このクイックスタートでは、次の作業を行います。

1. Docker Desktop のインストール
2. Windows 上の Linux コンテナーを使用した単純な Linux コンテナーの実行 (LCOW)

このクイック スタートは、Windows 10 に固有です。 その他のクイックスタートドキュメントは、このページの左側にある目次にあります。

## <a name="prerequisites"></a>前提条件

次の要件を満たしていることを確認してください。
- Windows 10 Professional、Windows 10 Enterprise、または Windows Server 2019 バージョン1809以降を実行している1台の物理コンピューターシステム
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)が有効になっていることを確認します。

***Hyper-v の分離:*** Windows 上の linux コンテナーでは、開発者が適切な Linux カーネルを使用してコンテナーを実行するために、Windows 10 で Hyper-v を分離する必要があります。 Hyper-v の分離の詳細については、「 [Windows コンテナーについて](../about/index.md)」ページを参照してください。

## <a name="install-docker-desktop"></a>Docker Desktop のインストール

[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードし、インストーラーを実行します (ログインする必要があります)。 アカウントをまだ作成していない場合は作成します)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

> Docker が既にインストールされている場合は、LCOW をサポートするためにバージョン18.02 以降がインストールされていることを確認してください。 `docker -v` を実行するか、 *Docker について*確認してください。

> LCOW コンテナーを実行するには、 *Docker 設定 > デーモン*の [試験的な機能] オプションをアクティブにする必要があります。

## <a name="run-your-first-lcow-container"></a>最初の LCOW コンテナーを実行する

この例では、BusyBox コンテナーがデプロイされます。 最初に、' Hello World ' BusyBox イメージを実行しようとします。

```console
docker run --rm busybox echo hello_world
```

Docker がイメージをプルしようとするとエラーが返されることに注意してください。 これは、行えが `--platform` フラグを使用して、イメージとホストオペレーティングシステムが適切に一致していることを確認するディレクティブが必要であるために発生します。 Windows コンテナーモードの既定のプラットフォームは Windows であるため、コンテナーをプルして実行するには `--platform linux` フラグを追加します。

```console
docker run --rm --platform linux busybox echo hello_world
```

指定されたプラットフォームでイメージがプルされると、`--platform` フラグは不要になります。 これをテストするためにコマンドを指定せずに実行します。

```console
docker run --rm busybox echo hello_world
```

`docker images` を実行して、インストールされているイメージの一覧を返します。 この場合は、Windows と Linux の両方のイメージです。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> おまけ: LCOW の実行については、Docker の対応する[ブログ](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)記事をご覧ください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルアプリを構築する方法について説明します](./building-sample-app.md)
