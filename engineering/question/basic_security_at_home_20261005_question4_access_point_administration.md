# 『おうちで学べるセキュリティの基本』2026-10-5 疑問4：アクセスポイントと管理者

## 疑問

> アクセスポイントとは何か？ アクセスポイントの管理者はルーターの管理者なのか？ ルーターの購入者なのか、それともプロバイダなのか？

## 回答

**アクセスポイント（AP）は、スマートフォンやPCをWi-Fiでネットワークにつなぐ機能、またはその機器です。** 一方、ルーターの主な役割は異なるネットワーク間の通信を転送することです。家庭用の「Wi-Fiルーター」は両方の機能を一台にまとめた製品が多いため、言葉が混同されがちです（[Cisco：アクセスポイントとは](https://www.cisco.com/site/us/en/learn/topics/small-business/what-is-an-access-point.html)、[Cisco：Wi-Fiルーターとは](https://www.cisco.com/site/us/en/learn/topics/networking/what-is-a-wireless-router-wi-fi.html)）。

```text
【一台にまとまった家庭用Wi-Fiルーター】
インターネット ── [Wi-Fiルーター：ルーター機能＋AP機能] ))) PC・スマホ

【ルーターとAPが別々】
インターネット ── [ルーター] ── [AP] ))) PC・スマホ
                                    ↑ 無線でLANに接続する入口
```

上の `)))` はWi-Fiの電波を表します。独立したAPは、通常、有線LANと無線LANをつなぐ役割を担います。家庭用Wi-Fiルーターを**APモード／ブリッジモード**で使うと、ルーター機能を停止して、上流のルーターに無線接続を追加できます。製品によって使える機能は異なるため、動作モードを確認します（[Cisco：APを既存の有線LANに追加する例](https://www.cisco.com/c/en/us/support/docs/smb/wireless/cisco-small-business-100-series-wireless-access-points/smb5531-add-a-wireless-network-to-an-existing-wired-network-using-a.html)、[バッファロー：ルーターモードとAPモード](https://www.buffalo.jp/support/faq/detail/16000.html)）。

### 「管理者」は誰？

ここでいう**管理者は、機器やWi-Fiの設定を変更する権限を持つ人・組織**です。機器を買った人という意味ではありません。実際に誰が管理するかは、機器の構成とサービス契約で決まります。

| 構成例 | APの設定を管理する人・場所 | ルーターの管理者との関係 |
| --- | --- | --- |
| 自分で購入・設置した家庭用Wi-Fiルーター | 通常は家のネットワークを設定する本人。製品の管理画面で設定する | 一台の管理画面でルーター機能とAP機能を設定できることが多い |
| 回線事業者などから提供されたルーター＋自分で設置したAP | APは本人、上流ルーターは契約・機器の条件に従う | 管理対象が別の機器なので、管理者も別になり得る |
| 店舗・会社のWi-Fi | 店舗・会社の担当者、委託先、契約した管理サービスなど | 複数APを専用の管理装置からまとめて設定することもある |

例えば、NTT東日本の法人向けWi-Fiサービスでは、事業者がAPを事前設定し、遠隔でサポートするプランがあります。反対に、自分で買った家庭用機器なら、自分で管理パスワードを変更することもできます。したがって、**「購入者＝管理者」「プロバイダ＝管理者」はどちらも一般則ではありません。** 購入・レンタル、契約内容、管理アカウントの権限を確認してください（[NTT東日本：ギガらくWi-Fiのサポート](https://business.ntt-east.co.jp/service/gigarakuwifi/support.html)、[バッファロー：管理パスワードの変更](https://www.buffalo.jp/support/faq/detail/16077.html)、[Cisco：APの集中管理](https://www.cisco.com/c/en/us/td/docs/wireless/access_point/csbap/CBW_WiFi_6/Admin_Guide/b_cisco_business_wifi_6_admin_guide/m_5_management.html)）。

APの管理権限があれば、そのAPのSSIDや接続方式、利用者向けの設定などを変更できます。ただし、**独立したAPの管理権限だけで上流ルーターや接続先PCまで管理できるわけではありません。** 一台のWi-Fiルーターに両機能が入っている場合は、その機器の管理アカウントに許された範囲を設定できます。管理権限の範囲は製品とアカウントの権限設定によります（[Cisco：AP管理画面と権限](https://www.cisco.com/c/en/us/td/docs/wireless/access_point/csbap/CBW_WiFi_6/Admin_Guide/b_cisco_business_wifi_6_admin_guide/m_5_management.html)、[バッファロー：Wi-Fiルーターの設定画面](https://www.buffalo.jp/support/faq/detail/129.html)）。

### Wi-Fiに接続するパスワードと管理パスワード

| 認証情報 | 何ができる？ | 誰に渡す？ |
| --- | --- | --- |
| **Wi-Fiのパスワード** | 許可されたWi-Fiネットワークに端末を接続する | 接続を許す利用者 |
| **機器の管理パスワード** | SSID、Wi-Fiのパスワード、通信や管理の設定などを変更する | 設定担当者だけ |

**Wi-Fiにつなげることと、設定画面に管理者として入れることは別です。** バッファローも両者のパスワードが異なると説明しています。機器によって初期ユーザー名・パスワードは違い、初期値をそのまま使うと第三者に設定を変えられる危険があります。自分が管理する機器では、管理パスワードを固有で強いものに変更し、ファームウェアも更新します。事業者が管理する機器なら、その契約の設定・更新手順に従います（[バッファロー：設定画面のパスワード](https://www.buffalo.jp/support/faq/detail/129.html)、[米国FTC：家庭のWi-Fiを守る方法](https://consumer.ftc.gov/articles/how-secure-your-home-wi-fi-network)）。

**自宅で確認する順番：** ①Wi-Fiにつながっている機器の型番を見る → ②その機器がルーターモードかAPモードか確認する → ③上流に別のルーターがあるか確認する → ④それぞれの機器の管理者と更新担当を確認する。バッファローは設定画面から動作モードを確認する方法を案内しています（[バッファロー：動作モードの確認](https://www.buffalo.jp/support/faq/detail/15989.html)）。
