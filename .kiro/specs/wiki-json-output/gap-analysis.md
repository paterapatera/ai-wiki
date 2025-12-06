# Gap Analysis: wiki-json-output

## Current State Investigation

### Existing Assets

#### コマンド定義ファイル
- **Location**: `.cursor/commands/ai-wiki/init.md`
- **Current Behavior**: 
  - GitHubリポジトリ（`${owner}/${repo}`）を分析
  - ファイルツリーとREADMEを読み取り
  - Wiki構造をXML形式で出力するよう指示
  - 包括的ビュー（`isComprehensiveView`）と簡潔ビューをサポート
  - 多言語対応（日本語、英語、中国語など）

#### アーキテクチャパターン
- **Markdownベースのコマンド定義**: CursorエージェントがMarkdownファイルを読み取り、`<instructions>`セクションの指示に従って処理を実行
- **動的変数展開**: `${variable}`形式で変数を展開（`${owner}`, `${repo}`, `${language}`, `${isComprehensiveView}`など）
- **出力形式の指示**: 現在はXML形式での出力を明示的に指示

#### 既存のパターンと制約
- コマンド定義は`.cursor/commands/`ディレクトリに配置
- 各コマンドは独立したMarkdownファイル
- ファイル出力やディレクトリ操作はエージェントの動的処理に依存（明示的な実装コードなし）

### Integration Surfaces

- **入力**: GitHubリポジトリ情報（`${owner}/${repo}`）またはファイルツリー（`${fileTree}`）
- **出力**: 現在はXML形式のテキスト出力（エージェントが生成）
- **設定**: 言語設定（`${language}`）、ビュー設定（`${isComprehensiveView}`）

## Requirements Feasibility Analysis

### Technical Needs from Requirements

#### Requirement 1: JSON形式でのWiki構造出力
- **Need**: XML形式からJSON形式への変更
- **Gap**: 現在のコマンド定義はXML形式を明示的に指示
- **Complexity**: 低（出力形式の指示変更のみ）

#### Requirement 2: 出力先ディレクトリの管理
- **Need**: `.wiki/`ディレクトリへのファイル出力
- **Gap**: 現在のコマンド定義には出力先の指定がない（エージェントが動的に処理）
- **Complexity**: 中（ディレクトリ作成とファイル書き込みの指示が必要）

#### Requirement 3: 再実行時の処理
- **Need**: 既存の`.wiki/`ディレクトリの削除
- **Gap**: 現在のコマンド定義には再実行時の処理がない
- **Complexity**: 低（削除処理の指示追加）

#### Requirement 4: 入力元ディレクトリの指定
- **Need**: `repo/<指定ディレクトリ>`形式での入力元指定
- **Gap**: 現在は`${owner}/${repo}`形式でGitHubリポジトリを想定
- **Complexity**: 中（ローカルディレクトリパスの処理が必要）

#### Requirement 5: JSONファイルの構造と内容
- **Need**: XML形式と同等の情報をJSON形式で提供
- **Gap**: XML構造からJSON構造へのマッピングが必要
- **Complexity**: 低（構造の変換ロジックは明確）

### Identified Gaps

#### Missing Capabilities
1. **JSON形式での出力指示**: 現在はXML形式のみ
2. **ファイル出力の明示的な指示**: 現在は出力先の指定がない
3. **ディレクトリ操作の指示**: `.wiki/`ディレクトリの作成・削除処理がない
4. **ローカルディレクトリパスの処理**: `repo/<指定ディレクトリ>`形式の入力処理がない

#### Unknowns / Research Needed
1. **Cursorエージェントのファイル操作能力**: 
   - ディレクトリ作成・削除の指示方法
   - ファイル書き込みの指示方法
   - ワークスペースパスの解決方法

2. **JSON構造の最適な形式**:
   - XML構造との1対1マッピングか、よりJSONらしい構造か
   - セクション階層の表現方法

#### Constraints
- **Markdownベースのコマンド定義**: 実装コードではなく、エージェントへの指示として記述する必要がある
- **既存の変数展開パターン**: `${variable}`形式を維持する必要がある
- **後方互換性**: 既存のXML出力機能への影響を考慮（要件では置き換えを想定）

## Implementation Approach Options

### Option A: Extend Existing Command File

**Rationale**: 既存の`init.md`を拡張して、JSON出力とファイル操作の指示を追加

**Which files/modules to extend**:
- `.cursor/commands/ai-wiki/init.md`: 
  - XML出力指示をJSON出力指示に変更
  - ファイル出力とディレクトリ操作の指示を追加
  - 入力元ディレクトリの処理を追加

