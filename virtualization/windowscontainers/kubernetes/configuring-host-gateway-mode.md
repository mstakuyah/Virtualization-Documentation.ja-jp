# <a name="host-gateway-mode"></a>ホスト ゲートウェイ モード #
Kubernetes ネットワー キングの使用可能なオプションの 1 つが*ホスト ゲートウェイ モード*です。このモードでは、すべてのノードのポッド サブネット間で静的なルートが構成されます。


## <a name="configuring-static-routes--linux"></a>静的なルートの構成 | Linux ##
これには、`iptables` を使用します。 すべてのポッドで使う簡略サブネットを使用して、`$CLUSTER_PREFIX` 変数の置き換えまたは設定を行います。

```bash
$CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

これにより、ポッドの基本的な NATing のセットアップのみが行われます。 ここで、ポッドに向かうすべてのトラフィックは、プライマリ インターフェイスを経由させる必要があります。 ここでも、必要に応じて `$CLUSTER_PREFIX` 変数と `eth0` (該当する場合) を置き換えます。

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

最後に、**ノード単位**ベースでネクストホップ ゲートウェイを追加する必要があります。 たとえば、最初のノードが `192.168.1.0/16` 上の Windows ノードの場合、次のようになります。

```bash
sudo route add -net $CLUSTER.1.0 netmask 255.255.255.0 gw $CLUSTER.1.2 dev eth0
```

クラスター内のすべてのノードで、クラスター内のすべてのノード用に、同様のルートを追加する必要があります****。


<a name="explanation-2-suffix"></a>
> [!Important]  
> Windows ノードの場合**のみ**、ゲートウェイはサブネットの `.2` です。 Linux の場合は、常に `.1` である可能性が高くなります。 この変則性が存在する理由は、ホスト ネットワークと仮想ポッド ネットワークの間のブリッジとなるネットワーク アダプターのゲートウェイとして、`.1` アドレスが予約されているためです。


## <a name="configuring-static-routes--windows"></a>静的なルートの構成 | Windows ##
これには、`New-NetRoute` を使用します。 [このリポジトリ](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1)に、自動化されたスクリプト `AddRoutes.ps1` があります。 *Linux マスター*の IP アドレスと、Windows ノードの*外部*アダプターの既定ゲートウェイ (ポッド ゲートウェイではなく) の IP アドレスが必要です。 次のようになります。


```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
wget $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3 -Gateway 10.1.3.1
```
