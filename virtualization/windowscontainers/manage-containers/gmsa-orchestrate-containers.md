---
title: GMSA でコンテナーを調整する
description: グループ管理サービスアカウント (gMSA) を使用して Windows コンテナーを調整する方法について説明します。
keywords: docker、コンテナー、active directory、gmsa、オーケストレーション、kubernetes、グループ管理サービスアカウント、グループ管理サービスアカウント
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b4dac775dc7a4ee6375f0d803e921527e66aae5b
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079741"
---
## <a name="orchestrate-containers-with-a-gmsa"></a>GMSA でコンテナーを調整する

運用環境では、多くの場合、アプリとサービスを展開して管理するためにコンテナーオーケストレータを使用します。 各 orchestrator には独自の管理パラダイムがあり、Windows コンテナープラットフォームに渡す資格情報の仕様を受け入れる必要があります。

グループ管理サービスアカウント (gMSAs) でコンテナーをオーケストレーションする場合は、次のことを確認します。

> [!div class="checklist"]
> * "ドメインは参加しています" というコンテナーを実行するようにスケジュールできるすべてのコンテナーホスト
> * コンテナーホストは、コンテナーで使用されるすべての gMSAs パスワードを取得するためのアクセス権を持っています。
> * 資格情報スペックファイルは、orchestrator で処理する方法に応じて、orchestrator に作成され、orchestrator にアップロードされるか、すべてのコンテナーホストにコピーされます。
> * コンテナーネットワークを使うと、コンテナーは Active Directory ドメインコントローラーと通信して gMSA チケットを取得することができます。

### <a name="how-to-use-gmsa-with-service-fabric"></a>Service Fabric で gMSA を使用する方法

アプリケーションマニフェストで資格情報の仕様の場所を指定した場合、Service Fabric は、gMSA を使用して Windows コンテナーの実行をサポートします。 Service Fabric で特定できるように、資格情報スペックファイルを作成し、各ホスト上の Docker データディレクトリの**Credentialspecs**サブディレクトリに配置する必要があります。 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)の一部である、 **credentialspec**コマンドレットを実行して、資格情報の仕様が適切な場所にあるかどうかを確認することができます。

アプリケーションの構成方法の詳細については、「[クイックスタート: windows コンテナーをサービスファブリックに展開する](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)」と「[サービスファブリックで実行されている Windows コンテナーの gMSA を設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)する」を参照してください。

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Docker 群れで gMSA を使用する方法

Docker 群れで管理されるコンテナーと共に gMSA を使用するには、次の`--credential-spec`パラメーターを使用して、 [docker サービスの create](https://docs.docker.com/engine/reference/commandline/service_create/)コマンドを実行します。

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker サービスでの資格情報の仕様の使い方の詳細については、 [docker の群れの例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)を参照してください。

### <a name="how-to-use-gmsa-with-kubernetes"></a>Kubernetes で gMSA を使用する方法

Kubernetes の gMSAs の Windows コンテナーのスケジュール設定は、Kubernetes 1.14 のアルファ機能として利用できます。 この機能に関する最新情報については、「 [Windows ポッドとコンテナー用に gMSA を構成](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)する」と「Kubernetes の配布でテストする方法」を参照してください。

## <a name="next-steps"></a>次のステップ

オーケストレーションコンテナーに加えて、次のような場合にも gMSAs 使うことができます。

- [アプリを構成する](gmsa-configure-app.md)
- [コンテナーを実行する](gmsa-run-container.md)

セットアップ時に問題が発生した場合は、可能な解決策については、[トラブルシューティングガイド](gmsa-troubleshooting.md)を確認してください。
