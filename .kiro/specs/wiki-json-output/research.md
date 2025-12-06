# Research & Design Decisions

---
**Purpose**: Capture discovery findings, architectural investigations, and rationale that inform the technical design.

**Usage**:
- Log research activities and outcomes during the discovery phase.
- Document design decision trade-offs that are too detailed for `design.md`.
- Provide references and evidence for future audits or reuse.
---

## Summary
- **Feature**: `wiki-json-output`
- **Discovery Scope**: Extension（既存コマンドの拡張）
- **Key Findings**:
  - 既存の`init.md`コマンドはXML形式で出力するよう指示している
  - CursorエージェントはMarkdown指示に従って動的に処理を実行する
  - ファイル操作（ディレクトリ作成・削除、ファイル書き込み）はエージェントの能力に依存
  - JSON構造はXML構造と1対1でマッピング可能

## Research Log

### Cursorエージェントのファイル操作能力
- **Context**: 要件2, 3で`.wiki/`ディレクトリの作成・削除とファイル出力が必要
- **Sources Consulted**: 
  - 既存のコマンド定義（`.cursor/commands/ai-wiki/init.md`）
  - プロジェクトのsteeringコンテキスト（`.kiro/steering/tech.md`）
- **Findings**: 
  - CursorエージェントはMarkdownファイルの`<instructions>`セクションの指示に従って処理を実行
  - エージェントはファイル操作ツール（write, delete_file, run_terminal_cmdなど）を使用可能
  - ディレクトリ操作は`run_terminal_cmd`で`mkdir -p`や`rm -rf`を実行する指示で実現可能
  - ファイル書き込みは`write`ツールを使用する指示で実現可能
- **Implications**: 
  - コマンド定義に明示的なファイル操作の指示を追加する必要がある
  - エラーハンドリング（ディレクトリ不存在、権限エラー）の指示も含める必要がある

### JSON構造の設計
- **Context**: 要件1, 5でXML形式からJSON形式への変換が必要
- **Sources Consulted**: 
  - 既存のXML構造（`init.md`の出力形式定義）
  - JSON標準仕様
- **Findings**: 
  - XML構造は`<wiki_structure>`, `<sections>`, `<pages>`などの階層構造
  - JSON構造はネストされたオブジェクトと配列で表現可能
  - セクション階層はネストされたオブジェクトで表現する方が自然
  - ページ情報は配列として管理し、セクション参照で関連付け
- **Implications**: 
  - XML構造をそのままJSONにマッピングするのではなく、JSONらしい構造に変換
  - セクション階層は`subsections`配列で表現
  - ページは`pages`配列で管理し、セクション参照は`parent_section`で表現

### 入力元ディレクトリの処理
- **Context**: 要件4で`repo/<指定ディレクトリ>`形式の入力処理が必要
- **Sources Consulted**: 
  - 既存のコマンド定義（`${owner}/${repo}`形式の変数展開）
  - プロジェクト構造（`repo/`ディレクトリの想定）
- **Findings**: 
  - 既存コマンドはGitHubリポジトリ（`${owner}/${repo}`）を想定
  - ローカルディレクトリパス（`repo/<指定ディレクトリ>`）の処理が必要
  - コマンド引数からディレクトリパスを取得し、変数展開で使用可能
  - ディレクトリ存在確認はエージェントのツールで実行可能
- **Implications**: 
  - コマンド引数から入力ディレクトリを取得する処理を追加
  - ディレクトリ存在確認とエラーハンドリングの指示を追加
  - ファイルツリー取得の方法を明確化

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| 既存コマンド拡張 | `init.md`を修正してJSON出力機能を追加 | 最小限の変更、既存パターン活用 | XML出力機能が失われる（要件では意図的） | 推奨アプローチ |
| 新規コマンド作成 | `init-json.md`を新規作成 | 既存機能保持 | コード重複、管理コスト増 | 要件と不一致 |

## Design Decisions

### Decision: JSON構造の設計
- **Context**: XML構造からJSON構造へのマッピング方法
- **Alternatives Considered**:
  1. XML構造をそのままJSONに変換（1対1マッピング）
  2. JSONらしい構造に再設計（ネストされたオブジェクト、配列の活用）
- **Selected Approach**: JSONらしい構造に再設計
- **Rationale**: 
  - JSONの特性（ネストされたオブジェクト、配列）を最大限活用
  - セクション階層を自然に表現可能
  - 将来の拡張性が高い
- **Trade-offs**: 
  - メリット: JSONらしい構造、拡張性、可読性
  - デメリット: XML構造との完全な1対1マッピングではない（ただし、情報は同等）
- **Follow-up**: 実装時にXML構造との情報の完全性を確認

### Decision: ファイル操作の指示方法
- **Context**: ディレクトリ作成・削除、ファイル書き込みの指示形式
- **Alternatives Considered**:
  1. エージェントに明示的な指示を記述（`mkdir -p`, `rm -rf`, `write`ツールの使用）
  2. エージェントの自動判断に任せる
- **Selected Approach**: エージェントに明示的な指示を記述
- **Rationale**: 
  - 動作が明確で予測可能
  - エラーハンドリングも明示的に指示可能
  - 既存のパターンと一貫性がある
- **Trade-offs**: 
  - メリット: 明確性、予測可能性、エラーハンドリング
  - デメリット: 指示が長くなる（ただし、可読性は維持）
- **Follow-up**: 実装時にエラーハンドリングの動作を確認

### Decision: 入力元ディレクトリの処理方法
- **Context**: `repo/<指定ディレクトリ>`形式の入力処理
- **Alternatives Considered**:
  1. コマンド引数から直接取得
  2. 変数展開（`${inputDir}`）を使用
- **Selected Approach**: コマンド引数から直接取得し、変数展開で使用
- **Rationale**: 
  - 既存の変数展開パターン（`${owner}`, `${repo}`）と一貫性がある
  - 柔軟性が高い
  - エラーハンドリングが容易
- **Trade-offs**: 
  - メリット: 一貫性、柔軟性、エラーハンドリング
  - デメリット: 変数展開の追加が必要
- **Follow-up**: 実装時にディレクトリ存在確認の動作を確認

## Risks & Mitigations
- **リスク**: Cursorエージェントのファイル操作能力が期待通りでない
  - **緩和策**: 明示的な指示を記述し、エラーハンドリングも含める
- **リスク**: JSON構造がXML構造と情報の完全性を保てない
  - **緩和策**: 設計時にXML構造の全要素をJSON構造にマッピングすることを確認
- **リスク**: 既存のXML出力機能を使用しているユーザーへの影響
  - **緩和策**: 要件ではXML出力の置き換えを想定しているため、これは意図的な変更

## References
- 既存コマンド定義: `.cursor/commands/ai-wiki/init.md`
- プロジェクトsteering: `.kiro/steering/tech.md`, `.kiro/steering/structure.md`
- JSON標準仕様: RFC 8259
