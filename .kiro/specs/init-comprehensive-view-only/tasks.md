# Implementation Plan


- [ ] 1. コマンド定義ファイルから簡潔ビュー関連のコードとドキュメントを削除
- [ ] 1.1 `<background_information>`セクションから簡潔ビューの説明を削除
  - 30-31行目の出力形式の説明から簡潔ビューの記述を削除
  - 包括的ビューの説明のみを残す
  - _Requirements: 2.3, 3.2_
- [ ] 1.2 `<instructions>`セクションから条件分岐と簡潔ビュー生成ロジックを削除
  - 96行目の`${isComprehensiveView ? \``を削除し、直接包括的ビューのコードを記述
  - 144行目の` : \``（簡潔ビューの開始）から168行目の`\`}`（条件分岐の終了）までを削除
  - 136行目の`IMPORTANT: For comprehensive view (${isComprehensiveView} is true):`を`IMPORTANT: For comprehensive view:`に変更
  - 162行目の`IMPORTANT: For concise view (${isComprehensiveView} is false):`から167行目までの簡潔ビュー関連の記述を削除
  - 181行目の`${isComprehensiveView ? '8-12' : '4-6'}`を`8-12`に固定
  - 181行目の`${isComprehensiveView ? 'comprehensive' : 'concise'}`を`comprehensive`に固定
  - _Requirements: 1.1, 1.4, 2.1, 2.2, 3.1, 3.3, 3.5_
- [ ] 1.3 `Output Description`セクションから簡潔ビューの記述を削除
  - 226-227行目の簡潔ビューに関する記述を削除
  - 包括的ビューの記述のみを残す
  - _Requirements: 2.3, 3.4_

- [ ] 2. 修正後のコマンド定義の動作を検証
- [ ] 2.1 既存の包括的ビュー生成ロジックが正常に動作することを確認
  - `/ai-wiki/init`コマンドを実行して、包括的ビュー形式のJSON構造が生成されることを確認
  - 生成されるJSON構造に`sections`配列が含まれることを確認
  - 生成されるJSON構造の各ページオブジェクトに`parent_section`プロパティが含まれることを確認
  - 生成されるJSON構造に8-12ページが含まれることを確認
  - 生成されるJSON構造にサブセクション階層構造が含まれることを確認
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 4.1, 4.2_
- [ ] 2.2 簡潔ビュー関連のコードが残っていないことを確認
  - `${isComprehensiveView}`変数参照が存在しないことを確認
  - 簡潔ビュー生成のコードパスが存在しないことを確認
  - 簡潔ビューに関するドキュメント記述が存在しないことを確認
  - _Requirements: 2.1, 2.2, 2.3, 3.1_
- [ ] 2.3 既存の変数とファイル操作パターンが維持されていることを確認
  - `${inputDir}`, `${fileTree}`, `${readme}`変数が正常に使用されていることを確認
  - `.wiki/`ディレクトリの準備とJSON検証が正常に動作することを確認
  - _Requirements: 4.3, 4.4_
