# コンテナー ホストの展開 - Nano Server

**この記事は暫定的な内容であり、変更される可能性があります。**

Windows コンテナー ホストの展開手順は、オペレーティング システムやホスト システムの種類 (物理または仮想) によって異なります。 このドキュメントの手順は、Windows コンテナー ホストを物理または仮想システム上の Nano Server に展開するために使用されます。 Windows コンテナー ホストを Windows Server にインストールするには、「[コンテナー ホストの展開 - Windows Server](./deployment.md)」を参照してください。

システム要件の詳細については、「[Windows コンテナーの要件](./system_requirements.md)」を参照してください。

PowerShell スクリプトを使用すると、Windows コンテナー ホストの展開を自動化できます。
- [新しい Hyper-V 仮想マシンでコンテナー ホストを展開する](../quick_start/container_setup.md)。
- [既存のシステムにコンテナー ホストを展開する](../quick_start/inplace_setup.md)。


# Nano Server ホスト

次の表に示された手順を使用して、Nano Server にコンテナー ホストを展開できます。 Windows Server コンテナーと Hyper-V コンテナーの両方に必要な構成が含まれています。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>展開アクション</strong></td>
<td width="70%"><strong>詳細情報</strong></td>
</tr>
<tr>
<td>[コンテナーのための Nano Server の準備](#nano)</td>
<td>コンテナーと Hyper-V の機能を持つ Nano Server VHD を準備します。</td>
</tr>
<tr>
<td>[仮想スイッチの作成](#vswitch)</td>
<td>コンテナーは、ネットワーク接続のために仮想スイッチに接続します。</td>
</tr>
<tr>
<td>[NAT の構成](#nat)</td>
<td>仮想スイッチが Network Address Translation (NAT) を使用して構成されている場合、NAT 自体に構成が必要です。</td>
</tr>
<tr>
<td>[コンテナー OS イメージのインストール](#img)</td>
<td>OS イメージは、コンテナー展開の基盤を提供します。</td>
</tr>
<tr>
<td>[Docker のインストール](#docker)</td>
<td>省略可能ですが、Docker を使用して Windows コンテナーの作成と管理を行うためには必要です。 </td>
</tr>
</table>

次の手順は、Hyper-V コンテナーを使用する場合には実行する必要があります。 * のマークが付いた手順は、コンテナー ホスト自体が Hyper-V 仮想マシンである場合にのみ必要です。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>展開アクション</strong></td>
<td width="70%"><strong>詳細情報</strong></td>
</tr>
<tr>
<td>[Hyper-V の役割の有効化](#hypv) </td>
<td>Hyper-V は Hyper-V コンテナーを使用する場合にのみ必要です。</td>
</tr>
<tr>
<td>[入れ子になった仮想化の有効化 *](#nest)</td>
<td>コンテナー ホスト自体が Hyper-V 仮想マシンの場合、入れ子になった仮想化を有効にする必要があります。</td>
</tr>
<tr>
<td>[仮想プロセッサの構成 *](#proc)</td>
<td>コンテナー ホスト自体が Hyper-V 仮想マシンの場合、少なくとも 2 つの仮想プロセッサを構成する必要があります。</td>
</tr>
<tr>
<td>[動的メモリの無効化 *](#dyn)</td>
<td>コンテナー ホスト自体が Hyper-V 仮想マシンの場合、動的メモリを無効にする必要があります。</td>
</tr>
<tr>
<td>[MAC アドレスのスプーフィングの構成 *](#mac)</td>
<td>コンテナー ホストが仮想化されている場合、MAC スプーフィングを有効にする必要があります。</td>
</tr>
</table>

## 展開の手順

### <a name=nano></a>Nano Server の準備

Nano Server の展開には、準備された仮想ハード ドライブの作成が含まれます。このドライブには、Nano Server オペレーティング システムと追加の機能パッケージが含まれます。 このガイドでは、Windows コンテナーに使用できる Nano Server 仮想ハード ドライブの準備について簡単に説明します。 Nano Server に関する詳細、および Nano Server のさまざまな展開オプションについては、[Nano Server のドキュメント](https://technet.microsoft.com/en-us/library/mt126167.aspx)を参照してください。

`nano` という名前のフォルダーを作成します。

```powershell
PS C:\> New-Item -ItemType Directory c:\nano
```

Windows Server メディア上の Nano Server フォルダーから `NanoServerImageGenerator.psm1` と `Convert WindowsImage.ps1` のファイルを見つけます。 これらのファイルを `c:\nano` にコピーします。

```powershell
#Set path to Windows Server 2016 Media
PS C:\> $WindowsMedia = "C:\Users\Administrator\Desktop\TP4 Release Media"

PS C:\> Copy-Item $WindowsMedia\NanoServer\Convert-WindowsImage.ps1 c:\nano

PS C:\> Copy-Item $WindowsMedia\NanoServer\NanoServerImageGenerator.psm1 c:\nano
```
次を実行して Nano Server 仮想ハード ドライブを作成します。 `–Containers` パラメーターは、コンテナー パッケージがインストールされることを示し、`–Compute` パラメーターは Hyper-V パッケージを処理します。 Hyper-V は Hyper-V コンテナーを使用する場合にのみ必要です。

```powershell
PS C:\> Import-Module C:\nano\NanoServerImageGenerator.psm1

PS C:\> New-NanoServerImage -MediaPath $WindowsMedia -BasePath c:\nano -TargetPath C:\nano\NanoContainer.vhdx -MaxSize 10GB -GuestDrivers -ReverseForwarders -Compute -Containers
```
完了したら、`NanoContainer.vhdx` ファイルから仮想マシンを作成します。 この仮想マシンは、オプションのパッケージとともに Nano Server OS を実行します。

### <a name=vswitch></a>仮想スイッチの作成

各コンテナーは、ネットワーク経由で通信するために仮想スイッチに接続する必要があります。 仮想スイッチは、`New-VMSwitch` コマンドで作成されます。 コンテナーは、`外部`または `NAT` の種類の仮想スイッチをサポートします。 Windows コンテナーのネットワークの詳細については、「[コンテナーのネットワーク](../management/container_networking.md)」を参照してください。

この例では、種類が NAT で、NAT サブネットが 172.16.0.0/12 の “Virtual Switch” という名前の仮想スイッチを作成します。

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```

### <a name=nat></a>NAT の構成

仮想スイッチの作成に加え、スイッチの種類が NAT の場合には、NAT オブジェクトを作成する必要があります。 これは `New-NetNat` コマンドを使用することで完了できます。 この例では、`ContainerNat` という名前の NAT オブジェクトと、コンテナー スイッチに割り当てられる NAT サブネットに一致するアドレスのプレフィックスを作成します。

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

### <a name=img></a>OS イメージのインストール

OS イメージは、任意の Windows Server コンテナーまたは Hyper-V コンテナーのベースとして使用されます。 このイメージは、コンテナーの展開に使用されます。展開されたコンテナーは、変更が可能で、新しいコンテナー イメージにキャプチャできます。 OS イメージは、基になるオペレーティング システムとして Windows Server Core と Nano Server の両方で作成されています。

コンテナー OS イメージは、ContainerProvider PowerShell モジュールを使用して検索およびインストールできます。 このモジュールを使用するには、先にインストールする必要があります。 モジュールをインストールするには、次のコマンドを使用できます。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

`Find-ContainerImage` を使用すると、PowerShell OneGet パッケージ マネージャーからイメージの一覧が返されます。

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```
**注** - この時点で Nano Server OS イメージと互換性があるのは Nano Server コンテナー ホストだけです。 Nano Server ベースの OS イメージをダウンロードしてインストールするには、次を実行します。

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

`Get-ContainerImage` コマンドを使用して、イメージがインストールされていることを確認します。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
```
コンテナー イメージの管理の詳細については、[Windows コンテナー イメージ](../management/manage_images.md)に関するドキュメントを参照してください。


### <a name=docker></a>Docker のインストール

Docker デーモンとコマンド ライン インターフェイスは、Windows に付属していません。また、Windows コンテナー機能と一緒にインストールされません。 Docker は、Windows コンテナーを使用するための要件ではありません。 Docker をインストールする場合は、この記事「[Docker と Windows](./docker_windows.md)」の手順に従います。


## Hyper-V コンテナーのホスト

### <a name=hypv></a>Hyper-V の役割の有効化

Nano サーバーでは、Nano Server イメージを作成するときに完了できます。 この手順については、「[コンテナーのための Nano Server の準備](#nano)」を参照してください。

### <a name=nest></a>入れ子になった仮想化の構成

コンテナー ホスト自体が Hyper-V 仮想マシン上で実行され、Hyper-V コンテナーもホストする場合、入れ子になった仮想化を有効にする必要があります。 この処理は、次の PowerShell コマンドを使って完了できます。

**注意** - このコマンドを実行するときには、仮想マシンをオフにする必要があります。

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>仮想プロセッサの構成

コンテナー ホスト自体が Hyper-V 仮想マシン上で実行され、Hyper-V コンテナーもホストする場合、仮想マシンには少なくとも 2 つのプロセッサが必要になります。 これは、仮想マシンの設定を通じて、または次のコマンドを使って構成できます。

**注意** - このコマンドを実行するときには、仮想マシンをオフにする必要があります。

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=dyn></a>動的メモリの無効化

コンテナー ホスト自体が Hyper-V 仮想マシンの場合、コンテナー ホストの仮想マシンで動的メモリを無効にする必要があります。 これは、仮想マシンの設定を通じて、または次のコマンドを使って構成できます。

**注意** - このコマンドを実行するときには、仮想マシンをオフにする必要があります。

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>MAC アドレスのスプーフィングの構成

最後に、コンテナー ホストが Hyper-V 仮想マシンの内部で実行されている場合には、MAC スプーフィングを有効化する必要があります。 これにより、各コンテナーが IP アドレスを受け取れるようになります。 MAC アドレスのスプーフィングを有効にするには、Hyper-V ホストで次のコマンドを実行します。 VMName プロパティは、コンテナー ホストの名前になります。

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```




<!--HONumber=Feb16_HO1-->
