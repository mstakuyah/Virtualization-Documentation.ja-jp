---
title: Windows Server のコンテナーの更新
description: Windows の複数のバージョン間で、ビルドとコンテナーを実行する方法について説明します。
keywords: メタデータ, コンテナー, バージョン
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 12a60398a12437ea733b2da31aae853a7afd2c57
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2020
ms.locfileid: "79037419"
---
# <a name="update-windows-server-containers"></a>Windows Server のコンテナーの更新

毎月、Windows Server のサービスの一部として、更新された Windows Server ベース OS コンテナーイメージを定期的に公開しています。 これらの更新プログラムを使用すると、更新されたコンテナーイメージの作成を自動化したり、最新バージョンをプルして手動で更新したりすることができます。 Windows Server のコンテナーには、Windows Server のようなサービススタックがありません。 Windows Server の場合と同じように、コンテナー内で更新プログラムを取得することはできません。 そのため、毎月、更新プログラムを使用して Windows Server ベース OS コンテナーイメージを再構築し、更新されたコンテナーイメージを発行します。

.NET や IIS などの他のコンテナーイメージは、更新された基本 OS コンテナーイメージに基づいて再構築され、毎月公開されます。

## <a name="how-to-get-windows-server-container-updates"></a>Windows Server コンテナーの更新プログラムを取得する方法

Windows Server ベース OS コンテナーイメージは、Windows サービスのリズムに合わせて更新されます。 更新されたコンテナーイメージは、毎月第2火曜日に公開されます。これは、"B" リリースと呼ばれることもあり、リリース月に基づくプレフィックス番号が付いています。 たとえば、2月の更新プログラム "2B" と、3月の更新プログラム "3B" を呼び出します。 この月次更新イベントは、新しいセキュリティ修正プログラムを含む唯一の標準リリースです。

コンテナーホストまたは単に "ホスト" と呼ばれるこれらのコンテナーをホストするサーバーは、"B" リリース以外の追加の更新イベント中にサービスを提供できます。 Windows update サービスの期間の詳細については、 [windows update のサービスリズム](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376)に関するブログの投稿を参照してください。

新しい Windows Server ベース OS コンテナーイメージは、Microsoft Container Registry (MCR) の毎月第2火曜日に午前10時 (太平洋標準時) に公開され、おすすめのタグは最新の B リリースを対象としています。 たとえば、次のものがあります。

