# 『おうちで学べるセキュリティの基本』2026-10-2 疑問2：`netstat`の業務での使いどころ

## 疑問

> `netstat`のユースケースは？ 業務の中でどのような時に使用されるのか？

## 回答

**`netstat`は、その端末で待ち受けているポートや、現在の通信状態を調べるときに使います。** たとえば「Webアプリを起動したのに接続できない」「8080番ポートが使用中と表示された」ときの切り分けに役立ちます。`netstat -n`の`-n`は、IPアドレスやポート番号を名前に置き換えず、数値で表示する指定です。**`-n`だけでは待ち受け中のポートをすべて表示する指定にはなりません。** 表示内容とオプションはOSによって違います（[Windowsのnetstat](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat)、[Linuxのnetstat](https://man7.org/linux/man-pages/man8/netstat.8.html)）。

### 仕事でよくある調査例

| 困っていること | まず調べること | 分かること・次の確認 |
| --- | --- | --- |
| アプリが「8080番ポートは使用中」で起動できない | ローカルの8080番で待ち受けるプロセス | 別のアプリが使っているかを確認し、起動するポートやプロセスを見直す |
| サーバーを起動したのに別のPCから接続できない | 8080番で`LISTEN`しているか、どのIPアドレスで待っているか | `127.0.0.1`だけなら同じ端末からの接続向け。外部から接続する設計なら待ち受け先の設定を確認する |
| 接続数が増えている、通信が切れない | 相手先IP・ポート、`ESTABLISHED`などの状態 | どの通信が残っているかを確認し、アプリのログや接続数の設定と照らす |

`LISTEN`はTCPで接続を待っている状態、`ESTABLISHED`はTCP接続が成立している状態です。たとえば次は**説明用の出力例**です。実際の列名や表示形式はOSで変わります。

```text
状態         ローカル側              相手側
LISTEN       127.0.0.1:8080          -
ESTABLISHED  192.0.2.10:53000        198.51.100.20:443
```

1行目は「この端末の`127.0.0.1:8080`で待ち受け中」、2行目は「この端末の一時的なポート`53000`から、相手の`443`番へTCP接続中」と読みます。**ローカル側と相手側のポートを区別する**ことが大切です。ポートの位置関係は[疑問13：PCの80番ポートとファイアウォール](./basic_security_at_home_20261002_question13.md)でも説明しています。

### OSごとの確認コマンド

| OS | 待ち受けポートの確認 | 使っているプロセスの確認 |
| --- | --- | --- |
| macOS | `netstat -an -p tcp`でTCPの一覧を表示し、`LISTEN`と`.8080`付近を探す | `lsof -nP -iTCP:8080 -sTCP:LISTEN`でプロセス名とPIDを確認する |
| Linux | `sudo ss -ltnp`でTCPの待ち受けとプロセスを表示する | 同じ出力の`users:`欄などを確認する。`netstat`を使う環境では`sudo netstat -lntp`も可能 |
| Windows | `netstat -ano`でポート・状態・PIDを表示し、ローカル側の`:8080`と`LISTENING`を探す | 表示されたPIDをタスクマネージャーで調べる |

macOSの`netstat`は、アドレスとポートの区切りを`127.0.0.1.8080`のように**ピリオド**で表示します。上の説明用出力例は読みやすさのため`:`で表しています。Windowsでは待ち受け状態を`LISTENING`と表示します。

たとえば「8080番のWebアプリへ接続できない」場合は、次の順に確認します。

1. **待ち受けの有無：** 8080番に`LISTEN`（Windowsでは`LISTENING`）があるか。なければ、アプリの起動状態とログを確認する。
2. **待ち受け先のIPアドレス：** `127.0.0.1`なら同じ端末からの接続向け。`0.0.0.0`ならその端末のすべてのIPv4アドレスで待ち受ける設定だが、外部から届くかは別途確認する。
3. **プロセス：** PIDやプロセス名が想定したアプリか確認する。別のアプリなら、そのアプリの設定とポートの割り当てを調べる。

ここでいう**PID**は、OSが実行中のプロセスに付ける番号です。macOSの`netstat -p tcp`の`-p`は**プロトコル指定**、Linuxの`netstat -p`は**プロセス表示**です。Windowsでは`-o`でPIDを表示します。同じ文字のオプションをOS間でそのまま使い回さないでください（macOSでは`man netstat`、[Linuxのnetstat](https://man7.org/linux/man-pages/man8/netstat.8.html)、[Windowsのnetstat](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat)、[lsofの説明](https://github.com/lsof-org/lsof/blob/master/docs/manpage.md)）。

Linuxでは`netstat`のマニュアルが後継として`ss`を案内しています。`ss`の`-l`は待ち受け、`-t`はTCP、`-n`は数値表示、`-p`はプロセス表示です。どのコマンドでも、ほかの利用者のプロセス情報を見るには権限が必要な場合があります（[Linuxのnetstat](https://man7.org/linux/man-pages/man8/netstat.8.html)、[ssのマニュアル](https://man7.org/linux/man-pages/man8/ss.8.html)）。

**調査の限界：** `netstat`や`ss`が示すのは、実行した端末から見たその時点の状態です。`LISTEN`があっても、別のPCから到達できるとは限りません。待ち受け先のIPアドレス、ファイアウォール、ルーター、クラウドの通信ルールなども確認します。また、`ESTABLISHED`だけでアプリの処理が成功したとは判断できません。アプリのログや実際の接続テストも合わせて見ます。
