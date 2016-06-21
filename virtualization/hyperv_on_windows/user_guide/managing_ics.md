---
title: &143268172 Hyper-V 統合サービスの管理
description: Hyper-V 統合サービスの管理
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1016392543 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 9cafd6cb-dbbe-4b91-b26c-dee1c18fd8c2
---

# Hyper-V 統合サービスの管理

統合サービス (多くの場合、統合コンポーネントと呼ばれます) は、仮想マシンから Hyper-V ホストに通信できるようにするサービスです。 このようなサービスの多くは便利なものですが (ゲスト ファイル コピーなど)、ゲスト オペレーティング システムが正しく機能するために重要なサービスもあります (時間の同期)。

この記事では、Hyper-V マネージャーと PowerShell の両方を Windows 10 で使用して、統合サービスを管理する方法について説明します。 各統合サービスの詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Integration Services (統合サービス)</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。

## Hyper-V マネージャーを使用して統合サービスを有効または無効にする

1. 仮想マシンを選択して設定を開きます。
  <g id="1" ctype="x-linkText"></g>

2. 仮想マシンの設定ウィンドウから、[管理] の [統合サービス] タブを開きます。

  <g id="1" ctype="x-linkText"></g>

  このタブには、Hyper-V ホストで使用できるすべての統合サービスが表示されます。 表示されているすべての統合サービスをゲスト オペレーティング システムがサポートしているかどうかを注意してください。

## PowerShell を使用して統合サービスを有効または無効にする

