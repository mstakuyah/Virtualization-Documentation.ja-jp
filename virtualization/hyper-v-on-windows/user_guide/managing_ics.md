---
title: "Hyper-V 統合サービスの管理"
description: "Hyper-V 統合サービスの管理"
keywords: "windows 10, hyper-v, 統合サービス, 統合コンポーネント"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 9cafd6cb-dbbe-4b91-b26c-dee1c18fd8c2
redirect_url: https://technet.microsoft.com/windows-server-docs/compute/hyper-v/manage/manage-Hyper-V-integration-services
ms.openlocfilehash: 83bcc4c2f47e2a3921be257f45a3a0e22dcba89a
ms.sourcegitcommit: fd6c5ec419aae425af7ce6c6a44d59c98f62502a
ms.translationtype: HT
ms.contentlocale: ja-JP
---
# <a name="managing-hyper-v-integration-services"></a>Hyper-V 統合サービスの管理

統合サービス (多くの場合、統合コンポーネントと呼ばれます) は、仮想マシンから Hyper-V ホストに通信できるようにするサービスです。 このようなサービスの多くは便利なものですが (ゲスト ファイル コピーなど)、仮想マシンが正しく機能するために重要なサービスもあります (時間の同期)。

この記事では、Hyper-V マネージャーと PowerShell の両方を Windows 10 で使用して、統合サービスを管理する方法について説明します。  

各統合サービスの詳細については、「[Integration Services](../reference/integration-services.md)」 (統合サービス) を参照してください。

## <a name="enable-or-disable-integration-services-using-hyper-v-manager"></a>Hyper-V マネージャーを使用して統合サービスを有効または無効にする

1. 仮想マシンを選択して設定を開きます。
  
2. 仮想マシンの設定ウィンドウから、[管理] の [統合サービス] タブを開きます。
  
  このタブには、Hyper-V ホストで使用できるすべての統合サービスが表示されます。  表示されているすべての統合サービスをゲスト オペレーティング システムがサポートしているかどうかを注意してください。 ゲスト オペレーティング システムのバージョン情報を特定するには、ゲスト オペレーティング システムにログオンしてコマンド プロンプトから次のコマンドを実行します。

REG QUERY "HKLM\Software\Microsoft\Virtual Machine\Auto" /v IntegrationServicesVersion

## <a name="enable-or-disable-integration-services-using-powershell"></a>PowerShell を使用して統合サービスを有効または無効にする

