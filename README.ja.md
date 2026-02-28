# cc-daily-report

Claude Code のセッション履歴から日報を自動生成するスキルです。

`~/.claude/projects/` 配下のセッションを対象日で絞り込み、構造化された Markdown レポートを出力します。

## インストール

```bash
git clone https://github.com/Suto-michimasa/cc-daily-report.git
cp -r cc-daily-report/daily-report ~/.claude/skills/daily-report
```

スキル、設定ファイル、テンプレートがまとめてコピーされます。出力先や言語を変更したい場合は `~/.claude/skills/daily-report/config.json` を編集してください。

`git pull` で更新を取り込みたい場合はシンボリックリンクも可:

```bash
ln -s /path/to/cc-daily-report/daily-report ~/.claude/skills/daily-report
```

## 使い方

```bash
# Claude Code 内で — 今日の日報を生成
> /daily-report

# 日付を指定
> /daily-report 2025-06-15

# headless 実行
claude -p "/daily-report" --output-format text > ~/daily-reports/$(date +%Y-%m-%d).md
```

レポートはデフォルトで `~/daily-reports/YYYY-MM-DD.md` に保存されます。

## 出力例

```markdown
# Daily Report — 2025-06-15

## 🏆 今日のハイライト
- ダッシュボードの読み込み時間を 3.2s → 0.8s に改善（N+1 クエリの解消）
- 通知システムの API 設計をチームでレビューし、方針が確定

## 📋 プロジェクト別タスク

### team-dashboard
_(3 sessions, ~3h 20m)_

- [x] ダッシュボード表示遅延の調査・修正（ユーザー一覧 API の N+1 クエリ解消）
- [x] PR #248 レビュー — 通知設定のマイグレーション
- [-] WebSocket によるリアルタイム更新（残: 再接続ロジック）

### personal-blog（個人開発）
_(1 session, ~50m)_

- [x] RSS フィードの全文配信対応
- [-] ダークモード切り替え（残: OS 設定の自動検出）
- [ ] OGP 画像の自動生成（ブロック: ライブラリ選定）

## 💡 学び・発見
- PostgreSQL の `EXISTS` サブクエリは `JOIN + DISTINCT` より「存在チェック」に強い — クエリ 1.2s → 40ms に短縮
- 通知 API の設計では、配信チャネル（email/push/in-app）と通知タイプを分離するとスキーマの拡張性が上がる

## ⏭️ ネクストアクション
- WebSocket 再接続に exponential backoff を実装（team-dashboard）
- OGP 画像生成ライブラリの選定: @vercel/og vs satori（personal-blog）
- 水曜のチーム定例でダッシュボード改善のデモ準備
```

## 設定

`~/.claude/skills/daily-report/config.json` を編集してカスタマイズできます:

```json
{
  "outputDir": "~/daily-reports",
  "templatePath": "",
  "language": "ja"
}
```

| キー | デフォルト | 説明 |
|---|---|---|
| `outputDir` | `~/daily-reports` | レポートの出力先ディレクトリ |
| `templatePath` | `""` | カスタムテンプレートのパス（空 = スキル内蔵テンプレートを使用） |
| `language` | `en` | テンプレート言語 — templatePath が空のとき `templates/{language}.md` を選択 |

すべてのキーは省略可能です。未指定のキーはデフォルト値が使われます。

## テンプレートのカスタマイズ

デフォルトテンプレートの出力セクション:

- **今日のハイライト** — その日の主な成果
- **プロジェクト別タスク** — プロジェクトごとの TODO 進捗（`[x]` 完了 / `[-]` 進行中 / `[ ]` ブロック）
- **学び・発見** — 技術的な知見やデバッグの教訓
- **ネクストアクション** — 翌日以降にやるべきこと

カスタムテンプレートを使う場合:

```bash
cp cc-daily-report/daily-report/templates/ja.md ~/.claude/daily-report-template.md
```

同じフォーマット（YAML フロントマター + `<!-- -->` 指示付き Markdown）で独自テンプレートを作成できます。

