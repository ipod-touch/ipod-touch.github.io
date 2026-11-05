# ipod-touch.github.io

個人開発アプリを紹介するGitHub Pages用の静的サイトです。HTMLと共通CSSで構成し、JavaScript、外部CDN、Cookie、アクセス解析は追加していません。

## ページ構成

- `index.html`：共通トップページとアプリ一覧、問い合わせ欄。
- `assets/style.css`：全ページ共通のレスポンシブCSS。
- `apps/voice-actor-deadline-manager-public/index.html`：声優・アーティスト締切管理の紹介。
- `apps/voice-actor-deadline-manager-public/privacy.html`：実装調査に基づくプライバシーポリシー。

既存のHTML・CSS・アプリページは存在しなかったため、共通ページを新設しました。既存の作業用ファイルは変更していません。

## 調査結果（2026年11月5日）

アプリ名は「声優・アーティスト締切管理」。Python 3.11以上、Raspberry Piでの定期実行を想定しています。公式ニュースの取得、Gemini APIによる公演・受付日時の抽出、SQLiteへの保存、Google Calendarへの同期、確認タスク管理、世代バックアップを実装しています。開発中で、抽出漏れ・誤判定・重複登録が既知の課題です。

調査根拠は対象アプリ内の次の相対パスです。アプリのコードは実行していません。

| 根拠ファイル | 確認した内容 |
| --- | --- |
| `README.md`, `pyproject.toml` | 名称、概要、開発状況、実行環境 |
| `src/voice_actor_deadline_manager/google_calendar.py` | Desktop OAuth、PKCE・state検証、トークンJSON保存・更新、Calendarの取得・作成・更新・削除 |
| `src/voice_actor_deadline_manager/calendar_sync.py`, `calendar_desired.py` | 管理対象予定の照合、送信項目、同期情報・ETagの保存 |
| `src/voice_actor_deadline_manager/llm.py`, `parse_worker.py`, `settings.py` | Geminiへの送信内容、トークン数見積もり、任意の証跡インターフェース、料金方針が設定依存であること |
| `src/voice_actor_deadline_manager/snapshots.py`, `migrations/001_initial.sql` | 記事本文、抽出・確認情報、同期データのSQLite保存 |
| `src/voice_actor_deadline_manager/backup.py`, `backup_remote.py` | SQLite全体の複製、rclone転送、ローカル・転送先の日次7世代・週次4世代の整理 |
| `src/voice_actor_deadline_manager/service_logging.py` | 操作ID・結果等の専用ログ、当日分と過去13世代の管理 |
| `deploy/voice-actor-deadline-manager.service` | systemd配備例の実行ユーザー・ディレクトリ権限等 |

要求するOAuthスコープは次の2つです。

- `https://www.googleapis.com/auth/calendar.events`：管理対象予定の取得・作成・更新・削除。
- `https://www.googleapis.com/auth/calendar.readonly`：設定カレンダーへのアクセス確認。実際の要求権限はこの用途より広く、最小権限の見直しはアプリ側の確認事項です。

Gmail API・Google Drive APIの直接利用は調査範囲では確認されませんでした。rcloneの転送先をGoogle Driveと断定していません。

Google Calendarの予定データを照合に使用し、カレンダーID・予定ID・同期予定／適用済みデータ・ETag等をSQLiteに保存します。OAuthトークンは別のローカルJSONです。DB全体のバックアップはCalendar同期情報も含みます。Geminiの通常解析経路には記事のタイトル・公開日・アーティスト名・正規化本文を送り、Calendarの取得データやOAuthトークンを渡す処理は確認されませんでした。

全データ削除機能、DB全体の自動保存期限、連携解除後の自動消去は確認できませんでした。終了時は定期実行・バックアップを停止し、Calendar上の予定、ローカルのトークン・DB・ログ・証跡・バックアップ、転送先のデータを個別に削除する必要があります。

## 読み取り制限の適用

- 対象パスの存在・ディレクトリであること・Gitルートを確認。
- ルートの `.gitignore` と `.git/info/exclude` を確認。グローバルの `core.excludesFile` 指定は検出されませんでした。探索した非除外ディレクトリ内に追加の `.gitignore` はありませんでした。
- 各候補に `git check-ignore --no-index` を適用し、追跡済みファイルにもignoreルールを適用。除外ディレクトリの内部は探索していません。
- ファイル名の一覧から通常のソース・文書・配備例だけを許可し、その一覧のファイルのみを読み取り・検索。`.env.example` も `.env.*` として読み取り対象から外しました。認証情報、DB、個人データ、バックアップ実体、用途を確認できない設定・JSONも読み取っていません。
- シンボリックリンク／リパースポイントを追跡せず、別リポジトリの内部を含めていません。Git履歴・オブジェクト・stashは未参照です。
- 対象リポジトリの変更、コード・テスト実行、依存関係インストールは行っていません。

