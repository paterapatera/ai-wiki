# Requirements Document

## Project Description (Input)
ユーザーはCursorのエージェントで`/ai-wiki/init`コマンドを実行したら、WikiトピックのXMLではなくJSONファイルを出力できる

出力先はこのワークスペースの`.wiki/`ディレクトリ内。

再実行する場合は`.wiki/`を削除してから実施する。

入力元に使用するディレクトリはこのワークスペースの`repo/<指定ディレクトリ>`とする。

## Requirements

### Requirement 1: JSON形式でのWiki構造出力
**Objective:** 開発者として、`/ai-wiki/init`コマンドを実行した際に、Wikiトピック構造をJSON形式で出力できるようにしたい。これにより、XMLではなく構造化されたJSONデータとしてWiki情報を扱えるようになる。

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドが実行される, the AI Wiki System shall Wiki構造をJSON形式で出力する
2. The AI Wiki System shall XML形式ではなくJSON形式でWiki構造を出力する
3. Where JSON形式で出力される, the AI Wiki System shall 有効なJSON構文に準拠した形式で出力する
4. The AI Wiki System shall 既存のXML出力形式と同等の情報をJSON形式で提供する

### Requirement 2: 出力先ディレクトリの管理
**Objective:** 開発者として、生成されたJSONファイルがワークスペースの`.wiki/`ディレクトリに保存されるようにしたい。これにより、出力先が明確になり、ファイル管理が容易になる。

#### Acceptance Criteria
1. When Wiki構造が生成される, the AI Wiki System shall `.wiki/`ディレクトリ内にJSONファイルを出力する
2. If `.wiki/`ディレクトリが存在しない, the AI Wiki System shall ディレクトリを自動作成してからJSONファイルを出力する
3. The AI Wiki System shall ワークスペースルートからの相対パス`.wiki/`にJSONファイルを出力する

### Requirement 3: 再実行時の処理
**Objective:** 開発者として、`/ai-wiki/init`コマンドを再実行する際に、既存の`.wiki/`ディレクトリを削除してから新しいJSONファイルを生成できるようにしたい。これにより、古い出力と新しい出力が混在することを防ぐ。

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドが再実行される, the AI Wiki System shall 既存の`.wiki/`ディレクトリを削除してから新しいJSONファイルを生成する
2. If `.wiki/`ディレクトリが存在する, the AI Wiki System shall 再実行時にディレクトリ全体を削除する
3. When `.wiki/`ディレクトリが削除される, the AI Wiki System shall 削除後に新しいJSONファイルを生成する

### Requirement 4: 入力元ディレクトリの指定
**Objective:** 開発者として、分析対象のディレクトリを`repo/<指定ディレクトリ>`形式で指定できるようにしたい。これにより、ワークスペース内の特定のリポジトリディレクトリを分析対象として指定できる。

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドが実行される, the AI Wiki System shall `repo/<指定ディレクトリ>`形式で指定されたディレクトリを入力元として使用する
2. If 指定されたディレクトリが存在しない, the AI Wiki System shall エラーを報告する
3. The AI Wiki System shall ワークスペースルートからの相対パス`repo/<指定ディレクトリ>`を入力元として認識する
4. When ディレクトリが指定される, the AI Wiki System shall 指定されたディレクトリの内容を分析してWiki構造を生成する

### Requirement 5: JSONファイルの構造と内容
**Objective:** 開発者として、生成されるJSONファイルが既存のXML形式と同等の情報を含み、かつJSONとして適切に構造化されていることを期待する。これにより、XMLからJSONへの移行がスムーズに行える。

#### Acceptance Criteria
1. The AI Wiki System shall JSONファイルにWiki構造のタイトル、説明、セクション、ページ情報を含める
2. The AI Wiki System shall JSONファイルに各ページのタイトル、説明、重要度、関連ファイル、関連ページ情報を含める
3. Where 包括的ビューが有効な場合, the AI Wiki System shall JSONファイルにセクション階層構造を含める
4. The AI Wiki System shall JSONファイルが既存のXML形式と同等の情報を提供する

