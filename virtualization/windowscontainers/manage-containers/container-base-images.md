---
title: Windows コンテナーの基本イメージ
description: Windows コンテナーの基本イメージの概要と、それらをいつ使用するかを説明します。
keywords: docker, コンテナー, ハッシュ
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909782"
---
# <a name="container-base-images"></a>コンテナーの基本イメージ

Windows には、ユーザーが作成できる4つのコンテナー基本イメージが用意されています。 各基本イメージは、Windows OS のさまざまなフレーバーで、ディスク上の別のフットプリントがあり、Windows API セットの量が異なります。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>Windows Server Core 
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">
                    <div class="has-padding-bottom-none">
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small"></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>は、従来の .NET framework アプリケーションをサポートしています。</p>
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
                    </div>Nano Server 
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">
                    <div class="has-padding-bottom-none">の 
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small"></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>.NET Core アプリケーション用に構築されています。</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">ウィンドウ</h3>
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
                    </div>Windows IoT Core 
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none"></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>IoT アプリケーション用に構築されています。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>イメージの検出

すべての Windows コンテナーの基本イメージは、 [Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images)を使用して検出できます。 Windows コンテナーの基本イメージ自体は、Microsoft Container Registry (MCR) の[mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/)から提供されます。 このため、Windows コンテナーの基本イメージのプルコマンドは次のようになります。

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR には独自のカタログエクスペリエンスがありません。 Docker Hub などの既存のカタログをサポートするように設計されています。 Azure の世界規模のフットプリントと Azure CDN との連携により、MCR は、一貫性のある高速なイメージのプルエクスペリエンスを提供します。 Azure のお客様は、Azure でワークロードを実行することで、ネットワーク内のパフォーマンスを向上させることができるだけでなく、MCR (Microsoft コンテナーイメージのソース)、Azure Marketplace、および Azure で提供されるサービスの数を増やすことで、展開パッケージ形式としてのコンテナー。

## <a name="choosing-a-base-image"></a>基本イメージの選択

構築する適切な基本イメージを選択するにはどうすればよいですか。 ほとんどのユーザーは、`Windows Server Core` と `Nanoserver` を使用するのが最適なイメージになります。

### <a name="guidelines"></a>ガイドライン

 希望するイメージを自由に選択できますが、次のガイドラインを参考にしてください。

- **アプリケーションに完全な .NET framework が必要ですか。** この質問に対する回答が「はい」の場合は、`Windows Server Core`を対象にする必要があります。
- **.NET Core に基づいて Windows アプリを構築していますか。** この質問に対する回答が「はい」の場合は、`Nanoserver`を対象にする必要があります。
- **IoT アプリケーションを構築していますか。** この質問に対する回答が「はい」の場合は、`IoT Core`を対象にする必要があります。
- **Windows Server Core コンテナーイメージにアプリに必要な依存関係がありませんか。** この質問に対する回答が「はい」の場合は、`Windows`のターゲットを指定する必要があります。 このイメージは、他の基本イメージよりもはるかに大きくなっていますが、多くのコア Windows ライブラリ (GDI ライブラリなど) が含まれています。
- **Windows Insider ですか。** 「はい」の場合、insider バージョンのイメージを使用することを検討してください。 次の「Windows insider の基本イメージ」を参照してください。

> [!TIP]
> 多くの Windows ユーザーは、.NET に依存しているアプリケーションをコンテナー化する必要があります。 Microsoft では、ここで説明する4つの基本イメージに加えて、 [.net framework](https://hub.docker.com/_/microsoft-dotnet-framework)イメージや[ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)イメージなど、一般的な Microsoft フレームワークで事前構成されたいくつかの Windows コンテナーイメージを発行します。

### <a name="base-images-for-windows-insiders"></a>Windows Insider の基本イメージ

Microsoft では、各コンテナーの基本イメージの "insider" バージョンを提供しています。 これらの内部コンテナーイメージは、コンテナーイメージで最新で最高の機能開発を行います。 Insider バージョンの Windows (Windows Insider または Windows Server Insider) のホストを実行している場合は、これらのイメージを使用することをお勧めします。 Insider イメージは、Docker Hub で入手できます。

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

詳細につい[ては、「Windows Insider プログラムでコンテナーを使用](../deploy-containers/insider-overview.md)する」を参照してください。

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core と Nanoserver

`Windows Server Core` と `Nanoserver` は、対象となる最も一般的な基本イメージです。 これらのイメージの主な違いは、Nanoserver の API サーフェイスが大幅に小さくなる点です。 PowerShell、WMI、および Windows サービススタックは、Nanoserver イメージには存在しません。

Nanoserver は、.NET core またはその他の最新のオープンソースフレームワークに依存するアプリを実行するための十分な API サーフェイスを提供するように構築されました。 小さい APi サーフェイスのトレードオフとして、Nanoserver イメージは、Windows の基本イメージの他の部分よりも、ディスク上のフットプリントが大幅に小さくなります。 Nano Server 上には、いつでもレイヤーを追加できることに注意してください。 この例については、[.NET Core の Nano Server の Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile) をご覧ください。
