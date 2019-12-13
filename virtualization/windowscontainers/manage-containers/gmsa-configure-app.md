---
title: グループの管理されたサービスアカウントを使用するようにアプリを構成する
description: Windows コンテナーに対してグループの管理されたサービスアカウント (gMSAs) を使用するようにアプリを構成する方法について説明します。
keywords: docker、コンテナー、active directory、gmsa、アプリ、アプリケーション、グループの管理されたサービスアカウント、グループの管理されたサービスアカウント、構成
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909792"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>GMSA を使用するようにアプリを構成する

一般的な構成では、コンテナーには、コンテナーコンピューターアカウントがネットワークリソースに対して認証を試みるたびに使用されるグループ管理サービスアカウント (gMSA) が1つだけ与えられます。 これは、gMSA id を使用する必要がある場合に、アプリを**ローカルシステム**または**ネットワークサービス**として実行する必要があることを意味します。

## <a name="run-an-iis-app-pool-as-network-service"></a>IIS アプリプールをネットワークサービスとして実行する

コンテナーで IIS web サイトをホストしている場合は、gMSA を利用するために必要なことは、アプリケーションプール id を**Network Service**に設定することだけです。 これは、次のコマンドを追加することによって、Dockerfile で行うことができます。

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

以前に IIS アプリプールに静的ユーザー資格情報を使用していた場合は、それらの資格情報の代わりとして gMSA を検討してください。 開発環境、テスト環境、運用環境の間で gMSA を変更できます。 IIS は、コンテナーイメージを変更しなくても、現在の id を自動的に取得します。

## <a name="run-a-windows-service-as-network-service"></a>Windows サービスをネットワークサービスとして実行する

コンテナー化されたアプリが Windows サービスとして実行される場合は、Dockerfile で**Network service**として実行するようにサービスを設定できます。

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>任意のコンソールアプリをネットワークサービスとして実行する

IIS または Service Manager でホストされていない汎用コンソールアプリでは、多くの場合、gMSA コンテキストをアプリが自動的に継承するように、コンテナーを**Network Service**として実行するのが最も簡単です。 この機能は、Windows Server バージョン1709以降で使用できます。

次の行を Dockerfile に追加して、既定で Network Service として実行されるようにします。

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

`docker exec`を使用すると、1回限りのネットワークサービスとしてコンテナーに接続することもできます。 これは、コンテナーが通常はネットワークサービスとして実行されていない場合に、実行中のコンテナーでの接続の問題をトラブルシューティングする場合に特に便利です。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>次のステップ

アプリを構成するだけでなく、gMSAs を使用して次のことを行うこともできます。

- [コンテナーの実行](gmsa-run-container.md)
- [コンテナーの調整](gmsa-orchestrate-containers.md)

セットアップ中に問題が発生した場合は、[トラブルシューティングガイド](gmsa-troubleshooting.md)で解決策を確認してください。
