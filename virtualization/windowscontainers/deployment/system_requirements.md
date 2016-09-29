---
title: "Windows コンテナーの要件"
description: "Windows コンテナーの要件"
keywords: "メタデータ、コンテナー"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: a17a0e4eca2596c8cebbef93ea91c854f09d0b76

---

# Windows コンテナーの要件

このガイドでは、Windows コンテナー ホストの要件を一覧で示します。

## OS 要件

- Windows コンテナーの機能は、Windows Server 2016 (Core、デスクトップ エクスペリエンス搭載)、Nano Server、Windows 10 Professional および Enterprise (Anniversary Edition) でのみ使用できます。
- Hyper-V コンテナーを実行するには、Hyper-V の役割をインストールする必要があります。
- Windows は Windows Server コンテナー ホストの C:\. にインストールされている必要があります。Hyper-V コンテナーのみを展開する場合、この制限は適用されません。

## 仮想化されたコンテナー ホスト

Windows コンテナー ホストが Hyper-V 仮想マシンから実行され、Hyper-V コンテナーもホストする場合、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- Windows Server 2016、またはホスト システム上の Windows 10、および Windows Server (Full、Core) または仮想マシンの Nano Server
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
- コンテナー ホスト VM には、少なくとも 2 つの仮想プロセッサも必要になります。

## サポートされる基本イメージ

Windows コンテナーには、Windows Server Core と Nano Server という 2 つのコンテナー イメージが付属しています。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

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
<td><center>Windows Server 2016 とデスクトップ</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>利用不可</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
</tbody>
</table>



<!--HONumber=Sep16_HO4-->


