# Requirements Document

## Project Description (Input)
ユーザーは`/ai-wiki/make` コマンドを実行することでWikiを作成できる。
作成したWikiの配置場所は引数で指定できる。
`.wiki/wiki_structure.json`を元にWIKIを生成する。

## Requirements

### Requirement 1: wiki_structure.jsonの読み込み
**Objective:** 開発者として、`/ai-wiki/make`コマンドが`.wiki/wiki_structure.json`ファイルを読み込んで、Wiki構造を取得できるようにしたい。これにより、`/ai-wiki/init`コマンドで生成された構造定義を基にWikiページを生成できるようになる。

#### Acceptance Criteria
1. When `/ai-wiki/make`コマンドが実行される, the AI Wiki System shall `.wiki/wiki_structure.json`ファイルを読み込む
2. If `.wiki/wiki_structure.json`ファイルが存在しない, the AI Wiki System shall エラーメッセージを報告し、処理を中断する
3. The AI Wiki System shall JSONファイルの構文を検証し、有効なJSON形式であることを確認する
4. If JSONファイルの構文が無効である, the AI Wiki System shall エラーメッセージを報告し、処理を中断する
5. The AI Wiki System shall JSONファイルから`sections`配列と`pages`配列を取得する
6. The AI Wiki System shall JSONファイルから`title`と`description`を取得する

### Requirement 2: 出力先ディレクトリの指定
**Objective:** 開発者として、生成されたWikiページの配置場所を引数で指定できるようにしたい。これにより、異なるプロジェクトや環境に応じて柔軟に出力先を変更できる。

#### Acceptance Criteria
1. When `/ai-wiki/make`コマンドが実行される, the AI Wiki System shall 引数として出力先ディレクトリパスを受け取る
2. If 出力先ディレクトリが指定されない, the AI Wiki System shall デフォルトの出力先ディレクトリ（例：`wiki/`）を使用する
3. If 指定された出力先ディレクトリが存在しない, the AI Wiki System shall ディレクトリを自動作成する
4. The AI Wiki System shall 出力先ディレクトリパスをワークスペースルートからの相対パスとして扱う
5. If 出力先ディレクトリの作成に失敗する, the AI Wiki System shall エラーメッセージを報告し、処理を中断する

### Requirement 3: Wikiページの生成
**Objective:** 開発者として、`wiki_structure.json`に定義された各ページに対応するMarkdown形式のWikiページを生成できるようにしたい。これにより、構造定義から実際のWikiコンテンツを自動生成できる。

#### Acceptance Criteria
1. When `wiki_structure.json`が読み込まれる, the AI Wiki System shall `pages`配列内の各ページオブジェクトに対してWikiページを生成する
2. The AI Wiki System shall 各ページの`title`をH1見出しとして使用する
3. The AI Wiki System shall 各ページの`relevant_files`に指定されたファイルを読み込んで、コンテンツ生成の基盤とする
4. The AI Wiki System shall 各ページの`description`をページの説明として使用する
5. The AI Wiki System shall 既存の`/ai-wiki/make`コマンド定義（`.cursor/commands/ai-wiki/make.md`）の指示に従って、各ページのMarkdownコンテンツを生成する
6. The AI Wiki System shall 生成される各ページに`<details>`ブロックを含め、使用した関連ファイルをリストアップする
7. The AI Wiki System shall 生成される各ページにMermaidダイアグラム、テーブル、コードスニペットを含める（該当する場合）
8. The AI Wiki System shall 生成される各ページのコンテンツを日本語で生成する
9. If `relevant_files`に指定されたファイルが存在しない, the AI Wiki System shall エラーメッセージを報告し、そのページの生成をスキップするか、利用可能なファイルのみを使用する

### Requirement 4: セクション構造の維持
**Objective:** 開発者として、生成されたWikiページが`wiki_structure.json`で定義されたセクション構造を反映するようにしたい。これにより、論理的な階層構造を持つWikiを生成できる。

