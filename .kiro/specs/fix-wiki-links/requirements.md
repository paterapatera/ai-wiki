# Requirements Document

## Project Description (Input)
`/ai-wiki/make`関連ページのリンクがおかしい

## Introduction

`/ai-wiki/make`コマンドで生成されるWikiページ内のMarkdownリンクが、`.md`拡張子を含んでいます（例: `[エージェントシステム](page-agent-system.md)`）。Markdownリンクでは通常`.md`拡張子を省略する必要があるため、リンク生成ロジックを修正し、既存のページも修正する必要があります。

## Requirements

### Requirement 1: リンク生成ロジックの修正

**Objective:** As a 開発者, I want `/ai-wiki/make`コマンドが生成するWikiページ内のリンクから`.md`拡張子を除去する, so that Markdownリンクが正しい形式で生成される

#### Acceptance Criteria
1. When `/ai-wiki/make`コマンドがページ間リンクを生成する時, the Wiki生成システム shall `.md`拡張子を含まないリンク形式（`[${relatedPageTitle}](${relatedPageId})`）で生成する
2. If リンク生成時にページIDが提供される時, the Wiki生成システム shall ページIDから`.md`拡張子を除去してリンクを生成する
3. The Wiki生成システム shall 既存のリンク生成ロジック（`make.md`のStep 4の4番目のステップ）を修正する

### Requirement 2: 既存ページの修正

**Objective:** As a ユーザー, I want 既存のWikiページ内のリンクから`.md`拡張子を除去する, so that すべてのページが一貫したリンク形式を持つ

#### Acceptance Criteria
1. When 既存のWikiページが`.md`拡張子を含むリンクを持つ時, the Wiki修正システム shall すべてのリンクから`.md`拡張子を除去する
2. If ページ内に複数のリンクが存在する時, the Wiki修正システム shall すべてのリンクを修正する
3. The Wiki修正システム shall 修正後のページを元の場所に保存する

### Requirement 3: リンク形式の検証

**Objective:** As a 開発者, I want 生成されたリンクが正しい形式であることを検証する, so that リンクが正しく機能することを保証する

#### Acceptance Criteria
1. When リンクが生成される時, the Wiki生成システム shall リンクが`.md`拡張子を含まないことを検証する
2. If リンクが`.md`拡張子を含む時, the Wiki生成システム shall エラーを報告するか、自動的に修正する
3. The Wiki生成システム shall リンク形式の検証を実行する
