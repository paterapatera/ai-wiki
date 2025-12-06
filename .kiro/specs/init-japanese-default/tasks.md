# Implementation Plan

- [x] 1. init.mdコマンド定義ファイルの修正
- [x] 1.1 86-96行目の多言語対応条件分岐を削除
  - `.cursor/commands/ai-wiki/init.md`の86-96行目にある`${language}`変数を使用した11行の条件分岐を削除
  - 多言語対応の三項演算子チェーン（'en', 'ja', 'zh', 'zh-tw', 'es', 'kr', 'vi', 'pt-br', 'fr', 'ru'）を完全に削除
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 2.2, 2.4, 3.2, 3.3_

- [x] 1.2 固定値"Japanese (日本語)"に置き換え
  - 削除した条件分岐の代わりに、固定文字列"IMPORTANT: The wiki content will be generated in Japanese (日本語) language."を1行で追加
  - 他の変数（`${inputDir}`, `${fileTree}`, `${readme}`, `${isComprehensiveView}`）は変更しない
  - _Requirements: 1.2, 2.1, 2.3, 3.4_

- [x] 1.3 修正後のMarkdown構文検証
  - 修正後の`init.md`ファイルが有効なMarkdown構文であることを確認
  - 他の変数（`${inputDir}`, `${fileTree}`, `${readme}`, `${isComprehensiveView}`）が正しく機能することを確認
  - ファイル全体の構造とセクションが維持されていることを確認
  - _Requirements: 1.4, 2.4_

- [ ] 2. 統合テストと動作確認
- [ ] 2.1 コマンド実行テスト
  - `/ai-wiki/init repo/<指定ディレクトリ>`コマンドを実行し、正常に動作することを確認
  - コマンド引数の形式（`repo/<指定ディレクトリ>`）が変更されていないことを確認
  - 既存の機能（ディレクトリ操作、ファイル読み取り、JSON出力）が正常に動作することを確認
  - _Requirements: 1.1, 1.3, 3.1_

- [ ] 2.2 日本語生成の確認
  - 実際のリポジトリディレクトリに対して`/ai-wiki/init`コマンドを実行
  - 生成された`wiki_structure.json`が日本語で記述されていることを確認
  - 多言語対応の条件分岐が削除され、固定値のみが使用されていることを確認
  - Wiki構造生成プロンプトに"Japanese (日本語)"が明示的に含まれていることを確認
  - _Requirements: 2.1, 2.3, 3.1, 3.4_
