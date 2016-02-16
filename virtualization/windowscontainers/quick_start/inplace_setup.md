# 既存の仮想システムまたは物理システムへの Windows コンテナー ホストの展開

このドキュメントでは、PowerShell スクリプトを使用して既存の物理システムまたは仮想システムに Windows コンテナー ロールを展開および構成する手順を説明します。

Windows コンテナー ホストとして構成された新しい Hyper-V 仮想マシンのスクリプト化された展開の手順を実行するには、「[新しい Hyper-V 仮想マシンへの Windows コンテナー ホストの展開](./container_setup.md)」を参照してください。

**コンテナー OS イメージのインストール前にお読みください。**Microsoft Windows Server のプレリリース版ソフトウェア (「ライセンス条項」) のライセンス条項が、Microsoft Windows コンテナー OS イメージの追加機能 (以下「追加ソフトウェア」) の使用に適用されます。 追加ソフトウェアをダウンロードして使用することで、ライセンス条項に同意したことになります。ライセンス条項に同意していない場合、そのソフトウェアを使用することはできません。 Windows Server のプレリリース版ソフトウェアと追加ソフトウェアはどちらも、米国 Microsoft Corporation によってライセンス供与されています。

このクイック スタートの Windows Server コンテナーおよび Hyper-V コンテナーの演習を完了するには、以下が必要です。

* Windows Server Technical Preview 4 以降を実行しているシステム。
* コンテナー ホスト イメージ、OS ベース イメージ、およびセットアップ スクリプトに使用可能な 10 GB の記憶域。
* システムでの管理者権限。

## コンテナーに既存の仮想マシンまたはベア メタル ホストを設定する

Windows コンテナーでは、コンテナー OS ベース イメージが必要です。 当社では、ダウンロードしてこれをインストールするためのスクリプトを用意しています。 システムを Windows コンテナー ホストとして構成するには、これらの手順に従います。 詳細については、[Windows Server 2016 Technical Preview](https://tnstage.redmond.corp.microsoft.com/en-US/library/dn765471.aspx#BKMK_nested) での Hyper-V の新機能に関するページを参照してください。

管理者として PowerShell セッションを開始します。 これは、コマンドラインから次のコマンドを実行することで行えます。

``` powershell
PS C:\> powershell.exe
```

ウィンドウのタイトルが「管理者: Windows PowerShell」になっていることを確認します。 管理者になっていない場合、このコマンドを実行して管理者権限で実行します。

``` powershell
PS C:\> start-process powershell -Verb runas
```

次のコマンドを使用して、セットアップ スクリプトをダウンロードします。 このスクリプトは、[構成スクリプト](https://aka.ms/tp4/Install-ContainerHost)から手動でダウンロードすることもできます。

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/Install-ContainerHost -OutFile C:\Install-ContainerHost.ps1
```

 ダウンロードが完了したら、スクリプトを実行します。
``` PowerShell
PS C:\> powershell.exe -NoProfile C:\Install-ContainerHost.ps1 -HyperV
```

同意すると、スクリプトによって Windows コンテナー コンポーネントのダウンロードと構成が開始されます。 これは大規模なダウンロードのため、このプロセスにはかなり時間がかかる場合があります。 処理中に、コンピューターが再起動する場合があります。 プロセスが完了すると、マシンが構成され、PowerShell と Docker の両方を使用して、Windows コンテナーと Windows コンテナー イメージの作成および管理ができるようになります。

 これらの項目が完了すると、システムは Windows コンテナー対応となります。

## 次の手順 - コンテナーの使用開始

これで Windows Server コンテナーの機能を実行する Windows Server 2016 システムが準備できたので、Windows Server コンテナーおよび Hyper-V コンテナーの使用を開始するには、次のガイドに移動します。

[Quick Start: Windows Containers and Docker (クイック スタート: Windows コンテナーと Docker)](./manage_docker.md)

[Quick Start: Windows Containers and PowerShell (クイック スタート: Windows コンテナーと PowerShell)](./manage_powershell.md)




<!--HONumber=Feb16_HO2-->
