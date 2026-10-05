# 『おうちで学べるセキュリティの基本』2026-10-5 疑問1〜3：PCのファイル共有とSMB

## 疑問

> 1. ファイルの共有設定はルーターではなく、パソコンで設定するもの？
> 2. ファイル共有をONにすると、同じネットワーク内のPCから見られてしまう？ どのように見られる？
> 3. ファイル共有を悪用した攻撃はある？ 攻撃者がSSHなどで侵入し、共有をONにして覗き見ることはある？

## 回答

**PCに保存されたファイルを共有するなら、基本的にはそのPCで設定します。共有をONにしただけで、PC内の文書や写真がすべて誰にでも見えるわけではありません。** 相手の端末から共有元へ通信でき、対象のフォルダが共有され、アクセス権もある場合に読めます。WindowsやMacのネットワークファイル共有では、主に **SMB（Server Message Block）** という通信方式を使います（[Microsoft：Windowsのネットワークファイル共有](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Apple：Macのファイル共有](https://support.apple.com/ja-jp/guide/mac-help/mh17131/27/mac/27)）。

```text
別のPC ──①到達できるか──→ 共有元PCのファイアウォール
                                  │
                             ②共有サービスが動いているか
                                  │
                             ③対象フォルダを共有しているか
                                  │
                             ④その利用者に読み取り権限があるか
                                  ↓
                          条件を満たす場合にファイルを読める
```

ここでいう「共有元PC」は、ファイルを保存し、ほかの端末からの接続を受けるPCです。家庭用ルーターやアクセスポイントはネットワークへの接続や転送を担いますが、**PC内のどのフォルダを誰に見せるかは、PC側で指定します。** ただし、USBストレージを接続してルーター自身をファイルサーバーとして使う製品もあります。その場合はルーター側で保存領域とアクセス権を設定します（[NETGEAR：ルーターに接続したUSBストレージの共有](https://kb.netgear.com/20088/ReadySHARE-Access)、[NETGEAR：共有の読み書き権限](https://kb.netgear.com/000038775/How-do-I-change-read-and-write-access-for-a-USB-drive-connected-to-my-router-or-gateway)）。

### 「共有をON」の意味はOSによって少し違う

| 設定する場所 | 設定するもの | ほかの端末に見せる範囲 |
| --- | --- | --- |
| **WindowsのPC** | ネットワークの種類、ネットワーク探索、ファイルとプリンターの共有、個々の共有フォルダと権限 | 指定した共有フォルダ。共有を有効にする設定だけで、全ファイルを無条件に公開するわけではない |
| **Mac** | 「システム設定 → 一般 → 共有 → ファイル共有」、共有フォルダ、利用者ごとの権限 | 共有リストと利用者の権限で決まる。ファイル共有を有効にすると、利用者の「パブリック」フォルダが共有対象になる点にも注意する |
| **ファイル共有機能付きルーター・NAS** | 機器自身の共有機能、保存領域、アカウントと権限 | その機器に保存したファイル。PCのファイル共有設定とは別 |

Windowsでは、エクスプローラーで対象フォルダを右クリックし、「アクセスを許可する」から共有相手を選びます。共有元のファイアウォールも接続を許可している必要があります。Macでは共有フォルダと利用者ごとに「読み/書き」「読み出しのみ」「アクセス不可」などを選びます。Macの「パブリック」フォルダや管理者アカウントの扱いは設定によって影響が大きいため、共有を有効にしたら一覧と権限を確認します（[Microsoft：共有フォルダの設定と確認](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Microsoft：SMBの通信制御](https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-secure-traffic)、[Apple：共有フォルダと利用者権限](https://support.apple.com/ja-jp/guide/mac-help/mh17131/27/mac/27)）。

### ほかのPCからはどうやって見る？

同じ家庭内LANなどで**端末間通信が許可されている**とき、たとえばWindowsのエクスプローラーから `\\共有元PC名\共有名` を開きます。MacならFinderの「ネットワーク」から共有元を探すか、「サーバへ接続」に `smb://共有元PC名/共有名` を入力します。必要なら共有元PCのユーザー名とパスワードを求められ、その利用者に許可された範囲のファイルが表示されます（[Microsoft：ネットワークドライブと共有フォルダ](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Apple：共有コンピュータへの接続](https://support.apple.com/guide/mac-help/connect-mac-shared-computers-servers-mchlp1140/mac)）。

| 状態 | 別のPCからの結果 |
| --- | --- |
| フォルダを共有し、通信でき、閲覧権限もある | 指定した共有フォルダの内容を読める |
| フォルダを共有していない | 通常の文書・写真は、共有機能をONにしただけでは読めない |
| フォルダを共有しているが、相手に権限がない | 接続を拒否されるか、対象を開けない |
| 共有と権限はあるが、ファイアウォールやWi-Fiの端末間分離で通信できない | 共有元へ接続できない |

**「ネットワーク一覧にPCが表示されること」と「共有フォルダに入れること」は別です。** Windowsのネットワーク探索は、共有元を一覧で見つけやすくする機能です。探索をOFFにしても、共有サービスへの通信が別に許可されていれば、アドレスを指定した接続まで遮断したことにはなりません。実際のアクセスは通信制御、共有設定、権限で判断します。また、Windowsのエクスプローラーで自分自身を `\\localhost` から見ると、自分のアカウントだから見えるフォルダも表示されます。そこに見えるものが全部、ほかの人に公開されているわけではありません（[Microsoft：ネットワーク探索と `\\localhost` の注意点](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Microsoft：SMBの通信制御](https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-secure-traffic)、[Apple：アドレスを指定した接続](https://support.apple.com/guide/mac-help/connect-mac-shared-computers-servers-mchlp1140/mac)）。

**「同じネットワーク」も十分条件ではありません。** Wi-Fiのアクセスポイントが利用者同士の通信を遮断していることもあります。カフェWi-Fiの場合は[疑問3：Wi-Fiのクライアント分離](./basic_security_at_home_20261005_question3_wifi_client_isolation.md)で詳しく説明します。

### 自分のPCで確認する手順

1. **Windows：** 「設定 → ネットワークとインターネット」で、接続中のネットワークが「パブリック」か「プライベート」かを確認します。知らない人が利用するネットワークでは「パブリック」にします。Windowsは通常、パブリック設定ではほかの端末から見えにくくし、ファイルとプリンターの共有を使えないようにします（[Microsoft：ネットワークの種類](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/essential-network-settings-and-tasks-in-windows)）。
2. **Windows：** 「設定 → ネットワークとインターネット → 詳細ネットワーク設定 → 共有の詳細設定」と、エクスプローラーで共有しているフォルダと共有相手を確認します。PowerShellの `Get-SmbShare` でもSMB共有を一覧できますが、`C$` など管理用の既定共有も表示されるため、一覧の件数だけで一般利用者への公開状態を判断しません（[Microsoft：共有の確認](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Microsoft：Get-SmbShare](https://learn.microsoft.com/en-us/powershell/module/smbshare/get-smbshare)）。
3. **Mac：** 「システム設定 → 一般 → 共有 → ファイル共有」でON/OFF、共有フォルダ、利用者ごとの権限、ゲストアクセスを確認します。必要がなければファイル共有をOFFにします（[Apple：ファイル共有の設定](https://support.apple.com/ja-jp/guide/mac-help/mh17131/27/mac/27)）。

共有が必要な場合は、**共有するフォルダと利用者を必要最小限にし、書き込み権限は必要な人だけに付けます。** Windowsで「パスワード保護共有」を無効にしたり、Macでゲストを許可したりすると、意図より広くアクセスされ得ます。共有の有無だけでなく、**誰が読めるか・変更できるか**まで確認してください（[Microsoft：パスワード保護共有](https://learn.microsoft.com/en-us/troubleshoot/windows-client/networking/set-up-your-small-business-network)、[Apple：ゲストと権限](https://support.apple.com/ja-jp/guide/mac-help/mh17131/27/mac/27)）。

### 疑問3：ファイル共有はどう悪用される？

**悪用例はあります。** ただし、共有がONであることと、ほかの人が全ファイルを読めることは同じではありません。攻撃者からの到達経路、共有フォルダの設定、認証・権限、共有サービスの脆弱性によって結果が変わります。

| 状況 | 起こり得ること | 防ぐために確認すること |
| --- | --- | --- |
| 広く読み取りを許す共有フォルダを作ってしまった | 到達できる相手に、フォルダ内の文書を読まれる | 共有対象、ゲストアクセス、利用者権限を確認する |
| 広い書き込み権限を付け、利用者のPCや認証情報が侵害された | 共有先のファイルを改ざん・削除される。ランサムウェアがアクセス可能な共有ドライブを暗号化することもある | 書き込み権限を絞り、バックアップと端末の保護を行う |
| 更新されていない共有サービスの欠陥があり、攻撃者から通信できる | 共有フォルダへの正規のアクセス権がなくても、欠陥自体を攻撃され得る | OSを更新し、不要な共有サービスや到達経路を閉じる |

共有ドライブへのランサムウェアの被害は、米国CISAも注意を促しています。過去には **SMBv1 の脆弱性 MS17-010** のように、細工された通信だけでリモートコード実行につながる欠陥もありました。これは「共有をONにすると必ず侵入される」という意味ではなく、**脆弱なサービスへ攻撃者が到達できる場合の例**です。MicrosoftはインターネットからのSMB接続を遮断することや、不要な端末間のSMB通信を制限することを推奨しています（[CISA：ランサムウェアと共有ドライブ](https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-291a)、[Microsoft：MS17-010](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)、[Microsoft：SMB通信の保護](https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-secure-traffic)）。

**SSHなどを通じて攻撃者が既にPC上でコマンドを実行できるなら、共有をONにする必要は必ずしもありません。** そのアカウントに読み取り権限があるファイルなら、攻撃者は侵入済みの経路から扱える可能性があります。一方、読めないファイルが共有をONにするだけで自動的に読めるようになるわけではありません。共有設定やファイアウォールを変更できるかも、得たアカウントの権限とOSの設定に左右されます。Windowsの共有設定では管理者の承認を求められる操作もあります。したがって、**「SSHで入れた」ことと「全ファイルを読める・共有設定を自由に変えられる」ことは別**です（[Microsoft：ファイルへのアクセス権](https://learn.microsoft.com/en-us/windows/win32/fileio/file-security-and-access-rights)、[Microsoft：ファイル共有の設定](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Apple：共有ユーザーの権限](https://support.apple.com/ja-jp/guide/mac-help/mh17131/27/mac/27)）。

共有サービスの古い方式にも注意します。とくに **SMBv1** は既知の問題が多く、Microsoftは使用しないことを強く勧めています。OSを更新し、不要な共有を止め、必要な場合も接続元と権限を絞ります（[Microsoft：SMBv1の扱い](https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3)、[Microsoft：不要なSMB接続の制限](https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-secure-traffic)）。

## 追加疑問（2026-10-6）：どの設定がそろうとファイルを見られる？

> LAN側の端末間の共有設定とPCのファイアウォールと共有設定次第では、PCのファイルを見ることができるという認識で良い？

**はい、概ねその認識で合っています。** 「LAN側の端末間の共有設定」は、正確には**端末同士の通信が許可されているか**という意味です。ルーターやWi-Fiのアクセスポイントが通信を遮断していれば、相手のPCへ接続できません。通信が許可されていても、さらに共有元PC側の条件を満たす必要があります。

| 確認する場所 | 必要な状態 | 満たさない場合 |
| --- | --- | --- |
| **LAN・Wi-Fi側** | 端末同士が通信できる。Wi-Fiのクライアント分離などで遮断されていない | 共有元PCまで接続できない |
| **共有元PCのファイアウォール** | ファイル共有への受信通信を許可している | ファイル共有への接続が遮断される |
| **共有元PCの共有設定と権限** | 共有サービスが有効で、対象フォルダが共有され、接続する人に読み取り権限がある | 対象の共有フォルダを開けない |

つまり、**「端末間で通信できる」→「PCが接続を受ける」→「共有フォルダへの閲覧を許す」** の順に確認します。すべてそろっても、見られるのは許可された共有フォルダ内のファイルです。認証が必要な共有なら、許可されたアカウントでのログインも必要です。PC内の全ファイルが自動で見えるわけではありません（[Microsoft：ファイル共有の設定](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Apple：共有フォルダと利用者の権限](https://support.apple.com/ja-jp/guide/mac-help/mh17131/27/mac/27)）。

カフェWi-Fiでの端末間通信は、[疑問3：Wi-Fiのクライアント分離](./basic_security_at_home_20261005_question3_wifi_client_isolation.md)も参照してください。なお、Windowsの「ネットワーク探索」は共有元を一覧で見つけやすくする設定です。探索をOFFにするだけでは、アドレスを直接指定した接続まで必ず遮断されるとは限りません（[Cisco Meraki：クライアント分離](https://documentation.meraki.com/Wireless/Operate_and_Maintain/How_Tos/Firewall_and_Traffic_Shaping/Wireless_Client_Isolation)、[Microsoft：SMB通信の制御](https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-secure-traffic)）。
