# 『おうちで学べるセキュリティの基本』2026-10-5 疑問3：カフェWi-Fiのクライアント分離

## 疑問

> カフェのWi-Fiでファイル共有をONにしていたら、ほかの客に丸見えになる？

## 回答

**共有がONで同じWi-Fi名に接続していても、すべてのファイルが無条件に見えるわけではありません。** カフェ側が端末間通信を遮断しているか、PC側が接続を許すか、どのフォルダを誰に共有しているかで結果が変わります。ファイル共有の仕組みと悪用例は[疑問1〜3：PCのファイル共有とSMB](./basic_security_at_home_20261005_question1_2_3_file_sharing_smb.md)にまとめています（[Microsoft：共有フォルダの設定](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)、[Cisco Meraki：クライアント分離](https://documentation.meraki.com/Wireless/Operate_and_Maintain/How_Tos/Firewall_and_Traffic_Shaping/Wireless_Client_Isolation)）。

### カフェのWi-Fiでは、ほかの客から見える？

**カフェの設定次第です。** 同じSSID（画面に出るWi-Fi名）に接続していても、アクセスポイントが利用者同士の通信を遮断する **クライアント分離** を有効にしていれば、通常、その経路ではほかの客のPCへ直接接続できません。分離されていない場合は、共有元PCのファイアウォールと共有権限によっては、別の客から共有サービスへ接続できる可能性があります。Ciscoは、公衆・ゲスト向けWi-Fiで端末間通信を遮断する機能を説明しています（[Cisco Meraki：クライアント分離](https://documentation.meraki.com/Wireless/Operate_and_Maintain/How_Tos/Firewall_and_Traffic_Shaping/Wireless_Client_Isolation)、[Cisco：ゲストWi-Fiの端末間通信](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3850/software/release/16-1/converged_access_deployement_guide/m_conAccess_deploy_guide/converged_access_wlan_configuration.html)）。

```text
カフェの客AのPC ─→ アクセスポイント ─→ カフェの客BのPC
                       │
                       ├─ クライアント分離ON：端末間通信を遮断
                       └─ クライアント分離OFF：通信できる可能性あり
                                                   ↓
                                  Bのファイアウォール・共有権限も必要
```

| カフェ側・PC側の状態 | ほかの客が共有フォルダを読めるか |
| --- | --- |
| クライアント分離が有効 | 通常、客同士の直接のファイル共有接続はできない |
| 分離なし、PCへの通信は可能、共有フォルダがあるが権限なし | 通信できても、フォルダの中身は読めない |
| 分離なし、PCへの通信も可能、広く閲覧を許す共有フォルダがある | その共有フォルダの内容を読まれる可能性がある |

**同じSSIDかどうかだけでは、分離の有無や接続できる範囲は判断できません。** また、クライアント分離の実装とネットワーク構成によっては、別の経路や例外があり得ます。「Wi-Fiが暗号化されているか」と「客同士が通信できるか」は別の設定です。暗号化については[2026-10-3 疑問4：公衆Wi-Fiの暗号化](./basic_security_at_home_20261003_question4_public_wifi.md)を参照してください（[Cisco Meraki：分離機能と通信範囲](https://documentation.meraki.com/Wireless/Operate_and_Maintain/How_Tos/Firewall_and_Traffic_Shaping/Wireless_Client_Isolation)）。

### カフェで使うときの実践

1. **Windowsでは接続中のWi-Fiを「パブリック」にする。** 通常、この設定ではPCをほかの端末から見えにくくし、ファイルとプリンターの共有を利用できないようにします。共有が必要な自宅など、信頼できるネットワークだけで「プライベート」を使います。ファイアウォールも有効か確認します（[Microsoft：パブリックとプライベートの違い](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/essential-network-settings-and-tasks-in-windows)、[Microsoft：ファイアウォール](https://support.microsoft.com/en-us/windows/security/windows-security/firewall-and-network-protection-in-the-windows-security-app)）。
2. **Macでは不要な「ファイル共有」をOFFにする。** 必要なときも、共有フォルダとゲスト・利用者権限を確認します（[Apple：ファイル共有の停止と権限](https://support.apple.com/ja-jp/guide/mac-help/mh17131/27/mac/27)）。
3. **ほかの客と通信できる設定か分からなくても、PC側で不要な共有を止める。** 端末間分離はカフェ側の設定なので、利用者が接続した画面だけで確実には確認できません。共有が必要なときも、共有するフォルダと相手の権限を絞ります（[Cisco Meraki：分離の設定](https://documentation.meraki.com/Wireless/Operate_and_Maintain/How_Tos/Firewall_and_Traffic_Shaping/Wireless_Client_Isolation)、[Microsoft：Windowsの共有設定](https://support.microsoft.com/en-us/windows/experience/connectivity-networking/file-sharing-over-a-network-in-windows)）。

**判断例：** カフェでファイル共有がONでも、端末間通信が遮断され、共有元のファイアウォールも受信を止めていれば、ほかの客がそのWi-Fi経由で共有フォルダを開くことは通常できません。反対に分離がなく、共有元が接続を許可し、誰でも読める共有フォルダがあれば、その中身は見られ得ます。カフェの設備設定は利用者から分かりにくいため、PC側のパブリック設定・共有範囲・権限を自分で確認するのが確実です。
