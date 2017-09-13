---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/02/2017
---
# Windows コンテナー

## コンテナーとは

コンテナーとは、分離された独自のボックスにアプリケーションをラップする方法です。 コンテナー内のアプリケーションは、ボックスの外に存在するアプリケーションやプロセスを認識しません。 コンテナー内には、アプリケーションの正常な実行に必要なすべての要素も存在します。  ボックスを移動しても、アプリケーションの実行に必要な要素がすべて揃っているため、アプリケーションが常に正常に実行されます。

これは、すべてが揃ったキッチンのようなものです。 すべての電化製品や家具、鍋やフライパン、食器用洗剤、ハンドタオルがすべてパッケージ化されたキッチン。 これがコンテナーです。

<center style="margin: 25px">![](media/box1.png)</center>

このコンテナーは移動して、どのようなアパートの部屋 (ホスト) にでも組み込むことができ、移動先で同じキッチンとして機能します。 必要な電化製品がすべて揃っているため、電気と水を接続すれば、すぐに料理を始められます。

<center style="margin: 25px">![](media/apartment.png)</center>

これとほとんど同じ意味で、コンテナーはこのキッチンに似ています。 このキッチンは、異なる部屋に組み込まれることもあれば、多くの同じ部屋に組み込まれることもあります。 重要なのは、コンテナーには必要なすべての要素がパッケージ化されて付属している点です。

簡単な概要を見る: [Windows ベースのコンテナー: エンタープライズ レベルの制御を使用した最新のアプリ開発](https://youtu.be/Ryx3o0rD5lY)。

## コンテナーの基礎

コンテナーは、分離されリソース制御された、移植可能なランタイム環境であり、ホスト マシンまたは仮想マシン上で実行されます。 コンテナー内で実行されるアプリケーションまたはプロセスは、必要なすべての必要な依存関係および構成ファイルと共にパッケージ化されており、コンテナーの外部で他のプロセスが一切実行されていないかのように動作します。

コンテナーのホストがコンテナーのリソースのセットをプロビジョニングし、コンテナーはこれらのリソースのみを使用します。 コンテナーの認識では、与えられた以外のリソースは存在しないため、近隣のコンテナー用にプロビジョニングされたリソースがあっても使用できません。

Windows コンテナーの作成と使用を開始する際には、次の主要な概念が役立ちます。

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. コンテナー ホストは、1 つ以上の Windows コンテナーを実行します。

**コンテナー イメージ:** ソフトウェアのインストールなどによってコンテナー ファイル システムまたはレジストリに変更が加えられると、それらの変更がサンドボックスでキャプチャされます。 これらの変更を継承する新しいコンテナーを作成できるように、この状態をキャプチャする場合がよくあります。 That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. このイメージがオペレーティング システム環境を提供します。 コンテナー OS イメージは不変です。 つまり、これを変更することはできません。

**コンテナー リポジトリ:** コンテナー イメージが作成されるたびに、コンテナー イメージとその依存関係がローカル リポジトリに格納されます。 これらのイメージは、コンテナー ホストで何度も再利用できます。 コンテナー イメージは、多くの異なるコンテナー ホスト間で使用できるように、DockerHub などのパブリック レジストリまたはプライベート レジストリにも格納できます。

<center>![](media/containerfund.png)</center>

仮想マシンに詳しいユーザーにとって、コンテナーは仮想マシンとほとんど同じように見えます。 A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. ただし、コンテナーの背後にあるテクノロジと概念は、仮想マシンのものとは大きく異なります。

それらの違いについては、Microsoft Azure の専門家である Mark Russinovich が[優れたブログ記事](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/)を公開しています。

## Windows コンテナーの種類

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. これらのコンテナーは、敵対的なセキュリティ境界を提供しないため、信頼されていないコードを分離するために使用しないでください。 これらのコンテナーはカーネルを共有しているため、カーネルのバージョンと構成を同一にする必要があります。

**Hyper-V による分離**: 高度に最適化した仮想マシン上で各コンテナーが実行されるため、Windows Server コンテナーの分離性が向上します。 In this configuration, the kernel of the container host is not shared with other containers on the same host. これらのコンテナーは、仮想マシンと同じセキュリティが保証されているため、敵対的なマルチテナント ホスティングに適しています。 これらのコンテナーは、ホストやホスト上の他のコンテナーとカーネルを共有しないため、異なるバージョンや構成 (サポートされている範囲内で) のカーネルを実行できます。たとえば、Windows 10 上のすべての Windows コンテナーで Hyper-V による分離を使用して、Windows Server のカーネル バージョンと構成を使用することができます。

Windows でコンテナーの実行に Hyper-V の分離を使用するかどうかは、実行時に決定されます。 最初に Hyper-V の分離を使用してコンテナーを作成し、後から実行時に、Windows Server コンテナーとして実行するように変更することもできます。

## Docker とは

コンテナーについての資料で、必ず言及されるのが Docker です。 Docker は、コンテナーのイメージをパッケージ化して配信するための容器です。 この自動プロセスによってイメージ (実際にはテンプレート) が生成され、生成されたイメージはどこでも (オンプレミス、クラウド内、または個人のコンピューター上のいずれでも) コンテナーとして実行できます。

<center>![](media/docker.png)</center>

Windows Server コンテナーは、他のコンテナーと同様、[Docker](https://www.docker.com) によって管理できます。

## 開発者向けのコンテナー ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## Windows Server コンテナーを使ってみる

コンテナーの優れた機能を試してみるには、 以下のリンクをクリックして、最初のコンテナーを実際に展開してみましょう。 <br/>
Windows Server をお使いの場合は、[Windows Server クイック スタートの概要](../quick-start/quick-start-windows-server.md)をご覧ください。 <br/>
Windows 10 をお使いの場合は、[Windows 10 クイック スタートの概要](../quick-start/quick-start-windows-10.md)をご覧ください。

