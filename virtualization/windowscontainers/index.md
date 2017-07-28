---
title: "Windows コンテナーに関するドキュメント"
description: "Windows コンテナーに関するドキュメント"
keywords: "Docker, コンテナー"
author: enderb-ms
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
ms.openlocfilehash: 9b630e372b575ef1721c0d0926ec2caec3e5ec34
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2017
---
# Windows コンテナーに関するドキュメント

Windows コンテナーは、1 つのシステムで複数の独立したアプリケーションを実行できるようにする、オペレーティング システム レベルの仮想化を実現します。 この機能では、2 種類のコンテナー ランタイムを使用します。それぞれ、アプリケーションの分離の度合いが異なります。 Windows Server コンテナーは、名前空間とプロセスの分離によって分離を実現します。 Hyper-V コンテナーは、軽量の仮想マシンで各コンテナーをカプセル化します。 このドキュメント セットでは、クイック スタート ガイド、展開ガイド、管理操作に関する詳しい技術情報を提供します。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:90%" cellpadding="25" cellspacing="5">
<tr>
<td ><center>![](media/try.png)</center></td>
<td>**クイック スタート**<br /><br />
Windows Server のクイック スタート<br /><br />
<ul>
<li>[手順 1 - 概念と用語](quick-start/index.md)<br /><br /></li>
<li>[手順 2 - Windows Server と最初のコンテナーを構成する](quick-start/quick-start-windows-server.md)<br /><br /></li>
<li>[手順 3 - コンテナー イメージを作成してプッシュする](quick-start/quick-start-images.md)<br /><br /></li>
</ul>
Windows 10 のクイック スタート<br /><br />
<ul>
<li>[手順 1 - 概念と用語](quick-start/index.md)<br /><br /></li>
<li>[手順 2 - Windows 10 と最初のコンテナーを構成する](quick-start/quick-start-windows-10.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td ><center>![](media/1.png)</center></td>
<td>**展開**<br /><br />
Windows Server 2016 と Nano Server で Windows コンテナーを展開する方法について確認できます。<br /><br />
<ul>
<li>[システム要件](deploy-containers/system-requirements.md)<br /><br /></li>
<li>[コンテナー ホストの展開 - Windows Server](deploy-containers/deploy-containers-on-server.md)<br /><br /></li>
<li>[コンテナー ホストの展開 - Nano Server](deploy-containers/deploy-containers-on-nano.md)<br /><br /></li>
<li>[ウイルス対策の最適化](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/explore.png)</center></td>
<td>**Windows 上の Docker**<br /><br />
Windows 上の Docker の管理について説明しています。<br /><br />
<ul>
<li>[Windows 上の Docker エンジン](manage-docker/configure-docker-daemon.md)<br /><br /></li>
<li>[Windows 上の Dockerfile](manage-docker/manage-windows-dockerfile.md)<br /><br /></li>
<li>[Dockerfile の最適化](manage-docker/optimize-windows-dockerfile.md)<br /><br /></li>
<li>[コンテナーのネットワーク](manage-containers/container-networking.md)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/video.png)</center></td>
<td>**視聴する**<br /><br />
Windows コンテナー チームのデモとインタビューに関心がありますか?<br /><br />
<ul>
<li>[コンテナー チャネル](https://channel9.msdn.com/Blogs/containers)</li>
</ul>
<br />
</td>
</tr>

<tr>
<td ><center>![](media/question.png)</center></td>
<td>**コミュニティ**<br /><br />
コミュニティでやり取りしたり、サンプルを試してみたりできるほか、その他のリソースを探すこともできます。<br /><br />
<ul>
<li>[コンテナー フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)<br /><br /></li>
<li>[コミュニティのビデオとブログ](communitylinks.md)<br /><br /></li>
<li>[コンテナー リソース](https://msdn.microsoft.com/virtualization/community/community_overview)<br /><br /></li>
</ul>
</td>
</tr>
</table>
