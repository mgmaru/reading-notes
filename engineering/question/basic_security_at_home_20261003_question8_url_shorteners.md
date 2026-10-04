# 『おうちで学べるセキュリティの基本』2026-10-3 疑問8：短縮URLの仕組みと用途

## 疑問

> 短縮URLとは何か？ どのようなところで使われるのか？

## 回答

**短縮URLは、長いURLへ転送するための短い別のURLです。** サービスに元のURLを登録すると、短い文字列を持つURLが発行されます。利用者がそれを開くと、まず短縮サービスにアクセスし、サービスが元のURLへ転送します。元のWebページ自体が短くなるわけではありません（[Bitly：短縮リンクの仕組み](https://support.bitly.com/hc/en-us/articles/230897368-How-does-a-shortened-link-work)、[RFC 9110：HTTPのリダイレクト](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.4)）。

```text
元のURL（例）：https://shop.example/products/123?campaign=autumn
短縮URL（例）：https://go.example/aB3

ブラウザ ── GET /aB3 ──→ go.example の短縮サービス
ブラウザ ←─ 301 Moved Permanently
            Location: https://shop.example/products/123?campaign=autumn
ブラウザ ── 元のURLへアクセス ──→ shop.example
```

このURLと応答は**仕組みを示す架空の例**です。`Location`ヘッダーが転送先を示し、ブラウザはそれに従って再度アクセスできます。Bitlyは`301`を使うと説明していますが、短縮サービスによって別のリダイレクト応答を使う場合もあります（[Bitly：301による転送](https://support.bitly.com/hc/en-us/articles/230897368-How-does-a-shortened-link-work)、[RFC 9110：Locationヘッダー](https://www.rfc-editor.org/rfc/rfc9110.html#section-10.2.2)）。

| 使用場所 | 短縮する理由・使い方 |
| --- | --- |
| SNS、チャット、SMS、メール | 長いURLを共有しやすくし、表示をすっきりさせる |
| チラシ、ポスター、QRコード | 印刷物へ載せやすくする。QRコードのリンク先として使うこともある |
| キャンペーン | リンクのクリック数などを集計し、配布先ごとの反応を調べる |

Bitlyの製品資料には、リンクの共有、QRコード化、クリック・スキャンの集計が具体例として載っています。集計できる項目や編集の可否はサービスや契約によって異なります（[Bitly：リンクの作成とQRコード](https://support.bitly.com/hc/en-us/articles/230897128-How-do-I-create-links-with-Bitly)、[Bitly：計測できる指標](https://support.bitly.com/hc/en-us/articles/20370474672141-What-metrics-are-available-in-Bitly)）。

### フィッシングとの関係

短縮URLの文字列からは、**最終的な行き先のドメインが分かりにくい**という特徴があります。攻撃者が偽ログイン画面へのリンクを短縮してメールに載せると、見た目から行き先を判断しづらくなります。ただし、短縮URLそれ自体が危険な技術という意味ではありません。CISAは**信頼できない短縮URL**をフィッシングの注意サインとして挙げています（[CISA：フィッシング対策](https://www.cisa.gov/sites/default/files/2024-09/Secure-Our-World-Phishing-Tip-Sheet.pdf)）。

1. 心当たりのないメッセージでは、短縮URLをそのまま開かず、公式アプリや自分で入力した公式サイトから確認する。
2. リンクを調べる必要がある場合は、**その短縮サービスが提供する確認機能**で転送先を確かめる。たとえばBitlyには[Link Checker](https://support.bitly.com/hc/en-us/articles/230650447-Can-I-check-a-Bitly-link-s-destination-before-clicking-on-it)がある。表示された転送先も偽装され得るので、ドメイン名を確認する。
3. 開いた後にログインを求められたら、アドレスバーの**最終的なドメイン名**を確かめる。短縮サービスのドメインや`https://`の表示だけで、転送先が本物とは判断できない。

疑問7の「メールなどでクリックを誘導する」手口では、短縮URLは**誘導先を隠しやすくする道具**の一つです。リンクを開くこと、偽サイトへ認証情報を入力すること、ファイルを実行することはそれぞれ異なる段階です（[MITRE ATT&CK：悪意あるリンクを利用者に開かせる手口](https://attack.mitre.org/techniques/T1204/001/)）。
