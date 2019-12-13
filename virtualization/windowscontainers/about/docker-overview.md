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
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910792"
---
# <a name="about-docker"></a>Docker について

コンテナーについての資料で、必ず言及されるのが Docker です。 Docker エンジンは、コンテナーイメージのパッケージ化と配信を行うコンテナー管理ツールセットです。 生成されたイメージは、コンテナーとして、オンプレミス、クラウド、または個人用コンピューターのいずれでも実行できます。

![](media/docker.png)

他のコンテナーと同じように、 [Docker](https://www.docker.com)を使用して Windows Server コンテナーを管理できます。

コンテナーを使用した名前空間の分離とリソースガバナンスの概念は、長い間、BSD 途中、Solaris ゾーン、および基本的な UNIX 変更ルート (chroot) メカニズムに戻りました。 Docker は、アプリケーションのコンテナー化と配布を簡略化する共通ツールセット、パッケージングモデル、および配置メカニズムを使用して、開発用の堅固な基盤となります。 これらのアプリケーションは、任意の Linux ホストおよび Windows 上の任意の場所で実行できます。

ユビキタスなパッケージモデルと展開テクノロジを使用すると、任意のホストに対して同じ管理コマンドを提供することで管理を簡素化し、シームレスな DevOps を実現するための独自の機会を作成できます。 また、開発者のデスクトップ、テスト用のコンピューター、または一連の実稼働コンピューターのいずれであっても、任意の環境で同じようにデプロイできる Docker イメージを作成することもできます。 これにより、docker で保持されているパブリックコンテナー化アプリケーションレジストリである DockerHub を使用して、Docker コンテナーにパッケージ化されたアプリケーションのエコシステムを大幅に拡大します。

次に、アプリケーションのエコシステムについて説明します。また、Docker の概念を構築して、ニーズに適した開発とデプロイのワークフローを作成する方法についても説明します。

## <a name="get-started-with-docker"></a>Docker の概要

Docker を使用してコンテナーを構築する方法については、「 [Windows 上の Docker エンジン](../manage-docker/configure-docker-daemon.md)」を参照してください。 Docker の使用方法の詳細については[、docker の web サイト](https://www.docker.com)を参照してください。