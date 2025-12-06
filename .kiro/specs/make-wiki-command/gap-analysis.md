# Gap Analysis: make-wiki-command

## Analysis Summary

- **Scope**: `/ai-wiki/make`コマンドを拡張し、`wiki_structure.json`を読み込んで全ページのWikiコンテンツを生成する機能を実装
- **Key Challenges**: 
  - 既存の`make.md`は単一ページ生成のテンプレートのみで、JSON読み込み・ループ処理・出力先指定機能が未実装
  - セクション構造の維持とページ間リンク生成の実装方法を決定する必要がある
- **Recommendations**: 
  - 既存の`make.md`を大幅に拡張するアプローチ（Option A）を推奨
  - `init.md`のパターン（JSON検証、ディレクトリ操作、エラーハンドリング）を再利用
  - セクション構造はフラットなディレクトリ構造で実装（シンプルさを優先）

## Document Status

この分析は`.kiro/settings/rules/gap-analysis.md`のフレームワークに従って実施されました。

## Current State Investigation

### Domain-Related Assets

**既存のコマンド定義ファイル**:
- `.cursor/commands/ai-wiki/init.md`: `wiki_structure.json`を生成するコマンド
  - JSON検証にDockerを使用するパターンが確立
  - ディレクトリ操作（`rm -rf`, `mkdir -p`）のパターンが確立
  - エラーハンドリングのパターンが確立
- `.cursor/commands/ai-wiki/make.md`: 単一ページ生成のテンプレートのみ
  - Wikiページ生成の指示は詳細に定義済み
  - しかし、JSON読み込み、ループ処理、出力先指定機能は未実装

**既存のデータ構造**:
- `.wiki/wiki_structure.json`: 既存のJSON構造ファイル（実例あり）
  - `title`, `description`, `sections`, `pages`を含む
  - 各ページには`id`, `title`, `description`, `importance`, `relevant_files`, `related_pages`, `parent_section`が含まれる
  - セクションには`id`, `title`, `pages`, `subsections`が含まれる

### Conventions and Patterns

**コマンド定義の構造**:
- `<meta>`セクション: `description`, `argument-hint`を含む
- `<background_information>`セクション: コマンドの目的と機能の説明
- `<instructions>`セクション: ステップバイステップの処理指示
- `<Tool Guidance>`セクション: 使用するツールの説明
- `<Output Description>`セクション: 出力の説明
- `<Safety & Fallback>`セクション: エラーハンドリング

**ツール使用パターン**:
- `read_file`: ファイル読み込み
- `write`: ファイル書き込み
- `run_terminal_cmd`: ディレクトリ操作、JSON検証（Docker使用）
- `list_dir`: ディレクトリ構造の取得
- `codebase_search`: コードベース検索（必要に応じて）

**エラーハンドリングパターン**:
- エラー発生時は処理を中断し、明確なエラーメッセージを日本語で報告
- Docker検証を使用したJSON構文チェック
- ディレクトリ操作の失敗時の適切なエラーメッセージ

**出力パターン**:
- 日本語での出力（`spec.json.language = "ja"`）
- Markdown形式でのWikiページ生成
- ファイルパスはワークスペースルートからの相対パス

### Integration Surfaces

**既存の統合ポイント**:
- `wiki_structure.json`の読み込み: `init.md`で生成されるJSONファイルを読み込む
- ファイル操作: 既存の`init.md`のパターンを再利用可能
- JSON検証: Dockerコンテナを使用した検証パターンが確立

## Requirements Feasibility Analysis

### Technical Needs from Requirements

**Requirement 1: wiki_structure.jsonの読み込み**
- **既存資産**: `read_file`ツールでJSONファイルを読み込む機能は利用可能
- **ギャップ**: JSON構文検証の実装が必要（`init.md`のパターンを再利用可能）
- **制約**: なし

**Requirement 2: 出力先ディレクトリの指定**
- **既存資産**: `run_terminal_cmd`で`mkdir -p`を使用したディレクトリ作成パターンが確立
- **ギャップ**: 引数から出力先ディレクトリを取得する処理が必要
- **制約**: なし

**Requirement 3: Wikiページの生成**
- **既存資産**: `make.md`に詳細なページ生成指示が定義済み
- **ギャップ**: 各ページをループ処理で生成する機能が必要
- **制約**: なし

