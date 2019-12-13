---
title: Windows コンテナーでの GPU アクセラレーション
description: Windows コンテナーに存在する GPU アクセラレーションのレベル
keywords: docker、コンテナー、デバイス、ハードウェア
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909912"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows コンテナーでの GPU アクセラレーション

多くのコンテナー化されたワークロードでは、CPU コンピューティングリソースが十分なパフォーマンスを提供します。 ただし、特定のクラスのワークロードでは、Gpu (グラフィックスプロセッシングユニット) によって提供される超並列のコンピューティング能力によって、処理速度が大幅に向上し、コストが削減され、スループット非常が向上します。

Gpu は、従来のレンダリングやシミュレーションから機械学習のトレーニングや推論まで、多くの一般的なワークロードに対して既に共通のツールです。 Windows コンテナーでは、DirectX の GPU アクセラレーションと、その上に構築されたすべてのフレームワークがサポートされています。

> [!NOTE]
> この機能は、Docker Desktop、バージョン2.1、Docker エンジン-Enterprise、バージョン19.03 以降で使用できます。

## <a name="requirements"></a>要件

この機能を機能させるには、環境が次の要件を満たしている必要があります。

- コンテナーホストでは、Windows Server 2019 または Windows 10、バージョン1809以降が実行されている必要があります。
- コンテナーの基本イメージは、 [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows)以降である必要があります。 Windows Server Core と Nano Server のコンテナーイメージは、現在サポートされていません。
- コンテナーホストは、Docker エンジン19.03 以降を実行している必要があります。
- コンテナーホストには、ディスプレイドライバーバージョン WDDM 2.5 以降を実行している GPU が必要です。

ディスプレイドライバーの WDDM バージョンを確認するには、コンテナーホストで DirectX 診断ツール (dxdiag) を実行します。 ツールの [表示] タブで、次に示すように、"ドライバー" セクションを確認します。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>GPU アクセラレータを使用してコンテナーを実行する

GPU アクセラレータを使用してコンテナーを開始するには、次のコマンドを実行します。

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (およびその上に構築されたすべてのフレームワーク) は、現在 GPU で高速化できる唯一の Api です。 サードパーティのフレームワークはサポートされていません。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v 分離 Windows コンテナーのサポート

Hyper-v 分離 Windows コンテナー内のワークロードの GPU アクセラレーションは現在サポートされていません。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v 分離された Linux コンテナーのサポート

Hyper-v 分離された Linux コンテナー内のワークロードの GPU アクセラレーションは、現在サポートされていません。

## <a name="more-information"></a>詳細

GPU アクセラレーションを活用するコンテナー化された DirectX アプリの完全な例については、「 [directx container sample](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)」を参照してください。
