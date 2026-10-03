# 『おうちで学べるセキュリティの基本』2026-10-2 疑問1：POP3は今も使われているか

## 疑問

> POP3は受信専用とのことだが、現在も使用されているのか？

## 回答

**はい。2026年10月現在もPOP3は使われています。** たとえば、Gmailは外部のメールソフトがGmailのメールをPOP3で取得する機能を提供しています。Exchange Onlineでも、管理者がメールボックスごとにPOP3の利用を有効・無効にできます。ただし、サービスや組織の設定によって利用できるかどうかは変わります（[Gmailヘルプ：外部メールソフトからのアクセス](https://support.google.com/mail/answer/17101213?hl=en)、[Microsoft Learn：Exchange OnlineのPOP3設定](https://learn.microsoft.com/en-us/troubleshoot/exchange/user-and-shared-mailboxes/pop3-imap-owa-activesync-office-365)）。

### 「受信専用」はどの部分を指すか

POP3は、**受信サーバーに届いたメールをメールソフトが取得する**ための通信手順です。メールを送るときは、通常、別の手順であるSMTPのメッセージ送信（Submission）を使います。「POP3では送信できない」は正しいですが、「POP3を使うメールソフトでは送信できない」という意味ではありません（[RFC 1939：POP3](https://www.rfc-editor.org/rfc/rfc1939.html)、[RFC 6409：メールの送信](https://www.rfc-editor.org/rfc/rfc6409.html)）。

```text
送信者のメールソフト ── SMTP Submission ──→ 送信用サーバー
                                              │ サーバー間で配送
                                              ↓
受信者のメールソフト ←────── POP3 ────── 受信者のメールサーバー
```

本のポート一覧にある**110番**はPOP3の標準ポートです。暗号化した通信を接続直後から始めるPOP3 over TLSでは、通常**995番**を使います。送信用のSubmissionでは通常**587番**を使います。実際の設定では、利用するサービスが指定するポート、暗号化方式、認証方法を確認します（[RFC 1939](https://www.rfc-editor.org/rfc/rfc1939.html)、[RFC 8314：メールアクセスのTLS](https://www.rfc-editor.org/rfc/rfc8314.html)、[RFC 6409](https://www.rfc-editor.org/rfc/rfc6409.html)）。

### POP3とIMAPの使い分け

| 観点 | POP3 | IMAP |
| --- | --- | --- |
| 基本の使い方 | サーバーからメールを取得し、端末側で読む | サーバー上のメールボックスを複数端末から扱う |
| フォルダーや既読状態 | 複数フォルダーや既読状態の共有には向かない | フォルダーや既読状態などをサーバー側で扱える |
| 複数端末での利用 | メールをサーバーから削除する設定だと、ほかの端末で読めなくなる | 複数端末で同じメールを扱いやすい |
| 向いている例 | 1台の端末へメールを取り込み、手元に保存したい場合 | PCとスマートフォンの両方でメールを確認したい場合 |

POP3でも、**取得後にメールをサーバーへ残す設定**は可能です。「POP3で受信すると必ずサーバーから消える」わけではありません。ただし、残しておくだけでは端末間の既読状態やフォルダー構成まで同期されません。これらを扱うならIMAPのほうが適しています（[RFC 1939：削除の仕組み](https://www.rfc-editor.org/rfc/rfc1939.html#section-6)、[Microsoft Learn：POP3とIMAP4の違い](https://learn.microsoft.com/en-us/exchange/clients/pop3-and-imap4/pop3-and-imap4)）。

### 現在のサービスで混同しやすい点

Gmailが**ほかのサービスのメールをPOP3で取り込む機能**は、2027年1月に終了する予定です。一方、**外部のメールソフトがGmailのメールをPOP3で読む機能**は、Googleの案内では引き続きサポートされます。これは別々の機能なので、「GmailでPOP3が全面的に廃止される」とは読めません（[Gmailヘルプ：変更対象と継続する機能](https://support.google.com/mail/answer/17101213?hl=en)）。

**覚え方：** POP3は「受信済みメールをサーバーから取り出す」、SMTP Submissionは「作成したメールを送信のために預ける」、IMAPは「サーバー上のメールを複数端末から扱う」です。
