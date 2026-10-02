# 『おうちで学べるセキュリティの基本』2026-10-2 疑問13：PCの80番ポートとファイアウォール

## 疑問

1. 自分のPCの80番ポートをファイアウォールで遮断すると、Webサーバーへのアクセスと、外部から自分のPCの80番ポートへのアクセスは両方できなくなるのか？
2. 自分のPCで80番ポートが使われていて、ファイアウォールでは遮断していない場合、外部のWebサーバーにはアクセスできるが、自分のPCへの80番ポート宛てのアクセスは遮断されるのか？
3. 内向き・ローカルポートと外向き・リモートポートは、どちらも自分のPCのファイアウォールで遮断できるのか？WindowsとLinuxで設定方法は違うのか？
4. Linuxサーバーの設定を、CLI以外の画面やOSSのツールで管理できるのか？

## 回答

**どちらも「自分のPCの80番ポート」という言葉だけでは決まりません。** 通信には送信元と宛先のポートが別々にあり、ファイアウォールも「外向きか内向きか」「自分側のポートか相手側のポートか」を条件にできます。TCPのヘッダーにも送信元ポートと宛先ポートが別々に入っています。詳しくは [RFC 9293 の TCP ヘッダー形式](https://www.rfc-editor.org/rfc/rfc9293.html#section-3.1) を参照してください。

たとえば、PCのブラウザから **HTTP** のWebサーバーへ接続する場合を考えます。`53000` は説明用に選んだ、PC側の一時的なポート番号です。

```text
接続を開始：自分のPC:53000 ──→ Webサーバー:80
応答を受信：自分のPC:53000 ←── Webサーバー:80

別の通信：外部の端末:任意のポート ──→ 自分のPC:80
```

最初の通信では、**80番は相手側（リモート）のポート**です。応答は自分のPCの `53000` 番へ戻ります。最後の通信では、**80番は自分側（ローカル）のポート**です。同じ「80番へのアクセス」でも、接続先が異なります。

| 自分のPCに設定するルールの例 | 外部のWebサーバーのTCP/80へ接続 | 外部から自分のPCのTCP/80へ新しく接続 |
| --- | --- | --- |
| **内向き**・**ローカル**ポート80を遮断 | 通常は可能 | 遮断される |
| **外向き**・**リモート**ポート80を遮断 | 遮断される | このルールだけでは遮断されない |
| 上の両方を遮断 | 遮断される | 遮断される |

この表は、示した遮断ルールの効果に着目した整理です。実際の通信は、ほかのルールや既定の動作にも左右されます。Windows ファイアウォールの公式資料でも、内向きのルールでは通常ローカルポート、外向きのルールでは通常リモートポートを指定すると説明されています（[Microsoft Learn：ファイアウォールルールの設定](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/configure)）。

### 1. PCの80番ポートを遮断した場合

「**内向き・ローカルポート80を遮断**」という設定なら、外部からPCの80番ポートへ始まる接続は遮断されます。一方、ブラウザが外部のWebサーバーの80番へ接続するとき、自分側では通常、別の一時的なポートを使うため、この設定だけでWeb閲覧は止まりません。

逆に「**外向き・リモートポート80を遮断**」すれば、外部のHTTPサーバーの80番への接続を止められます。さらに内向き・ローカルポート80も遮断すれば、質問のように両方向の接続を止められます。なお、HTTPS の既定の接続先ポートは **443** なので、TCP/80だけを遮断してもすべてのWeb閲覧が止まるわけではありません（[RFC 9110：HTTP と HTTPS の既定ポート](https://www.rfc-editor.org/rfc/rfc9110.html#section-4.2)）。

接続を許可した後の応答は、通信の状態を追跡するファイアウォールなら、その接続に属するパケットとして扱えます。「内向きの新しい接続を遮断する」ことと「自分から始めた接続の応答を受け取る」ことは両立します（[Microsoft Learn：Windows ファイアウォールの既定の動作](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/)、[Microsoft Learn：ステートフルフィルタリング](https://learn.microsoft.com/en-us/windows/win32/fwp/ale-stateful-filtering)）。

### 2. PCの80番ポートが使われているだけの場合

**80番ポートを使うプログラムがあることと、ファイアウォールが通信を遮断することは別です。** PCの80番ポートが使われていても、ブラウザは通常、別の一時的なポートから外部のWebサーバーへ接続できます。また、PCでWebサーバーが80番ポートを待ち受けており、ファイアウォールなどが接続を許可し、相手からPCへ通信が届くなら、外部からそのWebサーバーに接続できます。ポートが使われているだけで、外部からの接続が自動的に遮断されるわけではありません。

反対に、80番ポートで待ち受けるプログラムがなければ、そこへ新しいTCP接続は成立しません。ただし、これは「ファイアウォールが遮断した」という意味ではありません。また、待ち受けていても、ローカルホストだけに紐づけている場合や、ルーター・ファイアウォールで通信が届かない場合は、外部から接続できません。

### 3. 自分のPCで両方向を遮断できるか／OSによる違い

**はい。内向き・ローカルポート80も、外向き・リモートポート80も、自分のPCのファイアウォールで遮断できます。** ただし、それぞれ対象とする通信が違うため、両方を遮断したければ両方に対応するルールを設定します。

| 遮断したい通信 | Windowsで指定する条件 | UbuntuのUFWで指定する条件 |
| --- | --- | --- |
| 外部から自分のPCのTCP/80へ来る通信 | `Inbound`（内向き）＋`LocalPort 80` | `in`＋宛先ポート80 |
| 自分のPCから外部のTCP/80へ行く通信 | `Outbound`（外向き）＋`RemotePort 80` | `out`＋宛先ポート80 |

Windows では「ローカル／リモート」は**自分のPCから見た位置**を表します。一方、以下のUFWの例では `to any port 80` が**その通信の宛先ポート**を表すため、内向きなら自分側の80番、外向きなら相手側の80番になります。Windows では管理画面または PowerShell、Ubuntu では UFW などを使って設定します。Linux で使うツールはディストリビューションや環境によって異なります（[Microsoft Learn：ファイアウォールルールの設定](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/configure)、[Ubuntu：UFWマニュアル](https://manpages.ubuntu.com/manpages/jammy/man8/ufw.8.html)）。

設定例（**自分のPCで実行する場合**。各行は別々のルール）：

```powershell
# Windows PowerShell：外部から自分のPCのTCP/80への接続を遮断
New-NetFirewallRule -DisplayName "Block inbound TCP 80" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Block

# Windows PowerShell：自分のPCから外部のTCP/80への接続を遮断
New-NetFirewallRule -DisplayName "Block outbound TCP 80" -Direction Outbound -Protocol TCP -RemotePort 80 -Action Block
```

```bash
# Ubuntu（UFW）：外部から自分のPCのTCP/80への接続を遮断
sudo ufw deny in proto tcp to any port 80

# Ubuntu（UFW）：自分のPCから外部のTCP/80への接続を遮断
sudo ufw deny out proto tcp to any port 80
```

Windows の `New-NetFirewallRule` には `-Direction`・`-LocalPort`・`-RemotePort` を指定できます（[Microsoft Learn：New-NetFirewallRule](https://learn.microsoft.com/en-us/powershell/module/netsecurity/new-netfirewallrule)）。UFW のルールは、UFW が有効なときに適用されます。Ubuntu では `sudo ufw status` で状態を確認できます（[Ubuntu Server：ファイアウォール](https://ubuntu.com/server/docs/firewalls/)）。また、Windows ファイアウォールは既定で、要求していない内向き通信を遮断し、外向き通信を許可します。既定の動作や既存のルールも確認してから、必要なルールを判断します（[Microsoft Learn：Windows ファイアウォールの概要](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/)）。

### 4. Linuxサーバーの設定を管理するOSS

Linuxサーバーには、**ブラウザやデスクトップ画面から操作するツール**と、**設定をファイルに記述して適用するツール**があります。ファイアウォールだけでなく、サービスやユーザーなどを扱えるものもあります。

| ツール | 操作方法と主な用途 | 80番ポートの設定との関係 |
| --- | --- | --- |
| [Cockpit](https://cockpit-project.org/) | ブラウザからサービス、ログ、ストレージ、ネットワークなどを管理 | ファイアウォール画面は **firewalld** を使う。現在は既定ゾーンの定義済みサービスを表示・追加・削除する機能に限られ、UFWのルールや外向きTCP/80の遮断をその画面から設定する用途には合わない（[公式資料](https://docs.cockpit-project.org/cockpit-guide/latest/guide/feature-firewall.html)）。 |
| [Webmin](https://webmin.com/docs/intro/) | ブラウザからユーザー、パッケージ、サービス、ネットワーク設定などを管理 | [nftables用モジュール](https://webmin.com/docs/modules/nftables/)では `input`・`output` のルールや、送信元・宛先ポートを編集できる。 |
| [Gufw](https://github.com/costales/gufw) | デスクトップ画面からUFWを操作 | UFWのGUI。ブラウザで遠隔管理する画面ではなく、画面を備えたLinux環境向け。 |
| [Ansible](https://docs.ansible.com/projects/ansible/latest/getting_started/introduction.html) | YAML形式の設定ファイルを使い、SSH経由で複数のサーバーの設定を適用 | [UFW用モジュール](https://docs.ansible.com/projects/ansible/latest/collections/community/general/ufw_module.html)に `direction: in/out`、`to_port: '80'`、`rule: deny` を指定できる。 |

**今回のUFW設定を複数のUbuntuサーバーでそろえるなら、Ansibleが適しています。** 設定内容をファイルで管理でき、同じ設定を繰り返し適用できます。UFW用モジュールは `community.general` コレクションに含まれ、`ansible-core` だけには含まれません（[AnsibleのUFWモジュール資料](https://docs.ansible.com/projects/ansible/latest/collections/community/general/ufw_module.html)）。

一方、サーバーの状態をブラウザで確認しながら操作したいならCockpitやWebminが候補です。ただし、**操作画面によって管理するファイアウォールの仕組みが違います**。先ほどのUFWコマンドを使っているサーバーで、Cockpitのファイアウォール画面が同じUFWルールを編集するわけではありません。また、WebminのnftablesモジュールとUFWなど、複数のツールで有効なルールを同時に変更すると、変更が上書きされたり見えにくくなったりします。どの仕組みでルールを管理するかを決めて使います（[Cockpitのファイアウォール資料](https://docs.cockpit-project.org/cockpit-guide/latest/guide/feature-firewall.html)、[Webminのnftables資料](https://webmin.com/docs/modules/nftables/)）。

**覚え方：** ブラウザで外のHTTPサイトを見るときは「**相手の80番へ行く**」、自分のPCでHTTPサーバーを公開するときは「**自分の80番で待つ**」です。設定を確認するときは、`TCP`・`内向き/外向き`・`ローカル/リモートのポート`をセットで読みます。
