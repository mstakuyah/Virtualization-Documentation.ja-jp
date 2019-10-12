# [Windows ドキュメントのコンテナー](index.md) 

# 概要
## [Windows コンテナーについて](about/index.md)
## [システム要件](deploy-containers/system-requirements.md)
## [FAQ](about/faq.md)

# はじめに
## [環境をセットアップする](quick-start/set-up-environment.md)
## [最初のコンテナーを実行する](quick-start/run-your-first-container.md)
## [サンプルアプリの Containerize](quick-start/building-sample-app.md)

# チュートリアル
## Windows コンテナーを構築する
### [Dockerfile を作成する](manage-docker/manage-windows-dockerfile.md)
### [Dockerfile を最適化する](manage-docker/optimize-windows-dockerfile.md)
## Azure Kubernetes サービスで実行する
### [AKS で Windows コンテナークラスターを作成する](/azure/aks/windows-container-cli)
### [現在の制限事項](/azure/aks/windows-node-limitations)
## サービスファブリックで実行する
### [最初のコンテナーの展開](/azure/service-fabric/service-fabric-quickstart-containers)
### [Windows コンテナー内での .NET アプリケーションの展開](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Azure App Service で実行する
### [Azure App Service のクイックスタート](/azure/app-service/app-service-web-get-started-windows-container)
### [Windows コンテナーと Azure App Service を使用して ASP.NET アプリを移行する](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Windows の Linux コンテナー
### [概要](deploy-containers/linux-containers.md)
### [最初の LCOW コンテナーを実行する](quick-start/quick-start-windows-10-linux.md)

# 概念
## Windows Container の基礎
### [コンテナーの基本イメージ](manage-containers/container-base-images.md)
### [分離モード](manage-containers/hyperv-container.md)
### [バージョンの互換性](deploy-containers/version-compatibility.md)
### [リソースコントロール](manage-containers/resource-controls.md)
## Docker
### [Windows 上の Docker エンジン](manage-docker/configure-docker-daemon.md)
### [Windows Docker host のリモート管理](management/manage_remotehost.md)
## コンテナーのオーケストレーション
### [概要](about/overview-container-orchestrators.md)
### Windows で使用する Kubernetes
#### [Windows で使用する Kubernetes](kubernetes/getting-started-kubernetes-windows.md)
#### [Kubernetes マスターを作成する](kubernetes/creating-a-linux-master.md)
#### [ネットワークソリューションを選ぶ](kubernetes/network-topologies.md)
#### [Windows ワーカーに参加する](kubernetes/joining-windows-workers.md)
#### [Linux ワーカに参加する](kubernetes/joining-linux-workers.md)
#### [Kubernetes リソースの展開](kubernetes/deploying-resources.md)
#### [トラブルシューティング](kubernetes/common-problems.md)
#### [Kubernetes の Windows サービス](kubernetes/kube-windows-services.md)
#### [コンパイル Kubernetes バイナリ](kubernetes/compiling-kubernetes-binaries.md)
### サービスファブリック
#### [サービスファブリックとコンテナー](/azure/service-fabric/service-fabric-containers-overview)
#### [リソースガバナンス](/azure/service-fabric/service-fabric-resource-governance)
### Docker 群れ
#### [群れモード](manage-containers/swarm-mode.md)
## 配分
### グループ管理サービスアカウント
#### [gMSA を作成します](manage-containers/manage-serviceaccounts.md)
#### [GMSA を使用するようにアプリを構成する](manage-containers/gmsa-configure-app.md)
#### [GMSA でコンテナーを実行する](manage-containers/gmsa-run-container.md)
#### [GMSA でコンテナーを調整する](manage-containers/gmsa-orchestrate-containers.md)
#### [GMSAs のトラブルシューティング](manage-containers/gmsa-troubleshooting.md)
### [プリンターサービス](deploy-containers/print-spooler.md)
## ネットワーク
### [概要](container-networking/architecture.md)
### [ネットワークトポロジとドライバー](container-networking/network-drivers-topologies.md)
### [ネットワーク分離とセキュリティ](container-networking/network-isolation-security.md)
### [詳細なネットワークオプションを構成する](container-networking/advanced.md)
## ストレージ
### [概要](manage-containers/container-storage.md)
### [常設ストレージ](manage-containers/persistent-storage.md)
## デバイス
### [ハードウェアデバイス](deploy-containers/hardware-devices-in-containers.md)
### [GPU アクセラレータ](deploy-containers/gpu-acceleration.md)

# リファレンス
## [基本イメージのサービスライフサイクル](deploy-containers/base-image-lifecycle.md)
## [ウイルス対策の最適化](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [コンテナプラットフォームツール](deploy-containers/containerd.md)
## [コンテナー OS イメージ EULA](Images_EULA.md)

# 参考資料
## [コンテナーのサンプル](samples.md)
## [トラブルシューティング](troubleshooting.md)
## [コンテナーフォーラム](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [コミュニティのビデオとブログ](communitylinks.md)