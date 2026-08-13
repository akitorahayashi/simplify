# simplify

変更されたコードの品質を上げる Agent Skill を配布するプラグインです。差分を再利用、
単純化、効率、抽象度の4観点でレビューし、見つけた改善をその場でコードに適用します。
正しさのバグ探索は扱いません。

リポジトリのルートが Claude Code と Codex のマーケットプレースルート、`plugin/` が
Claude Code、Antigravity CLI、Codex のプラグインルートです。ひとつの `plugin/skills/` を
すべてのクライアントが共有し、各マニフェストはクライアント固有の識別情報だけを持ちます。

## 構成

```text
simplify/
├── .claude-plugin/
│   └── marketplace.json                # Claude Code マーケットプレースマニフェスト
├── .agents/
│   └── plugins/
│       └── marketplace.json            # Codex マーケットプレースマニフェスト
├── plugin/
│   ├── skills/
│   │   └── simplify/
│   │       ├── SKILL.md                # スキル本体。名前は frontmatter の name が決める
│   │       └── agents/
│   │           └── openai.yaml         # Codex 側の呼び出しポリシー
│   ├── .claude-plugin/
│   │   └── plugin.json                 # Claude Code マニフェスト
│   ├── .codex-plugin/
│   │   └── plugin.json                 # Codex マニフェスト
│   └── plugin.json                     # Antigravity CLI マニフェスト
└── README.md
```

マーケットプレースマニフェストは `./plugin` を指すインストール索引であり、プラグイン本体を
`plugins/` 配下に複製しません。コンポーネントディレクトリ（`skills/`、および将来の `hooks/`、
`agents/`、`commands/`、`.mcp.json`）はプラグインルートに置き、`.claude-plugin/` と
`.codex-plugin/` には `plugin.json` だけを置きます。

## スキルの動作

`plugin/skills/simplify/SKILL.md` が定義するワークフローは3段階です。

1. レビュー対象の取得 — `git diff @{upstream}...HEAD` を基本とし、upstream が無い場合は
   `main...HEAD` または `HEAD~1`。未コミットの変更があるか差分が空の場合は `git diff HEAD` も
   含めます。引数で PR 番号、ブランチ名、ファイルパスが渡された場合はそれを対象にします。
2. 4観点のレビュー — 再利用、単純化、効率、抽象度。サブエージェントを起動できる環境では
   4観点を並列に実行し、できない環境では順に実行して、その旨を報告に明記します。
3. 修正の適用 — 重複する指摘を統合し、残りを直します。意図された挙動を変える、レビュー範囲から
   大きく外れる、誤検知と判断した指摘はスキップし、スキップした事実を報告します。

## 呼び出しポリシー

このスキルはユーザーの明示的な呼び出しでのみ起動し、モデルが自律的に読み込むことはありません。
差分全体のレビューと修正適用を伴うため、起動の判断はユーザーが持ちます。

- Claude Code — `SKILL.md` の `disable-model-invocation: true`
- Codex — `plugin/skills/simplify/agents/openai.yaml` の `policy.allow_implicit_invocation: false`
- Antigravity CLI — `plugin.json` が閉じたスキーマであり、スキル単位の呼び出しポリシーを宣言する
  フィールドがないため、抑止は上記2クライアントで有効です。

明示呼び出し専用のため、`description` はモデルの起動判断のための条件文ではなく、スキルを選ぶ
ユーザーに向けた説明文です。

## 各マニフェストの要件

- Claude Code — `plugin/skills/` 配下のスキルは自動検出されるため、`.claude-plugin/plugin.json`
  に `skills` フィールドは不要です。`author`、`homepage`、`repository`、`license`、`keywords` は
  任意のメタデータです。
- Codex — `.codex-plugin/plugin.json` が `"skills": "./skills/"` を宣言します。
- Antigravity CLI — `plugin.json` は閉じたスキーマで、`name`（必須、`^[a-zA-Z0-9-_]+$`）と
  `description` だけが有効です。スキルは `skills/` から検出されるため、他のフィールドは
  追加しません。

## インストール

リポジトリのルートが GitHub 配布時のマーケットプレースルートです。

### Claude Code

```bash
claude plugin marketplace add akitorahayashi/simplify
claude plugin install simplify@simplify
```

プラグインのスキルはプラグイン名で名前空間化されるため、呼び出し名は `/simplify:simplify` です。
Claude Code に組み込まれている `/simplify` とは別物として共存します。

ローカル開発では `claude --plugin-dir ./plugin` で現在のセッションにプラグインルートを
読み込めます。

### Codex

```bash
codex plugin marketplace add akitorahayashi/simplify
codex plugin add simplify@simplify
```

インストールまたは有効化の後、新しい Codex セッションからスキルが利用可能になります。呼び出しは
`$simplify` です。

### Antigravity CLI

Antigravity は受け取ったパスからプラグインを取り込みます。`plugin/` に `plugin.json` と
`skills/` の両方があります。

```bash
agy plugin install ./plugin
```

## 検証

配布前にマニフェストを検証します。

```bash
claude plugin validate .
claude plugin validate ./plugin
agy plugin validate ./plugin
```
