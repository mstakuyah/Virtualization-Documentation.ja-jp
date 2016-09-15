---
title: "コンテナーのリソース管理"
description: "Windows コンテナーでコンテナー リソースを管理します。"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
redirect_url: https://docs.docker.com/engine/reference/run/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 7544e91856d969b09241df7a466d776d818a2968

---

# コンテナーのリソース管理

**この記事は暫定的な内容であり、変更される可能性があります。** 

Windows コンテナーには、コンテナーが消費できる CPU 使用量、ディスク IO、ネットワーク リソース、メモリ リソースを管理する機能があります。 コンテナー リソース消費を制限することで、ホスト リソースを効率的に使用し、過度な消費を防ぐことができます。 このドキュメントでは、Docker を使用してコンテナー リソースを管理する方法について説明します。

## Docker を使用したリソース管理 

Docker でコンテナー リソースの一部を管理する機能があります。 具体的には、ユーザーは複数のコンテナーで CPU を共有する方法を指定できます。 

### CPU

複数のコンテナー間の CPU 共有は、ランタイムで --cpu-shares フラグを指定して管理できます。 既定では、すべてのコンテナーが同割合の CPU 時間を利用します。 コンテナーが使用する CPU の相対的な共有量を変更するには、--cpu-shares フラグを 1 から 10000 の値を指定して実行します。 既定では、すべてのコンテナーに 5000 の重みが与えられます。 CPU 共有の制約の詳細については、[Docker Run のリファレンス]( https://docs.docker.com/engine/reference/run/#cpu-share-constraint)を参照してください。 

```none 
docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## 既知の問題

- 現在、CPU と IO リソース制御は、Hyper-V コンテナーでサポートされていません。
- 現在、IO リソース制御は、コンテナー データ ボリュームでサポートされていません。


<!--HONumber=Sep16_HO2-->


