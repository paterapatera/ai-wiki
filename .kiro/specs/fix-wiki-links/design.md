# Design Document

---
**Purpose**: Provide sufficient detail to ensure implementation consistency across different implementers, preventing interpretation drift.

**Approach**:
- Include essential sections that directly inform implementation decisions
- Omit optional sections unless critical to preventing implementation errors
- Match detail level to feature complexity
- Use diagrams and tables over lengthy prose
---

## Overview

この機能は、`/ai-wiki/make`コマンドで生成されるWikiページ内のMarkdownリンクから`.md`拡張子を除去し、既存のWikiページも修正する機能を提供します。

**Purpose**: この機能は、Wikiページ内のMarkdownリンクを正しい形式（`.md`拡張子なし）に修正することで、リンクの一貫性と互換性を保証します。

**Users**: 開発者が`/ai-wiki/make`コマンドを使用してWikiページを生成する際に、正しい形式のリンクが自動的に生成されます。また、既存のWikiページも一括修正されます。

**Impact**: 現在のリンク形式（`[タイトル](page-id.md)`）を正しい形式（`[タイトル](page-id)`）に変更します。

### Goals
- `/ai-wiki/make`コマンドで生成されるリンクから`.md`拡張子を除去する
- 既存のWikiページ内のリンクから`.md`拡張子を除去する
- リンク形式の検証機能を提供する

### Non-Goals
- リンクの動作確認やリンク切れの検出（将来の拡張として検討可能）
- 他のMarkdown構文の修正
- リンクの自動生成ロジックの大幅な変更

## Architecture

### Existing Architecture Analysis

**既存のアーキテクチャパターン**:
- `/ai-wiki/make`コマンドは`.cursor/commands/ai-wiki/make.md`で定義されている
- リンク生成ロジックは`make.md`のStep 4の4番目のステップ（200-206行目）で実装されている
- 現在のリンク生成形式: `[${relatedPageTitle}](${relatedPageId}.md)`

**統合ポイント**:
- `make.md`のリンク生成ロジックを修正する必要がある
- 既存のWikiページは`repo/{project-name}.wiki/`ディレクトリに保存されている

**技術的負債**:
- 既存のWikiページにも同様の問題が存在し、一括修正が必要

### Architecture Pattern & Boundary Map

この機能は既存システムの単純な拡張であり、新しいアーキテクチャパターンは不要です。既存のコマンド定義ファイル（`make.md`）を修正し、既存ページの一括修正スクリプトを提供するだけです。

**Architecture Integration**:
- 選択パターン: 既存システムの拡張（Extension Pattern）
- ドメイン/機能境界: コマンド定義ファイル（`make.md`）と既存Wikiページの修正処理
- 既存パターンの維持: `make.md`の既存構造と処理フローを維持
- 新規コンポーネントの理由: 既存ページの一括修正スクリプトが必要
- Steering準拠: Markdownベースのコマンド定義システムの原則を維持

### Technology Stack

| Layer | Choice / Version | Role in Feature | Notes |
|-------|------------------|-----------------|-------|
| Command Definition | Markdown | リンク生成ロジックの修正 | `.cursor/commands/ai-wiki/make.md` |
| Text Processing | 正規表現 | 既存ページの一括置換 | Markdownリンク構文に限定 |
| File System | Standard | 既存ページの読み書き | `repo/{project-name}.wiki/`ディレクトリ |

## Requirements Traceability

| Requirement | Summary | Components | Interfaces | Flows |
|-------------|---------|------------|------------|-------|
| 1.1 | リンク生成時に`.md`拡張子を含まない形式で生成 | Link Generation Logic | `make.md` Step 4.4 | Wiki生成フロー |
| 1.2 | ページIDから`.md`拡張子を除去してリンクを生成 | Link Generation Logic | `make.md` Step 4.4 | Wiki生成フロー |
| 1.3 | 既存のリンク生成ロジックを修正 | Link Generation Logic | `make.md` Step 4.4 | Wiki生成フロー |
| 2.1 | 既存ページ内のリンクから`.md`拡張子を除去 | Existing Page Fixer | 正規表現置換処理 | 一括修正フロー |
| 2.2 | 複数のリンクをすべて修正 | Existing Page Fixer | 正規表現置換処理 | 一括修正フロー |
| 2.3 | 修正後のページを元の場所に保存 | Existing Page Fixer | ファイル書き込み処理 | 一括修正フロー |
| 3.1 | リンクが`.md`拡張子を含まないことを検証 | Link Validator | 検証ロジック | 検証フロー |
| 3.2 | `.md`拡張子を含むリンクを検出 | Link Validator | 検証ロジック | 検証フロー |
| 3.3 | リンク形式の検証を実行 | Link Validator | 検証ロジック | 検証フロー |

