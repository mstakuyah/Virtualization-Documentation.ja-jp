---
title: Windows コンテナーに関する FAQ
description: Windows Server コンテナーよく寄せられる質問
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 69783f0fc3dcc80eb9614031dc6c9b2c35eeefd1
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380236"
---
# <a name="frequently-asked-questions"></a>よく寄せられる質問

## <a name="general"></a>全般的な情報

### <a name="what-is-wcow-what-is-lcow"></a>WCOW とは何ですか。 LCOW とは何ですか。

WCOW の Windows および LCOW Windows コンテナーを省略形は Windows の Linux コンテナーを省略形です。

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Linux コンテナーと Windows Server コンテナーの違いは何ですか。

Linux および Windows Server コンテナーは、カーネルとコア オペレーティング システム内のようなテクノロジを実装、両方に似ています。 違いは、プラットフォームと、コンテナー内で実行されるワークロードに由来します。  

お客様は、Windows Server コンテナーを使用する場合に統合することで既存の Windows .NET、ASP.NET、PowerShell などの技術します。

### <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>開発者が必要なコンテナーの種類ごとにアプリを改訂ですか。

できません。 Windows コンテナーの画像は、Windows Server コンテナーと HYPER-V 分離の両方で共通です。 コンテナーを開始すると、コンテナーの種類の選択が行われます。 開発の観点では、Windows Server コンテナーと HYPER-V 分離は同じの 2 種類があります。 同じの開発、プログラミングおよび管理のエクスペリエンスを提供する、開く] および [拡張、統合と Docker サポートの同じレベルに含めます。

開発者は、Windows Server コンテナーを使用してコンテナー イメージを作成し、HYPER-V 分離またはその逆以外の適切なランタイム フラグを指定するには、何も変更せずに展開します。

速度が時間と入れ子になった構成と比較して高速化の実行時のパフォーマンスを下の回転など、キーの場合は、Windows Server コンテナー密度とパフォーマンスの向上を提供します。 True の場合、その名前を HYPER-V 分離より大きい分離、1 つのコンテナーで実行されるコードが侵害またはホスト オペレーティング システム、または同じホストで実行されている他のコンテナーに影響を及ぼすできないことを確認が提供しています。 これは、SaaS アプリケーションや計算をホストしているなどのコードを信頼できないをホストするための要件をマルチ テナントの場合に便利です。

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>コンテナーの Windows を実行するための前提条件とは

コンテナーと Windows Server 2016 プラットフォームに導入されました。 コンテナーを使用するには、Windows Server 2016 または Windows 10 の記念日を必要があります (バージョン 1607) の更新以降。

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>実行できます Windows コンテナー プロセス分離モードで Windows 10 のエンタープライズまたは Professional のですか。

以降では、Windows 10 年 2018年 10 月されなくなったは、更新プロセスの分離で Windows コンテナーを実行しているユーザーを禁止します。 ただし、必要があります直接を要求するプロセス分離を使用して、`--isolation=process`を使って、コンテナーの実行時のフラグを設定する`docker run`します。

場合に興味のあるものは、エンジン 18.09 docker バージョンを使用して、ホストには、Windows 10 ビルド 17763 + が実行されていることを確認または新しいする必要があります。

> [!WARNING]
> この機能は、開発/テスト用ものだけです。 ホストと Windows Server を使用して、製品の展開を続行する必要があります。
>
> この機能を使用するも必ずと一致する、ホストとコンテナー バージョン タグを開始しますコンテナーが失敗する場合は、または未定義の動作が発生することができます。

## <a name="windows-container-management"></a>Windows コンテナー管理

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>作成する方法のコンテナーの画像利用可能な air gapped マシンでよいですか。

Windows コンテナーの基本イメージには、ライセンスの分布が制限されているアイテムが含まれています。 構成でデータ バインド画像をプライベートまたはパブリック レジストリにプッシュ配信すると、基本レイヤーはプッシュことがないことがわかります。 代わりに、実際の基本 Azure クラウド ストレージ内に存在するレイヤーを指す外部レイヤーの概念を使用します。

プライベート コンテナー レジストリのアドレスから画像を取得できるのみ air gapped のコンピューターがある場合は、問題が発生することができます。 ここでは外部基本イメージを取得するレイヤーをフォローする試行は失敗します。 外部のレイヤーの動作をオーバーライドするを使用できます、 `--allow-nondistributable-artifacts` Docker デーモンにフラグを設定します。

> [!IMPORTANT]
> このフラグの使用には、Windows コンテナーの基本イメージ ライセンス; の条項を遵守する義務は不能します。パブリックまたはサード パーティの再配布できるように Windows のコンテンツを投稿する必要があります。 独自の環境での使用が許可されています。

## <a name="microsofts-open-ecosystem"></a>Microsoft のオープン エコシステム

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft は Open Container Initiative (OCI) に参加していますか。

パッケージ形式の普遍性の維持を保証するため、Docker は先頃、コンテナー パッケージのオープン性と基盤主導の形式を維持することを目的に、Open Container Initiative (OCI) を組織しました。Microsoft は OCI の創設メンバーです。

> [!TIP]
> [よく寄せられる質問への追加の推奨事項があるか。 [コメント] セクションで、新しいフィードバックの問題を開くまたは GitHub を使用して、これらのドキュメントに対して取得要求を開きます。
