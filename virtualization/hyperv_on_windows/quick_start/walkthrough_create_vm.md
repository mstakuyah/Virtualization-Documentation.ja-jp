---
title: &494640063 Windows 仮想マシンを Windows 10 の Hyper-V に展開する
description: Windows 仮想マシンを Windows 10 の Hyper-V に展開する
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &239114363 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 66723f33-b12c-49d1-82cf-71ba9d6087e9
---

# Windows 仮想マシンを Windows 10 の Hyper-V に展開する

さまざまな方法で仮想マシンを作成し、オペレーティング システムをその仮想マシンに展開することができます。たとえば、Windows 展開サービスを使用したり、準備した仮想ハード ドライブを接続したり、手動でインストール メディアを使用したりします。 この記事では、オペレーティング システムのインストール メディアを使用して、仮想マシンを作成し、オペレーティング システムをその仮想マシンに展開する方法について説明します。

この演習を始める前に、展開するオペレーティング システムの .iso ファイルが必要です。 必要な場合は、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">TechNet Evaluation Center</g><g id="2CapsExtId3" ctype="x-title"></g></g> から Windows 8.1 または Windows 10 の評価版を取得します。

## Hyper-V マネージャーでの仮想マシンの作成

次のステップは、手動で仮想マシンを作成して、この仮想マシンにオペレーティング システムを展開する方法を説明しています。

1. Hyper-V マネージャーで、<g id="2" ctype="x-strong">[アクション]</g>、<g id="4" ctype="x-strong">[新規]</g>、<g id="6" ctype="x-strong">[仮想マシン]</g> の順にクリックし、新しい仮想マシン ウィザードを表示します。

2. [始める前に] のコンテンツを確認して、<g id="2" ctype="x-strong">[次へ]</g> をクリックします。

3. 仮想マシンに名前を付けます。
> <g id="1" ctype="x-strong">注:</g> この名前は、Hyper-V が仮想マシンに使用する名前であり、仮想マシン内に展開されるゲスト オペレーティング システムに指定されるコンピューター名ではありません。

4. 仮想マシン ファイルが格納される場所を選択します (<g id="2" ctype="x-strong">c:\virtualmachine</g> など)。 既定の場所を受け入れることもできます。 完了したら、<g id="2" ctype="x-strong">[次へ]</g> をクリックします。

  <g id="1" ctype="x-linkText"></g>

5. マシンの世代を選択し、<g id="2" ctype="x-strong">[次へ]</g> をクリックします。

  第 2 世代仮想マシンは、Windows Server 2012 R2 で導入されており、簡略化された仮想ハードウェア モデルといくつかの追加機能を提供します。 第 2 世代の仮想マシンには、64 ビット オペレーティング システムのみをインストールできます。 第 2 世代仮想マシンの詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">第 2 世代仮想マシンの概要</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。

> 新しい仮想マシンが第 2 世代として構成されている場合、Linux ディストリビューションを実行するには、セキュア ブートを無効にする必要があります。 セキュア ブートの詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">セキュア ブート</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。

6. <g id="4" ctype="x-strong">[起動メモリ]</g> の値として <g id="2" ctype="x-strong">[2048]</g> MB を選択し、<g id="6" ctype="x-strong">[動的メモリを使用します]</g> が選択されたままにします。 <g id="2" ctype="x-strong">[次へ]</g> をクリックします。

  メモリは、Hyper-V ホストとホスト上で実行されている仮想マシンの間で共有されます。 1 つのホスト上で実行できる仮想マシンの数は、使用可能なメモリにも依存します。 また、仮想マシンは動的メモリを使用するように、構成することもできます。 有効にすると、動的メモリは実行中の仮想マシンから未使用のメモリを解放します。 解放すると、より多くの仮想マシンをホスト上で実行できます。 動的メモリの詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Hyper-V 動的メモリの概要</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。

7. ネットワークの構成ウィザードで、仮想マシン用の仮想スイッチを選択し、<g id="2" ctype="x-strong">[次へ]</g> をクリックします。 詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">仮想スイッチを作成する</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。

