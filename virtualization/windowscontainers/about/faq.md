---
title: Windows コンテナーに関する FAQ
description: Windows コンテナーに関する FAQ
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 7ac5622ad0f6767fa8d55798e0bf99b84f0d0a2c
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973702"
---
# <a name="frequently-asked-questions"></a>よく寄せられる質問

## <a name="general"></a>全般的な情報

### <a name="what-is-wcow-what-is-lcow"></a>WCOW とは何ですか。 LCOW とは何ですか。

WCOW の Windows および LCOW Windows コンテナーを省略形は Windows の Linux コンテナーを省略形です。

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Linux コンテナーと Windows Server コンテナーの違いは何ですか。

Linux コンテナーと Windows Server コンテナーは似ています。どちらも同様のテクノロジをカーネルとコア オペレーティング システム内に実装します。 違いは、プラットフォームと、コンテナー内で実行されるワークロードに由来します。  
顧客が Windows Server のコンテナーを使用する場合は、.NET、ASP.NET、PowerShell などの既存の Windows テクノロジと統合できます。

### <a name="as-a-developer-do-i-have-to-re-write-my-app-for-each-type-of-container"></a>開発者ですが、自作のアプリケーションを、コンテナーの種類ごとに記述し直す必要はありますか。

いいえ。Windows コンテナー イメージは Windows Server コンテナーと Hyper-V コンテナーで共通です。 コンテナーを開始すると、コンテナーの種類の選択が行われます。 開発者の観点では、Windows Server のコンテナーおよびコンテナーの HYPER-V は、同じことの 2 つの種類があります。 オープンで拡張可能なは、および同じレベルの統合と Docker 経由でのサポートが含まれますが、同じ開発、プログラミング、および管理のエクスペリエンスを提供します。

開発者は、Windows Server のコンテナーを使用して、コンテナーのイメージを作成し、HYPER-V コンテナーまたはその逆の適切な実行時フラグを指定する以外には、何も変更せずとして展開できます。

Windows Server のコンテナーは、密度とパフォーマンス (入れ子になった構成と比較して、時間、高速なランタイムのパフォーマンスの低いスピンなど) が格段に向上が実現の速度がキーの場合。 HYPER-V のコンテナーでは、1 つのコンテナーで実行されるコードが危険にさらすことはできないかどうか、または、ホスト オペレーティング システムまたは同じホストで実行されている他のコンテナーに影響を与えることを保証する分離を向上を提供しています。 これは、SaaS アプリケーションやコンピューティングのホストなど、(信頼できないコードをホストするための要件を持つ) マルチテナント シナリオに便利です。

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>コンテナーの Windows を実行するための前提条件とは

Windows Server 2016 から開始するプラットフォームにコンテナーが導入されました。 Windows Server 2016 または Windows 10 の記念日を実行する必要があります (バージョン 1607) の更新またはコンテナーを使用して新しいします。

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>実行できます Windows コンテナー プロセス分離モードで Windows 10 のエンタープライズまたは Professional のですか。

以降で Windows 10 年 2018年 10 月されなくなったは、更新プロセス分離で Windows コンテナーを実行しているユーザーを禁止します。 直接を使用してプロセスを分離する要求する必要があります、`--isolation=process`を使って、コンテナーの実行時のフラグを設定する`docker run`します。

場合に興味のあるものは、エンジン 18.09 docker バージョンを使用して、ホストには、Windows 10 ビルド 17763 + が実行されていることを確認または新しいする必要があります。

> [!WARNING]
> この機能は、開発/テスト用ものだけです。 ホストと Windows Server を使用して、製品の展開を続行する必要があります。
>
> この機能を使用するも必ずと一致する、ホストとコンテナー バージョン タグを開始しますコンテナーが失敗する場合は、または未定義の動作が発生することができます。

## <a name="windows-container-management"></a>Windows コンテナー管理

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>作成する方法のコンテナーの画像利用可能な air gapped マシンでよいですか。

Windows コンテナーの基本イメージには、ライセンスの分布が制限されているアイテムが含まれています。 構成でデータ バインド画像をプライベートまたはパブリック レジストリにプッシュ配信すると、基本レイヤーはプッシュことがないことがわかります。 代わりに、実際の基本 Azure クラウド ストレージ内に存在するレイヤーを指す外部レイヤーの概念を使用します。

_プライベート コンテナー レジストリ_のアドレスから取得画像_のみ_を air gapped のコンピューターがある場合は、問題が発生することができます。 ここでは外部基本イメージを取得するレイヤーをフォローする試行は失敗します。 外部のレイヤーの動作をオーバーライドするを使用できます、 `--allow-nondistributable-artifacts` Docker デーモンにフラグを設定します。

> [!IMPORTANT]
> このフラグの使用には、Windows コンテナーの基本イメージ ライセンス; の条項を遵守する義務は不能します。パブリックまたはサード パーティの再配布できるように Windows のコンテンツを投稿する必要があります。 独自の環境での使用が許可されています。

## <a name="microsofts-open-ecosystem"></a>Microsoft のオープン エコシステム

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft は Open Container Initiative (OCI) に参加していますか。

パッケージ形式の普遍性の維持を保証するため、Docker は先頃、コンテナー パッケージのオープン性と基盤主導の形式を維持することを目的に、Open Container Initiative (OCI) を組織しました。Microsoft は OCI の創設メンバーです。

> ![ヒント][よく寄せられる質問への追加の推奨事項があるか。 新しいフィードバックの問題の下にあることをお勧めお場合です。 または、推奨事項のこれらのドキュメントに対して、現在価値を開く
