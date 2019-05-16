---
title: Windows のコンテナーでデバイスを使う
description: Windows のコンテナーにどのようなデバイスのサポートが存在します。
keywords: docker、コンテナー、デバイス、ハードウェア
author: cwilhit
ms.openlocfilehash: feff730ed21c439312cda65c7b5ccc1a6cf5ae86
ms.sourcegitcommit: 2b456022ee666863ef53082580ac1d432de86939
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/16/2019
ms.locfileid: "9657360"
---
# <a name="devices-in-containers-on-windows"></a>Windows のコンテナーでデバイスを使う

既定では、Windows コンテナーはホスト デバイス Linux コンテナーと同じように、最低限のアクセスを与えられます。 メリット - または - にアクセスして、ホスト ハードウェア デバイスとの通信も必要である特定のワークロードがあります。 このガイドは、コンテナーではサポートされているデバイスと作業を開始する方法について説明します。

> [!IMPORTANT]
> この機能をサポートしている Docker のバージョンを必要とする`--device`Windows コンテナーのコマンド ライン オプション。 正式な Docker サポートがスケジュールされている次回リリースされる Docker EE エンジン 19.03 します。 それまでは、Docker の[上位のソース](https://master.dockerproject.org/)には、必要なビットが含まれています。

## <a name="requirements"></a>要件

この機能を利用するには、現在の環境は、次の要件を満たす必要があります。
- コンテナーのホストでは、Windows Server 2019 または Windows 10、1809 以降のバージョンが実行されている必要があります。
- コンテナー基本イメージ バージョンは、1809 以降である必要があります。
- コンテナーは、プロセス分離モードで実行されている Windows コンテナーである必要があります。
- コンテナー ホストは、Docker エンジン 19.03 以降を実行している必要があります。

## <a name="run-a-container-with-a-device"></a>デバイスを使って、コンテナーを実行します。

デバイスを使用してコンテナーを起動するには、次のコマンドを使用します。

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

置き換える必要があります、`{interface class guid}`適切な[デバイス インターフェイス クラス GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)に含まれている次のセクションでします。

複数のデバイスには、コンテナーを開始するには、次のコマンドを使用して、つなげて複数`--device`引数。

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows では、すべてのデバイスの宣言を実装するインターフェイス クラスの一覧です。 Docker には、このコマンドを渡す] を要求されたクラスの実装として識別されるすべてのデバイスをコンテナーに組み込ますることを確認します。

つまり**ホストからデバイスを割り当てることが**できます。 代わりに、ホストがユーザーと共有しますコンテナーします。 同様に、クラス GUID を指定するため、そのコンテナーとその GUID を実装する_すべて_のデバイスが共有されます。

## <a name="what-devices-are-supported"></a>どのようなデバイスがサポートされています。

次のデバイス (およびクラス Guid インターフェイスがデバイス) はサポートされています。
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>デバイスの種類</center></th>
<th><center>インターフェイス クラス GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C バス</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM ポート</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>[SPI バス</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU アクセラレータ</center></td>
<td><center><a href="https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU アクセラレータ</a>ドキュメントを参照してください。</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> 上記のデバイスでは、Windows コンテナーで現在サポートされている_のみ_デバイス。 その他のすべてのクラス Guid を渡すしようとすると、開始に失敗しますコンテナーが発生します。

## <a name="hyper-v-isolated-windows-container-support"></a>Windows のハイパー V 分離コンテナーのサポート

デバイスの割り当てとデバイス ハイパー V 分離 Windows コンテナー内の作業負荷の共有が現在サポートされていません。

## <a name="hyper-v-isolated-linux-container-support"></a>ハイパー V 分離 Linux コンテナーのサポート

デバイスの割り当てとの共有ハイパー V 分離 Linux コンテナー内の作業負荷をデバイスが現在サポートされていません。
