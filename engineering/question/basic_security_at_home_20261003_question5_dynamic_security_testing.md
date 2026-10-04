# 『おうちで学べるセキュリティの基本』2026-10-3 疑問5：動作中のアプリを調べるDASTと自動テスト

## 疑問

> ソースコードを見なくても、ツールによってセキュリティホールを見つけられるか？

## 回答

**見つけられるものがあります。** 動作中のWebアプリに外側からリクエストを送り、応答を調べる方法を**DAST（動的アプリケーションセキュリティテスト）**と呼びます。通常、ソースコードは不要です。たとえばSQLインジェクション、XSS、危険な設定など、ツールが試せる種類の問題を探します。ただし、アプリ固有の権限や業務ルールは、想定される正しい動作を人が定義してテストする必要があります。そのテストの繰り返し実行は自動化できます（[OWASP：動的検査ツール](https://community.owasp.org/Vulnerability_Scanning_Tools)、[OWASP：ビジネスロジックの検査](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/10-Business_Logic_Testing/)）。

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
| **権限・業務ルールの手動・自動テスト** | 必須ではない | 「他人の注文を取得できない」「決済前に発送処理できない」などの仕様を確かめる | 試していない条件や経路。設計・コード内部の原因の特定には追加調査が必要 |
| **コードのSAST・レビュー** | 必要 | 実行せずに危険な処理経路を探し、権限判定の実装を読む | 本番相当の設定・通信や画面での挙動 |

ソースを読めなくても、**「本来は拒否されるべき要求が通る」**ことはテストで発見できます。しかし、どの要求を拒否すべきかは仕様を知らないツールには分かりません。OWASPは、ツールによる自動化を共通の弱点の検査に使い、業務ロジックは人による確認や、仕様に基づいて作った自動テストで補うことを勧めています（[OWASP：ビジネスロジックのセキュリティ](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html)、[OWASP：Web Security Testing Guide](https://wstg.owasp.org/v4.2/2-Introduction/)）。

### 実務での手順

1. **対象と期待する動作を決める。** URL・API、ログイン後の画面、利用者の権限、守るべきデータを洗い出す。権限ごとのテストアカウントも用意する。
2. **検証環境でDASTを実行する。** たとえばZAPはWebアプリの自動スキャンに使える。ログインが必要な箇所や到達できないAPIは、対象に含める設定や別のテストが必要（[ZAP：Quick Start](https://www.zaproxy.org/docs/desktop/addons/quick-start/)）。
3. **仕様に沿った拒否テストを作る。** 注文の例なら、Aで作った注文IDをBで指定し、`403` や `404` など設計した拒否結果になり、注文内容が返らないことを確認する。手順の飛ばし・繰り返し、金額や件数の境界値も検査する（[OWASP：ビジネスロジックのセキュリティ](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html)）。
4. **発見した問題を再現し、修正後に再検査する。** 影響を受ける処理と権限を確認し、必要ならソースコードや設定を調べて原因を直す。該当する拒否テストを継続的に実行する。

DASTの**アクティブスキャンは実際に攻撃に似たリクエストを送る**ため、管理している検証環境で実施し、データ変更や負荷を考慮します（[ZAP：Active Scan](https://www.zaproxy.org/docs/desktop/start/features/ascan/)）。ここではWebアプリを例にしました。サーバーやクラウドの設定、不要な公開、古いOS・ミドルウェアなども対象に含めるなら、構成・更新状況・実際の公開範囲を別途調べます（[OWASP：アプリケーション基盤の設定検査](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/02-Test_Application_Platform_Configuration/)）。SASTの仕組みとレビュー方法は[疑問5：ソースコードの解析とSAST](./basic_security_at_home_20261003_question5_source_code_review_sast.md)、依存ライブラリの既知の脆弱性を探すSCAは[疑問3：ライブラリの脆弱性を見つける方法](./basic_security_at_home_20261003_question3_dependency_vulnerability_management.md)を参照してください。

## 追加疑問（2026-10-4）：DASTを自動化する方法

> DASTは自動化できるか？ 人がテスト観点を決め、E2Eテストで繰り返し実行する方法のほかには何があるか？

**できます。** 自動化は大きく分けて、①ツールが共通の弱点を探す**自動スキャン**、②人が期待結果を決める**セキュリティ回帰テスト**、③多様な入力を生成する**ファジング**があります。いずれもCIや定期ジョブで繰り返せます。ただし、**何を対象に含め、何を「正しい結果」とするか**は人が決めます。

| 方法 | 具体例 | 見つけやすいこと・限界 |
| --- | --- | --- |
| **Webの自動スキャン** | ZAPのBaseline ScanをCIで実行する。アクティブスキャンを含むFull Scanは検証環境で実行する | 到達できた画面の既知の弱点や設定を調べる。独自の権限ルールは分からない |
| **API仕様からのスキャン** | OpenAPI/GraphQLの定義をZAP API Scanに読み込ませる | 画面にリンクがないAPIも対象に含めやすい。仕様にないAPIや業務ルールは別途確認する |
| **仕様に基づく自動テスト** | PlaywrightなどのE2E・APIテストで、権限の異なる利用者や操作順序を変えて試す | 自社の権限・業務ルールを正確に確認できるが、**書いたケースの範囲**だけを検査する |
| **ファジング・プロパティベーステスト** | SchemathesisなどでOpenAPIの型・境界から多様な入力を生成する | 予期しない入力や状態の不具合を探す。仕様に書かれていない「誰に何を許すか」は自動では分からない |

ZAPのDocker版には、通信を巡回して**受け取った内容を受動的に分析するBaseline Scan**、攻撃用リクエストも送るFull Scan、OpenAPI/GraphQL向けAPI Scanがあります。Baseline Scanもページを巡回するリクエストは送りますが、攻撃用のペイロードは送らない方式です。Full ScanとAPI Scanのアクティブ検査は、データ変更や負荷が起き得ることを考えて、管理する検証環境で実施します（[ZAP：Docker版の3種類のスキャン](https://www.zaproxy.org/docs/docker/about/)、[ZAP：Baseline Scan](https://www.zaproxy.org/docs/docker/baseline-scan/)）。認証が必要な画面ではテスト用アカウントとログイン方法を設定し、**実際に巡回・検査できた範囲**を確認します。

自分が管理する検証サイトでBaseline Scanを試すコマンドの例です。`staging.example.com` は実際の検証サイトに置き換えます。CIでも同じスキャンをジョブとして起動できます（[ZAP：Baseline Scanの実行例](https://www.zaproxy.org/docs/docker/baseline-scan/)）。

```bash
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t https://staging.example.com
```

### 人が決めた権限ルールを自動テストにする例

先ほどの注文APIなら、「所有者Aは自分の注文を読める」「利用者BはAの注文を読めない」を別々に確認します。UIでボタンが隠れることだけを見ると、APIを直接呼んだ場合の権限漏れを見逃します。Playwrightはブラウザ操作に加えてAPIリクエストも送れ、複数のログイン状態を使い分けられます（[Playwright：APIテスト](https://playwright.dev/docs/api-testing)、[Playwright：複数の役割での認証](https://playwright.dev/docs/auth)）。

```text
事前準備：テスト環境に「Aが所有する注文123」を作成し、A・Bのログイン状態を用意する
1. Aで GET /api/orders/123 → 200 と注文内容を確認する
2. Bで GET /api/orders/123 → 設計どおりの拒否（例：403 または 404）を確認する
3. 未ログインでも同じAPIを呼び、設計どおり拒否されることを確認する
4. これらをテストコードにして、プルリクエストごとにCIで実行する
```

**E2EテストでもAPIテストでも、このような拒否ケースを自動化できます。** 画面から一連の操作が守られるかを見るならE2E、権限判定を直接確認するならAPIテストが分かりやすいです。さらに、決済前の発送、クーポンの二重利用、順序を飛ばした操作などを、仕様に基づくテストケースとして追加します（[OWASP：ビジネスロジックのテスト](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html)）。

入力の探索を広げたい場合は、OpenAPIからテスト入力を生成するSchemathesisのような方法があります。型や境界値に関する不具合を探せますが、生成したテストだけで認可の正しさは証明できません（[Schemathesis：OpenAPIからの自動テスト](https://schemathesis.readthedocs.io/en/latest/explanations/pytest/)）。PRごとに検証環境を用意できるなら、**PR時に仕様テストと軽いスキャン、検証環境で定期的に深いスキャン**という組み合わせが始めやすいです。スキャン結果は再現して確認し、対象に含まれなかった画面・APIやログイン状態は別途テストします。
