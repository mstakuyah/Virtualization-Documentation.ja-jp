---
title: 基本イメージのサービスライフサイクル
description: Windows コンテナーのベースイメージのライフサイクルに関する情報。
keywords: windows コンテナー、コンテナー、ライフサイクル、リリース情報、基本イメージ、コンテナーベースイメージ
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: d3a8240dbba8af3c74ce5d185620e129d103ef81
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883185"
---
# <a name="base-image-servicing-lifecycles"></a>基本イメージのサービスライフサイクル

Windows コンテナーベースの画像は、半期チャネルリリースまたは Windows Server の長期的なサービスチャネルリリースに基づいています。 この記事では、両方のチャネルのさまざまなバージョンの基本イメージのサポート期間について説明します。

半期チャネルは、各リリースの18ヶ月のサービスタイムラインを含む年1回の機能更新プログラムのリリースです。 これにより、お客様は、アプリケーション (特にコンテナーとマイクロサービスに組み込まれているもの) とソフトウェア定義のハイブリッドデータセンターの両方で、新しいオペレーティングシステム機能を利用できるようになります。 詳細については、「 [Windows Server の半期チャネルの概要](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)」を参照してください。

サーバーのコアイメージの場合、ユーザーは、2 ~ 3 年ごとに新しいメジャーバージョンの Windows サーバーをリリースする長期サービスチャネルを使用することもできます。 長期サービスチャネルリリースでは、5年間のメインストリームサポートと5年間の延長サポートを受けることができます。 このチャネルは、より長いサービスオプションと機能の安定性が必要なシステムで動作します。

次の表は、基本イメージの種類、そのサービスチャネル、サポートが終了する期間を示しています。

|基本イメージ                       |サービス チャネル|Version|OS ビルド|対象|メインストリーム サポートの終了日|延長サポート日|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core、Nano Server、Windows|半期      |1903   |18362   |05/21/2019  |12/08/2020                 |該当せず                  |
|Server Core                      |長期        |1809   |17763   |11/13/2018  |01/09/2024                 |01/09/2029           |
|Server Core、Nano Server、Windows|半期      |1809   |17763   |11/13/2018  |05/12/2020                 |該当せず                  |
|Server Core、Nano Server         |半期      |1803   |17134   |04/30/2018  |11/12/2019                 |該当せず                  |
|Server Core、Nano Server         |半期      |1709   |16299   |2017 年 10 月 17 日  |2019 年 4 月 9日                 |該当なし                  |
|Server Core                      |長期        |1607   |14393   |10/15/2016  |2022 年 1 月 11 日                 |2027 年 1 月 11 日           |
|Nano Server                      |半期      |1607   |14393   |10/15/2016  |10/09/2018                 |該当せず                  |

サービス要件およびその他の追加情報については、「 [Windows ライフサイクルの FAQ](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)」、「 [windows Server リリース情報](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)」、「 [Windows ベース OS イメージの Docker hub リポジトリ](https://hub.docker.com/_/microsoft-windows-base-os-images)」を参照してください。
