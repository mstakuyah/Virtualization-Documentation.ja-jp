---
title: "入れ子になった仮想化"
description: "入れ子になった仮想化"
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.sourcegitcommit: 26a8adb426a7cf859e1a9813da2033e145ead965
ms.openlocfilehash: d17413fc572e59ec21ff513ef5de994c6716aa08

---

# 入れ子になった仮想化による仮想マシンでの Hyper-V の実行

入れ子になった仮想化は、Hyper-V 仮想マシン内での Hyper-V の実行を可能にする機能です。 つまり、入れ子になった仮想化により、Hyper-V ホスト自体を仮想化することができます。 入れ子になった仮想化の使用例として、仮想化されたコンテナー ホストで Hyper-V コンテナーを実行したり、仮想化環境に Hyper-V ラボをセットアップしたり、個別のハードウェアを必要とせずに複数コンピューターのシナリオをテストしたりする方法があります。 このドキュメントは、ソフトウェアとハードウェアの前提条件、および構成手順について詳しく説明し、トラブルシューティングの詳細を提供します。 Hyper-V を Windows Insider Preview ビルド 14361 以降で実行する場合は、[Windows Insider: ビルド 14361+ の入れ子になった仮想化プレビュー](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting#nested-virtualization-preview-for-windows-insiders-builds-14361-)をご覧ください。

## 必要条件

- ビルド 10565 またはそれ以降を実行する Windows Insider ビルド (Windows Server 2016 または Windows 10) を実行する Hyper-V ホスト。
- 両方のハイパーバイザー (親と子) が、同一の Windows のビルド (10565 またはそれ以降) を実行している必要があります。
- Intel VT-x および EPT テクノロジを使用する Intel プロセッサ。

## 入れ子になった仮想化の構成

最初に仮想マシンを作成します。**仮想マシンの電源は入れないでください**。 詳細については、[仮想マシンの作成](../quick_start/walkthrough_create_vm.md)に関するページを参照してください。

仮想マシンが作成されたら、物理的な Hyper-V ホスト上の次のコマンドを実行します。 これにより、仮想マシンでの入れ子になった仮想化が有効になります。

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
入れ子になった Hyper-V ホストを実行するときは、仮想マシンで動的メモリを無効にする必要があります。 これは、仮想マシンのプロパティで、または次の PowerShell スクリプトを使って構成できます。
```none
Set-VMMemory -VMName <VMName> -DynamicMemoryEnabled $false
```

上記の手順が完了すると、仮想マシンを起動し、Hyper-V をインストールすることができます。 Hyper-V のインストールの詳細については、[Hyper-V のインストール]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)に関するページを参照してください。

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


## 既知の問題

- Device Guard が有効になっているホストは、仮想化拡張機能をゲストに公開できません。
- 仮想化ベースのセキュリティ (VBS) が有効になった仮想マシンでは、入れ子も同時に有効にすることはできません。 入れ子になった仮想化を使用するには、まず VBS を無効にする必要があります。
- 仮想マシンで入れ子になった仮想化を有効にすると、次の機能とその VM との互換性がなくなります。  
  * ランタイム メモリのサイズ変更と動的メモリ
  * チェックポイント
  * Hyper-V が有効になった仮想マシンをライブ移行することはできません。

## よく寄せられる質問とトラブルシューティング

仮想マシンが起動しませんが、どうすればよいですか。

1. 動的メモリがオフになっていることを確認してください。
2. 実行できる Intel プロセッサがあることを確認してください。
3. 管理者特権でのプロンプトから、ホスト コンピューターで[この PowerShell スクリプト](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)を実行します。

## フィードバック

Windows フィードバック アプリ、[仮想化フォーラム](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv)、または [GitHub](https://github.com/Microsoft/Virtualization-Documentation) を使用してその他の問題を報告してください。

##Windows Insider: ビルド 14361+ の入れ子になった仮想化プレビュー
数か月前に、HYPER-V で入れ子になった仮想化の初期プレビューがビルド 10565 で発表されました。 期待されていた機能がついに発表されました。Windows Insider で更新プログラムを共有しましょう。

###入れ子になった仮想化に必要な新しい VM バージョン
14361 ビルド以降、入れ子になった仮想化が有効な VMには、バージョン 8.0 が必要になります。 このため、以前のホストで作成された VM の入れ子を有効にするには、バージョンの更新が必要になります。 

####VM バージョンの更新
入れ子になった仮想化の使用を続けるには、VM バージョンを 8.0 に更新する必要があります。 つまり、保存された状態を削除し、VM をシャット ダウンする必要があります。 次の PowerShell コマンドレットを実行すると VM バージョンが更新されます。
```none
Update-VMVersion -Name <VMName>
```
####入れ子になった仮想化の無効化
VM を更新しない場合でも、入れ子になった仮想化を無効にすれば VM を起動できます。
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

###VM バージョン 8.0 の新しい動作 
このプレビューでは、入れ子が有効な VM の動作方法に、いくつかの変更があります。
-   チェックポイントの作成と適用が、入れ子になった仮想化が有効な VM で動作するようになりました。
-   入れ子が有効な VM を保存および開始できるようになりました。
-   入れ子になった仮想化が有効な VM を、仮想化ベースのセキュリティ対応のホストで実行できるようになりました (デバイス ガード、資格情報のガードなど)。
-   既存の制限事項に関するエラー メッセージが改善されました。

###機能上の制限事項
-   入れ子になった仮想化は、HYPER-V 仮想マシン内で HYPER-V を実行するように設計されています。 サード パーティの仮想化アプリケーションはサポートされていないため、HYPER-V VM に失敗する可能性があります。
-   動的メモリは入れ子になった仮想化と互換性がありません。 VM 内部で HYPER-V を実行すると、VM は実行時にそのメモリを変更できません。 
-   ランタイム メモリのサイズ変更は入れ子になった仮想化と互換性がありません。 HYPER-V を内部で実行する場合、VM メモリのサイズは変更できません。 
-   入れ子になった仮想化は、Intel システムでのみサポートされます。

###既知の問題
ビルド 14361 には、第 2 世代の VM が次のエラーで起動に失敗する既知の問題があります。
```none
“Cannot modify property without enabling VirtualizationBasedSecurityOptOut”
```
この問題は、入れ子になった仮想化を無効にするか、仮想化ベースのセキュリティを無効にすることで一時的に修正できます。
```none
Set-VMSecurity -VMName <vmname> -VirtualizationBasedSecurityOptOut $true
```

###ご意見をお寄せください
いつものように、Windows フィードバック アプリからフィードバックをお送りください。 ご質問がある場合は、[GitHub](https://github.com/Microsoft/Virtualization-Documentation) ページのドキュメントでご報告ください。 



<!--HONumber=Jun16_HO4-->


