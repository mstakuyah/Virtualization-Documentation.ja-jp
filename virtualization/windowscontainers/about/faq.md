---
title: "Windows コンテナーに関する FAQ"
description: "Windows コンテナーに関する FAQ"
keywords: "Docker, コンテナー"
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 68f563d62090097b6fe2dd335c7841ae5ba4a9f5

---

# よく寄せられる質問

## Windows コンテナーについて

**Windows Server コンテナーとは**

Windows Server コンテナーは、同じコンテナー ホストで実行されている他のサービスからアプリケーションまたはサービスを分離するために使用される、軽量のオペレーティング システム仮想化の方法です。 これを可能にするため、各コンテナーには、オペレーティング システム、プロセス、ファイル システム、レジストリ、IP アドレスの独自のビューがあります。  

**Hyper-V コンテナーとは**

Hyper-V コンテナーは、Hyper-V パーティション内で実行される Windows Server コンテナーと考えることができます。

HYPER-V のコンテナーには、非常に効率的で高密度の Windows Server のコンテナーと、分離性の高いハードウェア仮想化-HYPER-V バーチャル マシンの間追加の展開オプションが提供されています。 さまざまなアプリケーションが、同じホスト上の境界を信頼環境では、追加の分離が必要な可能性があります。 HYPER-V のコンテナーでは、最適化された仮想化と、ホストのオペレーティング システムからは、コンテナーを分離する、Windows Server オペレーティング システムを使用して、高い分離を提供します。 どちらのコンテナー展開オプションも展開時に同じ管理 API、ツール、イメージ形式を利用しているため、お客様はご自身の要件に最適な展開モードを選択するだけで済みます。

**Linux コンテナーと Windows Server コンテナーの違いは何ですか。**

Linux コンテナーと Windows Server コンテナーは似ています。どちらも同様のテクノロジをカーネルとコア オペレーティング システム内に実装します。 違いは、プラットフォームと、コンテナー内で実行されるワークロードに由来します。  
顧客が Windows Server のコンテナーを使用する場合は、.NET、ASP.NET、PowerShell などの既存の Windows テクノロジと統合できます。

**開発者ですが、自作のアプリケーションを、コンテナーの種類ごとに記述し直す必要はありますか。**

いいえ。Windows コンテナー イメージは Windows Server コンテナーと Hyper-V コンテナーで共通です。 コンテナーを開始すると、コンテナーの種類の選択が行われます。 開発者の観点では、Windows Server のコンテナーおよびコンテナーの HYPER-V は、同じことの 2 つの種類があります。  オープンで拡張可能なは、および同じレベルの統合と Docker 経由でのサポートが含まれますが、同じ開発、プログラミング、および管理のエクスペリエンスを提供します。

開発者は、Windows Server のコンテナーを使用して、コンテナーのイメージを作成し、HYPER-V コンテナーまたはその逆の適切な実行時フラグを指定する以外には、何も変更せずとして展開できます。

Windows Server のコンテナーは、密度とパフォーマンス (入れ子になった構成と比較して、時間、高速なランタイムのパフォーマンスの低いスピンなど) が格段に向上が実現の速度がキーの場合。 HYPER-V のコンテナーでは、1 つのコンテナーで実行されるコードが危険にさらすことはできないかどうか、または、ホスト オペレーティング システムまたは同じホストで実行されている他のコンテナーに影響を与えることを保証する分離を向上を提供しています。 これは、SaaS アプリケーションやコンピューティングのホストなど、(信頼できないコードをホストするための要件を持つ) マルチテナント シナリオに便利です。

**Hyper-V/Windows Server コンテナーはアドオンですか、それとも Windows Server に統合されるものですか。**

コンテナー機能は Windows Server 2016 に統合されます。 次回にご期待の一般提供する詳細については近い。  

**Windows Server コンテナーと Drawbridge の関係はどのようなものですか?**

Drawbridge は、コンテナーに関する貴重な情報を取得するのに役立つ多くの研究プロジェクトのうちの 1 つでした。  Windows Server 2016 のコンテナー テクノロジの多くは、Drawbridge における経験から生み出されました。Windows Server 2016 で、世界クラスのコンテナー テクノロジをお客様にご提供いたします。

**Windows Server コンテナーおよび Hyper-V コンテナーの前提条件は何ですか。**

Windows Server コンテナーと Hyper-V コンテナーにはどちらも Windows Server 2016 が必要です。 これらのテクノロジは、以前のバージョンの Windows では機能しません。


## Windows コンテナー管理

**Hyper-V コンテナーは、Docker エコシステムでも利用できるようになりますか。**

はい。Hyper-V コンテナーは、Docker を使用して Windows Server コンテナーと同じレベルの統合と管理を提供します。  目標は、開く、一貫性のある、クロスプラット フォーム エクスペリエンスです。  
Docker プラットフォームを大幅に簡略化し、コンテナー オプション間で作業のエクスペリエンスを強化します。 Windows Server のコンテナーを使用して開発されたアプリケーションは、変更されることがなく、HYPER-V のコンテナーとして展開することができます。


## Microsoft のオープン エコシステム

**Microsoft は Open Container Initiative (OCI) に参加していますか。**

パッケージ形式の普遍性の維持を保証するため、Docker は先頃、コンテナー パッケージのオープン性と基盤主導の形式を維持することを目的に、Open Container Initiative (OCI) を組織しました。Microsoft は OCI の創設メンバーです。

**Microsoft は Docker と本当に提携しているのですか。**

はい。  
Docker とのパートナーシップでは、作成、管理、および同じ Docker ツール セットを使用して、Windows Server と Linux の両方のコンテナーを展開することができます。 Windows Server を対象とした開発者は、Windows Server のさまざまなテクノロジを使用して、コンテナー化アプリケーションの構築との間を選択するしなくなります。  

Docker は次の 2 つのプロジェクトおよび Docker、会社のオープン ソースのグループです。 私たちは、両方を含めるには、このパートナーシップを検討してください。 Docker 成功すると、一部をためは Docker コンテナー テクノロジに関する作成が強力なエコシステムです。 Docker プロジェクトは、Windows Server のコンテナーおよびコンテナーの HYPER-V のサポートを有効にするのには、Microsoft が関係しています。  

詳細については、ブログの投稿「[New Windows Server containers and Azure support for Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD)」 (新しい Windows Server コンテナーおよび Azure による Docker のサポート) を参照してください。



<!--HONumber=Sep16_HO4-->


