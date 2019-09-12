# [Windows ドキュメントのコンテナー](index.md) 

# 概要
## [Windows コンテナーについて](about/index.md)
## [システム要件](deploy-containers/system-requirements.md)
## [FAQ](about/faq.md)

# はじめに
## Windows 10
### [初めての WCOW コンテナーを実行する](quick-start/quick-start-windows-10.md)
### [サンプル アプリのビルド](quick-start/building-sample-app.md)
## Windows Server
### [Windows Server の初めてのコンテナーを実行します。](quick-start/quick-start-windows-server.md)
### [コンテナーのビルドを自動化する](quick-start/quick-start-images.md)
## Windows Insider
### [Insider 画像を使用する](quick-start/Using-Insider-Container-Images.md)
### [アプリケーションをビルドして実行する](quick-start/Nano-RS3-.NET-Core-and-PS.md)

# チュートリアル
## Windows コンテナーを構築する
### [Dockerfile を作成する](manage-docker/manage-windows-dockerfile.md)
### [Dockerfile を最適化する](manage-docker/optimize-windows-dockerfile.md)
## Windows で使用する Kubernetes
### [Windows で使用する Kubernetes](kubernetes/getting-started-kubernetes-windows.md)
### [Kubernetes マスターを作成する](kubernetes/creating-a-linux-master.md)
### [ネットワークソリューションを選ぶ](kubernetes/network-topologies.md)
### [Windows ワーカーに参加する](kubernetes/joining-windows-workers.md)
### [Linux ワーカに参加する](kubernetes/joining-linux-workers.md)
### [Kubernetes リソースの展開](kubernetes/deploying-resources.md)
### [トラブルシューティング](kubernetes/common-problems.md)
### [Kubernetes の Windows サービス](kubernetes/kube-windows-services.md)
### [コンパイル Kubernetes バイナリ](kubernetes/compiling-kubernetes-binaries.md)
## Windows のサービスファブリック
### [最初のコンテナーの展開](/azure/service-fabric/service-fabric-quickstart-containers)
### [Windows コンテナー内での .NET アプリケーションの展開](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Windows の Linux コンテナー
### [概要](deploy-containers/linux-containers.md)
### [最初の LCOW コンテナーを実行する](quick-start/quick-start-windows-10-linux.md)

# 概念
## Docker
### [Windows 上の Docker エンジン](manage-docker/configure-docker-daemon.md)
### [Docker 群れ](manage-containers/swarm-mode.md)
### [Windows Docker host のリモート管理](management/manage_remotehost.md)
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
## デバイス
### [ハードウェアデバイス](deploy-containers/hardware-devices-in-containers.md)
### [GPU アクセラレータ](deploy-containers/gpu-acceleration.md)
## [リソースコントロール](manage-containers/resource-controls.md)
## [Hyper-V による分離](manage-containers/hyperv-container.md)

# リファレンス
## [バージョンの互換性](deploy-containers/version-compatibility.md)
## [基本イメージのサービスライフサイクル](deploy-containers/base-image-lifecycle.md)
## [ウイルス対策の最適化](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [コンテナプラットフォームツール](deploy-containers/containerd.md)
## [コンテナー OS イメージ EULA](Images_EULA.md)

# 参考資料
## [コンテナーのサンプル](samples.md)
## [トラブルシューティング](troubleshooting.md)
## [コンテナーフォーラム](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [コミュニティのビデオとブログ](communitylinks.md)