8. 仮想ハード ドライブに名前を付け、場所を選択するか既定のままにして、最後にサイズを指定します。 準備ができたら、<g id="2" ctype="x-strong">[次へ]</g> をクリックします。

  仮想ハード ドライブは、物理ハード ドライブと同様に仮想マシンの記憶域を提供します。 仮想マシンにオペレーティング システムをインストールできるようにするには、仮想ハード ドライブが必要です。

  <g id="1" ctype="x-linkText"></g>

9. インストール オプション ウィザードで、<g id="2" ctype="x-strong">[ブート イメージ ファイルからオペレーティング システムをインストールする]</g> を選択して、オペレーティング システムの .iso ファイルを選択します。 完了したら、<g id="2" ctype="x-strong">[次へ]</g> をクリックします。

  仮想マシンの作成中に、いくつかのオペレーティング システムのインストール オプションを構成できます。 次の 3 つのオプションを使用できます。

  - <g id="1" ctype="x-strong">後でオペレーティング システムをインストールする</g> – このオプションは、仮想マシンに追加の構成変更を行いません。

  - <g id="1" ctype="x-strong">ブート イメージ ファイルからオペレーティング システムをインストールする</g> – これは物理コンピューターの物理的な CD-ROM ドライブに CD を挿入するのと似ています。 このオプションを構成するには、.iso イメージを選択します。 このイメージは、仮想マシンの仮想 CD-ROM ドライブにマウントされます。 仮想マシンのブート順は、CD-ROM ドライブから最初にブートされるように変更されます。

  - <g id="1" ctype="x-strong">ネットワーク ベースのインストール サーバーからオペレーティング システムをインストールする</g> – このオプションは、ネットワーク スイッチに仮想マシンを接続しない限り使用できません。 この構成では、仮想マシンはネットワークからブートを試行します。

10. 仮想マシンの詳細を確認し、<g id="2" ctype="x-strong">[完了]</g> をクリックして仮想マシンの作成を完了します。

## PowerShell での仮想スイッチの作成

1. 管理者として PowerShell ISE を開きます。

2. 以下のコマンドを実行します。

  ```powershell
  # Set VM Name, Switch Name, and Installation Media Path.
  $VMName = 'TESTVM'
  $Switch = 'External VM Switch'
  $InstallMedia = 'C:\Users\Administrator\Desktop\en_windows_10_enterprise_x64_dvd_6851151.iso'

  # Create New Virtual Machine
  New-VM -Name $VMName -MemoryStartupBytes 2147483648 -Generation 2 -NewVHDPath "D:\Virtual Machines\$VMName\$VMName.vhdx" -NewVHDSizeBytes 53687091200 -Path "D:\Virtual Machines\$VMName" -SwitchName $Switch

  # Add DVD Drive to Virtual Machine
  Add-VMScsiController -VMName $VMName
  Add-VMDvdDrive -VMName $VMName -ControllerNumber 1 -ControllerLocation 0 -Path $InstallMedia

  # Mount Installation Media
  $DVDDrive = Get-VMDvdDrive -VMName $VMName

  # Configure Virtual Machine to Boot from DVD
  Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
  ```

## オペレーティング システムの展開を完了する

仮想マシンの構築を完了するには、仮想マシンを起動して、オペレーティング システムのインストールを行う必要があります。

1. Hyper-V マネージャーで、仮想マシンをダブルクリックします。 VMConnect ツールが起動されます。

2. VMConnect で、緑色の [スタート] ボタンをクリックします。 これは物理コンピューター上の電源ボタンを押すのと似ています。 "CD または DVD からブートするには、いずれかのキーを押してください" というメッセージが表示される場合があります。 そのように押してください。
> <g id="1" ctype="x-strong">注:</g> VMConnect ウィンドウ内をクリックして、キーストロークが仮想マシンに送信されていることを確認することをお勧めします。

3. 仮想マシンでセットアップが起動され、物理コンピューターと同じように、インストールを行うことができます。

  <g id="1" ctype="x-linkText"></g>

> <g id="1" ctype="x-strong">注:</g> ボリューム ライセンス版の Windows を実行していない限り、仮想マシン内で実行されている Windows のライセンスを区別する必要があります。 仮想マシンのオペレーティング システムは、ホストのオペレーティング システムに依存しません。

## 次の手順 - PowerShell と Hyper-V の使用

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Hyper-V と Windows PowerShell</g><g id="1CapsExtId3" ctype="x-title"></g></g>





<!--HONumber=May16_HO2-->


