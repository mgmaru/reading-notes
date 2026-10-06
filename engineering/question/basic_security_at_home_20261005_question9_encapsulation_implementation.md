# 『おうちで学べるセキュリティの基本』2026-10-5 疑問9の追加質問：カプセル化・非カプセル化は誰が行う？

## 追加質問

> ルータであれ、サーバーであれ、クライアントであれ、カプセル化、非カプセル化するのはOSの役割という認識で良いですか？

## 回答

**PCやサーバーでは「主にOSのネットワーク機能が行う」でおおむね合っています。ただし、すべての層をOSだけが処理するわけではありません。** アプリケーション、TLSライブラリ、OSのネットワーク機能、ネットワークカード（NIC）が協力します。ルーターの転送も、OS上のソフトウェアで行う製品と、専用の転送回路で行う製品があります。RFC 1812はルーターの処理を定めていますが、ソフトウェア実装にも専用ハードウェア実装にも適用すると明記しています（[RFC 1812、2.2.3節](https://datatracker.ietf.org/doc/html/rfc1812#section-2.2.3)、[Linux Kernel：NICへの処理の委譲](https://docs.kernel.org/networking/segmentation-offloads.html)）。

### 送信側PCから受信側サーバーまで

以下は、TCP上でTLSを使い、途中に通常のIPルーターがある例です。**矢印はデータが各機能を通る順番**であり、特定の製品で必ず同じプログラム・チップが処理するという意味ではありません。

```text
送信側PC（クライアント）
  アプリ          メール本文などを作る
     ↓
  TLSの処理       TLSレコードを作り、データを暗号化する
     ↓
  OSのTCP/IP      TCPヘッダー・IPヘッダーを付ける
     ↓
  OSのドライバー／NIC  送信する区間のリンク層フレームを扱う
     ↓
============================= ネットワーク =============================
     ↓
ルーター
  受信側のリンク層を処理 → IP宛先を見て転送先を決める
                         → 次の区間のリンク層フレームを作って送る
  （通常の転送ではTCP接続やTLSを終端しない）
     ↓
============================= ネットワーク =============================
     ↓
受信側サーバー
  NIC／OSのドライバー → OSのIP・TCP → TLSの処理 → アプリ
  リンク層を処理       宛先と接続を処理  検証・復号   本文を読む
```

送信時にヘッダーを付けて包むのが**カプセル化**、受信側で必要なヘッダーを処理して上位層へ渡すのが**非カプセル化**です。TCPはアプリケーションのバイト列をセグメントにしてIPパケットで運びます。IPパケットは、その区間のリンク層フレームに入れて運ばれます（[RFC 9293、2.2節・3.1節](https://www.rfc-editor.org/rfc/rfc9293.html#section-2.2)、[RFC 894：IPパケットとEthernetフレーム](https://www.rfc-editor.org/rfc/rfc894.html)）。

| 機器 | 通常、どこまで処理する？ | 実行する場所の例 |
| --- | --- | --- |
| クライアント・サーバー | 通信の端点なので、リンク層からTCP、TLS、アプリケーションまで処理する | OSのネットワーク機能、TLSライブラリ、NIC |
| 中継するルーター | 入力側のリンク層フレームを処理し、IPで転送を判断して、出力側用のリンク層フレームを作る | ルーター上のソフトウェア、専用の転送回路 |

**「クライアント」「サーバー」はアプリケーション上の役割です。** 両者とも通信の端点なので、送信するときはカプセル化し、受信するときは逆向きに処理します。これに対し、**単に中継するルーターは、宛先アプリへデータを届ける端点ではありません。** 通常の転送ではTLSを復号したり、アプリケーション用のデータまで取り出したりしません（[RFC 1812、5.2.1節](https://datatracker.ietf.org/doc/html/rfc1812#section-5.2.1)、[RFC 8446、1節・5節](https://www.rfc-editor.org/rfc/rfc8446.html#section-5)）。

### OSだけ、と言い切れない理由

- **TLSの処理場所は実装によって異なる。** アプリはOS提供のTLS機能やユーザー空間のTLSライブラリを利用できます。LinuxにはTLSレコード処理をカーネルに任せる`kTLS`もあり、対応するNICへ暗号処理を委譲する構成もあります（[Microsoft：Schannel](https://learn.microsoft.com/en-us/windows-server/security/tls/tls-ssl-schannel-ssp-overview)、[Apple：ネットワークAPIの選び方](https://developer.apple.com/documentation/technotes/tn3151-choosing-the-right-networking-api)、[Linux Kernel：kTLS](https://docs.kernel.org/networking/tls.html)、[Linux Kernel：TLS offload](https://docs.kernel.org/networking/tls-offload.html)）。
- **NICも一部の通信処理を担える。** たとえばTCPの大きなデータを複数の送信フレームへ分ける処理を、対応するNICに委譲できます。したがって「ヘッダーを扱うのはすべてOSのCPU」とは限りません（[Linux Kernel：TCP Segmentation Offload](https://docs.kernel.org/networking/segmentation-offloads.html)）。
- **ルーターの転送方法は製品によって違う。** OSが制御情報を管理していても、通過する各パケットの転送はASICなどの専用回路が行う場合があります（[RFC 1812、2.2.3節](https://datatracker.ietf.org/doc/html/rfc1812#section-2.2.3)、[Cisco：ネットワークOSと転送回路](https://developer.cisco.com/docs/sonic/sonic-architecture/)）。

ここで説明したルーターは**他の機器への通信を中継する場合**です。ルーター自身の管理画面へHTTPSで接続する場合や、TLSを終端するプロキシ機能を持つ場合には、その機器が通信の端点となり、TLSまで処理します。機器の名前だけで処理範囲は決まりません（[NIST SP 800-41 Rev.1、2.1.4節：アプリケーションプロキシ](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-41r1.pdf)）。

**覚え方：** 「誰が処理するか」はOS・ライブラリ・NIC・専用回路で分担できます。「どこまで処理するか」は、その機器が**通信の端点か、途中の転送点か**でまず考えます。

## 追加質問：TLSの処理はOSには備わっていない？

> TLSの処理はOSには備わっていないという認識で良いですか？ 備わっていないので、アプリ側のライブラリで構築するという認識で良いですか？

**「OSにはTLSがない」は正確ではありません。** WindowsにはOSが提供する`Schannel`、AppleのOSには`URLSession`や`Network framework`から利用できるTLS機能があります。一方、LinuxではアプリがOpenSSLなどのTLSライブラリを使う構成が一般的です。つまり、アプリは**既存のTLS機能を利用する**のであって、TLSの暗号処理を毎回自作する必要はありません（[Microsoft：SchannelとSSPI](https://learn.microsoft.com/en-us/windows-server/security/tls/tls-ssl-schannel-ssp-overview)、[Apple：ネットワークAPIの選び方](https://developer.apple.com/documentation/technotes/tn3151-choosing-the-right-networking-api)、[Linux Kernel：ユーザー空間のTLSライブラリとkTLS](https://docs.kernel.org/networking/tls.html#integrating-in-to-userspace-tls-library)）。

| 環境 | アプリが利用できるTLS機能の例 | 補足 |
| --- | --- | --- |
| Windows | OS提供の`Schannel`（`SSPI`経由） | アプリがOSのTLS実装を呼び出せる |
| macOS・iOSなど | OS提供の`URLSession`や`Network framework` | HTTPSやTLS接続を扱うAPIがある |
| Linux | OpenSSLなどのTLSライブラリ。構成によっては`kTLS`も利用 | `kTLS`は主にハンドシェイク後のTLSレコード処理をカーネルへ移す |

```text
アプリ（TCP上でTLSを使う通信を選ぶ）
   ↓ TLS用のAPI・ライブラリを利用
TLS機能（OS提供の機能、またはアプリが利用するライブラリ）
   │ ハンドシェイク、必要な証明書の検証、データの暗号化・復号
   ↓ TLSレコード（暗号化されたアプリのデータ）
OSのTCP/IP機能 → NIC → ネットワーク

※ LinuxでkTLSを使う場合、ハンドシェイク後のレコード処理を
   カーネル（場合によっては対応NIC）へ任せることがある。
```

ここで区別したいのは、**「OSがTLS機能を提供するか」と「TCP/IPのカーネル処理がTLSを自動で行うか」は別の話**だという点です。通常のTCPソケットを作るだけではTLS通信にはなりません。アプリや利用中の通信APIがTLSを選び、必要なハンドシェイクと証明書の検証を行います。Linuxの`kTLS`も、まずハンドシェイクを終えた後に、その結果を使って送受信のTLSレコード処理をカーネルへ移す仕組みです（[Apple：BSDソケットとTLSの違い](https://developer.apple.com/documentation/technotes/tn3151-choosing-the-right-networking-api)、[Linux Kernel：kTLSの接続手順](https://docs.kernel.org/networking/tls.html#creating-a-tls-connection)）。

関連：[TLSとパケットのカプセル化](./basic_security_at_home_20261005_question9_tls_encapsulation.md)、[HTTPSサーバーを構築できるライブラリ](./basic_security_at_home_20261005_question9_https_server_libraries.md)
