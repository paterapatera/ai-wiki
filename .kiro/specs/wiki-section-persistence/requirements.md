# Requirements Document

## Project Description (Input)
ユーザーは前回のセクション情報を維持してWikiを更新できる

## Introduction

この機能は、`/ai-wiki/init`コマンド実行時に既存の`wiki_structure.json`ファイルから前回のセクション情報を読み込み、その構成を維持しながらWiki構造を更新する機能を提供します。ユーザーが`/ai-wiki/init`コマンドを再実行する際に、既存のセクション構造、ページ階層、関連ページの関係性を保持することで、一貫性のあるWiki管理を実現します。

## Requirements

### Requirement 1: 既存のwiki_structure.jsonの読み込みと検証
**Objective:** 開発者として、`/ai-wiki/init`コマンド実行時に既存の`wiki_structure.json`ファイルを読み込んで検証できるようにしたい。これにより、前回のセクション情報を取得できる。

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドが実行される, the AI Wiki System shall 出力先ディレクトリを削除する前に、`[出力先ディレクトリ]/wiki_structure.json`ファイルの存在を確認する
2. If `wiki_structure.json`ファイルが存在する, the AI Wiki System shall JSONファイルを読み込んで構文検証を実行する
3. If `wiki_structure.json`ファイルが存在しない, the AI Wiki System shall 新規作成モードとして処理を続行する
4. If JSON構文が無効である, the AI Wiki System shall エラーメッセージを報告し、処理を中断する
5. The AI Wiki System shall 読み込んだJSONデータから`sections`配列と`pages`配列を抽出する
6. The AI Wiki System shall 既存の`wiki_structure.json`を読み込んだ後、出力先ディレクトリの削除処理を実行する

### Requirement 2: セクション構成の維持
**Objective:** 開発者として、`/ai-wiki/init`コマンド実行時に既存のセクション構成（セクションID、タイトル、階層構造）を維持できるようにしたい。これにより、一貫性のあるWiki構造を保持できる。

#### Acceptance Criteria
1. When 既存の`wiki_structure.json`が読み込まれる, the AI Wiki System shall 既存の`sections`配列の構造を維持する
2. The AI Wiki System shall 既存のセクションIDを保持する
3. The AI Wiki System shall 既存のセクションタイトルを保持する
4. The AI Wiki System shall 既存の`subsections`配列の階層構造を維持する
5. When 新しいセクションが必要な場合, the AI Wiki System shall 既存のセクション構成を優先し、必要に応じて追加する
6. If 既存のセクション構成と新規分析結果が大きく乖離する場合, the AI Wiki System shall 構成を維持しなくてもよい（新規構成を採用可能）
7. When Wiki構造を生成する, the AI Wiki System shall 既存のセクション構成を参照して、新しいページを適切なセクションに配置する

### Requirement 3: ページとセクションの関連性の維持
**Objective:** 開発者として、`/ai-wiki/init`コマンド実行時に既存のページとセクションの関連性（`parent_section`、`related_pages`）を維持できるようにしたい。これにより、ページの論理的な配置とナビゲーション構造を保持できる。

#### Acceptance Criteria
1. When 既存の`wiki_structure.json`が読み込まれる, the AI Wiki System shall 既存のページIDとセクションIDのマッピング情報を保持する
2. When 新しいWiki構造を生成する, the AI Wiki System shall 既存のページIDが一致する場合、そのページの`parent_section`プロパティを維持する
3. The AI Wiki System shall 既存のページIDが一致する場合、そのページの`related_pages`プロパティを維持する
4. When 新しいページが生成される場合, the AI Wiki System shall 既存のセクション構造に基づいて適切な`parent_section`を割り当てる
5. When 既存のページIDと一致するページが生成される場合, the AI Wiki System shall 既存の`parent_section`と`related_pages`を優先的に使用する
6. If 既存のページが新規分析で検出されない場合, the AI Wiki System shall そのページを新しいWiki構造から除外する

### Requirement 4: 更新時の構成比較と判断
**Objective:** 開発者として、`/ai-wiki/init`コマンド実行時に既存の構成と新規分析結果を比較し、構成を維持するかどうかを判断できるようにしたい。これにより、内容が大きく変化した場合でも柔軟に対応できる。

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドが実行される, the AI Wiki System shall 既存の`wiki_structure.json`の構成と新規分析結果を比較する
2. The AI Wiki System shall 構成の類似度を評価する（セクション数、ページ数、階層構造の一致度、`project_root`の一致など）
3. If 構成の類似度が高い場合（`project_root`が一致し、セクション構造が類似している場合）, the AI Wiki System shall 既存のセクション構成を維持する
4. If 構成の類似度が低い場合（`project_root`が異なる、または内容が大きく乖離する場合）, the AI Wiki System shall 構成を維持しなくてもよい（新規構成を採用可能）
5. The AI Wiki System shall 構成維持の判断結果をユーザーに報告する（既存構成を維持した場合、または新規構成を採用した場合）

### Requirement 5: 更新されたwiki_structure.jsonの保存
**Objective:** 開発者として、`/ai-wiki/init`コマンド実行結果を`wiki_structure.json`ファイルに保存できるようにしたい。これにより、次回のコマンド実行時に最新の構成情報を利用できる。

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドのWiki構造生成が完了する, the AI Wiki System shall 生成されたWiki構造を`[出力先ディレクトリ]/wiki_structure.json`形式で保存する
2. The AI Wiki System shall 既存のセクション情報を維持した状態でJSONファイルを保存する（構成維持が選択された場合）
3. The AI Wiki System shall JSONファイルの構文検証を実行する（Dockerコンテナを使用）
4. If JSONファイルの保存に失敗する, the AI Wiki System shall エラーメッセージを報告し、処理を中断する
5. The AI Wiki System shall 保存された`wiki_structure.json`ファイルを次回の`/ai-wiki/init`コマンド実行時に利用可能にする
