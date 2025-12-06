# Research & Design Decisions Template

---
**Purpose**: Capture discovery findings, architectural investigations, and rationale that inform the technical design.

**Usage**:
- Log research activities and outcomes during the discovery phase.
- Document design decision trade-offs that are too detailed for `design.md`.
- Provide references and evidence for future audits or reuse.
---

## Summary
- **Feature**: `fix-wiki-links`
- **Discovery Scope**: Extension（既存システムの拡張）
- **Key Findings**:
  - `/ai-wiki/make`コマンドのリンク生成ロジックが`make.md`のStep 4の4番目のステップで定義されている
  - 現在のリンク形式は`[${relatedPageTitle}](${relatedPageId}.md)`で、`.md`拡張子を含んでいる
  - 既存のWikiページにも同様の問題が存在し、一括修正が必要
  - Markdownリンクでは通常`.md`拡張子を省略する必要がある

## Research Log

### 既存リンク生成ロジックの分析
- **Context**: `/ai-wiki/make`コマンドで生成されるWikiページ内のリンクが`.md`拡張子を含んでいる問題を調査
- **Sources Consulted**: 
  - `.cursor/commands/ai-wiki/make.md`のStep 4の4番目のステップ
  - 既存のWikiページ（`repo/slack-agent.wiki/`内のファイル）
- **Findings**: 
  - リンク生成ロジックは`make.md`の202-203行目で定義されている
  - 現在の形式: `[${relatedPageTitle}](${relatedPageId}.md)`
  - 修正後の形式: `[${relatedPageTitle}](${relatedPageId})`
  - 既存のWikiページにも同様の問題が20件以上存在
- **Implications**: 
  - `make.md`のリンク生成ロジックを修正する必要がある
  - 既存のWikiページも一括修正する必要がある
  - 修正は単純な文字列置換で実現可能

### Markdownリンク形式の調査
- **Context**: Markdownリンクで`.md`拡張子を含めるべきかどうかを確認
- **Sources Consulted**: 
  - Markdown仕様と一般的なベストプラクティス
  - 既存のWikiシステムの動作
- **Findings**: 
  - 多くのMarkdownレンダラーは`.md`拡張子を自動的に処理する
  - しかし、多くのWikiシステムやGitHubでは`.md`拡張子を省略するのが標準
  - リンクの一貫性を保つため、`.md`拡張子を除去する方が適切
- **Implications**: 
  - リンク生成ロジックから`.md`拡張子を除去する設計が適切
  - 既存ページも同様に修正する必要がある

## Architecture Pattern Evaluation

この機能は既存システムの単純な拡張であり、新しいアーキテクチャパターンは不要です。既存のコマンド定義ファイル（`make.md`）を修正するだけで実現可能です。

## Design Decisions

### Decision: リンク生成ロジックの修正方法
- **Context**: `/ai-wiki/make`コマンドで生成されるリンクから`.md`拡張子を除去する必要がある
- **Alternatives Considered**:
  1. リンク生成時に`.md`拡張子を除去する処理を追加する方法
  2. リンク生成後に正規表現で置換する方法
- **Selected Approach**: リンク生成時に`.md`拡張子を含めない形式で生成する（方法1）
- **Rationale**: 
  - 生成時に正しい形式で生成する方が、後処理よりも効率的
  - コードの意図が明確になる
  - パフォーマンスのオーバーヘッドが少ない
- **Trade-offs**: 
  - 利点: シンプルで明確な実装、パフォーマンスが良い
  - 欠点: なし
- **Follow-up**: 実装時にリンク生成ロジックが正しく修正されていることを確認

### Decision: 既存ページの修正方法
- **Context**: 既存のWikiページ内のリンクから`.md`拡張子を除去する必要がある
- **Alternatives Considered**:
  1. 手動で修正する方法
  2. スクリプトで一括置換する方法
  3. 新しいコマンドを作成する方法
- **Selected Approach**: 正規表現を使用した一括置換スクリプト（方法2）
- **Rationale**: 
  - 既存ページが多数存在するため、手動修正は非効率
  - 新しいコマンドを作成するほど複雑ではない
  - 正規表現による置換は安全で確実
- **Trade-offs**: 
  - 利点: 効率的、確実、再現可能
  - 欠点: 誤って他の`.md`を含む文字列を置換するリスク（ただし、Markdownリンク構文に限定することで回避可能）
- **Follow-up**: 置換処理が正しく動作することを確認し、バックアップを取る

## Risks & Mitigations
- **リスク**: 既存ページの修正時に誤って他の`.md`を含む文字列を置換する可能性
  - **対策**: 正規表現をMarkdownリンク構文（`](...md)`）に限定することで、誤置換を防止
- **リスク**: 修正後のリンクが正しく機能しない可能性
  - **対策**: 修正後にサンプルページでリンクの動作を確認
- **リスク**: 既存ページの修正が不完全な可能性
  - **対策**: 修正後に全ページをスキャンして、`.md`拡張子を含むリンクが残っていないことを確認

## References
- `.cursor/commands/ai-wiki/make.md` - 既存のリンク生成ロジック
- `repo/slack-agent.wiki/` - 既存のWikiページ（問題の実例）

