---
title: "入れ子になった仮想化"
description: "入れ子になった仮想化"
keywords: "windows 10、hyper-v"
author: theodthompson
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: 7d16fcf22187ae3ace25fe1bedbc02f3c6b63eb8
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2017
---
# 入れ子になった仮想化による仮想マシンでの Hyper-V の実行

入れ子になった仮想化は、Hyper-V 仮想マシン内での Hyper-V の実行を可能にする機能です。 つまり、入れ子になった仮想化により、Hyper-V ホスト自体を仮想化することができます。 入れ子になった仮想化の使用例として、仮想化されたコンテナー ホストで Hyper-V コンテナーを実行したり、仮想化環境に Hyper-V ラボをセットアップしたり、個別のハードウェアを必要とせずに複数コンピューターのシナリオをテストしたりする方法があります。 このドキュメントでは、ソフトウェアとハードウェアの前提条件、構成手順、および制限事項について詳しく説明します。 

## 必要条件

- Windows Server 2016 または Windows 10 Anniversary Update を実行する Hyper-V ホスト。
- Windows Server 2016 または Windows 10 Anniversary Update を実行する Hyper-V VM。
- 構成バージョン 8.0 以上の HYPER-V VM。
- Intel プロセッサ (VT-x/EPT テクノロジ搭載)。

## 入れ子になった仮想化の構成

1. 仮想マシンを作成します。 必要な OS と VM のバージョンについては、前述の前提条件をご覧ください。
2. 仮想マシンがオフ状態のときに、次のコマンドを物理的 Hyper-V ホスト上で実行します。 これで、この仮想マシンに対して入れ子になった仮想化が有効になります。

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 仮想マシンを開始します。
4. Hyper-V を仮想マシン内にインストールします。方法は物理サーバーの場合と同様です。 Hyper-V のインストールの詳細については、[Hyper-V のインストール](../quick-start/enable-hyper-v.md)に関するページを参照してください。

## 入れ子になった仮想化の無効化
停止している仮想マシンに対して、入れ子になった仮想化を無効にするには、次の PowerShell コマンドを使用します。
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## 動的メモリとランタイム メモリ サイズ変更
Hyper-V が仮想マシンの中で実行されているときに、仮想マシンのメモリを調整するには、仮想マシンをオフにする必要があります。 つまり、動的メモリが有効になっている場合でも、メモリの量が変動することはありません。 動的メモリが有効になっていない仮想マシンでは、仮想マシンがオンの状態でメモリの量を調整しようとすると失敗します。 

なお、入れ子になった仮想化を有効にするだけでは、動的メモリやランタイム メモリ サイズ変更への影響はありません。 これらに対応できなくなるのは、Hyper-V が VM の中で実行中のときに限られます。

## ネットワーク オプション
入れ子になった仮想マシンとのネットワーキングのオプションには、MAC アドレスのスプーフィングと NAT モードの 2 つがあります。

### MAC アドレスのスプーフィング
ネットワーク パケットを 2 つの仮想スイッチを通じてルーティングするには、MAC アドレスのスプーフィングを仮想スイッチの最初のレベルで有効にする必要があります。 この処理は、次の PowerShell コマンドを使って完了できます。

```none
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### ネットワーク アドレス変換
2 番目のオプションは、ネットワーク アドレス変換 (NAT) に依存します。 この方法は、MAC アドレスのスプーフィングを利用できない場合 (パブリック クラウド環境など) に最適です。

最初に、仮想 NAT スイッチをホストの仮想マシン ("中間"の VM) に作成する必要があります。 IP アドレスはただの一例で、環境によって異なることに注意してください。
```none
new-vmswitch -name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
次に、IP アドレスをネット アダプターに割り当てます。
```none
get-netadapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
入れ子になった仮想マシンごとに 1 つの IP アドレスと 1 つのゲートウェイが割り当てられている必要があります。 ゲートウェイ IP は、前の手順で説明した NAT アダプターをポイントする必要があります。 必要に応じて、DNS サーバーも割り当てます。
```none
get-netadapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## サード パーティの仮想化アプリ
Hyper-V 以外の仮想化アプリケーションは Hyper-V 仮想マシンではサポートされず、ほとんどの場合は実行に失敗します。 これには、ハードウェア仮想化拡張機能を必要とするソフトウェアも含まれます。
