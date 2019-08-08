---
title: Docker 用 PowerShell
description: PowerShell を使用して Docker コンテナーを管理する方法
keywords: docker, コンテナー, powershell
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 4a0e907d-0d07-42f8-8203-2593391071da
ms.openlocfilehash: bcbc2e4e76c48a3d9a1a9720b09ef366a396bf30
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998185"
---
### <a name="powershell-for-docker"></a>Docker 用 PowerShell

フォーラム、Twitter、GitHub を介したお客様との対話、さらには直接の対話を通して、"どうして PowerShell から Docker コンテナーを表示できないのか" という問いをよく耳にします。 

長所、短所、およびさまざまなオプションについて、お客様と話し合った結果、コンテナー PowerShell モジュールの更新が必要であるという結論に達しました。 そのため、Windows Server 2016 のプレビュー ビルドに同梱されているコンテナー PowerShell モジュールを非推奨とし、Docker 用の新しい PowerShell モジュールへの置き換え作業を開始しました。  この新しいモジュールの開発は既に進行中ですが、今までとは異なる方法 (つまり、作業をオープンにする) で進行しています。  このモジュールの目標はコミュニティ コラボレーションを実現することです。そうすれば、Docker エンジンでもコンテナーの優れた PowerShell エクスペリエンスを提供できます。  この新しいモジュールは Docker エンジンの REST インターフェイス上に直接ビルドされるため、ユーザーは Docker CLI か PowerShell、または両方を選択することができます。

優れた PowerShell モジュールをビルドするのは簡単なことではありません。すべてのコードを正しく記述してから、オブジェクトとパラメーター セットおよびコマンドレット名の適正なバランスを取るまでの作業はすべてとても重要です。  そのため、この新しいモジュールに着手しながら、エンド ユーザーや大規模な PowerShell と Docker のコミュニティに目を向け、このモジュールの形成に役立てています。  どのようなパラメーター セットが重要だと思いますか?  当社で "docker run" と同等のものを提供すべきでしょうか。それとも、お客様が new-container を start-container にパイプ処理するべきでしょうか。  このモジュールについて詳しく知り、開発に参加するには、GitHub のページhttps://github.com/Microsoft/Docker-PowerShell/)にアクセスして、参加してください。

開発が進み、品質の安定したモジュールができしだい、PowerShell ギャラリーに投稿し、このページを更新して使用手順を提供し続ける予定です。
