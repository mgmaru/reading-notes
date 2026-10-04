# 『おうちで学べるセキュリティの基本』2026-10-3 疑問5：動作中のアプリを調べるDASTと手動テスト

## 疑問

> ソースコードを見なくても、ツールによってセキュリティホールを見つけられるか？

## 回答

**見つけられるものがあります。** 動作中のWebアプリに外側からリクエストを送り、応答を調べる方法を**DAST（動的アプリケーションセキュリティテスト）**と呼びます。通常、ソースコードは不要です。たとえばSQLインジェクション、XSS、危険な設定など、ツールが試せる種類の問題を探します。ただし、アプリ固有の権限や業務ルールは、想定される正しい動作を人が定義し、利用者や操作の条件を変えて試す必要があります（[OWASP：動的検査ツール](https://community.owasp.org/Vulnerability_Scanning_Tools)、[OWASP：ビジネスロジックの検査](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/10-Business_Logic_Testing/)）。

```text
テスト環境のWebアプリ
    ↑ テスト用リクエストを送る
DASTツール（例：ZAP） ──→ 応答や設定から、既知の弱点を探す

利用者Aのアカウント ──→ Aの注文を取得できるか
利用者Bのアカウント ──→ Aの注文を取得できないか ← 仕様を知る人が確認
```

| 検査 | ソースコード | 得意なこと | 見落としやすいこと |
| --- | --- | --- | --- |
| **DASTの自動スキャン** | 通常不要 | 到達できた画面やAPIに対する既知の攻撃パターン、公開設定の問題を探す | 到達できない画面、必要なログイン状態、固有の業務ルール |
| **権限・業務ルールの手動テスト** | 必須ではない | 「他人の注文を取得できない」「決済前に発送処理できない」などの仕様を確かめる | 試していない条件や経路。設計・コード内部の原因の特定には追加調査が必要 |
| **コードのSAST・レビュー** | 必要 | 実行せずに危険な処理経路を探し、権限判定の実装を読む | 本番相当の設定・通信や画面での挙動 |

ソースを読めなくても、**「本来は拒否されるべき要求が通る」**ことはテストで発見できます。しかし、どの要求を拒否すべきかは仕様を知らないツールには分かりません。OWASPは、ツールによる自動化を共通の弱点の検査に使い、業務ロジックは人による確認や、仕様に基づいて作った自動テストで補うことを勧めています（[OWASP：ビジネスロジックのセキュリティ](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html)、[OWASP：Web Security Testing Guide](https://wstg.owasp.org/v4.2/2-Introduction/)）。

### 実務での手順

1. **対象と期待する動作を決める。** URL・API、ログイン後の画面、利用者の権限、守るべきデータを洗い出す。権限ごとのテストアカウントも用意する。
2. **検証環境でDASTを実行する。** たとえばZAPはWebアプリの自動スキャンに使える。ログインが必要な箇所や到達できないAPIは、対象に含める設定や別のテストが必要（[ZAP：Quick Start](https://www.zaproxy.org/docs/desktop/addons/quick-start/)）。
3. **仕様に沿った拒否テストを作る。** 注文の例なら、Aで作った注文IDをBで指定し、`403` や `404` など設計した拒否結果になり、注文内容が返らないことを確認する。手順の飛ばし・繰り返し、金額や件数の境界値も検査する（[OWASP：ビジネスロジックのセキュリティ](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html)）。
4. **発見した問題を再現し、修正後に再検査する。** 影響を受ける処理と権限を確認し、必要ならソースコードや設定を調べて原因を直す。該当する拒否テストを継続的に実行する。

DASTの**アクティブスキャンは実際に攻撃に似たリクエストを送る**ため、管理している検証環境で実施し、データ変更や負荷を考慮します（[ZAP：Active Scan](https://www.zaproxy.org/docs/desktop/start/features/ascan/)）。ここではWebアプリを例にしました。サーバーやクラウドの設定、不要な公開、古いOS・ミドルウェアなども対象に含めるなら、構成・更新状況・実際の公開範囲を別途調べます（[OWASP：アプリケーション基盤の設定検査](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/02-Test_Application_Platform_Configuration/)）。SASTの仕組みとレビュー方法は[疑問5：ソースコードの解析とSAST](./basic_security_at_home_20261003_question5_source_code_review_sast.md)、依存ライブラリの既知の脆弱性を探すSCAは[疑問3：ライブラリの脆弱性を見つける方法](./basic_security_at_home_20261003_question3_dependency_vulnerability_management.md)を参照してください。
