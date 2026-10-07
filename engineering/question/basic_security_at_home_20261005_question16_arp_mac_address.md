# 『おうちで学べるセキュリティの基本』2026-10-5 疑問16：ARPとMACアドレスによるLAN内の配送

## 疑問

> グローバルIPからローカルIPへデータを転送するのはルータの役目だと思うが、どのローカルIPに割り振るのか？識別方法は？MACアドレス？

## 回答

**MACアドレスは、特定した相手へLAN内で配送する段階で使います。** ただし、NAPTでグローバルIPを共有する複数PCのうち、返答先がどのPCかを特定する根拠は、**変換・接続の対応表**です。外部サーバーから元のPCのMACアドレスが渡されるわけではありません。

返答の宛先が `192.168.1.10` に戻った後、その宛先へ直接届けられるLAN上では、**ARP（Address Resolution Protocol）**により、そのIPv4アドレスに対応するMACアドレスを調べます。ARPの仕様も、ルーティングで次の配送先を決めた後、リンク上のアドレスを調べる流れを説明しています（[RFC 826：Packet Generation](https://www.rfc-editor.org/rfc/rfc826.html)）。

ここでは、IPv4をEthernetのLAN上で送る例を使います。NAPTの詳細は[NAT・NAPTの文書](./basic_security_at_home_20261005_question15_16_nat_napt.md)、経路選択は[IPルーティングの文書](./basic_security_at_home_20261005_question16_ip_routing.md)を参照してください。

## 1. IP、ポート、MACは、見る範囲が違う

| 情報 | 主な役割 | 返答がPC Aへ向かう例 |
| --- | --- | --- |
| IPアドレス | ネットワークをまたいで宛先へ向かうためのアドレス | 宛先IPは、NAPTで `192.168.1.10` に戻す |
| TCP/UDPのポート番号 | 端末上の通信の受け口を区別する情報 | 宛先TCPポートは、元の `53124` に戻す |
| MACアドレス | 同じリンク上でフレームを渡す相手のアドレス | LAN上でPC Aに届けるために、PC AのMACを使う |

**フレーム**は、Ethernetなどのリンクでデータを届けるための包みです。IPパケットを、そのリンクで使うフレームに包んで送ります。TCP/UDPのデータは、さらにIPパケットの中に入っています。

```text
Ethernetフレーム
┌─────────────────────────────────────────────┐
│ Ethernetヘッダー：送信元MAC、宛先MAC          │
│ ┌─────────────────────────────────────────┐ │
│ │ IPヘッダー：送信元IP、宛先IP              │ │
│ │ ┌─────────────────────────────────────┐ │ │
│ │ │ TCPヘッダー：送信元・宛先ポート       │ │ │
│ │ │ データ                              │ │ │
│ │ └─────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

これは主要な情報だけを示した図です。**宛先IPと宛先MACは別のヘッダーにあり、同じ機器を指すとは限りません。** 遠くのサーバー宛てなら、IPはサーバー、MACは次に渡すルーター、という組み合わせになります。ルーターは転送時、次のリンクに合うフレームへ包み直します（[RFC 1812、5.2.1.2節の手順10～11：リンク層のアドレスとフレーム](https://www.rfc-editor.org/rfc/rfc1812.html#section-5.2.1.2)）。

## 2. ARPで、IPからMACを調べる

NATの文書と同じく、PC AのIPv4アドレスを `192.168.1.10`、ルーターのLAN側を `192.168.1.1` とします。MACアドレスは説明用です。

| 機器 | IPv4アドレス | MACアドレス |
| --- | --- | --- |
| 家庭用ルーターのLAN側 | `192.168.1.1` | `02:00:00:00:01:01` |
| PC A | `192.168.1.10` | `02:00:00:00:00:0a` |
| PC B | `192.168.1.11` | `02:00:00:00:00:0b` |

ルーターは、例えば次の**ARPキャッシュ（ARP表）**を保持します。これは、最近得たIPとMACの対応関係です。

| LAN側のIPv4アドレス | 対応するMACアドレス |
| --- | --- |
| `192.168.1.10` | `02:00:00:00:00:0a` |
| `192.168.1.11` | `02:00:00:00:00:0b` |

使える記録があればそれを利用します。記録がなければ、次のように問い合わせます。

```mermaid
sequenceDiagram
    autonumber
    participant R as ルーターのLAN側
    participant A as PC A（192.168.1.10）
    participant B as PC B（192.168.1.11）
    Note over R: 192.168.1.10へ送りたい<br/>使えるARP記録がない
    R->>A: ARP Request：192.168.1.10を使う機器は？
    R->>B: 同じブロードキャストを受信
    Note over B: 自分のIPではないので応答しない
    A-->>R: ARP Reply：対応するMACは02:00:00:00:00:0a
    Note over R: IPとMACの対応関係をキャッシュ
    R->>A: PC AのMAC宛てに、IPパケットを包んだフレームを送信
```

最初の2本の矢印は、**一つのブロードキャストが同じLANの複数機器へ届くこと**を示しています。PC A・Bへ別々に問い合わせを作る意味ではありません。EthernetのARP Requestは通常ブロードキャストされ、対象のIPを使う機器がReplyを返します（[RFC 826：Packet Generation・Packet Reception](https://www.rfc-editor.org/rfc/rfc826.html)）。

この例ではARPへの返答を受けて送信します。ARP表に対応があれば、毎回問い合わせる必要はありません。

## 3. NAPTの対応表とARP表は別の情報

同じルーターが両方を持つため、混同しやすい部分です。

| 表 | 対応付けるもの | この例の1項目 | 分かること |
| --- | --- | --- | --- |
| NAPTの変換・接続記録 | 内側と外側のIP・ポートなど | TCP `192.168.1.10:53124 ↔ 203.0.113.10:61001` | この通信の返答をどのIP・ポートへ戻すか |
| ARP表 | 同じリンク上のIPv4アドレスとMAC | `192.168.1.10 → 02:00:00:00:00:0a` | 次に渡す相手のMACは何か |

根拠：[RFC 3022、3.1～3.2節：NAPTの対応関係](https://www.rfc-editor.org/rfc/rfc3022.html#section-3.1)、[RFC 826：ARPの対応表](https://www.rfc-editor.org/rfc/rfc826.html)。

**ARP表だけでは、外側IPの61001番への返答がPC Aの53124番に対応することは分かりません。** 逆に、NAPTの記録から宛先IPが分かっても、そのリンクへ送るにはMACなどのリンク層の情報が必要です。

PC Aから複数のTCP接続を作れば、NAPT側には複数の接続の記録ができます。一方、同じLAN上の同じPC Aに届けるための `IP → MAC` の対応は、複数の接続で共有できます。

## 4. PCのMACは、遠くのサーバーまで引き継がれない

行きの通信で、PC AからWebサーバーへ送る場合を見ます。

| 区間 | IPの送信元 → 宛先 | その区間のMACの送信元 → 宛先 |
| --- | --- | --- |
| PC A → 家庭用ルーター | `192.168.1.10 → 198.51.100.20` | PC AのMAC → 家庭用ルーターのLAN側MAC |
| 家庭用ルーター → ISP側の次のルーター | NAPT後の `203.0.113.10 → 198.51.100.20` | このWANもEthernetなら、家庭用ルーターのWAN側MAC → 次のルーターのMAC |

このように、**ルーターをまたぐと、Ethernetのフレームは次のリンクに合わせて作られます。** PC AのMACを、遠くのサーバーまでの転送先の識別に使い続けるわけではありません。なお、インターネット上の全リンクがEthernetとは限らず、MACを使わない接続方式もあります（[RFC 1812、3.2節・5.2.1.2節：リンクによる違いと包み直し](https://www.rfc-editor.org/rfc/rfc1812.html#section-3.2)）。

したがって、PC Aは外部サーバーのMACをARPで調べません。外部サーバーへの通信では、同じLAN上にある**次ホップのルーター**のMACを調べます。ARPの問い合わせが、ルーターを越えてインターネット中に届くこともありません。

返答の最後の区間では、ルーターがLAN側のフレームを作ります。

```text
ルーター → PC A

Ethernetの送信元MAC：ルーターのLAN側MAC
Ethernetの宛先MAC  ：PC AのMAC

IPの送信元        ：外部サーバーのIP 198.51.100.20
IPの宛先          ：NAPTで戻したPC AのIP 192.168.1.10
TCPの宛先ポート   ：NAPTで戻した53124
```

**IPの送信元は外部サーバーでも、直前のLAN区間での送信元MACはルーターです。** ここが、IPとMACの役割の違いをつかむポイントです。

## 5. 手元でARP表を確認する

次のコマンドは、保存された対応関係を表示します。

| OS | 表示コマンド |
| --- | --- |
| macOS | `arp -an` |
| Windows | `arp -a` |
| Linux（iproute2） | `ip -4 neigh show` |

macOSのオプションは、この環境の `man 8 arp` で確認しています。Windowsは[Microsoft公式：arp](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/arp)、Linuxは[ip-neighbourのマニュアル](https://man7.org/linux/man-pages/man8/ip-neighbour.8.html)を参照してください。

表示例は次のようなものです。形式やインターフェース名はOS・端末で異なります。

```text
192.168.1.1  →  02:00:00:00:01:01
```

PCで実行した場合に見えるのは、**そのPC自身が持つ表**です。家庭用ルーターのNAPT対応表を表示するコマンドではありません。また、最近やり取りしていない相手は表にないこともあるため、**LAN内の全機器一覧とは限りません**。

IPv6ではARPを使わず、**Neighbor Discovery（近隣探索、ND）**が同じリンク上のリンク層アドレスを調べる役割などを担います（[RFC 4861、概要・7.2節：IPv6のアドレス解決](https://www.rfc-editor.org/rfc/rfc4861.html#section-7.2)）。

## 追加質問：MACアドレスを知らないと、どこで送信が止まるのか

> なぜ、MACアドレスまで必要なのかがわからないです。
> MACアドレスを知らなかった場合を考えると良いかもしれません。

この質問の背景となったWAN側の構成とARPのシーケンスは、[NATの文書の追加質問7](./basic_security_at_home_20261005_question15_16_nat_napt.md)に載せています。ここでは、Ethernetの配送と、MACが分からない場合の動作に絞って説明します。

### 1. 宛先IPが分かっていても、Ethernetの宛先欄は別に必要

**Ethernetで通常の1対1の通信をするには、受け取り先のMACを指定します。** IPパケットはEthernetフレームの中に入るので、中の宛先IPだけでは、外側の宛先MACの代わりになりません（[RFC 894：Frame Format／Address Mappings](https://www.rfc-editor.org/rfc/rfc894.html)）。

NATの文書と同じく、ISP側から、宛先IP `203.0.113.10` のサーバーの返答をNATルーターへ渡す場面を考えます。WAN側NICのMACは、説明用の記号 `M-WAN` とします。

```text
MACが分からない状態                    MACが分かった状態

Ethernetの宛先MAC：???                 Ethernetの宛先MAC：M-WAN
└─ IPパケット                         └─ IPパケット
   宛先IP：203.0.113.10                   宛先IP：203.0.113.10
   内容  ：サーバーの返答                 内容  ：サーバーの返答

通常の1対1のフレームを作れない          宛先MACを指定して送れる
```

`???` は「送信に使うMACをまだ得ていない」という説明用の記号です。宛先欄を空欄にしたフレームが実際に送られる、という意味ではありません。ARPが担当するのは、IPの宛先を変更することではなく、このMACを得る処理です。

### 2. 通常のスイッチはMACを使って転送する

複数機器がスイッチにつながる構成では、**通常のレイヤー2のスイッチは、宛先MACを見て、フレームを転送するポートを選びます。** IPパケットの中の宛先IPを読んで経路を選ぶ処理とは別です。スイッチの詳細な構成例は、NATの文書に示しています。

| 情報 | 持つ機器 | この場面での例 | 用途 |
| --- | --- | --- | --- |
| ARP表 | ISP側のルーター | `203.0.113.10 → M-WAN` | 送信するフレームの宛先MACを得る |
| MACアドレス表 | スイッチ | `M-WAN → NATルーターにつながるポート` | 届いたフレームの転送先ポートを選ぶ |

ARP表とMACアドレス表は、対応付けるものが違います（[RFC 826：IPとMACの対応](https://www.rfc-editor.org/rfc/rfc826.html)、[Cisco公式：スイッチのMAC学習と転送](https://www.cisco.com/c/en/us/support/docs/lan-switching/ethernet/12006-chapter22.html)）。

なお、**送信元が宛先MACを知らない場合**と、**スイッチがそのMACの接続ポートをまだ学習していない場合**も別です。後者では、宛先MACが記入されたフレームを、同じVLANの他のポートへ広く転送することがあります。これは「送信元が宛先MACを書かなくてもよい」という仕組みではありません（[Cisco公式：Unknown UnicastのFlooding](https://www.cisco.com/c/en/us/support/docs/lan-switching/ethernet/12006-chapter22.html)）。

スイッチを使わずEthernetで2台を直結しても、フレームの形式は同じなので、宛先MACを指定します。**複数のルーターがあるから初めてMACが必要になる、ということではありません。**

### 3. MACが不明でもARPで分かれば送れる。最後まで分からなければ届かない

次の図は、ISP側のルーターが返答を受け取ってからの処理です。使えるARP記録などのIPとMACの対応があるかを確認し、なければ短時間保留してARPで調べます（[RFC 826：Packet Generation](https://www.rfc-editor.org/rfc/rfc826.html)、[RFC 1812、3.3.2節：ARP中の保留と解決失敗](https://www.rfc-editor.org/rfc/rfc1812.html#section-3.3.2)）。

```mermaid
flowchart TD
    P["ISP側のルーターがサーバーの返答を受信<br/>宛先IP：203.0.113.10"] --> C{"使えるIPとMACの対応がある？"}
    C -->|"ある"| F["宛先MACをM-WANにして<br/>返答をEthernetフレームに包んで送信"]
    C -->|"ない"| Q["返答を短時間保留し、ARP Requestを送る<br/>宛先MAC：FF:FF:FF:FF:FF:FF"]
    Q --> D{"ARPでMACを得られた？"}
    D -->|"得られた"| M["203.0.113.10 → M-WANを記録"]
    M --> F
    D -->|"問い合わせても解決できない"| X["送信を進められず、返答を最終的に破棄<br/>NATルーターには届かない"]
    F --> R["NATルーターが返答を受信<br/>Basic NAT：.10 → 192.168.1.10"]
    R --> A["LAN側で配送し、PC Aへ届く"]
```

**MACが最後まで分からなければ、NATルーターは返答を受け取れません。** そのため、NATルーターに正しい「`.10 → PC A`」の対応があっても、それを使う段階まで進めません。ARP表に記録がないだけで直ちに失敗するのではなく、ARPによる解決を試みたうえで、解決できなければ送信に失敗します。

### 4. 相手のMACが不明なのに、ARP自体はなぜ送れるのか

最初のARP Requestには、**同じリンク内の機器へ広く届ける特別な宛先MAC `FF:FF:FF:FF:FF:FF`**を使います。この既知のブロードキャスト用アドレスを指定できるため、個別の相手のMACが不明でも問い合わせを送れます。

対象のIPを担当する機器が応答すると、相手のMACが分かり、その後の返答データにはそのMACを指定できます。**ARPの問い合わせにも宛先MACはありますが、その宛先は特定のNATルーターではなく、リンク内へのブロードキャストです**（[RFC 826：ブロードキャストするRequestと、問い合わせ元へ返すReply](https://www.rfc-editor.org/rfc/rfc826.html)、[Cisco公式：ブロードキャストMACの転送](https://www.cisco.com/c/en/us/support/docs/lan-switching/ethernet/12006-chapter22.html)）。

## 追加質問：スイッチはARP Requestをどこまで転送するのか

> スイッチからARPをPC A、Bのルータおよび別のルータに転送していますが、この転送はスイッチにつながっている全てのルータに転送するんですか？

**通常のARP Requestは、同じVLANの他の転送可能なポートへ転送されます。** 機器の種類で「ルーターかどうか」を選ぶ処理ではありません。ブロードキャスト宛てのフレームを、同じVLAN内へ広く送る処理です。受信したポートへは送り返しません（[Cisco公式：VLAN／Transparent Bridging Algorithm](https://www.cisco.com/c/en/us/support/docs/lan-switching/ethernet/12006-chapter22.html)）。

VLANは、スイッチ上でネットワークを論理的に分ける仕組みです。同じ物理スイッチに接続された機器でも、所属するVLANが異なれば、ARP Requestの配送範囲は分かれます。以下では、説明用にVLAN 10とVLAN 20を使います。特別なフィルタリングは設定せず、例の各ポートは通信できる状態とします。

```mermaid
flowchart TD
    U["ISP側のルーター<br/>問い合わせ元／VLAN 10"] -->|"ポート1から受信<br/>ARP Request：.10のMACは？<br/>宛先MAC FF:FF:FF:FF:FF:FF"| W["1台のスイッチ"]
    subgraph V10["問い合わせ元と同じVLAN 10"]
        R["PC A・BのNATルーターのWAN側<br/>.10を担当するのでProxy ARPで応答<br/>MAC：M-WAN"]
        O["別のルーターの接続インターフェース<br/>.10を担当しないので応答しない"]
        H["同じWAN側VLANに直接つながるPC<br/>.10を担当しないので応答しない"]
    end
    subgraph V20["別のVLAN 20"]
        X["もう1台のルーター<br/>このARP Requestは届かない"]
    end
    W -->|"ポート2：転送"| R
    W -->|"ポート3：転送"| O
    W -->|"ポート4：転送"| H
    W -.->|"ポート5：接続あり／ARPを転送しない"| X
```

実線はARP Requestの受信・転送を示します。点線は、同じスイッチに接続していても、このARP Requestの転送先にはならないことを示します。図の `.10` は `203.0.113.10` の略です。ポート番号は説明用で、TCP／UDPのポート番号ではなく、スイッチの接続口を指します。

| スイッチのポート | 接続先 | VLAN | 今回のARP Requestを転送するか |
| --- | --- | --- | --- |
| 1 | 問い合わせ元のISP側ルーター | 10 | **しない。**このポートから受信したため、送り返さない |
| 2 | PC A・BのNATルーターのWAN側 | 10 | **する。**この例では、受信したNATルーターがProxy ARPで応答する |
| 3 | 別のルーター | 10 | **する。**ただし、この例では `.10` を担当せず、応答しない |
| 4 | WAN側VLANに直接接続したPC | 10 | **する。**ルーター以外も転送先になる |
| 5 | もう1台のルーター | 20 | **しない。**問い合わせ元とVLANが異なる |

**「受信する範囲」と「応答する機器」を区別すると、図を読みやすくなります。** スイッチは、ARPの対象IP `.10` を担当するルーターだけを探して問い合わせを届けるのではなく、ブロードキャストを同じVLAN内へ転送します。その問い合わせを受信した機器のうち、対象IPを担当する機器が応答します。今回のNAT用IPは、NATルーターがProxy ARPで代理応答する例です（[RFC 826：RequestとReply](https://datatracker.ietf.org/doc/html/rfc826)、[Cisco公式：NAT用IPへのProxy ARP](https://www.cisco.com/c/en/us/td/docs/security/asa/asa923/configuration/firewall/asa-923-firewall-config/nat-reference.html)）。

WAN側のARP Requestは、今回のNATルーターを越えて、その内側のPC A・Bへ配送されるものではありません。上の図の「WAN側VLANに直接つながるPC」は、LAN側のPC A・Bとは別の機器です。ARPの問い合わせは同じリンク内で行い、次のリンクへIPパケットを渡すときは、そのリンクで使う配送先のMACを調べます（[RFC 826：ルーティング後のリンク内のアドレス解決](https://datatracker.ietf.org/doc/html/rfc826)、[RFC 1812、5.2.1.2節の手順10～11](https://www.rfc-editor.org/rfc/rfc1812.html#section-5.2.1.2)）。

関連：[NAT・NAPTと返答先の識別](./basic_security_at_home_20261005_question15_16_nat_napt.md)、[IPルーティングと転送経路](./basic_security_at_home_20261005_question16_ip_routing.md)。

調査日：2026-10-07。根拠はIETF／RFC EditorのRFC、Cisco・Microsoft公式資料、OS・iproute2のマニュアルです。図・表・表示例は説明用です。