**Requirement 4: セクション構造の維持**
- **既存資産**: `wiki_structure.json`にセクション構造が定義済み
- **ギャップ**: セクション構造をファイルシステムに反映する方法を決定する必要がある
  - **研究が必要**: セクション構造をディレクトリ階層で表現するか、フラットな構造でインデックスファイルを使用するか
- **制約**: なし

**Requirement 5: ページ間のリンク生成**
- **既存資産**: `wiki_structure.json`に`related_pages`が定義済み
- **ギャップ**: ページIDからファイルパスへのマッピングとリンク生成の実装が必要
- **制約**: セクション構造の実装方法に依存

**Requirement 6: ファイル出力と構造化**
- **既存資産**: `write`ツールでファイル書き込み機能は利用可能
- **ギャップ**: ファイル名の生成ロジック（IDまたはタイトルベース）、無効文字のエスケープ処理が必要
- **制約**: なし

**Requirement 7: エラーハンドリングと検証**
- **既存資産**: `init.md`のエラーハンドリングパターンが確立
- **ギャップ**: 部分的な失敗時の処理方針を決定する必要がある（要件では「成功したページは保存」とあるが、実装方針による）
- **制約**: なし

**Requirement 8: コマンド定義の更新**
- **既存資産**: `make.md`の基本構造は存在
- **ギャップ**: 大幅な拡張が必要（JSON読み込み、ループ処理、出力先指定、セクション構造維持、リンク生成）
- **制約**: 既存のページ生成指示は維持する必要がある

### Complexity Signals

- **Simple**: ファイル読み込み、ディレクトリ作成、ファイル書き込み
- **Algorithmic Logic**: ページIDからファイルパスへのマッピング、リンク生成、ファイル名のエスケープ
- **Workflows**: 全ページのループ処理、エラーハンドリング、進捗報告
- **External Integrations**: なし（Dockerは既存パターン）

## Implementation Approach Options

### Option A: Extend Existing make.md

**概要**: 既存の`make.md`を大幅に拡張して、JSON読み込み、ループ処理、出力先指定、セクション構造維持、リンク生成の機能を追加する。

**拡張するファイル**:
- `.cursor/commands/ai-wiki/make.md`

**拡張内容**:
1. `<meta>`セクション: `description`と`argument-hint`を追加
2. `<background_information>`セクション: コマンドの目的と機能を説明
3. `<instructions>`セクション: 以下のステップを追加
   - Step 1: 出力先ディレクトリの取得と確認
   - Step 2: `wiki_structure.json`の読み込みと検証
   - Step 3: セクション構造の解析
   - Step 4: 各ページの生成ループ
   - Step 5: ページ間リンクの生成
   - Step 6: ファイル出力と完了報告
4. 既存のページ生成指示（`<instructions>`内の詳細な指示）は維持し、各ページ生成時に使用

**互換性評価**:
- ✅ 既存のページ生成指示を維持するため、後方互換性の問題なし
- ✅ 既存のパターン（`init.md`のJSON検証、ディレクトリ操作）を再利用
- ✅ 単一ファイルでの完結により、ナビゲーションが容易

**複雑さと保守性**:
- ⚠️ ファイルサイズが大きくなる可能性（200-300行程度）
- ✅ 単一責任の原則は維持（Wiki生成コマンドとしての責任）
- ✅ 既存のパターンを再利用するため、認知負荷は低い

**トレードオフ**:
- ✅ 最小限の新規ファイル、初期開発が速い
- ✅ 既存のパターンとインフラを活用
- ✅ 既存のページ生成指示を再利用
- ❌ ファイルサイズが大きくなる可能性
- ❌ 既存のロジックが複雑になる可能性

### Option B: Create New Components

**概要**: `make.md`を基本構造のみに保ち、ページ生成の詳細ロジックを別のテンプレートファイルに分離する。

**新規作成するファイル**:
- `.cursor/commands/ai-wiki/make.md`: メインコマンド定義（JSON読み込み、ループ処理、出力先指定）
- `.kiro/settings/templates/wiki/page-generation.md`: ページ生成の詳細指示テンプレート

**統合ポイント**:
- `make.md`がテンプレートファイルを読み込み、各ページ生成時に使用
- テンプレートファイルは変数展開（`${page.title}`, `${page.relevant_files}`など）をサポート

**責任の境界**:
- `make.md`: JSON読み込み、ループ処理、出力先指定、セクション構造維持、リンク生成
- `page-generation.md`: 個別ページのMarkdownコンテンツ生成指示

