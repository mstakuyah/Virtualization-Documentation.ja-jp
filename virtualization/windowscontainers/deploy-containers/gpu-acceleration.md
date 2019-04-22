---
title: Windows のコンテナー GPU アクセラレータ
description: GPU アクセラレータのレベルが Windows コンテナーに存在します。
keywords: docker、コンテナー、デバイス、ハードウェア
author: cwilhit
ms.openlocfilehash: 281241e07e4bc184e73c4e74a117b44253a775be
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380056"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows のコンテナー GPU アクセラレータ

多くのコンテナー化ワークロードの CPU 計算リソースは、十分なパフォーマンスを提供します。 ただし、特定のクラスのワークロードの Gpu (グラフィックス処理単位) によって提供される並列計算 power 時間を短縮できます操作桁、コストを停止、非常にスループットの向上します。

Gpu は、既に従来のレンダリングと機械学習トレーニングと自動推定シミュレーションから、多くの一般的なワークロードの一般的なツールです。 Windows コンテナー DirectX とその上に組み込まれているすべてのフレームワーク GPU アクセラレータをサポートします。

> [!IMPORTANT]
> この機能をサポートしている Docker のバージョンを必要とする`--device`Windows コンテナーのコマンド ライン オプション。 正式な Docker サポートがスケジュールされている次回リリースされる Docker EE エンジン 19.03 します。 それまでは、Docker の[上位のソース](https://master.dockerproject.org/)には、必要なビットが含まれています。

## <a name="requirements"></a>要件

この機能を利用するには、現在の環境は、次の要件を満たす必要があります。

- コンテナーのホストでは、Windows Server 2019 または Windows 10、1809 以降のバージョンが実行されている必要があります。
- コンテナーの基本イメージ必要があります[mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows)以降。 Windows Server Core と Nano サーバー コンテナーの画像は現在サポートされていません。
- コンテナー ホストは、Docker エンジン 19.03 以降を実行している必要があります。
- コンテナー ホスト GPU 実行表示ドライバー バージョン対応 2.5 以降が必要です。

ディスプレイ ドライバーの対応バージョンを確認するには、コンテナー ホストの DirectX 診断ツール (dxdiag.exe) を実行します。 ツールの「表示」] タブの [以下に示す「ドライバー」セクションで確認します。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>GPU アクセラレータとコンテナーを実行します。

GPU アクセラレータを使用して、コンテナーを起動するには、次のコマンドを実行します。

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (とその上に組み込まれているすべてのフレームワーク) GPU を高速化できる Api だけを今日です。 サード パーティ製フレームワークはサポートされません。

## <a name="hyper-v-isolated-windows-container-support"></a>Windows のハイパー V 分離コンテナーのサポート

ハイパー V 分離 Windows コンテナー内の作業負荷の GPU アクセラレータは現在サポートされていません。

## <a name="hyper-v-isolated-linux-container-support"></a>ハイパー V 分離 Linux コンテナーのサポート

ハイパー V 分離 Linux コンテナー内の作業負荷の GPU アクセラレータは現在サポートされていません。
