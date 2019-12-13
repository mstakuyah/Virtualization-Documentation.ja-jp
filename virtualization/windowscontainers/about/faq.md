---
title: Windows コンテナーに関する FAQ
description: Windows Server コンテナーに関する FAQ
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910802"
---
# <a name="frequently-asked-questions-about-containers"></a>コンテナーに関してよく寄せられる質問

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux と Windows Server のコンテナーの違いは何ですか。

Linux と Windows Server はどちらも、カーネルとコアオペレーティングシステム内に同様のテクノロジを実装しています。 違いは、プラットフォームと、コンテナー内で実行されるワークロードに由来します。  

顧客が Windows Server のコンテナーを使用する場合は、.NET、ASP.NET、PowerShell などの既存の Windows テクノロジと統合できます。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows でコンテナーを実行するための前提条件は何ですか。

コンテナーは、Windows Server 2016 でプラットフォームに導入されました。 コンテナーを使用するには、Windows Server 2016 または Windows 10 の記念日更新プログラム (バージョン 1607) 以降が必要です。 詳細については、[システム要件](../deploy-containers/system-requirements.md)を参照してください。

## <a name="what-are-wcow-and-lcow"></a>WCOW と LCOW とは

WCOW は、"Windows 上の Windows コンテナー" の略です。 LCOW は、"Windows 上の Linux コンテナー" の略です。

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>コンテナーはどのようにしてライセンス供与されますか。 実行できるコンテナーの数に制限はありますか。

Windows コンテナーイメージの[EULA](../images-eula.md)では、有効なライセンスを持つホスト OS を持つユーザーに依存する使用法について説明しています。 ユーザーが実行できるコンテナーの数は、ホスト OS のエディションとコンテナーが実行されている[分離モード](../manage-containers/hyperv-container.md)によって異なります。また、これらのコンテナーが開発/テスト目的または運用環境のどちらで実行されているかによっても異なります。

|ホスト OS                                                         |プロセス分離コンテナーの制限                   |Hyper-v 分離コンテナーの制限                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |無制限                                          |2                                                  |
|Windows Server Datacenter                                       |無制限                                          |無制限                                          |
|Windows 10 Pro および Enterprise                                   |無制限 *(テストまたは開発目的のみ)*|無制限 *(テストまたは開発目的のみ)*|
|Windows 10 IoT Core および Enterprise                             |無数                                         |無数                                          |

Windows Server コンテナーイメージの使用量は、その[エディション](/windows-server/get-started-19/editions-comparison-19.md)でサポートされている仮想化ゲストの数を確認することによって決定されます。 <br/>

>[!NOTE]
>windows 10 コアのランタイムイメージまたは Windows 10 IoT Enterprise デバイスライセンス (以下「Windows IoT コマーシャル契約」といいます) の Microsoft の商用使用条件に同意したかどうかによって、Windows の IoT エディションでのコンテナーの実稼働環境の使用が \*されます。 Windows IoT の商用契約書に記載されている追加の条件は、運用環境でのコンテナーイメージの使用に適用されます。 許可されている内容と何ではないものを正確に把握するには、[コンテナーイメージの EULA](../images-eula.md)を参照してください。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>開発者は、コンテナーの種類ごとにアプリを書き直す必要がありますか。

いいえ。 Windows コンテナーイメージは、Windows Server のコンテナーと Hyper-v の分離の両方で共通です。 コンテナーを開始すると、コンテナーの種類の選択が行われます。 開発者の観点から見ると、Windows Server のコンテナーと Hyper-v の分離は、同じことの2種類です。 同じ開発、プログラミング、および管理のエクスペリエンスを提供し、オープンで拡張可能で、Docker と同じレベルの統合とサポートが含まれています。

開発者は、Windows Server コンテナーを使用してコンテナーイメージを作成し、Hyper-v の分離またはその逆に展開できます。これに加えて、適切なランタイムフラグを指定する必要はありません。

Windows Server コンテナーでは、入れ子になった構成と比較して、スピンアップ時間が短縮され、実行時のパフォーマンスが向上するなど、速度が重要な場合の密度とパフォーマンスが向上しています。 Hyper-v の分離では、名前が真であるため、分離が向上し、1つのコンテナーで実行されているコードが、同じホスト上で実行されているホストオペレーティングシステムまたは他のコンテナーに対してセキュリティを侵害したり、影響を与えることがなくなります。 これは、SaaS アプリケーションやコンピューティングホスティングなど、信頼されていないコードをホストするための要件を持つマルチテナントシナリオに役立ちます。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Windows 10 でプロセス分離モードで Windows コンテナーを実行できますか。

Windows 10 10 月2018更新プログラム以降では、プロセス分離を使用して Windows コンテナーを実行できますが、まず、`docker run`でコンテナーを実行するときに `--isolation=process` フラグを使用してプロセス分離を直接要求する必要があります。 プロセス分離は、windows 10 Pro、Windows 10 Enterprise、Windows 10 IoT Core、Windows 10 IoT Enterprise と互換性があります。

この方法で Windows コンテナーを実行する場合は、ホストで Windows 10 build 17763 + が実行されていること、およびエンジン18.09 以降の Docker バージョンがあることを確認する必要があります。

> [!WARNING]
> IoT Core および IoT Enterprise ホスト (追加の使用条件に同意した後) とは別に、この機能は開発とテストのみを目的としています。 運用環境の展開では、引き続き Windows Server をホストとして使用する必要があります。 この機能を使用することにより、ホストとコンテナーのバージョンタグが一致するようにする必要があります。そうしないと、コンテナーの起動に失敗したり、未定義の動作が発生したりする可能性があります。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Gapped コンピューターでコンテナーイメージを使用できるようにする操作方法

Windows コンテナーの基本イメージには、配布がライセンスによって制限されている成果物が含まれています。 これらのイメージを構築してプライベートまたはパブリックレジストリにプッシュすると、基本レイヤーがプッシュされることはありません。 代わりに、Azure クラウドストレージに存在する実際の基本レイヤーを指す外部レイヤーの概念を使用します。

これにより、プライベートコンテナーレジストリのアドレスからイメージをプルすることしかできない gapped コンピューターがある場合に、処理が複雑になる可能性があります。 この場合、基本イメージを取得するために外部層に従う試行は機能しません。 外部レイヤーの動作をオーバーライドするには、Docker デーモンで `--allow-nondistributable-artifacts` フラグを使用します。

> [!IMPORTANT]
> このフラグを使用すると、Windows コンテナーの基本イメージライセンスの条項に準拠する義務を排除することはできません。パブリックまたはサードパーティの再配布のために Windows コンテンツを投稿することはできません。 自分の環境内で使用できます。

## <a name="additional-feedback"></a>その他のフィードバック

FAQ に何かを追加する必要がある場合は、 [コメント] セクションで新しいフィードバックの問題を開くか、GitHub を使用してこのページのプル要求を設定します。
