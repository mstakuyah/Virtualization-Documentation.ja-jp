



# コンテナーのネットワーク

**この記事は暫定的な内容であり、変更される可能性があります。**

ネットワークに関して言えば、Windows コンテナーの機能は仮想マシンと似ています。 それぞれのコンテナーが仮想ネットワーク アダプターを備え、それが仮想スイッチに接続され、そこで受信トラフィックと送信トラフィックが転送されます。 ネットワークの構成には次の 2 種類があります。

- **ネットワーク アドレス変換モード** – 各コンテナーは内部の仮想スイッチに接続され、内部 IP アドレスを受信します。 この内部アドレスは、NAT 構成によってコンテナー ホストの外部アドレスに変換されます。

- **透過モード** – 各コンテナーは外部の仮想スイッチに接続され、DHCP サーバーから IP アドレスを受け取ります。

このドキュメントでは、それぞれの利点と構成について詳しく説明します。

## NAT ネットワーク モード

**ネットワーク アドレス変換** – この構成は、NAT タイプの内部ネットワーク スイッチと WinNat から成ります。 この構成では、ネットワーク上の到達可能な "外部" IP アドレスがコンテナーのホストに割り当てられます。 コンテナーにはいずれも、ネットワーク上でアクセスすることのできない "内部" アドレスが割り当てられます。 この構成でコンテナーへのアクセスを確保するために、ホストの外部ポートがコンテナーの内部ポートにマップされます。 これらのマッピングは、NAT ポート マッピング テーブルに格納されています。 コンテナーには、そのホストの IP アドレスと外部ポートを介してアクセスすることができます。ホスト宛てのトラフィックが、コンテナーの内部 IP アドレスとポートに転送されます。 NAT の利点は、外部に公開された IP アドレスを 1 つだけ使用して、コンテナーのホストを拡張し、最大数百個のコンテナーを収容できることです。 加えて、まったく同じ通信ポートが必要とされるアプリケーションも、NAT であれば複数のコンテナーでホストすることができます。

### ホストの構成

ネットワーク アドレス変換を使用するようにコンテナーのホストを構成するには、次の手順を実行します。

