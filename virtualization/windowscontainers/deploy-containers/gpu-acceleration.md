---
title: Windows コンテナーでの GPU アクセラレータ
description: Windows コンテナーに存在する GPU アクセラレータのレベル
keywords: docker、コンテナー、デバイス、ハードウェア
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: da762ce138467e50dce22d5086ad407138b38e48
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "10261861"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows コンテナーでの GPU アクセラレータ

多くのコンテナーワークロードについては、CPU 計算リソースが十分なパフォーマンスを提供します。 ただし、特定のワークロードのクラスについては、Gpu (グラフィックス処理ユニット) で提供される強力な並列計算機能を使用すると、負荷の高い速度で操作を高速化して、コストを削減し、スループットの immensely を向上させることができます。

Gpu は、従来のレンダリングやシミュレーションからマシン学習トレーニング、推定など、多くの一般的なワークロードで既に一般的なツールとなっています。 Windows コンテナーでは、DirectX およびその上に構築されたすべてのフレームワークの GPU アクセラレータをサポートしています。

> [!NOTE]
> この機能は、Docker デスクトップ、バージョン2.1、Docker エンジン (エンタープライズ、バージョン19.03 以降) で利用できます。

## <a name="requirements"></a>要件

この機能が機能するためには、環境が次の要件を満たしている必要があります。

- コンテナーホストでは、Windows Server 2019 または Windows 10 バージョン1809以降を実行している必要があります。
- コンテナーの基本イメージは、 [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows)以上である必要があります。 Windows Server Core および Nano Server コンテナーのイメージは現在サポートされていません。
- コンテナーホストは、Docker Engine 19.03 以降を実行している必要があります。
- コンテナーホストには、ディスプレイドライバーバージョンの WDDM 2.5 以降を実行している GPU が必要です。

ディスプレイドライバーの WDDM バージョンを確認するには、コンテナーホストで DirectX 診断ツール (dxdiag) を実行します。 ツールの「表示」タブで、次に示す「Drivers」セクションを確認します。

![診断](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>GPU アクセラレータでコンテナーを実行する

GPU アクセラレータでコンテナーを開始するには、次のコマンドを実行します。

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (およびその上に構築されたすべてのフレームワーク) は、現在 GPU との間で加速できる唯一の Api です。 サードパーティのフレームワークはサポートされていません。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-分離された Windows コンテナーのサポート

Hyper-v のワークロードの GPU アクセラレータ (分離された Windows コンテナー) は、現在サポートされていません。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-分離された Linux コンテナーのサポート

Hyper-v でのワークロードの GPU アクセラレータ (分離された Linux コンテナー) は現在サポートされていません。

## <a name="more-information"></a>詳細情報

GPU アクセラレーションを活用する、コンテナーとなる DirectX アプリの完全な例については、「 [DirectX コンテナーのサンプル](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)」をご覧ください。
