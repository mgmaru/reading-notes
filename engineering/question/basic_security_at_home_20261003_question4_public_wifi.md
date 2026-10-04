# 『おうちで学べるセキュリティの基本』2026-10-3 疑問4：カフェのWi-Fiが「暗号化されていない」とき

## 疑問

> カフェのWi-Fiで「暗号化されていない」と表示されることがある。セキュリティ的に大丈夫なのか？

## 回答

**その表示だけで「Webサイトのパスワードが丸見え」とは限りません。** 多くの場合、表示が指すのは**端末とWi-Fiアクセスポイント（AP）の間**の暗号化です。訪問先のサイトが正しい **HTTPS** を使っていれば、通信内容は端末からそのサイトまで別途保護されます。ただし、オープンなWi-Fiでは、その保護がない通信を盗み見・改ざんされるおそれがあります。ネットワークそのものが偽物である可能性も考えます（[FTC：公共Wi-Fiの安全性](https://consumer.ftc.gov/articles/are-public-wi-fi-networks-safe-what-you-need-know)、[CISA：公共Wi-Fiの利用上の注意](https://www.cisa.gov/sites/default/files/publications/Best%20Practices%20for%20Using%20Public%20WiFi.pdf)）。

### 暗号化には「区間」がある

```text
端末 ──[ Wi-Fiの暗号化 ]── AP ── インターネット ── Webサイト
  └────────────[ HTTPS/TLSによる保護 ]───────────────┘
```

| 仕組み | 守る範囲 | 守れないもの |
| --- | --- | --- |
| Wi-Fiの暗号化（WPA2/WPA3など） | 端末とAPの間の無線通信 | APから先の通信。接続したサイトの正当性まで保証しない |
| HTTPS/TLS | 端末と接続先サイトとの通信内容、通信相手の証明書に基づく確認 | 端末が接続するWi-Fi自体の正当性。偽サイトを正規サイトに変えることもできない |

つまり、**Wi-Fiの暗号化がなくても、正しいHTTPS接続の本文、ログイン情報、URLのパスなどは通常、同じWi-Fiの利用者やAPの運営者には読めません。** 一方、HTTPSでは通信の存在や接続先IPアドレスなど、すべての情報が隠れるわけではありません。DNSの問い合わせなど、ほかの通信の保護状況も別です（[FTC：公共Wi-Fiの安全性](https://consumer.ftc.gov/articles/are-public-wi-fi-networks-safe-what-you-need-know)、[NCSC：HTTPSとVPNによる通信保護](https://www.ncsc.gov.uk/guidance/krack)）。

| Wi-Fiの方式 | 端末とAP間の状態 | 注意点 |
| --- | --- | --- |
| **オープン（暗号化なし）** | 無線区間の暗号化・参加者認証がない | 近くの人などに、保護されていない通信を見られる可能性がある |
| **Enhanced Open / OWE** | パスワードなしで無線区間を暗号化する | APの本人確認はしない。偽APや通信先への対策にはHTTPSが必要 |
| **WPA2/WPA3** | Wi-Fiへの接続と無線区間を保護する | 共有された接続パスワードやAPの管理者を、接続先サイトの安全性と同一視しない。HTTPSも使う |

「パスワードがないWi-Fi＝必ず無線区間が平文」とも言い切れません。OWEにはパスワードなしで暗号化する方式があります。ただし、OWEはAPを認証せず、Webサイトまでの暗号化も提供しません（[RFC 8110：OWEの仕組みと限界](https://www.rfc-editor.org/rfc/rfc8110.html)）。また、カフェの利用規約やログイン画面（**キャプティブポータル**）で認証しても、それだけで無線区間が暗号化されたことにはなりません（[Mozilla：キャプティブポータル](https://support.mozilla.org/en-US/kb/captive-portal)）。

### カフェでの具体的な使い方

1. **店が案内するSSID（Wi-Fi名）を確認する。** 似た名前の偽APを避けるため、掲示や店員の案内と照合する。SSIDが合っていても、その事実だけでAPの正当性は証明できない（[CISA：公共Wi-Fiの利用上の注意](https://www.cisa.gov/sites/default/files/publications/Best%20Practices%20for%20Using%20Public%20WiFi.pdf)）。
2. **ログインや決済では、正しいドメイン名と `https://` を確認する。** 証明書の警告を無視して続行しない。HTTPSでも偽サイトへの送信は防げないので、鍵の表示だけで判断しない（[FTC：公共Wi-Fiの安全性](https://consumer.ftc.gov/articles/are-public-wi-fi-networks-safe-what-you-need-know)）。
3. **HTTPのページにはパスワードなどを入力しない。** HTTPSへ切り替えられず警告が出るなら、その操作を中断する。対応ブラウザのHTTPS優先・HTTPSのみの設定も役立つ（[Mozilla：HTTPS-Only Mode](https://blog.mozilla.org/security/2020/11/17/firefox-83-introduces-https-only-mode/)）。
4. **端末とブラウザを更新し、仕事では会社指定の接続方法に従う。** 会社のVPNを指定されている場合は利用する。VPNは**VPNへ流した通信**を端末からVPNの接続先まで保護するが、HTTPSによるサイトとの通信保護や接続先確認の代わりにはならない（[FTC：更新の推奨](https://consumer.ftc.gov/articles/are-public-wi-fi-networks-safe-what-you-need-know)、[NCSC：VPNで保護される通信の範囲](https://www.ncsc.gov.uk/collection/device-security-guidance/infrastructure/virtual-private-networks)）。

**判断の例：** 店の本物のオープンWi-Fiから、正しいドメインのHTTPSサイトに接続し、証明書の警告も出ていなければ、ログイン情報の通信内容は通常HTTPSで保護されます。反対に、接続先が `http://` だったり、証明書の警告を無視したり、偽サイトへ入力したりすれば、Wi-Fiの種類にかかわらず危険です。接続先に確信が持てないときは、その操作には携帯回線を使います。