PowerShell で統合サービスを有効または無効にするには、[<g id="2" ctype="x-code">Enable-VMIntegrationService</g>](https://technet.microsoft.com/en-us/library/hh848500.aspx) と [<g id="4" ctype="x-code">Disable-VMIntegrationService</g>](https://technet.microsoft.com/en-us/library/hh848488.aspx) を実行します。

この例では、上の図のように "demovm" 仮想マシンのゲスト ファイル コピー統合サービスを有効にしてから、無効にします。

1. 実行中の統合サービスを確認します

  ``` PowerShell
  Get-VMIntegrationService -VMName "demovm"
  ```

  出力は次のようになります。
  ``` PowerShell
  VMName      Name                    Enabled PrimaryStatusDescription SecondaryStatusDescription
  ------      ----                    ------- ------------------------ --------------------------
  demovm      Guest Service Interface False   OK
  demovm      Heartbeat               True    OK                       OK
  demovm      Key-Value Pair Exchange True    OK
  demovm      Shutdown                True    OK
  demovm      Time Synchronization    True    OK
  demovm      VSS                     True    OK
  ```

2. <g id="2" ctype="x-code">ゲスト サービス インターフェイス</g>統合サービスを有効にします

   ``` PowerShell
   Enable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

   <g id="2" ctype="x-code">Get-VMIntegrationService -VMName "demovm"</g> を実行すると、そのゲスト サービス インターフェイス統合サービスが有効と表示されます。

3. <g id="2" ctype="x-code">ゲスト サービス インターフェイス</g>統合サービスを無効にします

   ``` PowerShell
   Disable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

統合サービスが機能するには、ホストとゲストの両方で有効にする必要があるように設計されました。 すべての統合サービスは Windows ゲスト オペレーティング システムの既定で有効ですが、無効にすることもできます。 その方法については、次のセクションを参照してください。


## ゲスト OS (Windows) から統合サービスを管理する

> <g id="1" ctype="x-strong">注:</g> 統合サービスを無効にすると、ホストの仮想マシンを管理する機能が深刻な影響を受けます。 統合サービスが機能するには、ホストとゲストの両方で有効にする必要があります。

統合サービスは Windows でサービスとして表示されます。 仮想マシン内から統合サービスを有効または無効にするには、Windows Server マネージャーを開きます。

<g id="1" ctype="x-linkText"></g>

Hyper-V を含むサービス名を検索します。 有効または無効にするサービスを右クリックして、サービスを開始または停止します。

または、PowerShell を使用してすべての統合サービスを表示するには、次を実行します。

```PowerShell
Get-Service -Name vm*
```

次のような一覧が返されます。

```PowerShell
Status   Name               DisplayName
------   ----               -----------
Running  vmicguestinterface Hyper-V Guest Service Interface
Running  vmicheartbeat      Hyper-V Heartbeat Service
Running  vmickvpexchange    Hyper-V Data Exchange Service
Running  vmicrdv            Hyper-V Remote Desktop Virtualizati...
Running  vmicshutdown       Hyper-V Guest Shutdown Service
Running  vmictimesync       Hyper-V Time Synchronization Service
Stopped  vmicvmsession      Hyper-V VM Session Service
Running  vmicvss            Hyper-V Volume Shadow Copy Requestor
```

[<g id="2" ctype="x-code">Start-Service</g>](https://technet.microsoft.com/en-us/library/hh849825.aspx) または [<g id="4" ctype="x-code">Stop-Service</g>](https://technet.microsoft.com/en-us/library/hh849790.aspx) を使用してサービスを開始または停止します。

たとえば、PowerShell Direct を無効にするには、<g id="2" ctype="x-code">Stop-Service -Name vmicvmsession</g> を実行できます。

既定では、ゲスト OS ですべての統合サービスが有効です。

## ゲスト OS (Linux) から統合サービスを管理する

一般的に、Linux 統合サービスは Linux カーネルで提供されます。

統合サービス ドライバーまたはデーモンが実行中かどうかを確認するには、Linux ゲスト オペレーティング システムで次のコマンドを実行します。

1. Linux 統合サービス ドライバーは "hv_utils" という名前です。 次を実行して、読み込まれているかどうかを確認します。

  ``` BASH
  lsmod | grep hv_utils
  ```

  出力は次のようになります。

  ``` BASH
  Module                  Size   Used by
  hv_utils               20480   0
  hv_vmbus               61440   8 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc
  ```

2. Linux ゲスト オペレーティング システムで次のコマンドを実行して、必要なデーモンが実行中かどうかを確認します。

  ``` BASH
  ps -ef | grep hv
  ```

  出力は次のようになります。

  ``` BASH
  root       236     2  0 Jul11 ?        00:00:00 [hv_vmbus_con]
  root       237     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  ...
  root       252     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  root      1286     1  0 Jul11 ?        00:01:11 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9333     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9365     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_vss_daemon
  scooley  43774 43755  0 21:20 pts/0    00:00:00 grep --color=auto hv          
  ```

  使用できるデーモンを表示するには、次を実行します。
  ``` BASH
  compgen -c hv_
  ```

  出力は次のようになります。

  ``` BASH
  hv_vss_daemon
  hv_get_dhcp_info
  hv_get_dns_info
  hv_set_ifconfig
  hv_kvp_daemon
  hv_fcopy_daemon     
  ```

  次のような統合サービス デーモンが表示されます。
  * **<g id="2" ctype="x-code">hv_vss_daemon</g>** - このデーモンは、Linux 仮想マシンのライブ バックアップを作成するために必要です。
  * **<g id="2" ctype="x-code">hv_kvp_daemon</g>** - このデーモンを使用すると、組み込みと外部のキーと値のペアを設定および照会できます。
  * **<g id="2" ctype="x-code">hv_fcopy_daemon</g>** - このデーモンは、ホストとゲスト間にファイル コピー サービスを実装します。

> <g id="1" ctype="x-strong">注:</g> 上の統合サービス デーモンを使用できない場合、システムでサポートされていないか、インストールできない可能性があります。 ディストリビューション固有の詳細については、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">こちら</g><g id="2CapsExtId3" ctype="x-title"></g></g>を参照してください。

この例では、KVP デーモン <g id="2" ctype="x-code">hv_kvp_daemon</g> を停止し、開始します。

上記の出力の 2 列目にある pid (プロセス ID) を使用して、デーモンのプロセスを停止します。 または、<g id="2" ctype="x-code">pidof</g> を使用して適切なプロセスを検索できます。 Hyper-V デーモンはルートとして実行されるので、ルート アクセス許可は必要ありません。

``` BASH
sudo kill -15 `pidof hv_kvp_daemon`
```

ここで <g id="2" ctype="x-code">ps -ef | hv</g> をもう一度実行すると、すべての <g id="4" ctype="x-code">hv_kvp_daemon</g> プロセスは表示されません。

デーモンを再起動するには、ルートとしてデーモンを実行します。

``` BASH
sudo hv_kvp_daemon
```

ここで <g id="2" ctype="x-code">ps -ef | hv</g> をもう一度実行すると、新しいプロセス ID の <g id="4" ctype="x-code">hv_kvp_daemon</g> プロセスが表示されます。


## 統合サービスのメンテナンス

可能な限り最高の仮想マシンのパフォーマンスと機能を実現するには、統合サービスを最新の状態に保ちます。

<g id="1" ctype="x-strong">Windows 10 ホストで実行されている仮想マシンの場合:</g>

> <g id="1" ctype="x-strong">注:</g> 統合コンポーネントを更新する際、ISO イメージ ファイルの vmguest.iso は不要になりました。 Windows 10 の Hyper-V では追加されません。

| ゲスト OS| 更新方法| メモ|
|:---------|:---------|:---------|
| Windows 10| Windows Update| |
| Windows 8.1| Windows Update| |
| Windows 8| Windows Update| データ交換統合サービスが必要です。<g id="2" ctype="x-strong">*</g>|
| Windows 7| Windows Update| データ交換統合サービスが必要です。<g id="2" ctype="x-strong">*</g>|
| Windows Vista (SP 2)| Windows Update| データ交換統合サービスが必要です。<g id="2" ctype="x-strong">*</g>|
| -| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| Windows Update| データ交換統合サービスが必要です。<g id="2" ctype="x-strong">*</g>|
| Windows Server 2008 R2 (SP 1)| Windows Update| データ交換統合サービスが必要です。<g id="2" ctype="x-strong">*</g>|
| Windows Server 2008 (SP 2)| Windows Update| Server 2016 での拡張サポートのみ (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">詳細</g><g id="2CapsExtId3" ctype="x-title"></g></g>)。|
| Windows Home Server 2011| Windows Update| Server 2016 ではサポートされません (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">詳細</g><g id="2CapsExtId3" ctype="x-title"></g></g>)。|
| Windows Small Business Server 2011| Windows Update| メインストリーム サポートではありません (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">詳細</g><g id="2CapsExtId3" ctype="x-title"></g></g>)。|
| -| | |
| Linux ゲスト| パッケージ マネージャー| Linux 用の統合コンポーネントはディストリビューションに組み込まれていますが、オプションで更新プログラムを使用できる場合があります。<g id="1" ctype="x-strong">****</g>|

<g id="1" ctype="x-strong">\*</g> データ交換統合サービスを有効にすることができない場合は、これらのゲストの統合コンポーネントは、ダウンロード センターの<g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">こちら</g><g id="3CapsExtId3" ctype="x-title"></g></g>で、キャビネット (cab) ファイルの形式で入手できます。 cab を適用する手順については、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">こちら</g><g id="2CapsExtId3" ctype="x-title"></g></g>を参照してください。


<g id="1" ctype="x-strong">Windows 8.1 ホストで実行されている仮想マシンの場合:</g>

| ゲスト OS| 更新方法| メモ|
|:---------|:---------|:---------|
| Windows 10| Windows Update| |
| Windows 8.1| Windows Update| |
| Windows 8| 統合サービス ディスク| |
| Windows 7| 統合サービス ディスク| |
| Windows Vista (SP 2)| 統合サービス ディスク| |
| Windows XP (SP 2、SP 3)| 統合サービス ディスク| |
| -| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| 統合サービス ディスク| |
| Windows Server 2008 R2| 統合サービス ディスク| |
| Windows Server 2008 (SP 2)| 統合サービス ディスク| |
| Windows Home Server 2011| 統合サービス ディスク| |
| Windows Small Business Server 2011| 統合サービス ディスク| |
| Windows Server 2003 R2 (SP 2)| 統合サービス ディスク| |
| Windows Server 2003 (SP 2)| 統合サービス ディスク| |
| -| | |
| Linux ゲスト| パッケージ マネージャー| Linux 用の統合コンポーネントはディストリビューションに組み込まれていますが、オプションで更新プログラムを使用できる場合があります。<g id="1" ctype="x-strong">****</g>|


<g id="1" ctype="x-strong">Windows 8 ホストで実行されている仮想マシンの場合:</g>

| ゲスト OS| 更新方法| メモ|
|:---------|:---------|:---------|
| Windows 8.1| Windows Update| |
| Windows 8| 統合サービス ディスク| |
| Windows 7| 統合サービス ディスク| |
| Windows Vista (SP 2)| 統合サービス ディスク| |
| Windows XP (SP 2、SP 3)| 統合サービス ディスク| |
| -| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| 統合サービス ディスク| |
| Windows Server 2008 R2| 統合サービス ディスク| |
| Windows Server 2008 (SP 2)| 統合サービス ディスク| |
| Windows Home Server 2011| 統合サービス ディスク| |
| Windows Small Business Server 2011| 統合サービス ディスク| |
| Windows Server 2003 R2 (SP 2)| 統合サービス ディスク| |
| Windows Server 2003 (SP 2)| 統合サービス ディスク| |
| -| | |
| Linux ゲスト| パッケージ マネージャー| Linux 用の統合コンポーネントはディストリビューションに組み込まれていますが、オプションで更新プログラムを使用できる場合があります。<g id="1" ctype="x-strong">****</g>|


Windows 8 および Windows 8.1 用統合サービス ディスクを使用した更新手順については、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">こちら</g><g id="2CapsExtId3" ctype="x-title"></g></g>を参照してください。

 <g id="1" ctype="x-strong">\****</g> Linux ゲストの詳細については、<g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">こちら</g><g id="3CapsExtId3" ctype="x-title"></g></g>を参照してください。






<!--HONumber=May16_HO1-->


