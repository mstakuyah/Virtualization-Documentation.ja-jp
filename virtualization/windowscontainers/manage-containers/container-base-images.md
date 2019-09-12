---
title: Windows コンテナーの基本イメージの履歴
description: SHA256 レイヤー ハッシュと共にリリースされた Windows コンテナー イメージの一覧
keywords: docker, コンテナー, ハッシュ
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107906"
---
# <a name="container-base-images"></a>コンテナーの基本イメージ

## <a name="supported-base-images"></a>サポートされている基本イメージ

Windows コンテナーは、Windows Server Core、Nano Server、Windows、IoT Core という4つのコンテナーベースのイメージで提供されます。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

|ホストオペレーティングシステム|Windows コンテナー|Hyper-V による分離|
|---------------------|-----------------|-----------------|
|Windows Server 2016 または Windows Server 2019 (標準またはデータセンター)|Server Core、Nano Server、Windows|Server Core、Nano Server、Windows|
|Nano Server|Nano Server|Server Core、Nano Server、Windows|
|Windows 10 Pro または Windows 10 Enterprise|使用できません|Server Core、Nano Server、Windows|
|IoT Core|IoT Core|使用できません|

> [!WARNING]  
> Windows Server バージョン1709以降では、Nano Server はコンテナーホストとしては利用できなくなりました。

## <a name="base-image-differences"></a>基本イメージの相違点

どのようにして適切な基本イメージを作成するか。 必要に応じてビルドを無料で行うことができます。各画像の一般的なガイドラインを次に示します。

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): アプリケーションで .net framework が完全に必要な場合は、この画像を使用するのが最適です。
- [Nano server](https://hub.docker.com/_/microsoft-windows-nanoserver): .net Core のみを必要とするアプリケーションの場合、nano server では、フルな画像が提供されます。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): アプリケーションは、サーバー Core または Nano server のイメージ (GDI ライブラリなど) に含まれていないコンポーネントまたは .dll に依存している可能性があります。 この画像は、すべての Windows の依存関係のセットを保持します。
- [Iot Core](https://hub.docker.com/_/microsoft-windows-iotcore): この画像は、 [iot アプリケーション](https://developer.microsoft.com/windows/iot)で使用することを目的としています。 IoT Core ホストをターゲットとする場合は、このコンテナーイメージを使用する必要があります。

ほとんどのユーザーは、Windows Server Core または Nano Server を使用するのに最適なイメージとなります。 Nano Server 上での構築について考えている場合は、次の点に注意してください。

- サービス スタックが削除されている
- .NET Core が含まれていない ([.NET Core の Nano Server イメージ](https://hub.docker.com/r/microsoft/dotnet/) は使用できます)
- PowerShell が削除されている
- WMI が削除されている
- Windows Server Version 1709 からは、アプリケーションがユーザー コンテキストで実行されるため、管理者特権を必要とするコマンドは失敗となります。 ユーザーフラグ (docker run--user ContainerAdministrator など) を使用して、コンテナー管理者アカウントを指定できますが、今後は NanoServer から管理者アカウントを完全に削除することを意図しています。

ここには特に重要な相違点が記載されていますが、すべてが網羅されているわけではありません。 上に記載されている以外にも、除外されているコンポーネントがあります。 Nano Server 上には、いつでもレイヤーを追加できることに注意してください。 この例については、[.NET Core の Nano Server の Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile) をご覧ください。
