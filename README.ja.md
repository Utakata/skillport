# ⚓ SkillPort

[English](README.md) | [日本語](README.ja.md)

<div align="center">

**Agent SkillsのためのSkillOpsツールキット**

SkillOps = 大規模なスキルの検証、管理、そして提供。

[![Python](https://img.shields.io/badge/Python-3.10+-blue)](https://python.org)
[![MCP](https://img.shields.io/badge/MCP-Enabled-green)](https://modelcontextprotocol.io)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

</div>

---

## なぜSkillPortなのか？

| こんな時... | SkillPortなら... |
|-------------|--------------|
| スキルをネイティブにサポートしていないコーディングエージェントを使う場合 | MCPやCLI経由で提供 |
| 独自のAIエージェントを構築する場合 | MCPサーバー、CLI、Pythonライブラリを提供 |
| 50以上のスキルがあり、適切なスキルをすぐに見つけたい場合 | 検索ファーストでの読み込み（[Tool Search Tool](https://www.anthropic.com/engineering/advanced-tool-use) パターン） |
| デプロイ前にスキルをチェックしたい場合 | CIで仕様に準拠しているか検証 |
| スキルのメタデータをプログラムで管理したい場合 | `meta` コマンドを提供 |
| GitHubでスキルを見つけた場合 | `add <url>` でインストール |

[Agent Skills仕様](https://agentskills.io/specification)と完全な互換性があります。

---

## 機能

### 検証

スキルが[Agent Skills仕様](https://agentskills.io/specification)に準拠しているかチェックします。

```bash
skillport validate                    # すべてのスキルを検証
skillport validate ./skills           # 特定のディレクトリを検証
skillport validate --json             # CI向けのJSON出力
```

欠落しているフィールド、命名規則の問題、仕様違反などを問題になる前に検出します。

### 管理

あらゆるソースからの完全なライフサイクル管理。

```bash
# GitHubから追加 (省略形)
skillport add anthropics/skills skills

# GitHubから追加 (完全なURL)
skillport add https://github.com/anthropics/skills/tree/main/skills

# ローカルパスまたはZIPファイルから追加
skillport add ./my-skills
skillport add skills.zip

# 更新、一覧表示、削除
skillport update                      # 元のソースからすべてを更新
skillport list                        # インストールされているスキルを表示
skillport remove <skill-id>           # アンインストール
```

### メタデータ

手動でファイルを編集せずにスキルのメタデータを更新できます。自動化や、チーム全体でスキルの一貫性を保つのに役立ちます。

```bash
skillport meta get <skill> <key>      # メタデータの値を取得
skillport meta set <skill> <key> <val> # メタデータの値を設定
skillport meta unset <skill> <key>    # メタデータのキーを削除
```

### 提供

Agent Skillsをネイティブにサポートしていないクライアント向けのMCPサーバーです。

Anthropicの[Tool Search Tool](https://www.anthropic.com/engineering/advanced-tool-use)パターンにインスパイアされています — 最初に検索し、必要に応じてロードします：

| ツール | 目的 |
|------|---------|
| `search_skills(query)` | 説明からスキルを検索 (全文検索) |
| `load_skill(skill_id)` | 完全な指示とパスを取得 |

**なぜ検索が重要なのか：** 50以上のスキルがある場合、すべてを事前に読み込むとコンテキストを消費し、精度が低下します。SkillPortはメタデータのみ（1スキルあたり約100トークン）を読み込み、完全な指示は必要になった時に読み込みます。

Cursor、Copilot、Windsurf、Cline、Codexなど、MCP互換のあらゆるクライアントで動作します。

---

## クイックスタート

### インストール

```bash
uv tool install skillport
# または: pip install skillport
```

### スキルの追加

```bash
# GitHubから追加
skillport add anthropics/skills skills

# またはカスタムスキルディレクトリを使用
skillport --skills-dir .claude/skills add anthropics/skills skills
```

### 検証

```bash
skillport validate
# ✓ All 5 skill(s) pass validation
```

---

## エージェントへの接続

AIエージェントにスキルを提供する方法を選択します：

| モード | こんな場合に最適 | セットアップ |
|------|----------|-------|
| [**CLIモード**](#cliモード) | シェルコマンドを実行できるエージェント (Cursor, Windsurf, Codexなど) | プロジェクト毎 |
| [**MCPモード**](#mcpモード) | MCP互換のクライアント、複数プロジェクト | 1回のみ |

### CLIモード

シェルコマンドを実行できるエージェント向けです。MCPの設定は必要ありません。

```bash
skillport init                        # プロジェクトを初期化
skillport doc                         # スキル表を含むAGENTS.mdを生成
skillport show <id>                   # スキルの完全な指示を読み込む
```

仕組み：
1. `skillport doc` が `AGENTS.md` にスキル表を生成します。
2. エージェントは `AGENTS.md` を読み、利用可能なスキルを発見します。
3. 必要に応じて、エージェントは `skillport show <id>` を実行し、完全な指示を読み込みます。

### MCPモード

MCP互換のクライアント向けです。サーバーをインストールします：

```bash
uv tool install skillport-mcp
```

クライアントの設定に追加します：

```json
{
  "mcpServers": {
    "skillport": {
      "command": "uvx",
      "args": ["skillport-mcp"],
      "env": { "SKILLPORT_SKILLS_DIR": "~/.skillport/skills" }
    }
  }
}
```

<details>
<summary>人気のクライアント向けのワンクリックインストール</summary>

**Cursor**

[![Install MCP Server](https://cursor.com/deeplink/mcp-install-dark.svg)](cursor://anysphere.cursor-deeplink/mcp/install?name=skillport&config=eyJjb21tYW5kIjoidXZ4IiwiYXJncyI6WyJza2lsbHBvcnQtbWNwIl0sImVudiI6eyJTS0lMTFBPUlRfU0tJTExTX0RJUiI6In4vLnNraWxscG9ydC9za2lsbHMifX0=)

**VS Code / GitHub Copilot**

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_MCP_Server-007ACC?logo=visualstudiocode)](https://insiders.vscode.dev/redirect/mcp/install?name=skillport&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22skillport-mcp%22%5D%2C%22env%22%3A%7B%22SKILLPORT_SKILLS_DIR%22%3A%22~/.skillport/skills%22%7D%7D)

**Kiro**

[![Add to Kiro](https://kiro.dev/images/add-to-kiro.svg)](https://kiro.dev/launch/mcp/add?name=skillport&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22skillport-mcp%22%5D%2C%22env%22%3A%7B%22SKILLPORT_SKILLS_DIR%22%3A%22~/.skillport/skills%22%7D%2C%22disabled%22%3Afalse%2C%22autoApprove%22%3A%5B%5D%7D)

**CLIエージェント**

```bash
# Codex
codex mcp add skillport -- uvx skillport-mcp

# Claude Code
claude mcp add skillport -- uvx skillport-mcp
```

</details>

---

## 構成

カテゴリやタグを使ってスキルを構成できます。CLIモードとMCPモードの両方で機能します。

### カテゴリとタグ

メタデータを使用してスキルを整理、フィルタリングします：

```yaml
# SKILL.md のフロントマター
metadata:
  skillport:
    category: development
    tags: [testing, quality]
    alwaysApply: true  # 常に利用可能 (コアスキル)
```

### クライアントごとのフィルタリング

異なるエージェントに対して異なるスキルを公開します：

```json
{
  "mcpServers": {
    "skillport-dev": {
      "command": "uvx",
      "args": ["skillport-mcp"],
      "env": { "SKILLPORT_ENABLED_CATEGORIES": "development,testing" }
    },
    "skillport-writing": {
      "command": "uvx",
      "args": ["skillport-mcp"],
      "env": { "SKILLPORT_ENABLED_CATEGORIES": "writing,research" }
    }
  }
}
```

---

## 設定

| 変数 | 説明 | デフォルト値 |
|----------|-------------|---------|
| `SKILLPORT_SKILLS_DIR` | スキルのディレクトリ | `~/.skillport/skills` |
| `SKILLPORT_ENABLED_CATEGORIES` | カテゴリでフィルタリング | all |
| `SKILLPORT_ENABLED_SKILLS` | スキルIDでフィルタリング | all |
| `SKILLPORT_ENABLED_NAMESPACES` | 名前空間でフィルタリング | all |
| `SKILLPORT_CORE_SKILLS_MODE` | コアスキルの動作 (`auto`/`explicit`/`none`) | `auto` |

---

## スキルの作成

YAMLフロントマターを持つ `SKILL.md` を作成します：

```markdown
---
name: my-skill
description: このスキルがすること
metadata:
  skillport:
    category: development
    tags: [example]
---
# マイ・スキル

AIエージェントへの指示。
```

ベストプラクティスについては、[Creating Skills Guide](guide/creating-skills.md) を参照してください。

---

## スキルソース

| ソース | 特徴 | ターゲット | URL |
|--------|----------|--------|-----|
| Anthropic Official | ドキュメントスキル (docx, pdf, pptx, xlsx)、デザイン、MCPビルダー | 全てのユーザー | [GitHub](https://github.com/anthropics/skills/tree/main/skills) |
| Awesome Claude Skills | コミュニティにより厳選されたコレクション、2,500以上のスター | 発見・探索 | [GitHub](https://github.com/ComposioHQ/awesome-claude-skills) |
| Hugging Face Skills | データセット作成、モデル評価、LLMトレーニング、論文公開 | ML/AIエンジニア | [GitHub](https://github.com/huggingface/skills) |
| Claude Scientific Skills | 128以上の科学的スキル (生物、化学、ML)、26以上のデータベース | 研究者 | [GitHub](https://github.com/K-Dense-AI/claude-scientific-skills) |
| ClaudeKit Skills | 30以上のスキル、認証、マルチモーダル、問題解決フレームワーク | フルスタック開発者 | [GitHub](https://github.com/mrgoonie/claudekit-skills) |
| Superpowers | TDD、デバッグ、並列エージェント、コードレビューのワークフロー | 品質重視の開発者 | [GitHub](https://github.com/obra/superpowers) |
| Kubernetes Operations | K8sのデプロイ、モニタリング、トラブルシューティング | DevOps/SRE | [GitHub](https://github.com/wshobson/agents/tree/main/plugins/kubernetes-operations/skills) |

---

## もっと詳しく知る

- [Configuration Guide](guide/configuration.md)
- [Creating Skills](guide/creating-skills.md)
- [CLI Reference](guide/cli.md)
- [Design Philosophy](guide/philosophy.md)

---

## 開発

> **ステータス:** 作業中。APIは変更される可能性があります。

```bash
git clone https://github.com/gotalab/skillport.git
cd skillport
uv sync

# MCPサーバーの実行
SKILLPORT_SKILLS_DIR=.skills uv run skillport-mcp

# CLIの実行
uv run skillport --help

# テストの実行
uv run pytest
```

---

## ライセンス

MIT
