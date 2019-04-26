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
ms.openlocfilehash: 71d91e12ae843bf96e1b4001915ffb55bb352ffd
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576853"
---
### <a name="powershell-for-docker"></a>Docker 用 PowerShell

フォーラム、Twitter、GitHub を介したお客様との対話、さらには直接の対話を通して、"どうして PowerShell から Docker コンテナーを表示できないのか" という問いをよく耳にします。 

長所、短所、およびさまざまなオプションについて、お客様と話し合った結果、コンテナー PowerShell モジュールの更新が必要であるという結論に達しました。 そのため、Windows Server 2016 のプレビュー ビルドに同梱されているコンテナー PowerShell モジュールを非推奨とし、Docker 用の新しい PowerShell モジュールへの置き換え作業を開始しました。  この新しいモジュールの開発は既に進行中ですが、今までとは異なる方法 (つまり、作業をオープンにする) で進行しています。  このモジュールの目標はコミュニティ コラボレーションを実現することです。そうすれば、Docker エンジンでもコンテナーの優れた PowerShell エクスペリエンスを提供できます。  この新しいモジュールは Docker エンジンの REST インターフェイス上に直接ビルドされるため、ユーザーは Docker CLI か PowerShell、または両方を選択することができます。

優れた PowerShell モジュールをビルドするのは簡単なことではありません。すべてのコードを正しく記述してから、オブジェクトとパラメーター セットおよびコマンドレット名の適正なバランスを取るまでの作業はすべてとても重要です。  そのため、この新しいモジュールに着手しながら、エンド ユーザーや大規模な PowerShell と Docker のコミュニティに目を向け、このモジュールの形成に役立てています。  どのようなパラメーター セットが重要だと思いますか?  当社で "docker run" と同等のものを提供すべきでしょうか。それとも、お客様が new-container を start-container にパイプ処理するべきでしょうか。  このモジュールについて詳しくは、参加の開発にしてくださいヘッド経由で GitHub ページに (https://github.com/Microsoft/Docker-PowerShell/)に参加するとします。

開発が進み、品質の安定したモジュールができしだい、PowerShell ギャラリーに投稿し、このページを更新して使用手順を提供し続ける予定です。