#### Acceptance Criteria
1. When Wikiページが生成される, the AI Wiki System shall `sections`配列の構造を維持する
2. The AI Wiki System shall 各セクションに対応するディレクトリまたはインデックスファイルを作成する（実装方法による）
3. The AI Wiki System shall 各ページの`parent_section`プロパティに基づいて、ページを適切なセクションに配置する
4. The AI Wiki System shall `subsections`配列に基づいて、サブセクションの階層構造を維持する
5. If セクション構造が無効である（存在しないセクションIDへの参照など）, the AI Wiki System shall エラーメッセージを報告し、処理を中断するか、無効な参照をスキップする

### Requirement 5: ページ間のリンク生成
**Objective:** 開発者として、生成されたWikiページ間で適切な相互リンクが生成されるようにしたい。これにより、関連するページ間を容易にナビゲートできる。

#### Acceptance Criteria
1. When Wikiページが生成される, the AI Wiki System shall 各ページの`related_pages`プロパティに基づいて、関連ページへのリンクを生成する
2. The AI Wiki System shall セクション内の他のページへのリンクを生成する（該当する場合）
3. The AI Wiki System shall 親セクションやサブセクションへのナビゲーションリンクを生成する（実装方法による）
4. The AI Wiki System shall リンクの形式をMarkdownリンク構文（`[Link Text](path/to/page.md)`）で生成する
5. If リンク先のページが存在しない, the AI Wiki System shall リンクを生成するが、存在しないページへの参照であることを示す（またはリンクをスキップする）

### Requirement 6: ファイル出力と構造化
**Objective:** 開発者として、生成されたWikiページが適切なファイル名とディレクトリ構造で保存されるようにしたい。これにより、生成されたWikiを容易に管理・閲覧できる。

#### Acceptance Criteria
1. When Wikiページが生成される, the AI Wiki System shall 各ページを個別のMarkdownファイルとして出力先ディレクトリに保存する
2. The AI Wiki System shall ファイル名をページの`id`または`title`に基づいて生成する（例：`page-id.md`または`page-title.md`）
3. The AI Wiki System shall ファイル名に使用できない文字（`/`, `\`, `:`, `*`, `?`, `"`, `<`, `>`, `|`）を適切にエスケープまたは置換する
4. The AI Wiki System shall セクション構造に基づいて、サブディレクトリを作成してページを配置する（実装方法による）
5. If ファイルの書き込みに失敗する, the AI Wiki System shall エラーメッセージを報告し、処理を継続するか中断する（実装方針による）

### Requirement 7: エラーハンドリングと検証
**Objective:** 開発者として、コマンド実行中に発生するエラーが適切に処理され、ユーザーに明確なフィードバックが提供されるようにしたい。これにより、問題の特定と解決が容易になる。

#### Acceptance Criteria
1. When エラーが発生する, the AI Wiki System shall 明確で理解しやすいエラーメッセージを日本語で報告する
2. The AI Wiki System shall エラーの種類（ファイルが見つからない、JSON構文エラー、ディレクトリ作成失敗など）に応じて適切なメッセージを表示する
3. If 一部のページの生成に失敗する, the AI Wiki System shall 成功したページは保存し、失敗したページについてのみエラーを報告する
4. The AI Wiki System shall 処理の完了時に、生成されたページ数と失敗したページ数（該当する場合）を報告する
5. The AI Wiki System shall 生成されたファイルのパスをユーザーに報告する

### Requirement 8: コマンド定義の更新
**Objective:** 開発者として、`/ai-wiki/make`コマンドの定義が`wiki_structure.json`の読み込みと全ページ生成に対応するように更新されることを期待する。これにより、コマンドが適切に機能する。

#### Acceptance Criteria
1. The AI Wiki System shall `.cursor/commands/ai-wiki/make.md`ファイルを更新して、`wiki_structure.json`の読み込み処理を含める
2. The AI Wiki System shall コマンド定義に出力先ディレクトリ引数の処理を含める
3. The AI Wiki System shall コマンド定義に全ページの生成ループ処理を含める
4. The AI Wiki System shall コマンド定義にセクション構造の維持処理を含める
5. The AI Wiki System shall コマンド定義にページ間リンクの生成処理を含める
6. The AI Wiki System shall コマンド定義の`<meta>`セクションに適切な`description`と`argument-hint`を設定する
