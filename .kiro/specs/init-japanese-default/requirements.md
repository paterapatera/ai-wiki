# Requirements Document

## Project Description (Input)
ユーザーは`ai-wiki/initに言語の指定をせずには日本語のみを対象となる

## Requirements

### Requirement 1: 言語パラメータの削除
**Objective:** 開発者として、`/ai-wiki/init`コマンドから言語パラメータを削除し、常に日本語のみを使用するようにしたい。これにより、言語指定の複雑さを排除し、日本語のみを対象としたシンプルな動作を実現する。

#### Acceptance Criteria
1. When `/ai-wiki/init`コマンドが実行される, the AI Wiki System shall 言語パラメータを受け付けない
2. The AI Wiki System shall `${language}`変数を使用せず、常に日本語（'ja'）を固定値として使用する
3. Where 言語パラメータが指定された場合, the AI Wiki System shall そのパラメータを無視し、日本語のみを使用する
4. The AI Wiki System shall コマンド定義から言語パラメータ関連の処理を削除する

### Requirement 2: 日本語のみの固定化
**Objective:** 開発者として、`/ai-wiki/init`コマンドが常に日本語（'ja'）のみを対象として動作するようにしたい。これにより、言語選択の余地を排除し、一貫した日本語でのWiki生成を保証する。

#### Acceptance Criteria
1. When Wiki構造が生成される, the AI Wiki System shall 常に日本語（'ja'）を使用してWiki構造を生成する
2. The AI Wiki System shall 言語判定ロジックを削除し、固定値'ja'を使用する
3. Where Wiki生成プロンプトが作成される, the AI Wiki System shall "Japanese (日本語)"を明示的に指定する
4. The AI Wiki System shall 多言語対応の条件分岐を削除し、日本語のみの処理フローを実装する

### Requirement 3: 日本語以外の言語指定の禁止
**Objective:** 開発者として、`/ai-wiki/init`コマンドで日本語以外の言語を指定できないようにしたい。これにより、意図しない言語でのWiki生成を防ぎ、日本語のみの動作を強制する。

#### Acceptance Criteria
1. When ユーザーが言語パラメータを指定しようとした場合, the AI Wiki System shall そのパラメータを無視し、エラーを報告しない（静かに無視する）
2. If コマンド定義に言語パラメータが含まれている, the AI Wiki System shall そのパラメータを処理せず、日本語のみを使用する
3. The AI Wiki System shall 日本語以外の言語コード（'en', 'zh', 'es'など）を認識しない
4. When 言語指定が試みられた場合, the AI Wiki System shall 日本語（'ja'）をデフォルトとして使用し続ける