統合サービスは、PowerShell を使用して [`Enable-VMIntegrationService`](https://technet.microsoft.com/en-us/library/hh848500.aspx) または [`Disable-VMIntegrationService`](https://technet.microsoft.com/en-us/library/hh848488.aspx) を実行して、有効または無効にすることもできます。

この例では、上の図のように "demovm" 仮想マシンのゲスト ファイル コピー統合サービスを有効にしてから、無効にします。

1. 実行中の統合サービスを確認します
  
  ``` PowerShell
  Get-VMIntegrationService -VMName "DemoVM"
  ```

  出力は次のようになります。  
  ``` PowerShell
  VMName      Name                    Enabled PrimaryStatusDescription SecondaryStatusDescription
  ------      ----                    ------- ------------------------ --------------------------
  DemoVM      Guest Service Interface False   OK
  DemoVM      Heartbeat               True    OK                       OK
  DemoVM      Key-Value Pair Exchange True    OK
  DemoVM      Shutdown                True    OK
  DemoVM      Time Synchronization    True    OK
  DemoVM      VSS                     True    OK
  ```

2. `Guest Service Interface` 統合サービスを有効にします。

   ``` PowerShell
   Enable-VMIntegrationService -VMName "DemoVM" -Name "Guest Service Interface"
   ```
   
   `Get-VMIntegrationService -VMName "DemoVM"` を実行すると、そのゲスト サービス インターフェイス統合サービスが有効と表示されます。
 
3. `Guest Service Interface` 統合サービスを無効にします。

   ``` PowerShell
   Disable-VMIntegrationService -VMName "DemoVM" -Name "Guest Service Interface"
   ```
   
統合サービスが機能するには、ホストとゲストの両方で有効にする必要があるように設計されました。  すべての統合サービスは Windows ゲスト オペレーティング システムの既定で有効ですが、無効にすることもできます。  その方法については、次のセクションを参照してください。


## <a name="manage-integration-services-from-guest-os-windows"></a>ゲスト OS (Windows) から統合サービスを管理する

> **注:** 統合サービスを無効にすると、ホストの仮想マシンを管理する機能が深刻な影響を受けます。  統合サービスが機能するには、ホストとゲストの両方で有効にする必要があります。

統合サービスは Windows でサービスとして表示されます。 仮想マシン内から統合サービスを有効または無効にするには、Windows Server マネージャーを開きます。

![](../user-guide/media/HVServices.png) 

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

[`Start-Service`](https://technet.microsoft.com/en-us/library/hh849825.aspx) または [`Stop-Service`](https://technet.microsoft.com/en-us/library/hh849790.aspx) を使用してサービスを開始または停止します。

たとえば、PowerShell Direct を無効にするには、`Stop-Service -Name vmicvmsession` を実行できます。

既定では、ゲスト OS ですべての統合サービスが有効です。

## <a name="manage-integration-services-from-guest-os-linux"></a>ゲスト OS (Linux) から統合サービスを管理する

一般的に、Linux 統合サービスは Linux カーネルで提供されます。

統合サービス ドライバーまたはデーモンが実行中かどうかを確認するには、Linux ゲスト オペレーティング システムで次のコマンドを実行します。

1. Linux 統合サービス ドライバーは "hv_utils" という名前です。  次を実行して、読み込まれているかどうかを確認します。

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
  * **`hv_vss_daemon`** - このデーモンは、Linux 仮想マシンのライブ バックアップを作成するために必要です。
  * **`hv_kvp_daemon`** - このデーモンを使用すると、組み込みと外部のキーと値のペアを設定および照会できます。
  * **`hv_fcopy_daemon`** - このデーモンは、ホストとゲスト間にファイル コピー サービスを実装します。

> **注:** 上の統合サービス デーモンを使用できない場合、システムでサポートされていないか、インストールできない可能性があります。  ディストリビューション固有の詳細については、[こちら](https://technet.microsoft.com/en-us/library/dn531030.aspx)を参照してください。  

この例では、KVP デーモン `hv_kvp_daemon` を停止し、開始します。

上記の出力の 2 列目にある pid (プロセス ID) を使用して、デーモンのプロセスを停止します。  または、`pidof` を使用して適切なプロセスを検索できます。  Hyper-V デーモンはルートとして実行されるので、ルート アクセス許可は必要ありません。

``` BASH
sudo kill -15 `pidof hv_kvp_daemon`
```

ここで `ps -ef | hv` をもう一度実行すると、すべての `hv_kvp_daemon` プロセスは表示されません。

デーモンを再起動するには、ルートとしてデーモンを実行します。

``` BASH
sudo hv_kvp_daemon
``` 

ここで `ps -ef | hv` をもう一度実行すると、新しいプロセス ID の `hv_kvp_daemon` プロセスが表示されます。


## <a name="integration-service-maintenance"></a>統合サービスのメンテナンス

Windows 10 での統合サービスのメンテナンスは、既定では、仮想マシンが Windows Update から重要な更新プログラムを受信できる限り実行されます。  

統合サービスを最新の状態に保つことで、仮想マシンのパフォーマンスと機能を最大限に引き出すことができます。

**Windows 10 ホストで実行されている仮想マシンの場合:**

> **注: **統合コンポーネントを更新する際、ISO イメージ ファイルの vmguest.iso は不要になりました。 Windows 10 の Hyper-V では追加されません。

| ゲスト OS | 更新方法 | 注 |
|:---------|:---------|:---------|
| Windows 10 | Windows Update | |
| Windows 8.1 | Windows Update | |
| Windows 8 | Windows Update | データ交換統合サービスが必要です。* |
| Windows 7 | Windows Update | データ交換統合サービスが必要です。* |
| Windows Vista (SP 2) | Windows Update | データ交換統合サービスが必要です。* |
| - | | |
| Windows Server 2012 R2 | Windows Update | |
| Windows Server 2012 | Windows Update | データ交換統合サービスが必要です。* |
| Windows Server 2008 R2 (SP 1) | Windows Update | データ交換統合サービスが必要です。* |
| Windows Server 2008 (SP 2) | Windows Update | Server 2016 での拡張サポートのみ ([詳細](https://support.microsoft.com/en-us/lifecycle?p1=12925))。 |
| Windows Home Server 2011 | Windows Update | Server 2016 ではサポートされません ([詳細](https://support.microsoft.com/en-us/lifecycle?p1=15820))。 |
| Windows Small Business Server 2011 | Windows Update | メインストリーム サポートではありません ([詳細](https://support.microsoft.com/en-us/lifecycle?p1=15817))。 |
| - | | |
| Linux ゲスト | パッケージ マネージャー | Linux 用の統合コンポーネントはディストリビューションに組み込まれていますが、オプションで更新プログラムを使用できる場合があります。 ******** |

>  \* データ交換統合サービスを有効にすることができない場合は、これらのゲストの統合コンポーネントは、ダウンロード センターの[こちら](https://support.microsoft.com/en-us/kb/3071740)で、キャビネット (cab) ファイルの形式で入手できます。  
  cab を適用する手順については、[こちら](http://blogs.technet.com/b/virtualization/archive/2015/07/24/integration-components-available-for-virtual-machines-not-connected-to-windows-update.aspx)を参照してください。


**Windows 8.1 ホストで実行されている仮想マシンの場合:**

| ゲスト OS | 更新方法 | 注 |
|:---------|:---------|:---------|
| Windows 10 | Windows Update | |
| Windows 8.1 | Windows Update | |
| Windows 8 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows 7 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Vista (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows XP (SP 2、SP 3) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| - | | |
| Windows Server 2012 R2 | Windows Update | |
| Windows Server 2012 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Server 2008 R2 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Server 2008 (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Home Server 2011 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Small Business Server 2011 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Server 2003 R2 (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Server 2003 (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| - | | |
| Linux ゲスト | パッケージ マネージャー | Linux 用の統合コンポーネントはディストリビューションに組み込まれていますが、オプションで更新プログラムを使用できる場合があります。 ** |


**Windows 8 ホストで実行されている仮想マシンの場合:**

| ゲスト OS | 更新方法 | メモ |
|:---------|:---------|:---------|
| Windows 8.1 | Windows Update | |
| Windows 8 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows 7 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Vista (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows XP (SP 2、SP 3) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| - | | |
| Windows Server 2012 R2 | Windows Update | |
| Windows Server 2012 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Server 2008 R2 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。|
| Windows Server 2008 (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Home Server 2011 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Small Business Server 2011 | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Server 2003 R2 (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| Windows Server 2003 (SP 2) | 統合サービス ディスク | 利用可能な手順については、[こちら](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4)を参照してください。 |
| - | | |
| Linux ゲスト | パッケージ マネージャー | Linux 用の統合コンポーネントはディストリビューションに組み込まれていますが、オプションで更新プログラムを使用できる場合があります。 ** |

 > Linux ゲストの詳細については、[こちら](https://technet.microsoft.com/en-us/library/dn531030.aspx)を参照してください。 
