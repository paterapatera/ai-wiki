# Project Structure

## Organization Philosophy

Spec-Driven Developmentとコマンド定義の分離を重視した構造。プロジェクト全体の知識（steering）と個別機能の仕様（specs）を明確に分離し、テンプレートとルールを一元管理します。

## Directory Patterns

### Steering（プロジェクト全体の知識）
**Location**: `.kiro/steering/`  
**Purpose**: プロジェクト全体に適用されるルール、パターン、技術スタックの定義  
**Example**: `product.md`, `tech.md`, `structure.md`

### Specifications（個別機能の仕様）
**Location**: `.kiro/specs/{feature-name}/`  
**Purpose**: 個別機能の要件、設計、タスク、実装計画を段階的に管理  
**Example**: `requirements.md`, `design.md`, `tasks.md`, `spec.json`

### Templates（テンプレート）
**Location**: `.kiro/settings/templates/`  
**Purpose**: 仕様書やsteeringファイルの生成に使用するテンプレート  
**Example**: `specs/design.md`, `steering/product.md`

### Rules（開発ルール）
**Location**: `.kiro/settings/rules/`  
**Purpose**: 開発プロセス、設計原則、検証ルールの定義  
**Example**: `steering-principles.md`, `design-principles.md`

### Commands（エージェントコマンド）
**Location**: `.cursor/commands/`  
**Purpose**: Cursorエージェント用のコマンド定義（Markdown形式）  
**Example**: `kiro/spec-init.md`, `ai-wiki/init.md`

## Naming Conventions

- **Specディレクトリ**: `kebab-case`（例: `wiki-topic-analysis-prompt`）
- **Markdownファイル**: `kebab-case.md`（例: `requirements.md`, `design.md`）
- **JSON設定ファイル**: `spec.json`（各specディレクトリに1つ）
- **コマンドファイル**: `kebab-case.md`（`.cursor/commands/`内）

## Import Organization

このプロジェクトは主にMarkdown/JSONファイルで構成され、コードベースのインポートは発生しません。代わりに、ファイルパスベースの参照を使用します：

- テンプレート参照: `.kiro/settings/templates/{category}/{file}`
- ルール参照: `.kiro/settings/rules/{rule-name}.md`
- Spec参照: `.kiro/specs/{feature-name}/{file}`

## Code Organization Principles

1. **段階的開発**: Requirements → Design → Tasks → Implementation の順序を厳守
2. **承認ワークフロー**: 各フェーズで人間の承認を必須とし、`-y`フラグは意図的な高速化のみ
3. **テンプレート駆動**: 新しいspecやsteeringファイルはテンプレートから生成
4. **言語一貫性**: `spec.json.language`で指定された言語で全てのドキュメントを生成
5. **分離の原則**: Steering（プロジェクト全体）とSpecs（個別機能）を明確に分離
6. **パターン重視**: ファイルリストではなく、組織パターンと命名規則を文書化

---
_Document patterns, not file trees. New files following patterns shouldn't require updates_
