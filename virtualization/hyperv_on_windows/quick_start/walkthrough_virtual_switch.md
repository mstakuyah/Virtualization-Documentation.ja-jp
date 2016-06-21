---
title: &262755780 仮想スイッチを作成する
description: 仮想スイッチを作成する
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1780170219 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 532195c6-564b-4954-97c2-5a5795368c09
---

# 仮想スイッチを作成する

Hyper-V で仮想マシンを作成する前に、この仮想マシンを物理ネットワークに接続する方法を指定することが必要になる場合があります。 Hyper-V にはソフトウェア ベースのネットワーク技術が搭載されており、これにより、仮想マシンのネットワーク カードを仮想スイッチに接続し、ネットワーク接続を確立することができます。 Hyper-V で作成した仮想スイッチはそれぞれ、次の 3 つの接続方法のうちのいずれかで構成できます。

- <g id="1" ctype="x-strong">外部ネットワーク</g> – 仮想スイッチは物理ネットワーク アダプターに接続されます。このアダプターにより物理ネットワークと、Hyper-V ホストと、仮想マシンとの間の接続が確立されます。 この構成では、物理的に接続されたネットワーク カードで通信するホスト機能の有効/無効を切り替えることもできます。 VM のトラフィックだけを特定の物理ネットワーク カードに分離する場合に便利です。

- <g id="1" ctype="x-strong">内部ネットワーク</g> – 仮想スイッチは物理ネットワーク アダプターに接続されません。 ただし、Hyper-V ホストと、このスイッチに接続されている任意の仮想マシンとの間にはネットワーク接続が存在します。

- <g id="1" ctype="x-strong">プライベート ネットワーク</g> – 仮想スイッチは物理ネットワーク アダプターに接続されません。Hyper-V ホストと、このスイッチに接続されている任意の仮想マシンとの間には接続が存在しません。

## 仮想スイッチを手動で作成する

この演習では、Hyper-V マネージャーを使用して外部仮想スイッチを作成する方法を順を追って説明します。 完了すると、Hyper-V ホストには、仮想マシンを物理ネットワークに接続するための仮想スイッチが追加されます。

1. Hyper-V マネージャーを開きます。

2. Hyper-V ホストの名前を右クリックし、<g id="2" ctype="x-strong">[仮想スイッチ マネージャー]</g> を選択します。

3. [仮想スイッチ] で、<g id="2" ctype="x-strong">[新しい仮想ネットワーク スイッチ]</g> を選択します。

4. [どの種類の仮想スイッチを作成しますか]で、<g id="2" ctype="x-strong">[外部]</g> を選択します。

5. <g id="2" ctype="x-strong">[仮想スイッチの作成]</g> ボタンを選択します。

6. [仮想スイッチのプロパティ] で、"<g id="2" ctype="x-strong">External VM Switch</g>" のような名前を新しいスイッチに付けます。

7. [接続の種類] で、<g id="2" ctype="x-strong">[外部ネットワーク]</g> が選択されていることを確認します。

8. 新しい仮想スイッチと組み合わせて使用する物理ネットワーク カードを選択します。 これは、ネットワークに物理的に接続されているネットワーク カードです。

    <g id="1" ctype="x-linkText"></g>

9. <g id="2" ctype="x-strong">[適用]</g> を選択し、仮想スイッチを作成します。 この時点で、通常は、次のようなメッセージが表示されます。 [<g id="2" ctype="x-strong">はい</g>] をクリックして続行します。

    <g id="1" ctype="x-linkText"></g>

10. <g id="2" ctype="x-strong">[OK]</g> を選択し、仮想スイッチ マネージャーのウィンドウを閉じます。

## PowerShell で仮想スイッチを作成する

次の手順は、PowerShell を使用して仮想スイッチと外部接続を作成する際に使用されます。

1. <g id="2" ctype="x-strong">Get-NetAdapter</g> を使用し、Windows 10 システムに接続されているネットワーク アダプターの一覧を取得します。

    ```powershell
    PS C:\> Get-NetAdapter

    Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
    ----                      --------------------                    ------- ------       ----------             ---------
    Ethernet 2                Broadcom NetXtreme 57xx Gigabit Cont...       5 Up           BC-30-5B-A8-C1-7F         1 Gbps
    Ethernet                  Intel(R) PRO/100 M Desktop Adapter            3 Up           00-0E-0C-A8-DC-31        10 Mbps  
    ```

2. Hyper-V スイッチで使用するネットワーク アダプターを選択し、<g id="2" ctype="x-strong">$net</g> という名前の変数にインスタンスを指定します。

    ```
    $net = Get-NetAdapter -Name 'Ethernet'
    ```

3. 次のコマンドを実行し、新しい Hyper-V 仮想スイッチを作成します。

    ```
    New-VMSwitch -Name "External VM Switch" -AllowManagementOS $True -NetAdapterName $net.Name
    ```

## 仮想スイッチとラップトップ

ノート PC で Windows 10 Hyper-V を実行する場合、イーサネットと無線ネットワーク カードの両方に対応する仮想スイッチを作成すると便利です。 この構成では、ネットワークへのノート PC の接続方法に応じて、これらのスイッチ間の仮想マシンを変更することができます。 仮想マシンが有線と無線を自動で切り替えることはありません。

## 次の手順 - 仮想マシンを作成する

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Windows 仮想マシンを作成する</g><g id="1CapsExtId3" ctype="x-title"></g></g>






<!--HONumber=May16_HO2-->


