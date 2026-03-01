# cc-session-tools

Claude Code のセッション履歴を活用した生産性トラッキングプラグインです。

`~/.claude/projects/` 配下のセッションデータから、日報・週報の生成、セッション検索、タイムシート推定を提供します。

## インストール

```bash
claude plugin install github:Suto-Michimasa/cc-session-tools
claude plugin enable session-tools
```

<details>
<summary>手動インストール（プラグインシステムを使わない場合）</summary>

```bash
git clone https://github.com/Suto-Michimasa/cc-session-tools.git
cp -r cc-session-tools/skills/* ~/.claude/skills/
```

</details>

## スキル一覧

### `/daily-report` — 日報生成

セッション履歴から構造化された日報を生成します。

```bash
> /daily-report
> /daily-report 2025-06-15
```

デフォルトで `~/daily-reports/YYYY-MM-DD.md` に保存されます。`config.json` で出力先、言語、テンプレートをカスタマイズできます。

<details>
<summary>出力例</summary>

```markdown
# Daily Report — 2025-06-15

## 今日のハイライト
- ダッシュボードの読み込み時間を 3.2s → 0.8s に改善（N+1 クエリの解消）

## プロジェクト別タスク

### team-dashboard — 3 sessions, 24 prompts
- [x] ダッシュボード表示遅延の調査・修正
- [-] WebSocket によるリアルタイム更新（残: 再接続ロジック）

## 学び・発見
- PostgreSQL の `EXISTS` サブクエリは `JOIN + DISTINCT` より「存在チェック」に強い

## ネクストアクション
- WebSocket 再接続に exponential backoff を実装
```

</details>

### `/weekly-report` — 週報生成

日報を集約して週次サマリーを生成します。1on1 やチームミーティングの準備に。

```bash
> /weekly-report
> /weekly-report 2026-W09
> /weekly-report 2026-02-24..2026-02-28
```

既存の日報があればそこから読み込み、なければセッションデータから直接生成します。`~/weekly-reports/YYYY-Www.md` に保存されます。

### `/session-search` — セッション検索

全セッションをキーワードで横断検索します。

```bash
> /session-search "N+1 クエリ"
> /session-search migration
```

結果はプロジェクト別にグループ化され、前後のコンテキスト付きで会話内に表示されます。

### `/timesheet` — 作業時間推定

セッションのタイムスタンプからプロジェクト別の作業時間を推定します。

```bash
> /timesheet
> /timesheet 2026-02-14
> /timesheet 2026-W09
> /timesheet 2026-02-01..2026-02-28
```

メッセージのタイムスタンプを基に計算し、30 分超のアイドル時間で作業ブロックを分割、15 分単位で丸めます。

## 設定

`daily-report` と `weekly-report` はスキルディレクトリ内に `config.json` があります。`session-search` と `timesheet` は設定不要（最適なデフォルト値が組み込み済み）です。

### daily-report の設定

| キー | デフォルト | 説明 |
|---|---|---|
| `outputDir` | `~/daily-reports` | レポートの出力先ディレクトリ |
| `templatePath` | `""` | カスタムテンプレートのパス（空 = 内蔵テンプレートを使用） |
| `language` | `en` | テンプレート言語 — `templates/{language}.md` を選択 |

### weekly-report の設定

| キー | デフォルト | 説明 |
|---|---|---|
| `dailyReportDir` | `~/daily-reports` | 既存日報の参照先ディレクトリ |
| `outputDir` | `~/weekly-reports` | 週報の出力先ディレクトリ |
| `templatePath` | `""` | カスタムテンプレートのパス |
| `language` | `en` | テンプレート言語 |

## テンプレートのカスタマイズ

テンプレートは YAML フロントマター + `<!-- -->` 指示付き Markdown 形式です。カスタマイズするには:

```bash
cp cc-session-tools/skills/daily-report/templates/ja.md ~/.claude/daily-report-template.md
```

対応する `config.json` の `templatePath` にパスを設定してください。

## License

MIT
