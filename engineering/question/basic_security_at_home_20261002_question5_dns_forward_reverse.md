# 『おうちで学べるセキュリティの基本』2026-10-2 疑問5：DNSの正引きと逆引き

## 疑問

> DNSはドメイン名をIPアドレスに結びつけるという認識で良い？
>
> - IPアドレスをドメイン名に紐づけることはある？？
> - ドメイン名は人間が覚えやすいようにつけた仕組みと理解しているが、この理解で良い？

## 回答

**基本的にはその理解で合っています。** Webサイトへ接続するときは、DNSで名前に対応するIPアドレスを調べます。逆方向に、IPアドレスから名前を調べる仕組みもあります。ただし、両方向は**別々のDNSレコード**で管理するため、一方を設定しただけで他方が自動的に決まるわけではありません（[RFC 1035：A・PTRレコード](https://www.rfc-editor.org/rfc/rfc1035.html)、[RFC 3596：AAAAレコードとIPv6逆引き](https://www.rfc-editor.org/rfc/rfc3596.html)）。

| 向き | 問い合わせの例 | 使用するレコード | 答えの例 |
| --- | --- | --- | --- |
| 正引き：名前 → IPアドレス | `www.example.com` のIPv4アドレスは？ | `A` | `192.0.2.10` |
| 正引き：名前 → IPアドレス | `www.example.com` のIPv6アドレスは？ | `AAAA` | `2001:db8::10` |
| 逆引き：IPアドレス → 名前 | `192.0.2.10` に対応する名前は？ | `PTR` | `www.example.com` |

表の名前とIPアドレスは**説明用**です。IPv4の逆引きでは、IPアドレスの数字を逆順に並べた `10.2.0.192.in-addr.arpa` という名前の `PTR` レコードを調べます。IPv6では `ip6.arpa` を使います（[RFC 1035：`in-addr.arpa`](https://www.rfc-editor.org/rfc/rfc1035.html#section-3.5)、[RFC 3596：`ip6.arpa`](https://www.rfc-editor.org/rfc/rfc3596.html#section-2.5)）。

```text
正引き： www.example.com ── A ──→ 192.0.2.10
逆引き： 192.0.2.10 ── PTR ──→ www.example.com  （PTRを設定した場合）
```

逆引きの `PTR` が設定されていないIPアドレスもあります。また、正引きと逆引きは異なるゾーンで管理されるため、結果が一致しない場合もあります。**PTRで名前が返ってきたことだけでは、接続先の身元を証明できません。** 1つの名前に複数のIPアドレスを登録することもできます（[RFC 1034：正引きと逆引きの不一致](https://www.rfc-editor.org/rfc/rfc1034.html)、[RFC 1035：複数のAレコード](https://www.rfc-editor.org/rfc/rfc1035.html#section-3.4.1)）。

### ドメイン名は何のためにあるか

人がIPアドレスの数字を覚えずに、`www.example.com` のような名前を使えることは大きな利点です。ただしDNSは「名前とIPアドレスだけの対応表」ではありません。たとえば `MX` レコードはメールの配送先、`NS` レコードはその名前を管理するDNSサーバーを示します。名前の管理を組織ごとに分担し、情報を分散して保存できることもDNSの重要な役割です。**すべてのドメイン名にIPアドレスのレコードが必要なわけではありません。**（[RFC 1034：DNSの設計目的](https://www.rfc-editor.org/rfc/rfc1034.html#section-2.2)、[RFC 1035：レコードの種類](https://www.rfc-editor.org/rfc/rfc1035.html#section-3.2.2)）

この「管理を分担する仕組み」は[疑問6〜8：DNSの階層と委任](./basic_security_at_home_20261002_question6_7_8_dns_hierarchy.md)で説明します。
