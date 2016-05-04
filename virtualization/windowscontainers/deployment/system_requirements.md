---
author: neilpeterson
---

# Windows コンテナーの要件

**この記事は暫定的な内容であり、変更される可能性があります。**

このガイドでは、Windows コンテナー ホストの要件を一覧で示します。

## 物理的システム上の Windows コンテナー

- Windows コンテナーの役割は、Windows Server 2016 TP4 (Full および Core) と Nano Server でのみ使用できます。
- Hyper-V コンテナーを実行するには、Hyper-V の役割をインストールする必要があります。

## 仮想システム上の Windows コンテナー

Windows コンテナー ホストが Hyper-V 仮想マシンから実行され、Hyper-V コンテナーもホストする場合、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- ホスト システム上の Windows Server 2016 Technical Preview 4 または Windows 10 ビルド 10565、および仮想マシン上の Windows Server Technical Preview 4 (Full、Core) または Nano Server。
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
- コンテナー ホスト VM には、少なくとも 2 つの仮想プロセッサも必要になります。


## サポートされる OS イメージ

Windows Server Technical Preview 4 は、Windows Server Core と Nano Server という 2 つのコンテナー OS イメージで提供されます。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>ホスト オペレーティング システム</center></th>
<th><center>Windows Server コンテナー</center></th>
<th><center>Hyper-V コンテナー</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 フル UI</center></td>
<td><center>Core OS イメージ</center></td>
<td><center>Nano OS イメージ</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Core OS イメージ</center></td>
<td><center> Nano OS イメージ</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Nano OS イメージ</center></td>
<td><center>Nano OS イメージ</center></td>
</tr>
</tbody>
</table>






<!--HONumber=Mar16_HO1-->