**トレードオフ**:
- ✅ 関心の分離が明確
- ✅ ページ生成ロジックのテストが容易
- ✅ 既存コンポーネントの複雑さを軽減
- ❌ より多くのファイルをナビゲートする必要がある
- ❌ テンプレートシステムの設計が必要（変数展開の実装方法）

### Option C: Hybrid Approach

**概要**: `make.md`を拡張しつつ、ページ生成の詳細指示は`make.md`内に保持するが、セクション構造維持とリンク生成のロジックを明確に分離する。

**拡張するファイル**:
- `.cursor/commands/ai-wiki/make.md`: メインコマンド定義を大幅に拡張

**分離する部分**:
- セクション構造の維持: フラットなディレクトリ構造で実装（シンプルさを優先）
- リンク生成: ページIDからファイルパスへのマッピングロジックを明確に定義

**段階的実装**:
- Phase 1: JSON読み込み、出力先指定、基本的なページ生成ループ
- Phase 2: セクション構造の維持、ページ間リンクの生成
- Phase 3: エラーハンドリングの強化、進捗報告の改善

**リスク軽減**:
- 各フェーズで動作確認可能
- 既存のページ生成指示を維持

**トレードオフ**:
- ✅ 複雑な機能に対するバランスの取れたアプローチ
- ✅ 反復的な改善が可能
- ❌ より複雑な計画が必要
- ❌ 適切に調整されない場合、一貫性の欠如の可能性

## Implementation Complexity & Risk

### Effort: **M (3-7 days)**

**根拠**:
- 既存のパターン（`init.md`のJSON検証、ディレクトリ操作）を再利用可能
- ページ生成の詳細指示は既に定義済み
- しかし、ループ処理、セクション構造維持、リンク生成の実装が必要
- セクション構造の実装方法の決定が必要（研究項目）

### Risk: **Medium**

**根拠**:
- 既存のパターンに基づいた実装のため、技術的な未知は少ない
- セクション構造の実装方法（ディレクトリ階層 vs フラット構造）の決定が必要
- ページ間リンクの生成ロジックの実装が必要
- 部分的な失敗時の処理方針の決定が必要

## Recommendations for Design Phase

### Preferred Approach: **Option A (Extend Existing make.md)**

**理由**:
1. 既存のページ生成指示を直接再利用できる
2. 既存のパターン（`init.md`のJSON検証、ディレクトリ操作）を再利用できる
3. 単一ファイルでの完結により、ナビゲーションが容易
4. 初期開発が速い

**主要な決定事項**:
1. **セクション構造の実装方法**: フラットなディレクトリ構造を推奨（シンプルさを優先）
   - 各ページを`{page-id}.md`形式で出力先ディレクトリに保存
   - セクション情報はページ内のメタデータとして保持（必要に応じて）
   - または、セクションごとにサブディレクトリを作成（`{section-id}/{page-id}.md`）
2. **ページ間リンクの生成**: ページIDからファイルパスへのマッピングを使用
   - `{page-id}.md`形式の場合: `[Link Text](page-id.md)`
   - セクションサブディレクトリの場合: `[Link Text](section-id/page-id.md)`
3. **エラーハンドリング方針**: 部分的な失敗時は成功したページを保存し、失敗したページについてのみエラーを報告

### Research Items to Carry Forward

1. **セクション構造の実装方法の詳細設計**:
   - フラット構造 vs ディレクトリ階層構造の比較
   - ナビゲーションの容易さ、ファイル管理の容易さの観点から評価
   - 既存のWikiシステム（GitHub Wiki、GitLab Wikiなど）の構造を参考にするか

2. **ページ間リンクの生成ロジックの詳細**:
   - 相対パス vs 絶対パスの選択
   - セクション構造に応じたリンクパスの生成方法
   - 存在しないページへのリンクの扱い（警告表示 vs リンクスキップ）

3. **ファイル名の生成ロジックの詳細**:
   - ページIDベース vs タイトルベースの選択
   - 無効文字のエスケープ方法（URLエンコード vs 置換）
   - ファイル名の長さ制限の考慮

## Next Steps

1. **設計フェーズ**: `/kiro/spec-design make-wiki-command`を実行して、技術設計ドキュメントを作成
2. **研究項目の調査**: セクション構造の実装方法、リンク生成ロジック、ファイル名生成ロジックの詳細を決定
3. **実装準備**: 設計が承認されたら、`/kiro/spec-tasks make-wiki-command`でタスクを生成
