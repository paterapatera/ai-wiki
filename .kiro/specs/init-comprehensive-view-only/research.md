# Research & Design Decisions: init-comprehensive-view-only

---
**Purpose**: Capture discovery findings, architectural investigations, and rationale that inform the technical design.

**Usage**:
- Log research activities and outcomes during the discovery phase.
- Document design decision trade-offs that are too detailed for `design.md`.
- Provide references and evidence for future audits or reuse.
---

## Summary
- **Feature**: `init-comprehensive-view-only`
- **Discovery Scope**: Extension（既存システムの拡張）
- **Key Findings**:
  - 既存のコマンド定義ファイル（`.cursor/commands/ai-wiki/init.md`）は`${isComprehensiveView}`変数による条件分岐で包括的ビューと簡潔ビューの両方をサポート
  - 包括的ビューの生成ロジック（96-143行目）は既に実装済みで動作確認済み
  - 簡潔ビューの生成ロジック（144-168行目）と関連ドキュメント記述を削除するだけで要件を満たせる
  - 既存の変数展開パターン（`${inputDir}`, `${fileTree}`, `${readme}`）は変更不要
  - 後方互換性を維持するため、既存の包括的ビュー機能は完全に維持される

## Research Log

### 既存コマンド定義の構造分析
- **Context**: 既存のコマンド定義ファイルの構造と条件分岐の実装を理解する必要があった
- **Sources Consulted**: `.cursor/commands/ai-wiki/init.md`ファイルの直接確認
- **Findings**:
  - 96行目から168行目まで`${isComprehensiveView}`変数による三項演算子形式の条件分岐が実装されている
  - 包括的ビューの生成ロジック（96-143行目）は完全に実装されており、そのまま使用可能
  - 簡潔ビューの生成ロジック（144-168行目）は削除対象
  - 30-31行目と226-227行目に簡潔ビューに関するドキュメント記述がある
  - 181行目にページ数の条件分岐がある（`${isComprehensiveView ? '8-12' : '4-6'}`）
- **Implications**: 条件分岐を削除し、包括的ビューのコードのみを残すことで要件を満たせる。既存の包括的ビュー機能への影響はない。

### Markdownベースのコマンド定義パターン
- **Context**: プロジェクトのアーキテクチャパターンを理解し、設計が既存パターンに準拠することを確認する必要があった
- **Sources Consulted**: `.kiro/steering/tech.md`, `.kiro/steering/structure.md`
- **Findings**:
  - プロジェクトはMarkdownベースのコマンド定義システムを使用
  - Cursorエージェントが`<instructions>`セクションを解釈して実行
  - 動的変数展開（`${variable}`形式）を使用
  - コマンド定義ファイルは`.cursor/commands/`ディレクトリに配置
- **Implications**: 既存のパターンを維持し、変数展開の仕組みを活用する。新規コンポーネントの作成は不要。

### 既存の包括的ビュー機能の確認
- **Context**: 既存の包括的ビュー機能が正常に動作していることを確認し、後方互換性を維持する必要があった
- **Sources Consulted**: `.wiki/wiki_structure.json`（既存の出力例）、`.cursor/commands/ai-wiki/init.md`の包括的ビュー生成ロジック
- **Findings**:
  - 既存の包括的ビュー機能は完全に実装されており、正常に動作している
  - JSON構造は`sections`配列、`pages`配列、`parent_section`プロパティを含む
  - 8-12ページの生成、サブセクション階層構造のサポートが実装されている
- **Implications**: 既存の包括的ビュー機能を完全に維持することで、後方互換性を保証できる。変更は簡潔ビューの削除のみ。

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| 既存コンポーネントの拡張 | `.cursor/commands/ai-wiki/init.md`を修正して条件分岐を削除 | 最小限の変更、既存パターンの維持、後方互換性 | なし | 推奨アプローチ |
| 新規コンポーネントの作成 | 新しいコマンド定義ファイルを作成 | 既存コードへの影響なし | 不要な複雑化、既存パターンからの逸脱 | 不適切 |

## Design Decisions

### Decision: 既存コマンド定義ファイルの修正による実装
- **Context**: 要件を満たすための実装アプローチを決定する必要があった
- **Alternatives Considered**:
  1. 既存コンポーネントの拡張 — `.cursor/commands/ai-wiki/init.md`を修正して条件分岐を削除
  2. 新規コンポーネントの作成 — 新しいコマンド定義ファイルを作成
- **Selected Approach**: 既存コンポーネントの拡張を選択。`.cursor/commands/ai-wiki/init.md`ファイルから`${isComprehensiveView}`変数による条件分岐を削除し、包括的ビューのコードのみを残す。
- **Rationale**: 
  - 既存の包括的ビュー機能は完全に実装されており、そのまま使用可能
  - 最小限の変更で要件を満たせる
  - 既存パターンを維持し、後方互換性を保証できる
  - 新規コンポーネントの作成は不要な複雑化を招く
- **Trade-offs**: 
  - ✅ 最小限の変更、高速な初期開発
  - ✅ 既存パターンとインフラの活用
  - ✅ 後方互換性の維持
  - ❌ 簡潔ビュー機能の完全な削除（要件により意図的）
- **Follow-up**: 
  - 実装後に既存の包括的ビュー機能が正常に動作することを確認
  - 生成されるJSON構造が既存の仕様と一致していることを確認

### Decision: 簡潔ビュー関連コードの完全削除
- **Context**: 簡潔ビュー機能を削除する方法を決定する必要があった
- **Alternatives Considered**:
  1. 完全削除 — 簡潔ビュー関連のコードとドキュメントをすべて削除
  2. コメントアウト — 将来の復元のためにコメントアウトで残す
- **Selected Approach**: 完全削除を選択。簡潔ビュー関連のコードとドキュメント記述をすべて削除する。
- **Rationale**: 
  - 要件により簡潔ビューは不要であり、将来の復元の可能性は低い
  - コメントアウトはコードの可読性を低下させる
  - 完全削除により、コマンド定義が簡素化され、包括的ビューのみに焦点を当てられる
- **Trade-offs**: 
  - ✅ コードの簡素化と可読性の向上
  - ✅ 将来の混乱の回避
  - ❌ 将来の復元が困難（要件により意図的）
- **Follow-up**: 
  - 削除後に簡潔ビュー関連のコードが残っていないことを確認

## Risks & Mitigations
- **既存の包括的ビュー機能への影響** — 既存の包括的ビュー生成ロジックを完全に維持することで影響を回避
- **JSON構造の変更** — 既存のJSON構造形式（`sections`配列、`pages`配列、`parent_section`プロパティ）を維持することで変更を回避
- **変数展開の破壊** — 既存の変数（`${inputDir}`, `${fileTree}`, `${readme}`）の使用を維持することで破壊を回避

## References
- `.cursor/commands/ai-wiki/init.md` — 既存のコマンド定義ファイル
- `.kiro/steering/tech.md` — 技術スタックとアーキテクチャパターン
- `.kiro/steering/structure.md` — プロジェクト構造と命名規則
- `.wiki/wiki_structure.json` — 既存の包括的ビュー出力例
