---
title: Docker について
description: Docker について説明します。
keywords: Docker, コンテナー
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: cdb7771a1293e7c3fd505103f0010bdfba47cc31
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674890"
---
# <a name="about-docker"></a>Docker について

コンテナーについての資料で、必ず言及されるのが Docker です。 Docker エンジンは、コンテナーイメージをパッケージ化して提供するコンテナー管理ツールセットです。 結果として得られた画像は、オンプレミス、クラウド、またはパーソナルコンピューターのいずれかの場所で、コンテナーとして実行できます。

![](media/docker.png)

他のコンテナーと同じように、 [Docker](https://www.docker.com)で Windows Server コンテナーを管理することができます。

コンテナーを介した名前空間分離とリソースガバナンスの概念は、長い間、BSD Jails、Solaris ゾーン、および基本的な UNIX の変更ルート (chroot) メカニズムに戻りました。 Docker は、共通のツールセット、パッケージモデル、アプリケーションの containerization と配布を簡素化する展開メカニズムを通じて、開発のための堅固な基盤を確立します。 これらのアプリケーションは、任意の Linux ホストと Windows でどこでも実行できます。

ユビキタスなパッケージモデルと展開テクノロジを使用すると、任意のホストに対して同じ管理コマンドが提供され、シームレスな DevOps を実現することができるため、管理が簡単になります。 また、開発者のデスクトップ、テスト用のコンピューター、一連の運用コンピューターなど、任意の環境で同じように展開される Docker イメージを作成することもできます。 この方法では、docker のコンテナーに含まれているアプリケーションの大規模で増大したエコシステムを、Docker が管理するパブリックコンテナーアプリケーションレジストリと共に作成しています。

次は、アプリケーションのエコシステムと、Docker の概念を基にして、ニーズに適した開発および展開のワークフローを作成する方法について説明します。

## <a name="get-started-with-docker"></a>Docker を使ってみる

Docker でコンテナーを構築する方法については、「 [Windows の Docker Engine](../manage-docker/configure-docker-daemon.md)」を参照してください。 Docker の使い方について詳しくは、 [docker の web サイト](https://www.docker.com)を参照してください。