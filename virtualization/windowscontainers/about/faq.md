---
title: Windows コンテナーに関する FAQ
description: Windows Server コンテナーについてよく寄せられる質問
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 0aa93b721ab1279cb789e3a18cad04bb668d2644
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998519"
---
# <a name="frequently-asked-questions-about-containers"></a>コンテナーについてよく寄せられる質問

## <a name="what-are-wcow-and-lcow"></a>WCOW と LCOW とは何ですか?

WCOW は、"windows 上の Windows コンテナー" には短いものです。 LCOW は、"Linux の Windows 上のコンテナー" には短いものです。

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux と Windows Server コンテナーの違いは何ですか?

Linux と Windows Server はどちらも、カーネルとコアオペレーティングシステム内に同様のテクノロジを実装しています。 違いは、プラットフォームと、コンテナー内で実行されるワークロードに由来します。  

顧客が Windows Server コンテナーを使っている場合は、.NET、ASP.NET、PowerShell などの既存の Windows テクノロジと統合することができます。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>開発者として、コンテナーの種類ごとにアプリを書き換える必要がありますか?

できません。 Windows コンテナーの画像は、Windows Server コンテナーと Hyper-v 分離で共通しています。 コンテナーを開始すると、コンテナーの種類の選択が行われます。 開発者の観点から見ると、Windows Server コンテナーと Hyper-v 分離は、同じものの2つのフレーバーです。 これにより、開発、プログラミング、管理のエクスペリエンスが変わり、オープンで拡張可能になり、Docker と同じレベルの統合とサポートが提供されます。

開発者は、Windows Server コンテナーを使ってコンテナーイメージを作成し、Hyper-v 分離で展開するか、適切なランタイムフラグを指定する以外に何も変更せずにそのまま展開できます。

Windows Server コンテナーでは、スピンアップ時間の短縮や入れ子になった構成とのランタイムのパフォーマンスの高速化などの速度が重要な場合に、高い密度とパフォーマンスを実現しています。 Hyper-v 分離は、その名前に対して true であり、分離性が高くなり、1つのコンテナーで実行されているコードが、同じホスト上で実行されているホストオペレーティングシステムや他のコンテナーに悪影響を与えたり、影響を受ける可能性があります。 これは、SaaS アプリケーションやコンピューティングホスティングなど、信頼されていないコードをホストするための要件を持つマルチテナントシナリオで役立ちます。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows でコンテナーを実行するための前提条件を教えてください。

コンテナーは、Windows Server 2016 でプラットフォームに導入されました。 コンテナーを使用するには、Windows Server 2016 または Windows 10 記念日更新プログラム (バージョン 1607) 以降が必要です。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>Windows 10 Enterprise または Professional で、プロセス分離モードで Windows コンテナーを実行できますか?

Windows 10 年 2018 10 月の更新プログラムでは、プロセス分離を使用して Windows コンテナーを実行でき`--isolation=process` `docker run`ますが、まず、コンテナーを実行するときにフラグを使用してプロセスの分離を要求する必要があります。

このようにして Windows コンテナーを実行する場合は、ホストが Windows 10 ビルド 17763 + を実行していて、エンジン18.09 以降を搭載した Docker バージョンがインストールされていることを確認する必要があります。

> [!WARNING]
> この機能は、開発/テストのみを目的としています。 引き続き Windows Server を運用展開用のホストとして使用する必要があります。 この機能を使うには、ホストとコンテナーのバージョンタグが一致していることを確認する必要があります。そうでないと、コンテナーの開始に失敗したり、未定義の動作が発生したりする可能性があります。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Gapped コンピューターでコンテナーの画像を使用できるようにするにはどうすればよいですか?

Windows コンテナーのベースイメージには、配布がライセンスによって制限されている成果物が含まれています。 これらの画像を構築してプライベートまたは公開レジストリにプッシュすると、基本レイヤーがプッシュされることはありません。 代わりに、Azure クラウドストレージに存在する実際のベースレイヤーを指す、外部レイヤーの概念を使います。

これは、プライベートコンテナーレジストリのアドレスからのみ画像を取得できる air-gapped のコンピューターを使用している場合に、より複雑なものになることがあります。 この場合は、外部レイヤーに従って基本イメージを取得しようとしても機能しません。 外部レイヤーの動作をオーバーライドするには、Docker `--allow-nondistributable-artifacts`デーモンのフラグを使うことができます。

> [!IMPORTANT]
> このフラグを使用すると、Windows コンテナーベースのイメージライセンスの条項に準拠する義務を排除することはできません。パブリックまたはサードパーティの再配布には、Windows のコンテンツを投稿しないでください。 独自の環境内での使用が許可されています。

## <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft は Open Container Initiative (OCI) に参加していますか。

パッケージ形式が普遍的であることを保証するために、Docker はオープンコンテナーイニシアチブ (OCI) をまとめて整理したものであり、コンテナーのパッケージ化はオープンで、基盤となるメンバーのいずれかになります。

## <a name="additional-feedback"></a>その他のフィードバック

FAQ に何かを追加しますか? [コメント] セクションで新しいフィードバックの問題を開くか、GitHub でこのページのプル要求をセットアップします。