## Components and Interfaces

| Component | Domain/Layer | Intent | Req Coverage | Key Dependencies (P0/P1) | Contracts |
|-----------|--------------|--------|--------------|--------------------------|-----------|
| Link Generation Logic | Command Definition | リンク生成時に`.md`拡張子を除去 | 1.1, 1.2, 1.3 | `make.md` (P0) | Text Processing |
| Existing Page Fixer | Text Processing | 既存ページ内のリンクを一括修正 | 2.1, 2.2, 2.3 | File System (P0) | File I/O |
| Link Validator | Validation | リンク形式の検証 | 3.1, 3.2, 3.3 | File System (P1) | Validation |

### Command Definition Layer

#### Link Generation Logic

| Field | Detail |
|-------|--------|
| Intent | `/ai-wiki/make`コマンドで生成されるリンクから`.md`拡張子を除去する |
| Requirements | 1.1, 1.2, 1.3 |
| Owner / Reviewers | (optional) |

**Responsibilities & Constraints**
- リンク生成時に`.md`拡張子を含めない形式で生成する
- 既存のリンク生成ロジック（`make.md`のStep 4の4番目のステップ）を修正する
- ページIDから直接リンクを生成する（`.md`拡張子を付与しない）

**Dependencies**
- Inbound: `make.md` Step 4.4 — リンク生成処理の修正対象 (P0)
- Outbound: なし
- External: なし

**Contracts**: Text Processing [✓]

##### Text Processing Contract
- **入力**: ページID（`${relatedPageId}`）、ページタイトル（`${relatedPageTitle}`）
- **出力**: Markdownリンク形式（`[${relatedPageTitle}](${relatedPageId})`）
- **前提条件**: ページIDとページタイトルが提供される
- **事後条件**: `.md`拡張子を含まないリンクが生成される
- **不変条件**: リンク形式はMarkdown標準に準拠する

**Implementation Notes**
- **Integration**: `make.md`の202-203行目を修正する
  - 変更前: `[${relatedPageTitle}](${relatedPageId}.md)`
  - 変更後: `[${relatedPageTitle}](${relatedPageId})`
- **Validation**: 生成されたリンクが`.md`拡張子を含まないことを確認
- **Risks**: 既存のリンク生成ロジックを誤って変更するリスク（慎重に修正する必要がある）

### Text Processing Layer

#### Existing Page Fixer

| Field | Detail |
|-------|--------|
| Intent | 既存のWikiページ内のリンクから`.md`拡張子を除去する |
| Requirements | 2.1, 2.2, 2.3 |
| Owner / Reviewers | (optional) |

**Responsibilities & Constraints**
- 既存のWikiページ内のすべてのリンクから`.md`拡張子を除去する
- 正規表現を使用してMarkdownリンク構文に限定して置換する
- 修正後のページを元の場所に保存する

**Dependencies**
- Inbound: File System — 既存Wikiページの読み込み (P0)
- Outbound: File System — 修正後のページの保存 (P0)
- External: なし

**Contracts**: File I/O [✓]

##### File I/O Contract
- **入力**: 既存のWikiページファイル（`.md`形式）
- **出力**: 修正後のWikiページファイル（`.md`拡張子を除去したリンク）
- **前提条件**: 既存のWikiページファイルが存在する
- **事後条件**: すべてのリンクから`.md`拡張子が除去される
- **不変条件**: ファイルの他の内容は変更されない

