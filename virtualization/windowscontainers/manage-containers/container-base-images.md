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
ms.openlocfilehash: 0ec6eccbcf69532d583c32136f1a0c50c9811a8b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998369"
---
# <a name="windows-container-base-image-history"></a>Windows コンテナーの基本イメージの履歴

すべての Windows コンテナーは、Microsoft から提供されている基本 OS 上に構築されます。 コンテナーが構築されている Windows のバージョンがわからない場合は、`docker inspect <tag>` を実行し、1 行目または 2 行を以下の表と照合してください。

たとえば、`docker inspect microsoft/windowsservercore:10.0.14393.447` を実行すると次のように示されます。

```
...
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67",
        "sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15"
    ]
}
```

これは、Microsoft によって提供されたイメージ内に 2 つのレイヤーがあることを表しています。 最初のレイヤーは一定であり、最初の Windows Server リリースを表しています。2 つ目のレイヤーは、含まれている最新の累積的な更新プログラムに基づいて変化します。

各バージョンの変更内容を確認する場合は、「[Windows 10 および Windows Server 2016 の更新履歴](https://support.microsoft.com/help/12387/windows-10-update-history)」で該当するバージョンのサポート技術情報をご覧ください。


## <a name="tools-to-simplify-this-process"></a>このプロセスを簡略化するためのツール

コンテナー全体をダウンロードせずにイメージ マニフェストを読み取ってバージョンを判定できるツールが、Stefan Scherer 氏によって作成されています。 詳しくは、同氏の[ブログ](https://stefanscherer.github.io/winspector/)と [GitHub](https://github.com/StefanScherer/winspector) リポジトリをご覧ください。


## <a name="image-versions"></a>イメージのバージョン

<table>
    <tr>
        <th>Windows のバージョン</th>
        <th>microsoft/windowsservercore</th>
        <th>microsoft/nanoserver</th>
    </tr>
    <tr>
        <td>10.0.14393.206</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063</td>
    </tr>
    <tr>
        <td>10.0.14393.321</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:cc6b0a07c696c3679af48ab4968de1b42d35e568f3d1d72df21f0acb52592e0b</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:2c195a33d84d936c7b8542a8d9890a2a550e7558e6ac73131b130e5730b9a3a5</td>
    </tr>
    <tr>
        <td>10.0.14393.447</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:c8606bedb07a714a6724b8f88ce85b71eaf5a1c80b4c226e069aa3ccbbe69154</td>
    </tr>
    <tr>
        <td>10.0.14393.576</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:de57d9086f9a337bb084b78f5f37be4c8f1796f56a1cd3ec8d8d1c9c77eb693c</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:0d812bf7a7032db75770c3d5b92c0ac9390ca4a9efa0d90ba2f55ccb16515381</td>
    </tr>
    <tr>
        <td>10.0.14393.693</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:c28d44287ce521eac86e0296c7677f5d8ca1e86d1e45e7618ec900da08c95df3</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:dd33c5d8d8b3c230886132c328a7801547f13de1dac9a629e2739164a285b3ab</td>
    </tr>
</table>

