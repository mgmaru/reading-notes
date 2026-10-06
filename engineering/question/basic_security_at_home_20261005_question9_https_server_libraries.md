# 『おうちで学べるセキュリティの基本』2026-10-5 疑問9の追加質問：WebアプリでHTTPSを構築できるライブラリ

## 追加質問

> WebアプリケーションのライブラリでHTTPSを構成できるものはありますか？HTTPであればたくさんある気がしますが、HTTPSはあまりないような気がします。調査して具体的に構築できるライブラリをあげてください。

## 回答

**あります。** ここではWebアプリを公開するサーバー側を扱います。WebアプリがTCP上のHTTPSを提供するときは、HTTPのリクエストを扱う機能と、TLSで接続を保護する機能を組み合わせます。両方を一つのAPIで提供するものも、WebフレームワークとTLS対応サーバーを組み合わせるものもあります。「HTTPS」という名前のWebフレームワークを別に探す必要はありません。Node.jsの公式文書もHTTPSを「TLS上のHTTP」と説明しています（[Node.js：HTTPS](https://nodejs.org/api/https.html)）。

```text
ブラウザ ── HTTPS（TLSで保護したHTTP）──> HTTPSサーバー
                                         ├─ TLS：接続の確立・暗号化／復号
                                         └─ HTTP：URLの振り分け・応答の生成
```

ExpressのようにHTTPの経路を定義するライブラリでは、TLSのAPIが目立たないことがあります。Expressの`app.listen()`はHTTPサーバーを作るため、アプリが直接HTTPSを受ける場合はNode.jsの`https.createServer()`にExpressの`app`を渡します。Expressの公式文書もこの構成を示しています（[Express：`app.listen()`の説明](https://expressjs.com/en/4x/api/application/#app.listen)、[Node.js：`https.createServer()`](https://nodejs.org/api/https.html#httpscreateserveroptions-requestlistener)）。

### HTTPSサーバーを構築できる具体例

| 言語・環境 | ライブラリ／フレームワーク | HTTPSにする方法 |
| --- | --- | --- |
| JavaScript／Node.js | [標準の`node:https`](https://nodejs.org/api/https.html#httpscreateserveroptions-requestlistener) ＋ [Express](https://expressjs.com/en/4x/api/application/#app.listen) | 証明書と秘密鍵を渡して`https.createServer(options, app)`を呼ぶ。ExpressがHTTPの経路を処理する |
| Python | [aiohttpのWebサーバー](https://docs.aiohttp.org/en/stable/web_reference.html#aiohttp.web.run_app) ＋ [標準の`ssl`](https://docs.python.org/3/library/ssl.html#ssl.SSLContext.load_cert_chain) | 証明書・秘密鍵を設定した`ssl.SSLContext`を`web.run_app(app, ssl_context=context)`へ渡す |
| Go | [標準の`net/http`](https://pkg.go.dev/net/http#ListenAndServeTLS) | `http.ListenAndServeTLS(":8443", "cert.pem", "key.pem", handler)`を呼ぶ |
| Java | [Spring Bootの組み込みWebサーバー](https://docs.spring.io/spring-boot/how-to/webserver.html#howto.webserver.configure-ssl) | `server.ssl.*`で証明書と秘密鍵を設定し、HTTPSで待ち受ける |
| C#／.NET | [ASP.NET CoreのKestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-10.0#configure-https) | HTTPSエンドポイントと証明書を設定する。コードでは`UseHttps()`も使える |

Expressを使う場合の最小例です。`server-key.pem`と`server-cert.pem`は、対応する秘密鍵とサーバー証明書を**あらかじめ用意したファイル**です。`app.get()`がHTTPの経路、`https.createServer()`がTLS付きの待ち受けを担当します（[Expressの公式例](https://expressjs.com/en/4x/api/application/#app.listen)、[Node.jsの公式例](https://nodejs.org/api/https.html#httpscreateserveroptions-requestlistener)）。

```js
const express = require('express');
const https = require('node:https');
const fs = require('node:fs');

const app = express();
app.get('/', (_req, res) => res.send('Hello HTTPS'));

https.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
}, app).listen(8443);
```

### アプリの外側でTLSを処理する構成

実運用では、CaddyやNginxなどのリバースプロキシでHTTPSを受け、Webアプリへ転送する構成もあります。これならアプリのコードにHTTPSの起動処理が見えなくても、ブラウザからプロキシまでの通信はHTTPSです。**プロキシからアプリまでの区間は別の通信**なので、HTTPで転送する設定ならその区間は暗号化されません（[Caddy：HTTPS対応のリバースプロキシ](https://caddyserver.com/docs/quick-starts/reverse-proxy)、[Nginx：HTTPSサーバーの設定](https://nginx.org/en/docs/http/configuring_https_servers.html)）。

```text
ブラウザ ── HTTPS ──> リバースプロキシ（TLSを終端） ── HTTPまたはHTTPS ──> Webアプリ
```

つまり、**「Webアプリ自身がTLSを終端する」方法と「手前のサーバーがTLSを終端する」方法の両方があります。** フレームワークの説明にHTTPの記述しかなくても、どの機器・ソフトウェアでTLSを終端するかを確認すると、HTTPSの構成が見えてきます。

## 追加質問：アプリ側とプロキシ側でTLSを処理すると、何が違う？

> 1. アプリ側でTLSを構築する場合と、アプリの外側でTLSを構築する場合を、暗号化される範囲が分かる図で見たいです。
> 2. それぞれのメリット・デメリットは何ですか？ アプリ側でTLSを構築した方が、アプリとリバースプロキシの間も暗号化されて安全ではないでしょうか？

**ポイントは「TLSをどこで終端（復号）するか」と「プロキシからアプリへ何の通信方式で送るか」です。** アプリにTLS機能があっても、プロキシからアプリへの転送をHTTPに設定すれば、その区間は暗号化されません。逆に、プロキシでブラウザ側のTLSを終端しても、アプリへの転送をHTTPSに設定すれば、両区間の通信を暗号化できます（[Caddy：上流へのHTTP/HTTPS転送](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy)、[NGINX：TLS終端と再暗号化](https://docs.nginx.com/nginx/deployment-guides/migrate-hardware-adc/f5-big-ip-configuration/)）。

### どこまで暗号化されるか

以下は**TCP上のHTTPS**の例です。`════`の区間はHTTPの内容がTLSで保護され、`────`の区間はHTTPの内容が平文で流れます。TLSで保護される区間でも、転送に必要なIPアドレスやTCPポート番号までTLSで暗号化されるわけではありません（[TLSとパケットのカプセル化](./basic_security_at_home_20261005_question9_tls_encapsulation.md)）。

```text
A. アプリがTLSを終端する（プロキシなし、またはTLSを復号しないTCP中継）
   ブラウザ ════ TLS ════ [TCP中継プロキシ：任意・復号しない] ════ 同じTLS ════ [アプリ：復号]
                     ↑ ブラウザからアプリまでHTTPの内容を暗号化 ↑

B. プロキシがTLSを終端し、アプリへHTTPで転送する
   ブラウザ ════ TLS ════ [プロキシ：復号] ──── HTTP ──── [アプリ]
                 暗号化された区間              平文の区間

C. プロキシがTLSを終端し、アプリへHTTPSで転送する（再暗号化）
   ブラウザ ════ TLS接続① ════ [プロキシ：復号→再暗号化] ════ TLS接続② ════ [アプリ：復号]
                 暗号化された区間               別の暗号化された区間
```

AのTCP中継プロキシはTLSの中身を復号せず、そのままアプリへ渡します。そのためHTTPのURLパスや本文を見た振り分けはできませんが、TLSの接続開始時に得られる情報などを使った振り分けは可能です（[NGINX：TLSを終端せずにSNIなどを読む仕組み](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html)）。Cでは**途中に平文のネットワーク区間はありませんが、プロキシの内部ではHTTPの内容が一度復号されます**。ブラウザからアプリまで一つのTLS接続が続くわけではなく、①と②は別のTLS接続です（[NGINX：再暗号化の説明](https://docs.nginx.com/nginx/deployment-guides/migrate-hardware-adc/f5-big-ip-configuration/)）。

### メリットとデメリット

| 構成 | メリット | デメリット・注意点 |
| --- | --- | --- |
| A：アプリでTLSを終端 | ブラウザからアプリまで同じTLS通信を保てる。TCP中継プロキシはHTTPの内容を読めない | アプリ側で証明書・秘密鍵の管理が必要。中継プロキシではURLパスなどHTTPの内容に基づく処理ができない |
| B：プロキシでTLSを終端→アプリへHTTP | アプリ側のTLS設定が不要。プロキシに証明書管理やHTTPの振り分けをまとめられる | プロキシとアプリの間は平文。この区間を通る通信を盗聴・改ざんされるリスクを別途評価する必要がある |
| C：プロキシでTLSを終端→アプリへHTTPS | 両方のネットワーク区間を暗号化しながら、プロキシでHTTPの振り分けもできる | プロキシはHTTPの内容を読める。アプリ側の証明書、プロキシ側の接続設定・証明書検証など、運用する項目が増える |

この比較は、[Caddyの証明書自動管理](https://caddyserver.com/docs/automatic-https)、[CaddyのHTTP転送とHTTPS転送](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy)、[NGINXのTLSを終端しない転送](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html)、[NGINXの再暗号化](https://docs.nginx.com/nginx/deployment-guides/migrate-hardware-adc/f5-big-ip-configuration/)から、各構成で必要になる機能と設定を整理したものです。

**「アプリ側のTLSなら必ずより安全」とは言い切れません。** プロキシとアプリが別のホストにあり、その間のネットワークも保護したいなら、AのようにTLSをアプリまで通すか、Cのようにプロキシからアプリへの通信もHTTPSにします。一方、BのようにHTTPで転送する場合は、その区間を暗号化しない設計です。どの区間の盗聴・改ざんを防ぐ必要があるかで選びます。Cでもプロキシ自身には平文が見えるため、**プロキシにも内容を見せたくない**要件ならAが適します（[NGINX：各構成と上流ネットワークの考え方](https://docs.nginx.com/nginx/deployment-guides/migrate-hardware-adc/f5-big-ip-configuration/)）。

Cを選ぶときは、**プロキシがアプリ側の証明書を検証すること**も確認します。たとえばNGINXでは、上流HTTPSサーバーの証明書を検証する`proxy_ssl_verify`の既定値は`off`です。必要に応じて信頼するCAを指定し、検証を有効にします（[NGINX：`proxy_ssl_verify`の既定値](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_verify)、[NGINX：上流サーバーの証明書検証設定](https://docs.nginx.com/nginx/admin-guide/security-controls/securing-http-traffic-upstream/)）。

関連：[カプセル化・非カプセル化の実装とTLSの役割](./basic_security_at_home_20261005_question9_encapsulation_implementation.md)
