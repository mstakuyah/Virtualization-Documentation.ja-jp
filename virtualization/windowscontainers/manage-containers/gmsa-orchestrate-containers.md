---
title: GMSA を使用してコンテナーを調整する
description: グループの管理されたサービスアカウント (gMSA) を使用して Windows コンテナーを調整する方法。
keywords: docker、コンテナー、active directory、gmsa、orchestration、kubernetes、グループの管理されたサービスアカウント、グループの管理されたサービスアカウント
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910262"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>GMSA を使用してコンテナーを調整する

運用環境では、多くの場合、コンテナー orchestrator を使用してアプリとサービスをデプロイし、管理します。 各 orchestrator には独自の管理パラダイムがあり、Windows コンテナープラットフォームに渡す資格情報の仕様を受け入れる役割を担います。

グループの管理されたサービスアカウント (gMSAs) を使用してコンテナーを調整している場合は、次のことを確認してください。

> [!div class="checklist"]
> * GMSAs がドメインに参加しているコンテナーを実行するようにスケジュールできるすべてのコンテナーホスト
> * コンテナーホストは、コンテナーが使用するすべての gMSAs パスワードを取得するためのアクセス権を持っています。
> * 資格情報の仕様ファイルは、orchestrator がどのように処理するかに応じて、orchestrator に作成されてアップロードされるか、すべてのコンテナーホストにコピーされます。
> * コンテナーネットワークを使用すると、コンテナーは Active Directory ドメインコントローラーと通信して gMSA チケットを取得することができます。

## <a name="how-to-use-gmsa-with-service-fabric"></a>Service Fabric で gMSA を使用する方法

アプリケーションマニフェストで資格情報の仕様の場所を指定すると、Service Fabric では gMSA を使用した Windows コンテナーの実行がサポートされます。 資格情報の仕様ファイルを作成し、各ホストの Docker データディレクトリの**Credentialspecs**サブディレクトリに配置して、Service Fabric が検出できるようにする必要があります。 資格情報の仕様が正しい場所にあるかどうかを確認するには、 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)の一部である**Get credentialspec**コマンドレットを実行します。

アプリケーションを構成する方法の詳細については、「[クイックスタート: Service Fabric に windows コンテナーを展開する](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)」および「 [Service Fabric で実行されている Windows コンテナーの gMSA を設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)する」を参照してください。

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Docker の群れで gMSA を使用する方法

Docker 群れで管理されているコンテナーで gMSA を使用するには、`--credential-spec` パラメーターを指定して[docker service create](https://docs.docker.com/engine/reference/commandline/service_create/)コマンドを実行します。

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker サービスで資格情報の仕様を使用する方法の詳細については、 [docker の群れの例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)を参照してください。

## <a name="how-to-use-gmsa-with-kubernetes"></a>Kubernetes で gMSA を使用する方法

Kubernetes の gMSAs での Windows コンテナーのスケジュール設定のサポートは、Kubernetes 1.14 のアルファ機能として利用できます。 この機能に関する最新情報と、Kubernetes 配布でテストする方法については、「 [Configure gMSA For Windows ポッド and コンテナー](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) 」を参照してください。

## <a name="next-steps"></a>次のステップ

コンテナーを調整するだけでなく、gMSAs を使用して次のことを行うこともできます。

- [アプリの構成](gmsa-configure-app.md)
- [コンテナーの実行](gmsa-run-container.md)

セットアップ中に問題が発生した場合は、[トラブルシューティングガイド](gmsa-troubleshooting.md)で解決策を確認してください。
