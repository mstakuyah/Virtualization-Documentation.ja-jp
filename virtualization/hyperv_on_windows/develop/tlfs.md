# ハイパーバイザーの仕様

## ハイパーバイザーのトップレベル機能仕様

Hyper-V ハイパーバイザー トップレベル機能仕様 (TLFS) では、他のオペレーティング システム コンポーネントに対する、外部から参照可能なハイパーバイザーの動作を説明します。 この仕様は、ゲスト オペレーティング システムの開発者にとって役立ちます。

> この仕様は、Microsoft Open Specification Promise の下で提供されます。 [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications) に関する詳細については、次をお読みください。

#### ダウンロード

 リリース| マニュアル名の正式名称
--- | ---
 Windows Server 2012 R2 (リビジョン B)| [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
 Windows Server 2012 R2| [Hypervisor Top Level Functional Specification v4.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
 Windows Server 2012| [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
 Windows Server 2008 R2| [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## Microsoft ハイパーバイザー インターフェイスを実装するための要件

Windows オペレーティング システムには、ゲスト仮想マシンで実行するための限られたハイパーバイザー インターフェイスのセット ("HV#1" インターフェイスとも呼ばれます) が必要です。 また、いくつかのオプション機能は Microsoft と互換性のあるハイパーバイザーによって実装できます。 これらのオプションにより、仮想マシンでの Windows の動作が変更されます。 "Microsoft ハイパーバイザー インターフェイスを実装するための要件" では、Microsoft と互換性のあるハイパーバイザーによって実装される必須機能およびオプション機能の両方について説明します。

#### ダウンロード

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)



<!--HONumber=Mar16_HO2-->
