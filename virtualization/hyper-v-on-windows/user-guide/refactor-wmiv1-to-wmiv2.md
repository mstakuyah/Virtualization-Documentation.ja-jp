---
title: Hyper-V WMIv1 から WMIv2 への移植
description: Hyper-V WMIv1 から WMIv2 に移植する方法について説明します。
keywords: windows 10, hyper-v, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: 963a1cc356c34c8d051c427a069c49021e3c0d27
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439339"
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>Hyper-V WMI v1 から WMI v2 への移行

Windows Management Instrumentation (WMI) は、Hyper-V マネージャーと Hyper-V の PowerShell コマンドレットの基になる管理インターフェイスです。  ほとんどのユーザーは、マイクロソフトの PowerShell コマンドレットまたは Hyper-V マネージャーを使用しますが、開発者は直接 WMI を必要とする場合がありました。  

Hyper-V WMI 名前空間 (つまり Hyper-V WMI API のバージョン) は 2 つあります。
* WMI v1 名前空間 (root\virtualization) は、Windows Server 2008 で導入され、Windows Server 2012 まで使用することができました。
* WMI v2 名前空間 (root\virtualization\v2) は、Windows Server 2012で導入されました。

このドキュメントには、以前の WMI 名前空間と通信するコードを、新しい名前空間に変換するためのリソースへのリファレンス情報が含まれています。  この記事では、最初に API 情報のリポジトリとなる情報を示し、Hyper-V WMI API を使用するプログラムやスクリプトを、v1 名前空間から v2 名前空間に移植するために使用できるサンプル コードやスクリプトを示します。

## <a name="msdn-samples"></a>MSDN サンプル

[Hyper-v 仮想マシンの移行のサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Hyper-v 仮想ファイバーチャネルのサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Hyper-v の計画された仮想マシンのサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Hyper-v アプリケーションの正常性監視のサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[バーチャルハードディスクの管理のサンプル](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Hyper-v レプリケーションのサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Hyper-v メトリックのサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Hyper-v 動的メモリのサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Hyper-v 拡張可能スイッチ拡張機能フィルタードライバー](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Hyper-v ネットワークのサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Hyper-v リソースプールの管理のサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Hyper-v 回復スナップショットのサンプル](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>ブログからのサンプル

[Hyper-v WMI V2 名前空間を使用した VM へのネットワークアダプターの追加](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-v WMI V2 名前空間を使用して VM ネットワークアダプターをスイッチに接続する](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-v WMI V2 名前空間を使用して NIC の MAC アドレスを変更する](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-v WMI V2 名前空間を使用した VM へのネットワークアダプターの削除](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-v WMI V2 名前空間を使用して VM に VHD を接続する](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-v WMI V2 名前空間を使用した VM からの VHD の削除](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-v WMI V2 名前空間を使用した VM の作成](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)