## 公開前の確認事項

実装調査では確認できない運用情報を次にまとめています。必要に応じて本文に反映してください。

1. Google Cloud ConsoleのOAuth同意画面の正式アプリ名と、このサイトの名称が一致するか。
2. 運営者、公開可能な問い合わせ先、サポートメール。現在は窓口を「公開準備中」と表示しています。
3. 個人利用に限定するか、第三者にも提供するか。対象ユーザーと実際の公開状態。
4. DB・トークン・証跡・OSログの保存期間、削除手順と通知・再同意の運用。
5. rclone転送の実使用、転送先事業者・所在・アクセス権・暗号化・保持／削除条件、利用者の同意。
6. Geminiの実利用プラン、入力の保存・学習利用条件、任意の証跡出力の有効化状況。
7. 実環境の権限・保護措置、Googleユーザーデータへの人のアクセスとLimited Useへの対応。
8. 要求スコープの必要性と最小権限、必要なGoogleの検証手続き。

## 公開手順とOAuth用URL

この作業ではpush・公開設定変更・デプロイを行っていません。実際のGitHub Pages設定は未確認です。公開する際は上の確認事項を解決し、サイトの変更をレビューしてGitHubへ反映してください。GitHubのリポジトリ設定で Pages → Build and deployment → Deploy from a branch を使う場合は、公開対象のブランチ（現在のローカルブランチは `main`）と `/(root)` を指定します。既存のActionsによる公開設定がある場合はそれに従ってください。

リモートのリポジトリ名から想定されるURL（公開の到達性は未確認）：

- 共通トップ：<https://ipod-touch.github.io/>
- アプリホーム：<https://ipod-touch.github.io/apps/voice-actor-deadline-manager-public/>
- プライバシーポリシー：<https://ipod-touch.github.io/apps/voice-actor-deadline-manager-public/privacy.html>

OAuthのホームページURLにはアプリホーム、プライバシーポリシーURLにはポリシーのページを設定します。静的な紹介ページはOAuthリダイレクト先ではありません。

2026年11月5日に次の公式資料を確認しました。

- [Google OAuth Branding Verification](https://developers.google.com/identity/protocols/oauth2/production-readiness/brand-verification)：アプリとの関連が明確な公開ホームページ、同一ドメインのポリシーと相互リンク、表示名、承認済みドメイン・所有権の確認。
- [Google API Services User Data Policy](https://developers.google.com/terms/api-services-user-data-policy)：取得・利用・保存・共有の透明な説明、必要最小限の権限、該当するLimited Use要件。
- [Google Auth Platform — Manage App Audience](https://support.google.com/cloud/answer/15549945)：対象ユーザーとTesting／In productionの設定、公開ステータスと検証の関係。

`github.io` は共有ドメインです。Search ConsoleでURLの所有権を確認できることと、OAuthの承認済みドメイン要件を満たすことは同一ではありません。実際のホスト名についてGoogleの登録可能なドメイン単位の要件とConsole上の受け入れ状況を確認し、必要なら所有する独自ドメインを検討してください。アプリの本番公開、ブランド検証、スコープ検証、GitHub Pages公開は別の手続きです。ページを作成しただけでOAuth検証に合格するとは限りません。

## ローカル検証結果

Python標準ライブラリのHTMLパーサーで3ページのタグ構造・日本語指定・タイトル・見出し・viewport、29件の相対リンク／ページ内リンク、共通CSS参照、トップページへの導線を確認しました。CSSの括弧とレスポンシブ指定も確認しています。作成・更新したHTML・CSS・READMEはUTF-8（BOMなし）で、機密情報の代表的な形式・メールアドレス・ローカル絶対パスは検出されませんでした。目視による本文確認も行っています。

`git diff --check` は問題ありませんでした。Computer Use・GUIブラウザー・スクリーンショットは使用していません。画面上のレイアウト、外部リンクの到達性、実際のGitHub Pages公開状態とGoogle OAuth設定は未検証です。
