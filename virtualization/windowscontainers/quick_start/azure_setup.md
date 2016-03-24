---
author: neilpeterson
---

# Azure クイック スタート

Azure で Windows Server コンテナーの作成と管理をするには、事前に、Windows Server コンテナー機能にあらかじめ構成されている Windows Server 2016 Technical Preview イメージをデプロイする必要があります。 このガイドでは、このプロセスについて説明します。

> Microsoft Azure は、Hyper-V コンテナーをサポートしていません。 Hyper-V コンテナーの演習を完了するには、オンプレミスのコンテナー ホストが必要です。

## Azure ポータルの使用を開始します。

Azure アカウントをお持ちの場合は、この手順をスキップして[コンテナー ホスト VM の作成](#CreateacontainerhostVM)に移動します。

1. [azure.com](https://azure.com) に移動して、[Azure 無料試用版](https://azure.microsoft.com/en-us/pricing/free-trial/)の手順に従います。
2. Microsoft アカウントでサインインします。
3. アカウントの準備ができたら、[Azure 管理ポータル](https://portal.azure.com)にサインインします。

## コンテナーのホスト仮想マシンを作成します。

Azure Marketplace で「containers」を検索すると、「Windows Server 2016 Core with Containers Tech Preview 4」という検索結果が返されます。

![](./media/newazure1.png)

イメージを選択し、`[作成]` をクリックします。

![](./media/tp41.png)

により、仮想マシンの名前は、ユーザー名とパスワードを選択します。

![](media/newazure2.png)

オプションの構成の選択 > エンドポイント > 以下に示すように、プライベートおよびパブリック ポートは 80 の HTTP エンドポイントを入力します。 ときに完了した] をクリックには、2 回が [ok] です。

![](./media/newazure3.png)

`[作成]` ボタンを選択して、仮想マシンのデプロイ プロセスを開始します。

![](media/newazure2.png)

VM の展開が完了すると、Windows Server のコンテナーのホストで RDP セッションを開始する [接続] ボタンを選択します。

![](media/newazure6.png)

ユーザー名と、VM の作成ウィザードの中に指定されたパスワードを使用して VM にログインします。 Windows のコマンド プロンプトで、そのでする検索は 1 回ログに記録します。

![](media/newazure7.png)

## Docker エンジンの更新

`docker pull` を Azure Windows コンテナー Technical Preview イメージで使用するには、Docker エンジンを更新する必要があります。 この更新を実行するには、次の PowerShell コマンドを Azure 仮想マシンで実行します。

```powershell
PS C:\> wget https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/live/windows-server-container-tools/Update-ContainerHost/Update-ContainerHost.ps1 -OutFile Update-ContainerHost.ps1

PS C:\> ./Update-ContainerHost.ps1
```

## ビデオ チュートリアル

<iframe src="https://channel9.msdn.com/Blogs/containers/Quick-Start-Configure-Windows-Server-Containers-in-Microsoft-Azure/player#ccLang=ja" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>


## 次の手順 - コンテナーの使用開始

これで Windows Server コンテナーの機能を実行する Windows Server 2016 システムが準備できたので、Windows Server コンテナーおよび Windows Server コンテナー イメージの使用を開始するため、次のガイドに移動します。

[Quick Start: Windows Containers and Docker (クイック スタート: Windows コンテナーと Docker)](./manage_docker.md)  
[Quick Start: Windows Containers and PowerShell (クイック スタート: Windows コンテナーと PowerShell)](./manage_powershell.md)



<!--HONumber=Mar16_HO3-->
