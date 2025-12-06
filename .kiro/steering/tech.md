# Technology Stack

## Architecture

Markdownベースのコマンド定義システム。CursorエージェントがMarkdownファイルを読み取り、定義された指示に従って処理を実行するアーキテクチャです。コードベースは主に設定ファイル、テンプレート、仕様書で構成され、実行時ロジックはエージェントが動的に解釈します。

## Core Technologies

- **Language**: Markdown, JSON
- **Framework**: Cursorエージェントコマンドシステム
- **Runtime**: Cursor IDE環境

## Key Libraries

プロジェクトは外部ライブラリに依存せず、Cursorエージェントの機能とMarkdown/JSONの標準形式のみを使用します。

## Development Standards

### Type Safety

- JSONファイルは有効なJSONスキーマに準拠
- Markdownファイルは一貫した構造とフォーマットを維持

### Code Quality

- Markdownファイルは明確なセクション構造を持つ
- コマンド定義は`<meta>`, `<background_information>`, `<instructions>`の標準セクションで構成
- 日本語を主要言語として使用（`spec.json.language`で管理）

### Testing

- 仕様書ベースの検証プロセス（`validate-gap`, `validate-design`, `validate-impl`）
- 各フェーズでの承認ワークフローによる品質保証

## Development Environment

### Required Tools

- Cursor IDE（エージェント機能が必要）
- 標準的なテキストエディタ（Markdown/JSON編集用）

### Common Commands

```bash
# Spec初期化: /kiro/spec-init "description"
# 要件生成: /kiro/spec-requirements {feature}
# 設計生成: /kiro/spec-design {feature}
# タスク生成: /kiro/spec-tasks {feature}
# 実装: /kiro/spec-impl {feature} [tasks]
# ステータス確認: /kiro/spec-status {feature}
```

## Key Technical Decisions

1. **Markdownベースのコマンド定義**: コードではなく設定ファイルとしてコマンドを定義することで、柔軟性と可読性を確保
2. **Spec-Driven Development**: Requirements → Design → Tasks → Implementation の段階的承認プロセスで、品質と一貫性を保証
3. **テンプレートシステム**: `.kiro/settings/templates/`でテンプレートを管理し、一貫した構造を維持
4. **日本語優先**: プロジェクトの主要言語として日本語を使用し、すべてのドキュメントと出力を日本語で生成
5. **Steering vs Specs**: プロジェクト全体のルール（`.kiro/steering/`）と個別機能の仕様（`.kiro/specs/`）を分離

---
_Document standards and patterns, not every dependency_
