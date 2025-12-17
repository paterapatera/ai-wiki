# Research & Design Decisions Template

---
**Purpose**: Capture discovery findings, architectural investigations, and rationale that inform the technical design.

**Usage**:
- Log research activities and outcomes during the discovery phase.
- Document design decision trade-offs that are too detailed for `design.md`.
- Provide references and evidence for future audits or reuse.
---

## Summary
- **Feature**: `wiki-section-persistence`
- **Discovery Scope**: Extension（既存システムの拡張）
- **Key Findings**:
  - 既存の`init.md`コマンドは出力先ディレクトリを削除する前に既存の`wiki_structure.json`を読み込む処理がない
  - `make.md`コマンドでは`.wiki/wiki_structure.json`を読み込むパターンが確立されている
  - 既存のJSON検証パターン（Docker使用）を再利用可能
  - セクション構成の維持には、既存のJSON構造と新規分析結果の比較ロジックが必要

## Research Log

### 既存のinit.mdコマンドの処理フロー分析
- **Context**: `/ai-wiki/init`コマンドの現在の実装を理解し、統合ポイントを特定する必要があった
- **Sources Consulted**: `.cursor/commands/ai-wiki/init.md`
- **Findings**:
  - Step 1: 入力ディレクトリの取得と確認
  - Step 2: 出力ディレクトリの準備（既存ディレクトリを削除して再作成）
  - Step 3: 入力ディレクトリの分析（ファイルツリーとREADME読み取り）
  - Step 4: Wiki構造の生成（JSON形式）
  - Step 5: JSONファイルの出力と検証
  - 現在、Step 2でディレクトリを削除する前に既存の`wiki_structure.json`を読み込む処理がない
- **Implications**: Step 2の前に新しいステップを追加して、既存の`wiki_structure.json`を読み込む必要がある

### make.mdコマンドのJSON読み込みパターン
- **Context**: 既存のコマンドで`wiki_structure.json`を読み込むパターンを確認
- **Sources Consulted**: `.cursor/commands/ai-wiki/make.md`
- **Findings**:
  - `make.md`では`.wiki/wiki_structure.json`を読み込むパターンが確立されている
  - JSON構文検証にDockerコンテナ（`python:3-alpine`）を使用
  - ファイルが存在しない場合のエラーハンドリングパターンが定義されている
- **Implications**: 同様のパターンを`init.md`に適用可能。ただし、`init.md`では`[出力先ディレクトリ]/wiki_structure.json`のパスを使用する必要がある

### wiki_structure.jsonの構造分析
- **Context**: 既存のJSON構造を理解し、セクション情報を維持する方法を検討
- **Sources Consulted**: `.wiki/wiki_structure.json`
- **Findings**:
  - JSON構造には`sections`配列と`pages`配列が含まれる
  - 各セクションには`id`、`title`、`pages`、`subsections`が含まれる
  - 各ページには`id`、`title`、`parent_section`、`related_pages`が含まれる
  - `project_root`フィールドでプロジェクトのルートディレクトリを識別
- **Implications**: セクション構成を維持するには、既存の`sections`配列を保持し、新しいページを既存のセクション構造にマッピングする必要がある

### 構成比較と判断ロジック
- **Context**: 既存の構成と新規分析結果を比較し、構成を維持するかどうかを判断する方法を検討
- **Sources Consulted**: 要件ドキュメント、既存のJSON構造
- **Findings**:
  - `project_root`の一致が構成類似度の重要な判断基準
  - セクション数、ページ数、階層構造の一致度も評価対象
  - 内容が大きく乖離する場合は構成を維持しなくてもよい
- **Implications**: 類似度評価ロジックを実装し、閾値に基づいて構成維持の判断を行う必要がある

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| 既存コマンド拡張パターン | `init.md`の既存処理フローに新しいステップを追加 | 既存パターンを維持、一貫性が保たれる | 処理フローが複雑になる可能性 | 既存の`make.md`パターンと整合性がある |
| 独立モジュールパターン | セクション維持ロジックを別モジュールとして分離 | 関心の分離、再利用性 | コマンド定義システムでは適用困難 | Markdownベースのコマンド定義システムでは不適切 |

## Design Decisions

### Decision: 既存のwiki_structure.jsonの読み込みタイミング
- **Context**: 出力先ディレクトリを削除する前に既存のJSONファイルを読み込む必要がある
- **Alternatives Considered**:
  1. Step 2の前に新しいステップ（Step 1.5）を追加 — 既存のJSONを読み込む
  2. Step 2の削除処理の前に読み込み処理を挿入 — Step 2内で処理
- **Selected Approach**: Step 1とStep 2の間に新しいステップ（Step 1.5）を追加し、既存の`wiki_structure.json`を読み込む
- **Rationale**: 処理フローが明確になり、既存のステップ番号を変更する必要がない
- **Trade-offs**: ステップ数が増えるが、可読性と保守性が向上する
- **Follow-up**: 実装時に既存のステップ番号との整合性を確認

### Decision: 構成類似度の評価方法
- **Context**: 既存の構成と新規分析結果を比較し、構成を維持するかどうかを判断する必要がある
- **Alternatives Considered**:
  1. 単純な一致チェック（`project_root`のみ） — シンプルだが柔軟性に欠ける
  2. 複合的な類似度評価（`project_root`、セクション数、ページ数など） — 柔軟だが複雑
- **Selected Approach**: `project_root`の一致を必須条件とし、セクション構造の類似度も評価する複合的なアプローチ
- **Rationale**: `project_root`が異なる場合は明らかに別プロジェクトなので、構成維持は不要。同じプロジェクトでも内容が大きく変化した場合は柔軟に対応できる
- **Trade-offs**: 評価ロジックが複雑になるが、より適切な判断が可能
- **Follow-up**: 実装時に類似度の閾値を調整可能にする

### Decision: セクション構成の維持方法
- **Context**: 既存のセクション構成を維持しながら新しいWiki構造を生成する方法
- **Alternatives Considered**:
  1. 既存の`sections`配列をそのまま使用 — シンプルだが新規セクション追加が困難
  2. 既存のセクションをベースに、必要に応じて新規セクションを追加 — 柔軟だが複雑
- **Selected Approach**: 既存の`sections`配列をベースに維持し、新規分析で必要なセクションのみ追加する
- **Rationale**: 既存のセクション構造を最大限維持しつつ、プロジェクトの変化に対応できる
- **Trade-offs**: 新規セクションの追加ロジックが必要になるが、既存構成の維持と柔軟性のバランスが取れる
- **Follow-up**: 実装時に新規セクション追加の判断基準を明確にする

## Risks & Mitigations
- **既存のJSONファイルが破損している場合** — JSON構文検証でエラーを検出し、新規作成モードにフォールバック
- **構成類似度の評価が不適切な場合** — 初期実装では保守的なアプローチ（`project_root`一致を必須）を採用し、必要に応じて調整
- **既存のページIDと新規分析結果のマッチングが困難な場合** — ページIDの生成ロジックを既存のパターンと整合性を保つように設計

## References
- `.cursor/commands/ai-wiki/init.md` — 既存のinitコマンド実装
- `.cursor/commands/ai-wiki/make.md` — JSON読み込みパターンの参考
- `.wiki/wiki_structure.json` — 既存のJSON構造定義
- `.kiro/specs/make-wiki-command/design.md` — 類似機能の設計参考

