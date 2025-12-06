# Requirements Document

## Project Description (Input)
`/ai-wiki/make`でwikiのサイドバー(_sidebar.md)も作成する

## Requirements

### Requirement 1: サイドバーファイルの生成
**Objective:** 開発者として、`/ai-wiki/make`コマンド実行時に、wikiページと同時にサイドバーファイル（`_sidebar.md`）も生成されるようにしたい。これにより、生成されたWikiにナビゲーション機能を提供できる。

#### Acceptance Criteria
1. When `/ai-wiki/make`コマンドが実行され、Wikiページの生成が完了する, the AI Wiki System shall サイドバーファイル（`_sidebar.md`）を出力先ディレクトリに生成する
2. The AI Wiki System shall サイドバーファイルをMarkdown形式で生成する
3. The AI Wiki System shall サイドバーファイルのファイル名を`_sidebar.md`とする
4. The AI Wiki System shall サイドバーファイルをWikiページと同じ出力先ディレクトリに配置する
5. If サイドバーファイルの生成に失敗する, the AI Wiki System shall エラーメッセージを報告し、処理を継続するか中断する（実装方針による）

### Requirement 2: セクション構造に基づくナビゲーション階層
**Objective:** 開発者として、サイドバーが`wiki_structure.json`のセクション構造を反映した階層的なナビゲーションを提供するようにしたい。これにより、ユーザーがWikiの構造を理解しやすくなる。

#### Acceptance Criteria
1. When サイドバーが生成される, the AI Wiki System shall `wiki_structure.json`の`sections`配列に基づいて、セクションごとにナビゲーション項目を生成する
2. The AI Wiki System shall 各セクションの`title`をセクション見出しとして使用する
3. The AI Wiki System shall 各セクションの`pages`配列に含まれるページIDを参照して、そのセクションに属するページをリストアップする
4. The AI Wiki System shall セクションの階層構造（`subsections`配列）を反映した階層的なナビゲーションを生成する（該当する場合）
5. The AI Wiki System shall セクションの順序を`sections`配列の順序に従う
6. If `sections`配列が存在しない（簡潔ビューの場合）, the AI Wiki System shall セクション構造なしで、すべてのページをフラットなリストとして表示する

### Requirement 3: ページリンクの生成
**Objective:** 開発者として、サイドバー内の各ページ項目が適切なMarkdownリンクとして生成されるようにしたい。これにより、ユーザーがサイドバーから直接ページにアクセスできる。

#### Acceptance Criteria
1. When サイドバーが生成される, the AI Wiki System shall 各ページの`id`を基にファイルパスを生成する（形式：`{page-id}`、`.md`拡張子は含めない）
2. The AI Wiki System shall 各ページの`title`をリンクテキストとして使用する
3. The AI Wiki System shall Markdownリンク構文（`[Page Title](page-id)`）で各ページへのリンクを生成する（`.md`拡張子は含めない）
4. The AI Wiki System shall ページの順序を、各セクションの`pages`配列の順序に従う
5. If ページの`title`が存在しない, the AI Wiki System shall ページの`id`をリンクテキストとして使用する（フォールバック）

### Requirement 4: サイドバーのMarkdown形式
**Objective:** 開発者として、生成されるサイドバーファイルが一般的なWikiシステム（GitBook、Docsifyなど）で使用可能な標準的なMarkdown形式であるようにしたい。これにより、生成されたWikiを様々なWikiシステムで利用できる。

#### Acceptance Criteria
1. The AI Wiki System shall サイドバーをMarkdown形式で生成する
2. The AI Wiki System shall セクション見出しをMarkdownの見出し構文（`## Section Title`）で表現する
3. The AI Wiki System shall ページリンクをMarkdownのリスト構文（`- [Page Title](page-id)`）で表現する（`.md`拡張子は含めない）
4. The AI Wiki System shall 階層構造をインデントで表現する（該当する場合）
   - トップレベルのページリンク: `- [Page Title](page-id)`
   - サブレベルのページリンク: `  - [Page Title](page-id)`（2スペースのインデント）
5. The AI Wiki System shall サイドバーの内容を日本語で生成する（セクションタイトルとページタイトル）

### Requirement 5: 既存コマンドとの統合
**Objective:** 開発者として、サイドバー生成機能が既存の`/ai-wiki/make`コマンドにシームレスに統合されるようにしたい。これにより、追加のコマンド実行なしでサイドバーも生成される。

#### Acceptance Criteria
1. The AI Wiki System shall サイドバー生成処理を`/ai-wiki/make`コマンドの処理フローに統合する
2. The AI Wiki System shall サイドバー生成をWikiページ生成の完了後、または適切なタイミングで実行する
3. The AI Wiki System shall サイドバー生成に必要な情報（`wiki_structure.json`の内容）を既存の処理から取得する
4. The AI Wiki System shall サイドバー生成のエラーが発生しても、Wikiページ生成の結果に影響を与えない（または適切にエラーハンドリングする）
5. The AI Wiki System shall コマンド定義（`.cursor/commands/ai-wiki/make.md`）にサイドバー生成の処理を追加する

### Requirement 6: エラーハンドリングと検証
**Objective:** 開発者として、サイドバー生成中に発生するエラーが適切に処理され、ユーザーに明確なフィードバックが提供されるようにしたい。これにより、問題の特定と解決が容易になる。

#### Acceptance Criteria
1. When サイドバー生成中にエラーが発生する, the AI Wiki System shall 明確で理解しやすいエラーメッセージを日本語で報告する
2. The AI Wiki System shall エラーの種類（ファイル書き込み失敗、構造解析エラーなど）に応じて適切なメッセージを表示する
3. If サイドバー生成に失敗する, the AI Wiki System shall Wikiページの生成結果には影響を与えず、サイドバー生成の失敗のみを報告する
4. The AI Wiki System shall サイドバー生成の成功/失敗を完了報告に含める
5. The AI Wiki System shall 生成されたサイドバーファイルのパスをユーザーに報告する

### Requirement 7: サイドバーの構造とフォーマット
**Objective:** 開発者として、生成されるサイドバーが読みやすく、論理的な構造を持つようにしたい。これにより、ユーザーがWikiを効率的にナビゲートできる。

#### Acceptance Criteria
1. The AI Wiki System shall サイドバーの先頭にWiki全体のタイトル（`wiki_structure.json`の`title`）を含める（オプション、実装方針による）
2. The AI Wiki System shall 各セクションを明確に区別できる形式で表示する
3. The AI Wiki System shall セクション内のページを適切にインデントして表示する
4. The AI Wiki System shall 空行を適切に使用して、可読性を向上させる
5. The AI Wiki System shall サイドバーの全体構造が`wiki_structure.json`の構造を正確に反映する

