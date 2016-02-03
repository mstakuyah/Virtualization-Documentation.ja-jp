# 新しい Hyper-V 仮想マシンへの Windows コンテナー ホストの展開

このドキュメントでは、PowerShell スクリプトを使用して、Windows コンテナー ホストとして構成される、新しい Hyper-V 仮想マシンを展開する手順を説明します。

Windows コンテナー ホストの既存の物理システムまたは仮想システムへのスクリプトによる展開を進めるには、[Windows コンテナー ホストのインプレース展開](./inplace_setup.md)に関するページを参照してください。

**コンテナー OS イメージのインストール前にお読みください。**Microsoft Windows Server のプレリリース版ソフトウェア (「ライセンス条項」) のライセンス条項が、Microsoft Windows コンテナー OS イメージの追加機能 (以下「追加ソフトウェア」) の使用に適用されます。 追加ソフトウェアをダウンロードして使用することで、ライセンス条項に同意したことになります。ライセンス条項に同意していない場合、そのソフトウェアを使用することはできません。 Windows Server のプレリリース版ソフトウェアと追加ソフトウェアはどちらも、米国 Microsoft Corporation によってライセンス供与されています。

このクイック スタートの **Windows Server** および **Hyper-V コンテナー**の演習を完了するには、以下が必要です。

* Windows 10 ビルド 10586 以降または Windows Server Technical Preview 4 以降を実行しているシステム。
* Hyper-V の役割が有効になっていること ([手順を参照](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install#UsingPowerShell)))。
* コンテナー ホスト イメージ、OS ベース イメージ、セットアップ スクリプトに使用可能な 20 GB の記憶域。
* Hyper-V ホストに対する管理者権限。

> Hyper-V コンテナーを実行する仮想化されたコンテナー ホストには、入れ子になった仮想化が必要です。 物理ホストと仮想ホストの両方で、入れ子になった仮想化をサポートしている OS を実行している必要があります。 詳細については、[Windows Server 2016 Technical Preview](https://technet.microsoft.com/library/dn765471.aspx#BKMK_nested) での Hyper-V の新機能に関するページを参照してください。

## 新しい仮想マシンでの新しいコンテナー ホストの設定

Windows コンテナーは、Windows コンテナー ホストやコンテナー OS ベース イメージなどの複数のコンポーネントで構成されます。 当社では、ダウンロードしてこれらの項目を構成するためのスクリプトを用意しています。 これらの手順に従って、新しい Hyper-V 仮想マシンを展開し、このシステムを Windows コンテナー ホストとして構成します。

管理者として PowerShell セッションを開始します。 これを行うには、PowerShell アイコンを右クリックし、[管理者として実行] を選択するか、任意の PowerShell セッションから次のコマンドを実行します。

``` powershell
PS C:\> start-process powershell -Verb runAs
```

スクリプトをダウンロードして実行する前に、外部の Hyper-V 仮想スイッチが作成されていることを確認します。 それがないと、このスクリプトは失敗します。

外部仮想スイッチの一覧を返すには、次を実行します。 何も返されない場合は、新しい外部仮想スイッチを作成し、このガイドの次の手順に進みます。

```powershell
PS C:\> Get-VMSwitch | where {$_.SwitchType –eq “External”}
```

構成スクリプトをダウンロードするには、次のコマンドを使用します。 このスクリプトは、[構成スクリプト](https://aka.ms/tp4/New-ContainerHost)から手動でダウンロードすることもできます。

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/New-ContainerHost -OutFile c:\New-ContainerHost.ps1
```

コンテナー ホストを作成して構成するには、次のコマンドを実行します。`&lt;containerhost&gt;` は仮想マシンの名前を指定します。

``` powershell
PS C:\> c:\New-ContainerHost.ps1 –VmName <containerhost> -WindowsImage ServerDatacenterCore -Hyperv
```

スクリプトの開始時に、パスワードの入力を求めるメッセージが表示されます。 これは管理者アカウントに割り当てられているパスワードになります。

次に、ライセンス条項を読んで同意することを求められます。

```
Before installing and using the Windows Server Technical Preview 4 with Containers virtual machine you must:
    1. Review the license terms by navigating to this link: http://aka.ms/tp4/containerseula
    2. Print and retain a copy of the license terms for your records.
By downloading and using the Windows Server Technical Preview 4 with Containers virtual machine you agree to such
license terms. Please confirm you have accepted and agree to the license terms.
[N] No  [Y] Yes  [?] Help (default is "N"):
```

同意すると、スクリプトによって Windows コンテナー コンポーネントのダウンロードと構成が開始されます。 これは大規模なダウンロードのため、このプロセスにはかなり時間がかかる場合があります。 プロセスが完了すると、仮想マシンが構成され、PowerShell と Docker の両方を使用して、Windows コンテナーと Windows コンテナー イメージの作成および管理ができるようになります。

構成スクリプトが完了したら、構成プロセス中に指定したパスワードを使用して仮想マシンにログインし、仮想マシンに有効な IP アドレスがあることを確認します。 これらの項目が完了すると、システムは Windows コンテナー対応となります。

## 次の手順 - コンテナーの使用開始

これで Windows Server コンテナーの機能を実行する Windows Server 2016 システムが準備できたので、Windows Server コンテナーおよび Hyper-V コンテナーの使用を開始するには、次のガイドに移動します。

[Quick Start: Windows Containers and PowerShell (クイック スタート: Windows コンテナーと PowerShell)](./manage_powershell.md)  
[Quick Start: Windows Containers and Docker (クイック スタート: Windows コンテナーと Docker)](./manage_docker.md)




<!--HONumber=Jan16_HO2-->
