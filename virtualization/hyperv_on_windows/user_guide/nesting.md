---
title: 入れ子になった仮想化
description: 入れ子になった仮想化
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# 入れ子になった仮想化

入れ子になった仮想化は、仮想化環境で Hyper-V ホストを実行する機能を提供します。 つまり、入れ子になった仮想化により、Hyper-V ホスト自体を仮想化することができます。 入れ子になった仮想化の使用例として、仮想化環境で Hyper-V ラボを実行したり、個別のハードウェアを必要とせずに仮想化サービスを提供したりする方法があります。またWindows コンテナーのテクノロジは、仮想化されたコンテナー ホストで Hyper-V コンテナーを実行する際、入れ子になった仮想化に依存します。 このドキュメントは、ソフトウェアとハードウェアの前提条件、および構成手順について詳しく説明し、トラブルシューティングの詳細を提供します。

> 入れ子になった仮想化はプレビュー リリースであるため、運用環境では使用しないでください。

## 必要条件

- ビルド 10565 またはそれ以降を実行する、Windows Insider ビルド (Windows Server 2016、Nano Server または Windows 10)。
- 両方のハイパーバイザー (親と子) が、同一の Windows のビルド (10565 またはそれ以降) を実行している必要があります。
- 最低 4 GB の RAM が使用可能である必要があります。
- Intel VT-x テクノロジを使用する Intel プロセッサ。

## 入れ子になった仮想化の構成

最初にホストと同じビルドを実行する仮想マシンを作成します。**仮想マシンの電源は入れないでください**。 詳細については、[仮想マシンの作成](../quick_start/walkthrough_create_vm.md)に関するページを参照してください。

仮想マシンが作成されたら、親ハイパーバイザーで次のコマンドを実行します。これにより、入れ子になった仮想化が仮想マシンで有効になります。

```none
Set-VMProcessor -VMName <virtual machine> -ExposeVirtualizationExtensions $true -Count 2
```

入れ子になった Hyper-V ホストを実行するときは、仮想マシンで動的メモリを無効にする必要があります。 これは、仮想マシンのプロパティで、または次の PowerShell スクリプトを使って構成できます。

```none
Set-VMMemory <virtual machine> -DynamicMemoryEnabled $false
```

入れ子になった仮想マシンで IP アドレスを受信するためには、MAC アドレスのスプーフィングが有効になっている必要があります。 この処理は、次の PowerShell コマンドを使って完了できます。

```none
Get-VMNetworkAdapter -VMName <virtual machine> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

上記の手順が完了すると、仮想マシンを起動し、Hyper-V をインストールすることができます。 Hyper-V のインストールの詳細については、[Hyper-V のインストール]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)に関するページを参照してください。

## ダウンロード

または、次のスクリプトを使用して、入れ子になった仮想化を有効にして構成することができます。 次のコマンドはスクリプトをダウンロードし、実行します。
  
```none
# download script
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Enable-NestedVm.ps1 -OutFile .\Enable-NestedVm.ps1 

# run script
.\Enable-NestedVm.ps1 -VmName "DemoVM"
```

## 既知の問題

- Device Guard が有効になっているホストは、仮想化拡張機能をゲストに公開できません。
- Virtualization Based Security (VBS) が有効になっているホストは、仮想化拡張機能をゲストに公開できません。 入れ子になった仮想化を使用するには、まず VBS を無効にする必要があります。
- 空白のパスワードを使用した場合、仮想マシン接続が失われる可能性があります。 パスワードを変更すると、問題が解決されます。
- 仮想マシンで入れ子になった仮想化を有効にすると、次の機能とその VM との互換性がなくなります。  
  * ランタイム メモリのサイズ変更。
  * 実行中の仮想マシンへのチェックポイントの適用。
  * 他の仮想マシンをホストしている仮想マシンのライブ マイグレーションができなくなる。
  * 保存と復元が機能しない。

## よく寄せられる質問とトラブルシューティング

仮想マシンが起動しませんが、どうすればよいですか。

1. 動的メモリがオフになっていることを確認してください。
2. 管理者特権でのプロンプトから、ホスト コンピューターで[この PowerShell スクリプト](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)を実行します。
  
## フィードバック

Windows フィードバック アプリ、[仮想化フォーラム](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv)、または [GitHub](https://github.com/Microsoft/Virtualization-Documentation) を使用してその他の問題を報告してください。



<!--HONumber=Jun16_HO2-->


