---
title: "Windows コンテナーの要件"
description: "Windows コンテナーの要件"
keywords: metadata, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: cc216f56acd5e547d05a48beea57450ba5fcb28b
ms.openlocfilehash: 12ae565f012dc87a2cab883c0486322db42b1dcc

---

# Windows コンテナーの要件

**この記事は暫定的な内容であり、変更される可能性があります。** 

このガイドでは、Windows コンテナー ホストの要件を一覧で示します。

## OS 要件

- Windows コンテナーの役割は、Windows Server 2016 TP5 (Full および Core)、Nano Server、Windows 10 (insider ビルド 14352 以降) でのみ使用できます。
- Hyper-V コンテナーを実行するには、Hyper-V の役割をインストールする必要があります。
- Windows Server コンテナー ホストでは、Windows を c: \\ にインストールする必要があります。 Hyper-V コンテナーのみを展開する場合、この制限は適用されません。

## 仮想化されたコンテナー ホスト

Windows コンテナー ホストが Hyper-V 仮想マシンから実行され、Hyper-V コンテナーもホストする場合、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- ホスト システム上の Windows Server 2016 Technical Preview 5 または Windows 10 ビルド 10565、および仮想マシン上の Windows Server Technical Preview 5 (Full、Core) または Nano Server。
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
- コンテナー ホスト VM には、少なくとも 2 つの仮想プロセッサも必要になります。

## サポートされる OS イメージ

Windows Server Technical Preview 5 は、Windows Server Core と Nano Server という 2 つのコンテナー OS イメージで提供されます。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

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
<td><center>Server Core イメージ</center></td>
<td><center>Nano Server イメージ</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core イメージ</center></td>
<td><center> Nano Server イメージ</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Nano Server イメージ</center></td>
<td><center>Nano Server イメージ</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Insider Releases</center></td>
<td><center>利用不可</center></td>
<td><center>Nano Server イメージ</center></td>
</tr>
</tbody>
</table>



<!--HONumber=Jun16_HO4-->


