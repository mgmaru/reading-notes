# 『おうちで学べるセキュリティの基本』2026-10-3 疑問6：PCの遠隔操作に使うプロトコル

## 疑問

> PCの遠隔操作の主要プロトコルは何か？ Telnet？ もしくはSSH？

## 回答

**操作のしかたで選ぶプロトコルが変わります。** サーバーにログインしてコマンドを実行するなら **SSH**、Windowsなどの画面を見ながら操作するなら **RDP** や **VNC（通信規格はRFB）** が代表例です。Telnetも文字ベースの遠隔操作に使えますが、通常のTelnetは通信を保護しないため、現在の管理用途ではSSHを選ぶのが基本です。攻撃者による遠隔操作は、これらの管理用プロトコルだけを使うとは限りません。遠隔操作型マルウェアについては[疑問7：遠隔操作型マルウェアの仕組み](./basic_security_at_home_20261003_question7_remote_access_malware.md)で説明します（[RFC 4251：SSHの構成](https://www.rfc-editor.org/rfc/rfc4251.html)、[RFC 2946：Telnetの通信保護](https://www.rfc-editor.org/rfc/rfc2946.html)、[MITRE ATT&CK：Web通信を使う指令](https://attack.mitre.org/techniques/T1071/001/)）。

| 方式 | 主な操作 | 一般的な待受ポート | 通信保護と注意点 |
| --- | --- | --- | --- |
| **SSH** | 端末にログインしてシェルやコマンドを実行する。ファイル転送などにも使う | TCP 22 | 通信を暗号化し、サーバーと利用者を認証する。認証設定や接続先の確認も必要 |
| **RDP** | 主にWindowsのデスクトップ画面を見て、マウス・キーボードで操作する | TCP/UDP 3389 | 画面・入力を遠隔転送する。利用者認証と安全な公開範囲の設定が必要 |
| **VNC / RFB** | 他のPCの画面を見て操作する | 通常 TCP 5900 | 製品・設定により認証や暗号化方式が異なる。通信保護を設定する |
| **Telnet** | 文字ベースの端末操作 | TCP 23 | 通常の利用ではパスワードや操作内容が平文で流れるため、管理用途では避ける |

ポートは**標準的な値**で、設定により変更できます。SSH/Telnetのポートは[IANAの登録表](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)、RDPは[Microsoftのポート一覧](https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/ports-used-by-rds)、VNC/RFBは[RFC 6143](https://www.rfc-editor.org/rfc/rfc6143.html)を参照してください。RDPが画面と入力を転送する仕組みは[MicrosoftのRDP解説](https://learn.microsoft.com/en-us/windows/win32/termserv/remote-desktop-protocol)、VNCとRFBの関係は[RFC 6143](https://www.rfc-editor.org/rfc/rfc6143.html)に説明があります。

```text
コマンド操作：手元の端末 ── SSH ──→ 接続先のシェル ──→ コマンド実行・結果を返す
画面操作　：手元の端末 ←─ RDP / VNC ─→ 接続先の画面・キーボード・マウス
```

たとえばLinuxサーバーでログを確認するときはSSHが自然です。自分のWindows PCのデスクトップを遠隔で操作したいときはRDPが候補です。**SSHは「画面を操作するための唯一のプロトコル」ではなく、RDPも「乗っ取り専用の仕組み」ではありません。** どちらも許可された管理作業に使える技術で、攻撃者が資格情報を盗んだ場合には悪用もされます（[RFC 4254：SSHのコマンド実行](https://www.rfc-editor.org/rfc/rfc4254.html)、[Microsoft：リモートデスクトップの利用条件](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/remote-desktop-allow-access)、[MITRE ATT&CK：正規の遠隔操作ツールの悪用](https://attack.mitre.org/techniques/T1219/)）。

実務では、接続を許可する利用者を絞り、強い認証を設定し、不要な遠隔操作サービスは無効にします。RDPなどをインターネットへ直接公開しないことも重要です（[CISA：ランサムウェア対策ガイド](https://www.cisa.gov/stopransomware/ransomware-guide)、[Microsoft：リモートデスクトップの設定](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/remote-desktop-allow-access)）。
