---
title: Windows コンテナーについて
description: コンテナーは、オンプレミスのさまざまな環境で、またはクラウドでアプリをパッケージ化して実行するためのテクノロジです。 このトピックでは、Docker と Azure Kubernetes サービスの使用など、コンテナーでのアプリの開発と展開について説明します。
keywords: Docker, コンテナー
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: acce214cc8991f20c979b6dbe636590416841cb9
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254292"
---
# <a name="windows-and-containers"></a>Windows とコンテナー

コンテナーは、オンプレミスとクラウドでのさまざまな環境で Windows アプリと Linux アプリケーションをパッケージ化して実行するためのテクノロジです。 コンテナーは、アプリの開発、展開、管理を簡単にするための軽量な分離環境を提供します。 コンテナーはすぐに起動して停止するため、変更要求に迅速に対応する必要があるアプリに最適です。 コンテナーの軽量な性質も、インフラストラクチャの密度と使用率を向上させるのに便利なツールです。

![クラウドまたはオンプレミスでコンテナーを実行する方法を示すグラフィック。ほぼすべての言語で記述されたモノリシックアプリまたはマイクロサービスをサポートしています。](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Microsoft コンテナーエコシステム

Microsoft には、コンテナーでアプリを開発して展開するためのツールとプラットフォームが数多く用意されています。

- Windows <strong>10 で windows ベースのコンテナーまたは Linux ベースのコンテナーを実行</strong>し、 [Docker デスクトップ](https://store.docker.com/editions/community/docker-ce-desktop-windows)を使用して開発およびテストを行い、windows に組み込まれているコンテナー機能を使用します。 [Windows Server でネイティブにコンテナーを実行](../quick-start/set-up-environment.md?tabs=Windows-Server)することもできます。
- Docker、Docker の作成、Kubernetes、ヘルム、その他の便利な機能を備えた[Visual studio](https://docs.microsoft.com/visualstudio/containers/overview)と[visual studio コード](https://code.visualstudio.com/docs/azure/docker)での強力なコンテナーのサポートを使用して、 <strong>Windows ベースのコンテナーの開発、テスト、公開、展開</strong>を行います。各種.
- 他のユーザーが使用するために、<strong>アプリをコンテナーイメージとして</strong>公開されるように公開します。または、組織の開発と展開のためのプライベート[Azure container レジストリ](https://azure.microsoft.com/services/container-registry/)に、Visual Studio と visual studio コード内から直接プッシュおよびプルします。.
- Azure またはその他のクラウド<strong>のスケールでコンテナーを展開</strong>します。

  - Azure Container レジストリなどのコンテナーレジストリからアプリ (コンテナーイメージ) を取得し、 [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes)などのオーケストレータ (Windows ベースのアプリの場合はプレビュー) または azure サービスなどの orchestrator を使用して展開および管理します。 [ファブリック](https://docs.microsoft.com/azure/service-fabric/)。
  - Azure Kubernetes Service は、コンテナー、数百、または数千のコンテナーなど、さまざまな方法で、Azure 仮想マシンにコンテナーを展開して管理します。 Azure 仮想マシンは、カスタマイズされた Windows Server イメージ (Windows ベースのアプリを展開している場合)、またはカスタマイズされた Ubuntu Linux イメージ (Linux ベースのアプリを展開している場合) のいずれかを実行します。
- [AKS エンジン](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)(プレビュー版は Linux コンテナー)、または openshift を使った[Azure Stack](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack)で azure stack を使用して、<strong>オンプレミスのコンテナーを展開</strong>します。 Windows Server で Kubernetes をセットアップすることもできます (「 [windows の Kubernetes](../kubernetes/getting-started-kubernetes-windows.md)」を参照してください)。また、windows[コンテナーを RedHat Openshift コンテナープラットフォームで](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821)も実行できるように取り組んでいます。

## <a name="how-containers-work"></a>コンテナーのしくみ

コンテナーは、ホストオペレーティングシステム上でアプリケーションを実行するための、分離された簡易サイロです。 コンテナーは、次の図に示すように、ホストオペレーティングシステムのカーネル (オペレーティングシステムの埋め込み機能とも呼ばれます) の上に構築されます。

![コンテナーがカーネル上でどのように動作するかを示すアーキテクチャ図](media/container-diagram.svg)

コンテナーは、ホストオペレーティングシステムのカーネルを共有していますが、コンテナーは unfettered アクセスを取得しません。 代わりに、コンテナーは分離を取得します。また、一部の状況では、システムの表示が仮想化されています。 たとえば、コンテナーはファイルシステムとレジストリの仮想化されたバージョンにアクセスできますが、変更はコンテナーにのみ影響し、停止すると破棄されます。 データを保存するために、コンテナーは[Azure ディスク](https://azure.microsoft.com/services/storage/disks/)やファイル共有 ( [azure Files](https://azure.microsoft.com/services/storage/files/)を含む) などの固定記憶域をマウントできます。

コンテナーはカーネルの上に構築されますが、カーネルでは、アプリで実行する必要があるすべての Api とサービスを提供するわけではありません。これらのほとんどは、カーネルの上でユーザーモードで実行されるシステムファイル (ライブラリ) によって提供されます。 コンテナーは、ホストのユーザーモード環境から分離されているため、コンテナーには、基本イメージと呼ばれるものにパッケージ化された、ユーザーモードシステムファイルの独自のコピーが必要です。 基本イメージは、コンテナーが構築されている基礎レイヤーとして機能し、カーネルによって提供されていないオペレーティングシステムサービスを提供します。 ただし、後では、コンテナの画像について詳しく説明します。

## <a name="containers-vs-virtual-machines"></a>コンテナーと仮想マシン

コンテナーとは異なり、仮想マシン (Vm) では、次の図に示すように、独自のカーネルを含む完全なオペレーティングシステムを実行します。

![Vm がホストオペレーティングシステムの横に完全なオペレーティングシステムを実行する方法を示すアーキテクチャ図](media/virtual-machine-diagram.svg)

各コンテナーと仮想マシンはそれぞれの用途を持っています。実際には、コンテナーの多くの展開では、ハードウェア上で直接実行されるのではなく、仮想マシンをホストオペレーティングシステムとして使用します。特に、クラウドでコンテナーを実行している場合です。

これらの補完的なテクノロジの類似点と相違点について詳しくは、「[コンテナーと仮想マシン](containers-vs-vm.md)」をご覧ください。

## <a name="container-images"></a>コンテナーイメージ

すべてのコンテナーは、コンテナイメージから作成されます。 コンテナーの画像は、ローカルコンピューターまたはリモートコンテナーレジストリに存在する一連のレイヤーに編成されたファイルのバンドルです。 コンテナーのイメージは、アプリをサポートするために必要なユーザーモードのオペレーティングシステムファイル、アプリ、アプリのランタイムまたは依存関係、およびアプリで適切に実行する必要があるその他の構成ファイルで構成されます。

Microsoft には、独自のコンテナーイメージを作成するための出発点として使用できる、いくつかの画像 (基本イメージ) が用意されています。

* <strong>Windows</strong> -windows api とシステムサービスの完全なセット (-server の役割) が含まれています。
* <strong>Windows Server Core</strong> -Windows server api のサブセット、つまり完全な .net framework を含む小さな画像。 また、ほとんどのサーバーの役割も含まれていますが、Fax サーバーではありません。
* <strong>Nano Server</strong> -.Net Core api と一部のサーバーの役割をサポートする最小の Windows サーバーイメージ。
* <strong>Windows 10 IoT Core</strong> -ARM または x86 プロセッサを搭載した、小規模インターネット用のハードウェアメーカーによって使用される windows のバージョン。

前に説明したように、コンテナーの画像は、一連のレイヤーで構成されています。 各レイヤーには、コンテナーの画像を表す一連のファイルが含まれています。 コンテナーのレイヤーの性質上、Windows コンテナーを構築するために常に基本イメージをターゲットにする必要はありません。 代わりに、目的のフレームワークを既に持っている別のイメージをターゲットにすることもできます。 たとえば、.net チームは .net core runtime を含む[.net core イメージ](https://hub.docker.com/_/microsoft-dotnet-core)を公開します。 .NET core のインストールプロセスを複製する必要がなくなり、このコンテナーイメージのレイヤーを再利用することができます。 .NET core イメージ自体は、Nano Server に基づいて構築されます。

詳しくは、「[コンテナーの基本イメージ](../manage-containers/container-base-images.md)」をご覧ください。

## <a name="container-users"></a>コンテナーユーザー

### <a name="containers-for-developers"></a>開発者のためのコンテナー

コンテナーを使うと、開発者はより高品質のアプリを迅速に構築および出荷することができます。 コンテナーでは、開発者は、環境に合わせて、数秒で展開されるコンテナーイメージを作成できます。 コンテナーは、チーム間でコードを共有するための簡単なメカニズムとして機能し、ホストのファイルシステムに影響を及ぼすことなく、開発環境をブートストラップします。

コンテナーには、ポータブルで汎用性があり、任意の言語で記述されたアプリを実行でき、Windows 10、バージョン1607以降、または Windows 2016 以降を実行しているコンピューターと互換性があります。 開発者は、自分のノート pc またはデスクトップでローカルにコンテナーを作成してテストし、その同じコンテナーイメージを会社のプライベートクラウド、パブリッククラウド、またはサービスプロバイダーに展開することができます。 コンテナーの自然なアジリティは、大規模な仮想化されたクラウド環境での先進のアプリ開発パターンをサポートしています。

### <a name="containers-for-it-professionals"></a>IT プロフェッショナル向けのコンテナー

コンテナーを使うと、管理者は簡単に更新して維持することができ、ハードウェアリソースをより完全に活用できます。 IT プロフェッショナルは、コンテナーを使用して、開発、QA、および運用チーム向けに標準化された環境を提供できます。 コンテナーを使用することで、システム管理者はオペレーティングシステムのインストールと基盤となるインフラストラクチャの相違点を抽象化します。

## <a name="container-orchestration"></a>コンテナーのオーケストレーション

オーケストレーションの基盤は、コンテナーベースの環境をセットアップするときに重要なインフラストラクチャです。 Docker と Windows を使って、いくつかのコンテナーを手動で管理することもできますが、多くの場合、アプリは、オーケストレーションの提供元である5、10、または数百のコンテナーを使用します。

コンテナーオーケストレーションは、スケーラビリティと運用環境でのコンテナーの管理を支援するために構築されました。 オーケストレーションは、次の機能を提供します。

- 縮尺での展開
- ワークロードのスケジュール設定
- 正常性の監視
- ノードに障害が発生した場合のフェールオーバー
- 拡大縮小
- ネットワーク
- サービス検出
- アプリのアップグレードの調整
- クラスターノードのアフィニティ

Windows コンテナーで使うことができるさまざまなオーケストレーションがあります。Microsoft が提供するオプションを次に示します。
- [Azure Kubernetes service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) -管理された azure Kubernetes サービスを使用する
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) -マネージサービスを使用する
- [AKS Engine での Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) -オンプレミスの Azure Kubernetes サービスの使用
- [Windows 上の Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) -Windows で Kubernetes をセットアップする

## <a name="try-containers-on-windows"></a>Windows でコンテナーを試す

Windows Server または Windows 10 のコンテナーの使用を開始するには、次の情報を参照してください。
> [!div class="nextstepaction"]
> [はじめに: コンテナーの環境の構成](../quick-start/set-up-environment.md)

シナリオに適した Azure サービスを決定する方法については、「 [azure container services](https://azure.microsoft.com/product-categories/containers/) 」および「[アプリケーションをホストするために使用する Azure サービスを選択](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)する」を参照してください。
