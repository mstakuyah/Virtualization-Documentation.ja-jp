---
title: グループ管理サービスアカウントを使用するようにアプリを構成する
description: Windows コンテナーのグループ管理サービスアカウント (gMSAs) を使用するようにアプリを構成する方法について説明します。
keywords: docker、コンテナー、active directory、gmsa、アプリ、アプリケーション、グループ管理サービスアカウント、グループ管理サービスアカウント、構成
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 234909d7f0cb0f30ee7fbf4796dd0381bfbff89f
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079749"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>GMSA を使用するようにアプリを構成する

一般的な構成では、コンテナーには、コンテナーコンピューターアカウントがネットワークリソースへの認証を試みるときに使用される、1つのグループ管理サービスアカウント (gMSA) のみが含まれます。 つまり、gMSA id を使用する必要がある場合は、アプリを**ローカルシステム**または**ネットワークサービス**として実行する必要があります。

## <a name="run-an-iis-app-pool-as-network-service"></a>IIS アプリプールをネットワークサービスとして実行する

コンテナーで IIS web サイトをホストしている場合、gMSA を利用するために必要な操作は、アプリプール id を**Network Service**に設定することだけです。 次のコマンドを追加して、Dockerfile でその操作を行うことができます。

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

以前に IIS アプリプールに対して静的ユーザー資格情報を使用したことがある場合は、それらの資格情報の代わりとして gMSA を検討してください。 Dev、テスト、および運用環境の間で gMSA を変更することができ、IIS はコンテナーイメージを変更せずに、自動的に現在の id を取得します。

## <a name="run-a-windows-service-as-network-service"></a>Windows サービスをネットワークサービスとして実行する

コンテナーアプリが Windows サービスとして実行されている場合は、Dockerfile で**Network service**として実行するようにサービスを設定することができます。

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>ネットワークサービスとして任意のコンソールアプリを実行する

IIS または Service Manager でホストされていない一般的なコンソールアプリの場合、コンテナーを**ネットワークサービス**として実行すると、アプリは自動的に gMSA コンテキストを継承するため、最も簡単です。 この機能は、Windows Server バージョン1709で利用できます。

既定でネットワークサービスとして実行するには、次の行を Dockerfile に追加します。

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

また、を使用`docker exec`して、コンテナーにネットワークサービスとして接続することもできます。 これは、コンテナーが通常ネットワークサービスとして実行されていない場合に、実行中のコンテナーでの接続の問題をトラブルシューティングする場合に特に役立ちます。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>次のステップ

アプリを構成するだけでなく、gMSAs 使用することもできます。

- [コンテナーを実行する](gmsa-run-container.md)
- [オーケストレーションコンテナー](gmsa-orchestrate-containers.md)

セットアップ時に問題が発生した場合は、可能な解決策については、[トラブルシューティングガイド](gmsa-troubleshooting.md)を確認してください。
