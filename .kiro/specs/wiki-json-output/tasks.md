# Implementation Plan

## Task Format Template

Use whichever pattern fits the work breakdown:

### Major task only
- [ ] {{NUMBER}}. {{TASK_DESCRIPTION}}{{PARALLEL_MARK}}
  - {{DETAIL_ITEM_1}} *(Include details only when needed. If the task stands alone, omit bullet items.)*
  - _Requirements: {{REQUIREMENT_IDS}}_

### Major + Sub-task structure
- [ ] {{MAJOR_NUMBER}}. {{MAJOR_TASK_SUMMARY}}
- [ ] {{MAJOR_NUMBER}}.{{SUB_NUMBER}} {{SUB_TASK_DESCRIPTION}}{{SUB_PARALLEL_MARK}}
  - {{DETAIL_ITEM_1}}
  - {{DETAIL_ITEM_2}}
  - _Requirements: {{REQUIREMENT_IDS}}_ *(IDs only; do not add descriptions or parentheses.)*

> **Parallel marker**: Append `(P)` only to tasks that can be executed in parallel. Omit the marker when running in `--sequential` mode.
>
> **Optional test coverage**: When a sub-task is deferrable test work tied to acceptance criteria, mark the checkbox as `- [ ]*` and explain the referenced requirements in the detail bullets.

## Implementation Tasks

- [x] 1. コマンド定義の修正準備と入力ディレクトリ処理の実装
- [x] 1.1 既存のinit.mdコマンド定義を確認し、変更箇所を特定する
  - 既存のXML出力指示を確認
  - 変数展開パターン（${language}, ${isComprehensiveView}など）を確認
  - 変更が必要なセクションを特定
  - _Requirements: 4.1, 4.3_

- [x] 1.2 コマンド引数から入力ディレクトリを取得する指示を追加する
  - コマンド引数から入力ディレクトリパスを取得する処理を追加
  - 取得したパスを${inputDir}変数として設定する指示を追加
  - ワークスペースルートからの相対パスとして解決する処理を追加
  - _Requirements: 4.1, 4.3_

- [x] 1.3 入力ディレクトリの存在確認とエラーハンドリングを実装する
  - ${inputDir}で指定されたディレクトリの存在確認指示を追加
  - ディレクトリが存在しない場合のエラーメッセージ報告指示を追加
  - エラー時の処理中断指示を追加
  - _Requirements: 4.2, 4.4_

- [x] 2. ディレクトリ操作機能の実装
- [x] 2.1 再実行時のディレクトリ削除処理を実装する
  - .wiki/ディレクトリの存在確認指示を追加
  - 存在する場合にrm -rf .wiki/を実行する指示を追加
  - 削除処理のエラーハンドリングを追加
  - _Requirements: 3.1, 3.2_

- [x] 2.2 出力ディレクトリの自動作成処理を実装する
  - .wiki/ディレクトリが存在しない場合の確認処理を追加
  - mkdir -p .wiki/を実行する指示を追加
  - ディレクトリ作成のエラーハンドリングを追加
  - _Requirements: 2.2, 2.3_

- [x] 3. JSON出力形式への変更
- [x] 3.1 XML出力指示をJSON出力指示に変更する
  - 既存のXML形式の出力指示を削除
  - JSON形式での出力を指示する内容に変更
  - 有効なJSON構文に準拠することを明示
  - _Requirements: 1.1, 1.2, 1.3_

- [x] 3.2 包括的ビュー時のJSON構造生成指示を実装する
  - ${isComprehensiveView}がtrueの場合の処理を追加
  - sections配列を含む完全なJSON構造を生成する指示を追加
  - セクション階層構造（subsections配列）を含める指示を追加
  - ページのparent_sectionプロパティを含める指示を追加
  - _Requirements: 5.1, 5.3_

- [x] 3.3 簡潔ビュー時のJSON構造生成指示を実装する
  - ${isComprehensiveView}がfalseの場合の処理を追加
  - sectionsプロパティを省略したJSON構造を生成する指示を追加
  - ページのparent_sectionプロパティを省略する指示を追加
  - pages配列のみを含むJSON構造を生成する指示を追加
  - _Requirements: 5.1, 5.3_

- [x] 3.4 XML構造と同等の情報をJSON形式で提供する指示を実装する
  - XML構造の全要素をJSON構造にマッピングする指示を追加
  - タイトル、説明、セクション、ページ情報を含める指示を追加
  - 各ページのタイトル、説明、重要度、関連ファイル、関連ページ情報を含める指示を追加
  - _Requirements: 1.4, 5.2, 5.4_

- [x] 4. ファイル出力機能の実装
- [x] 4.1 JSON構造をファイルに書き込む指示を実装する
  - 生成されたJSON構造を.wiki/wiki_structure.jsonに書き込む指示を追加
  - writeツールを使用するようエージェントに指示
  - ファイル書き込みのエラーハンドリングを追加
  - _Requirements: 2.1, 2.3_

- [x] 4.2 ファイル出力の検証と完了通知を実装する
  - JSONファイルが正しく生成されたことを確認する指示を追加
  - 有効なJSON構文に準拠していることを確認する指示を追加
  - 完了通知をユーザーに報告する指示を追加
  - _Requirements: 1.3, 3.3_

- [x] 5. 統合と最終検証
- [x] 5.1 全機能の統合テストを実施する
  - コマンド実行からJSONファイル生成までの完全なフローを検証
  - 包括的ビューと簡潔ビューの両方でJSON構造が正しく生成されることを確認
  - 再実行時のディレクトリ削除と再生成が正しく動作することを確認
  - 入力ディレクトリ不存在時のエラーハンドリングが正しく動作することを確認
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 2.1, 2.2, 2.3, 3.1, 3.2, 3.3, 4.1, 4.2, 4.3, 4.4, 5.1, 5.2, 5.3, 5.4_
