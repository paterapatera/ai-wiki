# Requirements Document

## Introduction

この機能は、`/ai-wiki/init`コマンドを「包括的ビュー」のみを生成するように制限します。既存のコマンド定義では`${isComprehensiveView}`変数による条件分岐で包括的ビューと簡潔ビューの両方をサポートしていますが、この機能により簡潔ビューのオプションを削除し、常に包括的ビュー（セクション階層構造とページの親子関係を含む完全なJSON構造）のみを生成するようにします。

## Requirements

### Requirement 1: 包括的ビューの固定化

**Objective:** As a 開発者, I want `/ai-wiki/init`コマンドが常に包括的ビューを生成すること, so that 一貫したWiki構造形式を保証し、簡潔ビューと包括的ビューの選択の複雑さを排除できる

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドが実行される, the Wiki構造生成コマンド shall 常に包括的ビュー形式のJSON構造を生成する
2. The Wiki構造生成コマンド shall 生成されるJSON構造に`sections`配列を含む
3. The Wiki構造生成コマンド shall 生成されるJSON構造の各ページオブジェクトに`parent_section`プロパティを含む
4. The Wiki構造生成コマンド shall 8-12ページを含む包括的なWiki構造を生成する
5. The Wiki構造生成コマンド shall サブセクション階層構造をサポートする

### Requirement 2: 簡潔ビュー機能の削除

**Objective:** As a 開発者, I want 簡潔ビューのオプションが削除されること, so that コマンド定義が簡素化され、包括的ビューのみに焦点を当てられる

#### Acceptance Criteria
1. The Wiki構造生成コマンド shall `${isComprehensiveView}`変数による条件分岐を含まない
2. The Wiki構造生成コマンド shall 簡潔ビュー生成のためのコードパスを含まない
3. The Wiki構造生成コマンド shall 簡潔ビューに関するドキュメント記述を含まない
4. If 簡潔ビュー形式のJSON構造が要求される, then the Wiki構造生成コマンド shall 包括的ビュー形式を生成する（簡潔ビューは利用不可）

### Requirement 3: コマンド定義の更新

**Objective:** As a 開発者, I want コマンド定義ファイルが包括的ビューのみを扱うように更新されること, so that コマンドの動作が明確で一貫性が保たれる

#### Acceptance Criteria
1. The Wiki構造生成コマンド shall `.cursor/commands/ai-wiki/init.md`ファイル内の`${isComprehensiveView}`変数参照を削除する
2. The Wiki構造生成コマンド shall コマンド定義の`<background_information>`セクションから簡潔ビューの説明を削除する
3. The Wiki構造生成コマンド shall コマンド定義の`<instructions>`セクションから簡潔ビュー生成ロジックを削除する
4. The Wiki構造生成コマンド shall コマンド定義の`Output Description`セクションから簡潔ビューの記述を削除する
5. The Wiki構造生成コマンド shall 包括的ビューの生成ロジックのみを含む

### Requirement 4: 後方互換性の維持

**Objective:** As a 既存ユーザー, I want 既存の包括的ビュー機能が維持されること, so that 既存のワークフローに影響を与えずに機能を利用できる

#### Acceptance Criteria
1. The Wiki構造生成コマンド shall 既存の包括的ビュー生成ロジックを維持する
2. The Wiki構造生成コマンド shall 既存のJSON構造形式（`sections`配列、`pages`配列、`parent_section`プロパティ）を維持する
3. The Wiki構造生成コマンド shall 既存の変数（`${inputDir}`, `${fileTree}`, `${readme}`）の使用を維持する
4. The Wiki構造生成コマンド shall 既存のファイル操作パターン（`.wiki/`ディレクトリの準備、JSON検証）を維持する
