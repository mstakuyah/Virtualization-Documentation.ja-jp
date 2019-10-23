---
title: Windows コンテナーの基本イメージ
description: Windows コンテナーの基本イメージの概要と、その使用方法を説明します。
keywords: docker, コンテナー, ハッシュ
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254272"
---
# <a name="container-base-images"></a>コンテナーの基本イメージ

Windows では、ユーザーが作成できるコンテナーベースの画像が4つ用意されています。 各基本イメージは、Windows OS のさまざまなフレーバーであり、ディスク上のメモリ領域が異なり、Windows API セットのさまざまな量が含まれています。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>従来の .NET framework アプリケーションをサポートしています。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>.NET コアアプリケーション向けに開発されています。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>完全な Windows API セットを提供します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>IoT アプリケーション向けに設計された用途</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>イメージの検出

すべての Windows コンテナーベース画像は、 [Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images)から見つけることができます。 Windows コンテナーの基本イメージは、 [mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/)、Microsoft container Registry (mcr) から提供されます。 Windows コンテナーベースの画像の pull コマンドは、次のようになります。

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR には、独自のカタログ操作はありません。また、Docker Hub などの既存のカタログをサポートすることを目的としています。 Azure のグローバルなフットプリントと Azure CDN を組み合わせることで、MCR は一貫して高速のイメージプルエクスペリエンスを提供します。 Azure でのワークロードの実行、azure での作業負荷の実行、ネットワーク内でのパフォーマンスの強化、および MCR (Microsoft コンテナイメージのソース) との緊密な統合、Azure Marketplace、および提供されている Azure でのサービスの拡張数の向上展開パッケージ形式としてのコンテナー。

## <a name="choosing-a-base-image"></a>基本イメージの選択

作成する適切な基本イメージを選択する方法を教えてください。 ほとんどのユーザーの`Windows Server Core`場合`Nanoserver`は、最も適切な画像が使用されます。

### <a name="guidelines"></a>ガイドライン

 任意の画像をターゲットにすることはできますが、次のガイドラインを参考にしてください。

- **アプリケーションには完全な .NET framework が必要ですか?** この質問への回答が「はい」の場合は`Windows Server Core`、ターゲットを指定する必要があります。
- **.NET Core に基づいて Windows アプリを構築していますか?** この質問への回答が「はい」の場合は`Nanoserver`、ターゲットを指定する必要があります。
- **IoT アプリケーションを構築していますか?** この質問への回答が「はい」の場合は`IoT Core`、ターゲットを指定する必要があります。
- **Windows Server Core コンテナの画像にアプリの依存関係がありませんか?** この質問への回答が「はい」の場合は、ターゲット`Windows`を指定してください。 この画像は、他の基本イメージよりも大幅に大きくなっていますが、主要な Windows ライブラリの多く (GDI ライブラリなど) を備えています。
- **Windows Insider の場合** [はい] の場合は、insider バージョンの画像の使用を検討してください。 以下の「Windows insider 用の基本イメージ」を参照してください。

> [!TIP]
> 多くの Windows ユーザーは、.NET に依存するアプリケーションを containerize したいと思います。 ここで説明する4つの基本イメージに加えて、Microsoft では、 [.net framework](https://hub.docker.com/_/microsoft-dotnet-framework)イメージや[ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)イメージなどの一般的な Microsoft フレームワークで事前構成されているいくつかの Windows コンテナーイメージを公開しています。

### <a name="base-images-for-windows-insiders"></a>Windows Insider の基本イメージ

Microsoft には、各コンテナーの基本イメージの "insider" バージョンが用意されています。 これらの insider コンテナーの画像には、コンテナーイメージ内の最新の最大機能開発が含まれています。 Insider バージョンの Windows (Windows Insider または Windows Server Insider) であるホストを実行している場合は、これらの画像を使うことをお勧めします。 Insider の画像は、Docker Hub で入手できます。

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

詳細につい[ては、「Windows Insider プログラムでコンテナーを使用](../deploy-containers/insider-overview.md)する」を参照してください。

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs Nanoserver

`Windows Server Core` これ`Nanoserver`は、ターゲットとなる最も一般的な基本イメージです。 これらの画像の主な違いは、Nanoserver の API サーフェスが大幅に小さくなることです。 PowerShell、WMI、および Windows サービススタックが Nanoserver のイメージから省略されています。

Nanoserver は、.NET core またはその他のモダンオープンソースフレームワークに依存するアプリを実行するために十分な API サーフェイスを提供するように設計されています。 小さい APi サーフェスに対するトレードオフとして、Nanoserver の画像は、Windows の基本イメージの他の領域よりも、ディスクの使用量が大幅に小さくなります。 Nano Server 上には、いつでもレイヤーを追加できることに注意してください。 この例については、[.NET Core の Nano Server の Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile) をご覧ください。