**Compatibility assessment**:
- ✅ 既存の変数展開パターン（`${variable}`）を維持
- ✅ 既存の多言語対応を維持
- ✅ 既存の包括的ビュー/簡潔ビューのロジックを維持
- ⚠️ XML出力機能は置き換えられる（要件に合致）

**Complexity and maintainability**:
- ファイルサイズが増加するが、単一責任（Wiki構造生成）は維持
- JSON出力とファイル操作の指示を追加するだけなので、認知負荷は低い

**Trade-offs**:
- ✅ 最小限の変更で実現可能
- ✅ 既存のパターンとインフラを活用
- ✅ 単一ファイルで管理が容易
- ❌ ファイルサイズが増加
- ❌ XML出力機能が失われる（要件では意図的）

### Option B: Create New Command File

**Rationale**: 新しいコマンドファイル（例: `init-json.md`）を作成して、JSON出力専用の実装を提供

**Integration points**:
- 既存の`init.md`とは独立したコマンドとして動作
- 同じ変数展開パターンを使用
- 同じ入力処理ロジックを共有（`repo/<指定ディレクトリ>`形式）

**Responsibility boundaries**:
- 新コマンド: JSON形式でのWiki構造生成とファイル出力
- 既存コマンド: XML形式でのWiki構造生成（維持）

**Trade-offs**:
- ✅ 既存機能を保持（後方互換性）
- ✅ 責任の分離が明確
- ✅ テストとメンテナンスが容易
- ❌ コード重複の可能性
- ❌ 2つのコマンドを管理する必要がある
- ❌ 要件ではXML出力の置き換えを想定（新規作成は要件と不一致）

### Option C: Hybrid Approach

**Rationale**: 既存コマンドを拡張しつつ、出力形式を設定可能にする

**Combination strategy**:
- 既存の`init.md`を拡張
- 出力形式を変数（`${outputFormat}`）で制御可能にする
- デフォルトをJSON形式に設定（要件に合致）

**Phased implementation**:
- Phase 1: JSON出力機能を追加（XML出力は維持）
- Phase 2: デフォルトをJSONに変更、XML出力は非推奨
- Phase 3: XML出力を削除（要件に合致）

**Trade-offs**:
- ✅ 段階的な移行が可能
- ✅ 既存ユーザーへの影響を最小化
- ❌ 実装が複雑になる
- ❌ 要件ではXML出力の置き換えを想定（段階的移行は不要かも）

## Implementation Complexity & Risk

### Effort: **S (1-3 days)**

**Justification**:
- 既存のコマンド定義パターンに従うだけ
- 出力形式の指示変更とファイル操作の指示追加のみ
- 実装コードは不要（Markdown指示の修正のみ）
- 既存の変数展開とロジックを再利用

### Risk: **Low**

**Justification**:
- 既存のパターンと技術スタックを使用
- 明確なスコープ（出力形式の変更とファイル操作）
- 実装コードが不要（エージェントが動的に処理）
- 既存機能への影響が限定的（要件では置き換えを想定）

## Recommendations for Design Phase

### Preferred Approach: **Option A (Extend Existing Command File)**

**Key Decisions**:
1. 既存の`init.md`を拡張してJSON出力機能を実装
2. XML出力指示をJSON出力指示に置き換え
3. ファイル出力とディレクトリ操作の指示を追加
4. 入力元ディレクトリの処理を`repo/<指定ディレクトリ>`形式に対応

**Rationale**:
- 要件ではXML出力の置き換えを想定しているため、新規作成（Option B）や段階的移行（Option C）は不要
- 既存のパターンを最大限活用し、最小限の変更で実現可能
- 単一ファイルで管理が容易

### Research Items to Carry Forward

1. **Cursorエージェントのファイル操作指示方法**:
   - ディレクトリ作成・削除の指示形式
   - ファイル書き込みの指示形式
   - ワークスペースパスの解決方法
   - エラーハンドリングの指示方法

2. **JSON構造の設計**:
   - XML構造とのマッピング方法
   - セクション階層の表現方法（ネストされたオブジェクト vs フラットな配列）
   - メタデータの表現方法

3. **入力元ディレクトリの処理**:
   - `repo/<指定ディレクトリ>`形式のパース方法
   - ディレクトリ存在確認の指示方法
   - ファイルツリー取得の方法

### Design Phase Focus Areas

1. **JSON構造の詳細設計**: XML構造からJSON構造への完全なマッピング
2. **ファイル操作の指示設計**: エージェントへの明確な指示形式
3. **エラーハンドリング**: ディレクトリ不存在、権限エラーなどの処理
4. **変数展開の拡張**: 必要に応じて新しい変数（`${outputDir}`, `${inputDir}`など）の追加