- ltsc2019 [(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc): docker pull mcr.microsoft.com/windows/servercore:ltsc2019
- 1909 [(SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker pull mcr.microsoft.com/windows/servercore:1909

MCR よりも Docker Hub を使い慣れている方は、こちらの[ブログ投稿](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/)でより詳細な説明を得ることができます。  

各リリースでは、各コンテナーイメージも、特定のコンテナーイメージのリビジョンをターゲットとするために、リビジョン番号とサポート技術情報の記事番号の2つのタグを追加して発行されます。 例 :

- docker pull mcr.microsoft.com/windows/servercore:10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore:1809-KB4546852

これらの例では、どちらも Windows Server 2019 Server Core コンテナーイメージを2月18日のセキュリティリリース更新プログラムでプルします。  

Windows Server ベース OS コンテナーイメージ、バージョン、およびそれぞれのタグの完全な一覧については、Docker Hub のこの[Windows ベース os コンテナーイメージ](https://hub.docker.com/_/microsoft-windows-base-os-images)を参照してください。

Microsoft によって Azure Marketplace でリリースされる月単位のサービス対象の Windows Server イメージには、プレインストールされた基本 OS コンテナーイメージも付属しています。 詳細については、 [Windows Server Azure Marketplace の価格](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice)に関するページを参照してください。 通常、これらのイメージは、"B" リリースから5営業日以内に更新されます。

Windows Server のイメージとバージョンの完全な一覧については、「 [Azure Marketplace の更新履歴の Windows server リリース](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history)」を参照してください。

## <a name="host-and-container-version-compatibility"></a>ホストとコンテナーのバージョンの互換性

Windows コンテナーの分離モードには、プロセスの分離と Hyper-v の分離の2種類があります。 Hyper-v の分離は、ホストとコンテナーのバージョンの互換性に関しては、より柔軟です。 詳細については、「[バージョンの互換性](version-compatibility.md)と[分離モード](../manage-containers/hyperv-container.md)」を参照してください。 このセクションでは、特に明記されていない限り、プロセス分離コンテナーに焦点を当てます。

月単位の更新プログラムを使用してコンテナーホストまたはコンテナーイメージを更新する場合、ホストとコンテナーのイメージが両方ともサポートされていれば (Windows Server バージョン1809以降)、コンテナーを開始するためにホストとコンテナーイメージのリビジョンが一致する必要はありません。正常に実行します。

ただし、2020年2月11日のセキュリティ更新プログラムのリリース ("2B" とも呼ばれます) またはそれ以降の毎月のセキュリティ更新プログラムのリリースで Windows Server コンテナーを使用すると、問題が発生する場合があります。 詳細については、[この Microsoft サポートの記事](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t)をご覧ください。 これらの問題は、アプリケーションのセキュリティを確保するために、ユーザーモードとカーネルモードの間のインターフェイスを変更する必要があるセキュリティの変更に起因します。 この問題が発生するのは、プロセス分離コンテナーでは、コンテナーホストとカーネルモードが共有されるためです。 つまり、更新されたユーザーモードコンポーネントのないコンテナーイメージは、セキュリティで保護されていないため、新しいセキュリティで保護されたカーネルインターフェイスと互換性がありません。

2020年2月18日の時点で修正プログラムをリリースしました。 この新しいリリースでは、"新しい基準" が確立されました。 この新しい基準は、次の規則に従います。

- 2B より前のホストとコンテナーの任意の組み合わせが機能します。
- 2B の後にあるホストとコンテナーの任意の組み合わせが機能します。
- 新しいベースラインの異なる辺にホストとコンテナーを組み合わせて使用することはできません。 たとえば、3B ホストと1B コンテナーは機能しません。

これらの新しい互換性規則がどのように機能するかを示す例として、2020年3月のセキュリティ更新プログラムのリリースを使用してみましょう。 次の表では、2020年3月のセキュリティ更新プログラムのリリースは "3B" と呼ばれていますが、2020更新プログラムは "2B" で、1月の2020更新プログラムは "1B" です。

| ホスト | コンテナー | 互換性 |
|---|---|---|
| 3B | 3B | はい |
| 3B | 2B | はい |
| 3B | 1B 以前 | いいえ |
| 2B | 3B | はい |
| 2B | 2B | はい |
| 2B | 1B 以前 | いいえ |
| 1B 以前 | 3B | いいえ |
| 1B 以前 | 2B | いいえ |
| 1B 以前 | 1B 以前 | はい |

次の表では、Windows Server 2016 から最新の Windows Server バージョン1909リリースまでの異なる主要 OS リリースにわたる、1か月ごとのセキュリティ更新プログラムのリリースを含む、基本 OS コンテナーイメージのバージョン番号を示します。

| Windows Server のバージョン (浮動タグ) | 1/14/20 リリース (1B) の更新プログラムのバージョン| 2/18/20 リリース (2B) の更新プログラムのバージョン | 3/10/20 リリース (3B) の更新プログラムのバージョン |
|---|---|---|---|
| Windows Server 2016 (ltsc2016) | 10.0.14393.3443 (B4534271) | 10.0.14393.3506 (KB4546850) | 今後の予定日にリリースされるコンテナー |
| Windows Server バージョン 1803 (1803) | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | このバージョンのサポートが終了しました。 詳細については、「[基本イメージサービスライフサイクル](base-image-lifecycle.md)」を参照してください。|
| Windows Server 2019 (ltsc2019) | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server バージョン 1903 (1903) |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server バージョン 1909 (1909) | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>ホストとコンテナーイメージの不一致のトラブルシューティング

開始する前に、[バージョンの互換性](version-compatibility.md)に関する情報をよく理解しておいてください。 この情報は、修正プログラムの不一致によって問題が発生したかどうかを判断するのに役立ちます。 原因に一致しない修正プログラムがある場合は、このセクションの手順に従って問題のトラブルシューティングを行うことができます。

### <a name="query-the-version-of-your-container-host"></a>コンテナーホストのバージョンを照会する

コンテナーホストにアクセスできる場合は、`ver` コマンドを実行して、その OS バージョンを取得できます。 たとえば、最新の Feb 2020 セキュリティ更新プログラムのリリースで Windows Server 2019 を実行しているシステムで `ver` を実行すると、次のような結果が表示されます。

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>この例のリビジョン番号は1040ではなく1039と表示されます。これは、Windows Server 用の帯域外の2B リリースが、2 2020 月のセキュリティ更新プログラムのリリースに含まれていなかったためです。 コンテナーには帯域外の2B リリースのみがありました。これはリビジョン番号が1040でした。

コンテナーホストに直接アクセスできない場合は、IT 管理者に確認してください。クラウドで実行している場合は、クラウドプロバイダーの web サイトで、実行されているコンテナーホスト OS バージョンを確認してください。 たとえば、Azure Kubernetes Service (AKS) を使用している場合は、 [AKS のリリースノート](https://github.com/Azure/AKS/releases)でホスト OS のバージョンを見つけることができます。

### <a name="query-the-version-of-your-container-image"></a>コンテナーイメージのバージョンを照会する

次の手順に従って、コンテナーが実行されているバージョンを確認します。

1. PowerShell で次のコマンドレットを実行します。

    ```powershell
    docker images
    ```

    出力は次のようになります。

     ```powershell
     REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
     mcr.microsoft.com/windows/servercore   ltsc2019            b456290f487c        4 weeks ago         4.84GB
     mcr.microsoft.com/windows              1809                58229ca44fa7        4 weeks ago         12GB
     mcr.microsoft.com/windows/nanoserver   1809                f519d4f3a868        4 weeks ago         251M

2. Run the `docker inspect` command for the Image ID of the container image that isn't working. This will tell you which version the container image is targeting.

   For example, let's say we `run docker inspect` for an ltsc 2019 container image:

   ```powershell
   docker inspect b456290f487c

       "Architecture": "amd64",

        "Os": "windows",

        "OsVersion": "10.0.17763.1039",

        "Size": 4841309825,

        "VirtualSize": 4841309825,
    ```

    この例では、コンテナーの OS バージョンが `10.0.17763.1039`として表示されます。

    コンテナーを既に実行している場合は、コンテナー内で `ver` コマンドを実行してバージョンを取得することもできます。 たとえば、Windows Server 2019 の Server Core コンテナーイメージで、最新の Feb 2020 セキュリティ更新プログラムのリリースを使用して `ver` を実行すると、次のような内容が表示されます。

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```