**Implementation Notes**
- **Integration**: 正規表現 `]([^)]+\.md)` を使用してMarkdownリンク構文を検出し、`.md`拡張子を除去
- **Validation**: 置換後に`.md`拡張子を含むリンクが残っていないことを確認
- **Risks**: 誤って他の`.md`を含む文字列を置換するリスク（正規表現をMarkdownリンク構文に限定することで回避）

### Validation Layer

#### Link Validator

| Field | Detail |
|-------|--------|
| Intent | 生成されたリンクが正しい形式であることを検証する |
| Requirements | 3.1, 3.2, 3.3 |
| Owner / Reviewers | (optional) |

**Responsibilities & Constraints**
- リンクが`.md`拡張子を含まないことを検証する
- `.md`拡張子を含むリンクを検出して報告する
- 検証結果を提供する

**Dependencies**
- Inbound: File System — Wikiページファイルの読み込み (P1)
- Outbound: なし
- External: なし

**Contracts**: Validation [✓]

##### Validation Contract
- **入力**: Wikiページファイルまたはリンク文字列
- **出力**: 検証結果（エラーまたは成功）
- **前提条件**: 検証対象のファイルまたは文字列が提供される
- **事後条件**: 検証結果が報告される
- **不変条件**: 検証ロジックは一貫性を保つ

**Implementation Notes**
- **Integration**: 正規表現を使用して`.md`拡張子を含むリンクを検出
- **Validation**: 検証ロジックが正しく動作することを確認
- **Risks**: 検証ロジックが不完全な場合、問題のあるリンクを見逃す可能性

## Data Models

この機能は既存のデータ構造を変更せず、テキスト処理のみを行います。新しいデータモデルは不要です。

### Domain Model

- **Wiki Page**: 既存のWikiページエンティティ（変更なし）
- **Link**: Markdownリンク（形式のみ変更: `.md`拡張子を除去）

### Logical Data Model

**構造定義**:
- WikiページはMarkdown形式のテキストファイル
- リンクはMarkdownリンク構文（`[タイトル](リンク先)`）で表現される
- リンク先はページID（`.md`拡張子なし）

**一貫性と整合性**:
- リンク形式の一貫性を保つ（すべてのリンクから`.md`拡張子を除去）
- ファイルの他の内容は変更しない

## Error Handling

### Error Strategy

**エラーカテゴリと対応**:

1. **リンク生成エラー**:
   - ページIDが提供されない場合: エラーメッセージを報告し、リンク生成をスキップ
   - ページタイトルが提供されない場合: ページIDをタイトルとして使用

2. **ファイル処理エラー**:
   - ファイルが存在しない場合: エラーメッセージを報告し、そのファイルをスキップ
   - ファイルの読み込みに失敗した場合: エラーメッセージを報告し、処理を中断
   - ファイルの書き込みに失敗した場合: エラーメッセージを報告し、処理を中断

3. **検証エラー**:
   - `.md`拡張子を含むリンクが検出された場合: エラーメッセージを報告し、該当リンクをリストアップ

### Monitoring

- リンク生成の成功/失敗をログに記録
- 既存ページの修正数を記録
- 検証結果を記録

## Testing Strategy

### Unit Tests
- リンク生成ロジックのテスト: ページIDとタイトルから正しいリンク形式を生成することを確認
- 正規表現置換のテスト: Markdownリンク構文から`.md`拡張子を正しく除去することを確認
- 検証ロジックのテスト: `.md`拡張子を含むリンクを正しく検出することを確認

### Integration Tests
- 既存ページの一括修正フロー: 複数のWikiページファイルを読み込み、リンクを修正し、保存することを確認
- リンク生成と検証の統合: 生成されたリンクが検証を通過することを確認

### E2E Tests
- `/ai-wiki/make`コマンドの実行: 生成されたWikiページのリンクが`.md`拡張子を含まないことを確認
- 既存ページの修正: 修正後のページでリンクが正しく機能することを確認

## Supporting References

- `.cursor/commands/ai-wiki/make.md` - 既存のリンク生成ロジック
- `repo/slack-agent.wiki/` - 既存のWikiページ（問題の実例）
- `.kiro/specs/fix-wiki-links/research.md` - 詳細な調査結果と設計決定