"NAT" タイプの仮想スイッチを作成し、内部サブネットを含む構成を行います。 **New-VMSwitch** コマンドの詳細については、[New-VMSwitch のリファレンス](https://technet.microsoft.com/en-us/library/hh848455.aspx)を参照してください。

```powershell
New-VMSwitch -Name "NAT" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```
ネットワーク アドレス変換オブジェクトを作成します。 このオブジェクトが NAT アドレス変換を実行します。 **New-NetNat** コマンドの詳細については、[New-NetNat のリファレンス](https://technet.microsoft.com/en-us/library/dn283361.aspx)を参照してください。

```powershell
New-NetNat -Name NAT -InternalIPInterfaceAddressPrefix "172.16.0.0/12" 
```

### コンテナーの構成

Windows コンテナーを作成するときに、そのコンテナーに使用する仮想スイッチを選択することができます。 NAT を使用するように構成された仮想スイッチにコンテナーを接続すると、変換済みのアドレスがそのコンテナーに割り当てられます。

この例で作成するコンテナーは、NAT を有効にした仮想スイッチに接続します。

```powershell
New-Container -Name DemoNAT -ContainerImageName WindowsServerCore -SwitchName "NAT"
```

コンテナーが起動済みであれば、コンテナー内からその IP アドレスを確認することができます。

```powershell
[DemoNAT]: PS C:\> ipconfig

Windows IP Configuration
Ethernet adapter vEthernet (Virtual Switch-527ED2FB-D56D-4852-AD7B-E83732A032F5-0):
   Connection-specific DNS Suffix  . : contoso.com
   Link-local IPv6 Address . . . . . : fe80::384e:a23d:3c4b:a227%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.240.0.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Windows コンテナーの起動と接続の詳細については、[コンテナーの管理](./manage_containers.md)に関するページを参照してください。

### ポートのマッピング

"NAT を有効" にしたコンテナー内のアプリケーションにアクセスするためには、コンテナーとそのホストとの間にポート マッピングを作成する必要があります。 マッピングを作成するには、コンテナーの IP アドレス、"内部" コンテナー ポート、"外部" ホスト ポートが必要です。

この例では、ホストのポート **80** と、IP アドレスが **172.16.0.2** であるコンテナーのポート **80** とを対応付けるマッピングを作成しています。

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
```

この例では、コンテナーのホストのポート **82** と、IP アドレスが **172.16.0.3** であるコンテナーのポート **80** を対応付けるマッピングを作成しています。

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.3 -InternalPort 80 -ExternalPort 82
```
> 対応するファイアウォール規則が、外部ポートごとに必要となります。 これは、`New-NetFirewallRule` で作成できます。 詳細については、[New-NetFirewallRule のリファレンス](https://technet.microsoft.com/en-us/library/jj554908.aspx)を参照してください。

ポート マッピングの作成後、コンテナーのホスト (物理または仮想) に割り当てられている IP アドレスと公開されている外部ポートを使用して、コンテナーのアプリケーションにアクセスすることができます。 たとえば、以下の図は、コンテナー ホストの外部ポート **82** を要求の宛先とした NAT 構成を表しています。 この要求は、ポート マッピングに基づいて、コンテナー 2 でホストされているアプリケーションを返します。

![](./media/nat1.png)

以下の図は、この要求をインターネット ブラウザーから見たところです。

![](./media/portmapping.png)

## 透過ネットワーク モード

**透過ネットワーク** – この構成には、外部ネットワーク スイッチが使用されます。 この構成では、各コンテナーが DHCP サーバーから IP アドレスを受け取ります。コンテナーには、その IP アドレスでアクセスすることができます。 このモードの利点は、ポート マッピング テーブルの管理が不要であることです。

### ホストの構成

DHCP サーバーから IP アドレスを受け取ることができるようにコンテナーのシステムを構成するには、物理または仮想のネットワーク アダプターに接続する仮想スイッチを作成します。

次の例では、Ethernet という名前のネットワーク アダプターを使用して、DHCP という名前の仮想スイッチを作成しています。

```powershell
New-VMSwitch -Name DHCP -NetAdapterName Ethernet
```

コンテナーのホスト自体が仮想マシンである場合は、そのコンテナーのスイッチに接続されるネットワーク アダプターの MacAddressSpoofing を有効にする必要があります。 次の例では、その設定を `DemoVm` という名前の VM に対して行っています。

```powershell
Get-VMNetworkAdapter -VMName DemoVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
外部仮想スイッチをコンテナーに接続すれば、DHCP サーバーから IP アドレスを受け取ることができます。 この構成で、コンテナーにホストされているアプリケーションにアクセスするときは、そのコンテナーに割り当てられている IP アドレスを使用することになります。

## Docker の構成

Docker デーモンを起動するときに、ネットワーク ブリッジを選択できます。 Docker を Windows で実行している場合、外部仮想スイッチまたは NAT 仮想スイッチが該当します。 次の例では、`Virtual Switch` という名前の仮想スイッチを指定して Docker デーモンを起動しています。

```powershell
Docker daemon -D -b “Virtual Switch” -H 0.0.0.0:2375
```

コンテナーのホストが展開済みで、かつ Windows コンテナー クイック スタートで提供されているスクリプトを使用して Docker を展開してある場合、NAT タイプの内部仮想スイッチが作成され、Docker サービスが作成されてそのスイッチを使用するように事前構成されます。 Docker サービスで使用する仮想スイッチを変更するには、Docker サービスを停止し、構成ファイルに変更を加えてから、もう一度サービスを開始する必要があります。

サービスを停止するには、次の PowerShell コマンドを実行します。

```powershell
Stop-Service docker
```

構成ファイルは `c:\programdata\docker\runDockerDaemon.cmd` にあります。 次の行を編集します。`Virtual Switch` の部分は、Docker サービスで使用する仮想スイッチの名前に置き換えてください。

```powershell
docker daemon -D -b “New Switch Name"
```
最後に、サービスを開始します。

```powershell
Start-Service docker
```

## ネットワーク アダプターを管理する

コンテナーのネットワーク アダプターと仮想スイッチ接続は、ネットワーク構成 (NAT または透過) に関係なく、いくつかの PowerShell コマンドを使って管理できます。

コンテナーのネットワーク アダプターを管理する

- Add-ContainerNetworkAdapter - コンテナーにネットワーク アダプターを追加します。
- Set-ContainerNetworkAdapter - コンテナーのネットワーク アダプターに変更を加えます。
- Remove-ContainerNetworkAdapter - コンテナーのネットワーク アダプターを削除します。
- Get-ContainerNetworkAdapter - コンテナーのネットワーク アダプターに関するデータを返します。

コンテナーのネットワーク アダプターと仮想スイッチとの間の接続を管理する

- Connect-ContainerNetworkAdapter - コンテナーを仮想スイッチに接続します。
- Disconnect-ContainerNetworkAdapter - コンテナーを仮想スイッチから切断します。

それぞれのコマンドの詳細については、[コンテナーの PowerShell リファレンス](https://technet.microsoft.com/en-us/library/mt433069.aspx)を参照してください。






<!--HONumber=Feb16_HO4-->


