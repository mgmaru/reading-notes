# 『おうちで学べるセキュリティの基本』2026-10-3 疑問5：ソースコードの解析とSAST

## 疑問

> セキュリティホールを見つけるには、ソースコードを人が一行ずつ解析する必要があるか？ ツールでカバーできるか？

## 回答

**人が全行を目視する必要はありませんが、ツールだけで完全には検査できません。** ソースコードを自動で調べる方法は**SAST（静的アプリケーションセキュリティテスト）**です。実行前のコードから、危険なデータの流れや既知のパターンを探します。一方、どの利用者に何を許すべきかという**仕様上の判断**には、開発者の確認と、その仕様を確かめるテストが必要です（[OWASP：ソースコード解析ツールの強みと限界](https://community.owasp.org/Source_Code_Analysis_Tools)、[OWASP：セキュアコードレビュー](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html)）。

```text
ソースコード ── SAST ──→ 危険そうなコード箇所を示す
      │
      └─ 人によるレビュー ──→ 「本来、誰に何を許すか」を仕様と照合する

動作中のアプリ ── DAST・権限別テスト ──→ 実際に許されない操作ができないか確認する
依存ライブラリ ── SCA ──→ 公開済みの脆弱性情報とバージョンを照合する
```

| 方法 | 主な対象 | 例 | 主な限界 |
| --- | --- | --- | --- |
| **SAST** | 自社で書いたコードなど。ソースまたは解析可能なビルド成果物が必要 | ユーザー入力が危険なSQL実行処理に届く経路を検出する | 業務上の権限・手順や実行環境の設定を理解しきれない。誤検出・見逃しがある |
| **人によるコードレビュー** | 設計、権限判定、入力処理、重要な変更 | 「注文の所有者だけが詳細を見られるか」を処理の流れで確認する | 時間と知識が必要。レビューだけでは実際の設定や動作を確かめきれない |
| **SCA** | 外部・内部の依存関係 | `npm audit` などで既知の脆弱性を照合する | 自社実装の欠陥や、まだ登録されていない欠陥は原則として検出できない |

SASTの「静的」は**アプリを実行せずに解析する**という意味です。SAST自体がソースコードレベルの解析を自動化します。OWASPは、SQLインジェクションなど一部の既知パターンを検出できる一方、認証・アクセス制御・設定の問題には弱く、誤検出も起こると説明しています（[OWASP：ソースコード解析ツール](https://community.owasp.org/Source_Code_Analysis_Tools)）。SCAの詳しい使い方は[疑問3：ライブラリの脆弱性を見つける方法](./basic_security_at_home_20261003_question3_dependency_vulnerability_management.md)にまとめています。

### ツールだけで判断できない例：他人の注文を見る

```text
GET /orders/123  → 自分の注文なので閲覧できる
GET /orders/124  → 他人の注文なので拒否されるべき
```

サーバーが「ログイン済みか」は確認していても、「注文124の所有者か」を確認していなければ、他人の注文が見えるかもしれません。コード上でデータベースへのアクセス自体は正常に見えるため、一般的なSASTのルールだけでは、その注文番号を利用者が閲覧してよいか判断できません。**仕様としての権限ルールを明文化し、該当する処理をレビューして、異なる利用者で拒否されることをテストします。** OWASPも、手動レビューが業務ロジックや状況に依存する欠陥の確認を補うとしています（[OWASP：セキュアコードレビュー](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html)、[OWASP：アクセス制御の不備](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/)）。

### 開発での進め方

1. **守るルールを決める。** 例：「注文は所有者と担当者だけが閲覧できる」。権限・機密データ・重要な操作を洗い出す。
2. **SASTを変更時に実行する。** 対応言語と解析対象を確認して、指摘を実際のコード経路・入力条件と照合する。GitHubで対応言語を使うならCodeQLのコードスキャンが一例（[GitHub：CodeQLによるコードスキャン](https://docs.github.com/en/code-security/concepts/code-scanning/codeql/codeql-code-scanning)）。
3. **重要な処理を人がレビューする。** とくに認証、認可、金額、状態遷移、外部入力と機密情報の扱いを見る。変更箇所だけでなく、その前後の呼び出しや設定も必要に応じて確認する（[OWASP：セキュアコードレビュー](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html)）。
4. **実際の動作も確認する。** 異なる権限の利用者で要求を送り、拒否されるべき操作が拒否されるかをテストする。動作中の検査は[疑問5：DASTと自動テスト](./basic_security_at_home_20261003_question5_dynamic_security_testing.md)で説明する。

**スキャン結果が0件でも、安全の証明にはなりません。** ツールの対応言語・ルール・解析できた経路を確認し、結果に疑問があれば手動で再現・確認します。逆に指摘が出ても、実際に悪用可能かは確認が必要です（[OWASP：SASTの限界](https://community.owasp.org/Source_Code_Analysis_Tools)）。

## 追加疑問（2026-10-4）：具体的なSASTツールと自動化

> SASTには具体的にどのようなツールがあるか？ 検査を自動化できるか？

**できます。SASTツールそのものがコード解析を自動化し、さらにCIに組み込めばコードの変更時に毎回実行できます。** 例として次のものがあります。`npm audit` は依存パッケージの既知の脆弱性を調べる**SCA**であり、自社コードを解析するSASTとは対象が違います。

| ツール | 対象・使い方の例 | 自動化の方法 |
| --- | --- | --- |
| **CodeQL** | GitHubで管理する、対応言語のコード。JavaScript/TypeScriptやPythonなどを解析する | GitHubの**コードスキャンのデフォルト設定**を有効にすると、対象ブランチへのpush・プルリクエスト時などに実行できる |
| **Semgrep Community Edition** | 複数言語のコードを、既存または自作のルールで検査する | `semgrep scan --config auto .` を手元やCIで実行する |
| **Bandit** | Pythonコードにある、よくある危険な書き方を検査する | `bandit -r src/` を手元やCIで実行する（`src/` は対象ディレクトリの例） |

根拠：[GitHub：CodeQLの対象言語](https://docs.github.com/en/code-security/concepts/code-scanning/codeql/codeql-code-scanning)と[デフォルト設定の実行契機](https://docs.github.com/en/code-security/concepts/code-scanning/setup-types)、[Semgrep：Community Editionと実行例](https://semgrep.dev/products/community-edition)・[CIの設定例](https://semgrep.dev/docs/semgrep-ci/sample-ci-configs)、[Bandit：解析対象とコマンド](https://bandit.readthedocs.io/en/latest/man/bandit.html)。ツールごとに対応言語、ルール、解析の深さが違います。CodeQLのGitHub上での利用条件はリポジトリの公開設定や契約によって異なるため、導入前に[公式の利用条件](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/configure-code-scanning/configure-code-scanning)を確認します。

```text
開発者がコードを変更 → プルリクエスト作成 → CIがSASTを実行 → 指摘を確認・修正
                                         ↑
                             定期実行や主ブランチの検査も設定できる
```

たとえばCodeQLなら、対象リポジトリの **Settings → Advanced Security → CodeQL analysis → Set up → Default** で設定できます。対応言語のコードがあるか、実際に解析が成功したかを確認します（[GitHub：デフォルト設定の手順と条件](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/configure-code-scanning/configure-code-scanning)）。Semgrepなら対象プロジェクトで上のコマンドを試し、選んだルールがその言語やフレームワークに合うか確認してからCIに入れます（[Semgrep：CI設定例](https://semgrep.dev/docs/semgrep-ci/sample-ci-configs)）。

自社フレームワークに固有の危険な使い方がある場合は、**「このAPIを権限確認なしで呼ばない」など、チェックできる形のルールを追加する**方法もあります。ただし、ルールに書いていない業務上の判断まで自動で理解するわけではありません（[Semgrep：カスタムルールの例](https://semgrep.dev/docs/writing-rules/rule-ideas)、[OWASP：SASTの限界](https://community.owasp.org/Source_Code_Analysis_Tools)）。人がテスト観点を決めて実行を自動化する方法は、[疑問5：DASTと自動テスト](./basic_security_at_home_20261003_question5_dynamic_security_testing.md)に追記しました。
