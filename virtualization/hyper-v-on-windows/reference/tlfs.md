---
title: "ハイパーバイザーの仕様"
description: "ハイパーバイザーの仕様"
keywords: "windows 10、hyper-v"
author: theodthompson
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: a2a6727289b5c1ecea6ce863d78d4edbb3426b82
ms.sourcegitcommit: 04563fde5017e8d9e8b8ab2bbce4bf2bdf29b419
ms.translationtype: HT
ms.contentlocale: ja-JP
---
# <a name="hypervisor-specifications"></a>ハイパーバイザーの仕様

## <a name="hypervisor-top-level-functional-specification"></a>ハイパーバイザーのトップレベル機能仕様

Hyper-V ハイパーバイザー トップレベル機能仕様 (TLFS) では、他のオペレーティング システム コンポーネントに対する、外部から参照可能なハイパーバイザーの動作を説明します。 この仕様は、ゲスト オペレーティング システムの開発者にとって役立ちます。
  
> この仕様は、Microsoft Open Specification Promise の下で提供されます。  [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications) の詳細については、以下をご覧ください。  

#### <a name="download"></a>ダウンロード
リリース | マニュアル名の正式名称
--- | ---
Windows Server 2016 (リビジョン B) | [Hypervisor Top Level Functional Specification v5.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0b.pdf)
Windows Server 2012 R2 (リビジョン B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Microsoft ハイパーバイザー インターフェイスを実装するための要件

Windows オペレーティング システムには、ゲスト仮想マシンで実行するための限られたハイパーバイザー インターフェイスのセット ("HV#1" インターフェイスとも呼ばれます) が必要です。 また、いくつかのオプション機能は Microsoft と互換性のあるハイパーバイザーによって実装できます。 これらのオプションにより、仮想マシンでの Windows の動作が変更されます。 "Microsoft ハイパーバイザー インターフェイスを実装するための要件" では、Microsoft と互換性のあるハイパーバイザーによって実装される必須機能およびオプション機能の両方について説明します。

#### <a name="download"></a>ダウンロード

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)