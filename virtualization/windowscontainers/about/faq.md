---
title: Windows コンテナーに関する FAQ
description: Windows Server コンテナーについてよく寄せられる質問
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 19ff54ec032d61b24aea9fec4f14e8fce301d33a
ms.sourcegitcommit: 347d7c9d34f4c1d2473eb6c94c8ad6187318a037
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257954"
---
# <a name="frequently-asked-questions-about-containers"></a>コンテナーについてよく寄せられる質問

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux と Windows Server コンテナーの違いは何ですか?

Linux と Windows Server はどちらも、カーネルとコアオペレーティングシステム内に同様のテクノロジを実装しています。 違いは、プラットフォームと、コンテナー内で実行されるワークロードに由来します。  

顧客が Windows Server コンテナーを使っている場合は、.NET、ASP.NET、PowerShell などの既存の Windows テクノロジと統合することができます。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows でコンテナーを実行するための前提条件を教えてください。

コンテナーは、Windows Server 2016 でプラットフォームに導入されました。 コンテナーを使用するには、Windows Server 2016 または Windows 10 記念日更新プログラム (バージョン 1607) 以降が必要です。 詳細については、[システム要件](../deploy-containers/system-requirements.md)を参照してください。

## <a name="what-are-wcow-and-lcow"></a>WCOW と LCOW とは何ですか?

WCOW は、"windows 上の Windows コンテナー" には短いものです。 LCOW は、"Linux の Windows 上のコンテナー" には短いものです。

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>コンテナーのライセンスについて 実行できるコンテナーの数に制限はありますか?

Windows コンテナーイメージ[EULA](../images-eula.md)は、有効なライセンスを持つホスト OS を使用しているユーザーに依存する使用法を示しています。 ユーザーが実行できるコンテナーの数は、ホストの OS エディションと、コンテナーが実行されている[分離モード](../manage-containers/hyperv-container.md)、およびこれらのコンテナーが開発/テスト目的で実行されているか、または運用環境で実行されているかによって異なります。

|ホスト OS                                                         |プロセス-分離されたコンテナーの制限                   |Hyper-v-分離されたコンテナーの制限                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |無制限                                          |両面                                                  |
|Windows Server Datacenter                                       |無制限                                          |無制限                                          |
|Windows 10 Pro と Enterprise                                   |無制限 *(テストまたは開発目的のみ)*|無制限 *(テストまたは開発目的のみ)*|
|Windows 10 IoT Core および Enterprise)                             |無制限 *(テストまたは開発目的のみ)*|無制限 *(テストまたは開発目的のみ)*|

Windows Server コンテナーイメージの使用状況は、その[エディション](/windows-server/get-started-19/editions-comparison-19.md)でサポートされている仮想ゲストの数を読み取ることによって決定されます。 Windows の IoT edition でのコンテナーの運用方法は、追加のライセンスの制限によって異なります。 何が許可されているかを正確に把握するには、[コンテナイメージの EULA](../images-eula.md)をお読みください。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>開発者として、コンテナーの種類ごとにアプリを書き換える必要がありますか?

できません。 Windows コンテナーの画像は、Windows Server コンテナーと Hyper-v 分離で共通しています。 コンテナーを開始すると、コンテナーの種類の選択が行われます。 開発者の観点から見ると、Windows Server コンテナーと Hyper-v 分離は、同じものの2つのフレーバーです。 これにより、開発、プログラミング、管理のエクスペリエンスが変わり、オープンで拡張可能になり、Docker と同じレベルの統合とサポートが提供されます。

開発者は、Windows Server コンテナーを使ってコンテナーイメージを作成し、Hyper-v 分離で展開するか、適切なランタイムフラグを指定する以外に何も変更せずにそのまま展開できます。

Windows Server コンテナーでは、スピンアップ時間の短縮や入れ子になった構成とのランタイムのパフォーマンスの高速化などの速度が重要な場合に、高い密度とパフォーマンスを実現しています。 Hyper-v 分離は、その名前に対して true であり、分離性が高くなり、1つのコンテナーで実行されているコードが、同じホスト上で実行されているホストオペレーティングシステムや他のコンテナーに悪影響を与えたり、影響を受ける可能性があります。 これは、SaaS アプリケーションやコンピューティングホスティングなど、信頼されていないコードをホストするための要件を持つマルチテナントシナリオで役立ちます。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>Windows 10 Enterprise または Professional で、プロセス分離モードで Windows コンテナーを実行できますか?

Windows 10 年 2018 10 月の更新プログラムでは、プロセス分離を使用して Windows コンテナーを実行でき`--isolation=process` `docker run`ますが、まず、コンテナーを実行するときにフラグを使用してプロセスの分離を要求する必要があります。

このようにして Windows コンテナーを実行する場合は、ホストが Windows 10 ビルド 17763 + を実行していて、エンジン18.09 以降を搭載した Docker バージョンがインストールされていることを確認する必要があります。

> [!WARNING]
> この機能は、開発とテストのみを目的としています。 引き続き Windows Server を運用展開用のホストとして使用する必要があります。 この機能を使うには、ホストとコンテナーのバージョンタグが一致していることを確認する必要があります。そうでないと、コンテナーの開始に失敗したり、未定義の動作が発生したりする可能性があります。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Gapped コンピューターでコンテナーの画像を使用できるようにするにはどうすればよいですか?

Windows コンテナーのベースイメージには、配布がライセンスによって制限されている成果物が含まれています。 これらの画像を構築してプライベートまたは公開レジストリにプッシュすると、基本レイヤーがプッシュされることはありません。 代わりに、Azure クラウドストレージに存在する実際のベースレイヤーを指す、外部レイヤーの概念を使います。

これは、プライベートコンテナーレジストリのアドレスからのみ画像を取得できる air-gapped のコンピューターを使用している場合に、より複雑なものになることがあります。 この場合は、外部レイヤーに従って基本イメージを取得しようとしても機能しません。 外部レイヤーの動作をオーバーライドするには、Docker `--allow-nondistributable-artifacts`デーモンのフラグを使うことができます。

> [!IMPORTANT]
> このフラグを使用すると、Windows コンテナーベースのイメージライセンスの条項に準拠する義務を排除することはできません。パブリックまたはサードパーティの再配布には、Windows のコンテンツを投稿しないでください。 独自の環境内での使用が許可されています。

## <a name="additional-feedback"></a>その他のフィードバック

FAQ に何かを追加しますか? [コメント] セクションで新しいフィードバックの問題を開くか、GitHub でこのページのプル要求を設定します。
