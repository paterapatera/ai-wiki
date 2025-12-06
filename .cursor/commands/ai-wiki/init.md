<meta>
description: 指定されたディレクトリを分析して、Wiki構造をJSON形式で生成し、.wiki/wiki_structure.jsonに出力します。プロジェクトのファイルツリーとREADMEを基に、包括的なWiki構造（セクションとページ）を自動生成します。
argument-hint: repo/<指定ディレクトリ>
</meta>

# Wiki トピック生成

<background_information>
このコマンドは、指定されたリポジトリディレクトリを分析して、そのプロジェクトのWiki構造をJSON形式で生成するためのコマンドです。

## 主な機能
- 指定ディレクトリ（`repo/<指定ディレクトリ>`形式）のファイルツリーを分析
- READMEファイルの内容を読み取り
- プロジェクトの構造と内容に基づいて、論理的なWiki構造を自動生成
- 生成されたWiki構造をJSON形式で`.wiki/wiki_structure.json`に保存

## 生成されるWiki構造の特徴
- **セクション構造**: プロジェクトの主要な側面をカバーするセクション（概要、アーキテクチャ、コア機能、バックエンド、デプロイメント、拡張性など）
- **ページ定義**: 各セクションに属するページで、タイトル、説明、重要度、関連ファイル、関連ページへの参照を含む
- **視覚化に適したページ**: アーキテクチャ概要、データフロー、コンポーネント関係図など、図解が有効なページを自動的に含める

## 処理フロー
1. 入力ディレクトリの存在確認
2. `.wiki/`ディレクトリの準備（既存の場合は削除して再作成）
3. ファイルツリーとREADMEの読み取り
4. プロジェクト内容に基づくWiki構造の生成
5. JSON形式での出力と検証

## 出力形式
- 包括的ビュー: セクション階層とページの親子関係を含む完全な構造

生成されるJSONは、後続のWiki生成プロセスで使用される構造定義として機能します。
</background_information>

<instructions>
## Step 1: 入力ディレクトリの取得と確認

1. コマンド引数から入力ディレクトリを取得します。
   - コマンド引数の形式: `repo/<指定ディレクトリ>`
   - 例: `/ai-wiki/init repo/my-project` の場合、入力ディレクトリは `repo/my-project`
   - 取得したディレクトリパスを `${inputDir}` 変数として設定します
   - ワークスペースルートからの相対パスとして `${inputDir}` を解決します（例: `repo/my-project` → `./repo/my-project`）

2. `${inputDir}` で指定されたディレクトリの存在確認を実行します。
   - ディレクトリが存在しない場合: エラーメッセージ「指定されたディレクトリ `${inputDir}` が存在しません。」を報告し、処理を中断します
   - ディレクトリが存在する場合: 次のステップに進みます

## Step 2: 出力ディレクトリの準備

1. `.wiki/` ディレクトリが存在する場合、削除します。
   - `rm -rf .wiki/` を実行してディレクトリ全体を削除します
   - エラーが発生した場合は、エラーメッセージを報告し、処理を中断します

2. `.wiki/` ディレクトリを作成します。
   - `mkdir -p .wiki/` を実行してディレクトリを作成します
   - ディレクトリ作成に失敗した場合は、エラーメッセージを報告し、処理を中断します

## Step 3: 入力ディレクトリの分析

1. `${inputDir}` ディレクトリの完全なファイルツリーを読み取ります。
   - `list_dir` ツールを使用して、ディレクトリ内のすべてのファイルとサブディレクトリの構造を取得します
   - 取得したファイルツリーを `${fileTree}` 変数として設定します

2. `${inputDir}` ディレクトリ内の README ファイルを読み取ります。
   - `read_file` ツールを使用して、README.md, README.txt, README などの一般的なREADMEファイルを探します
   - 見つかった場合は内容を読み取り、見つからない場合は空文字列として扱います
   - 取得したREADME内容を `${readme}` 変数として設定します

## Step 4: Wiki構造の生成

以下の情報を基に、Wiki構造をJSON形式で生成します：

1. The complete file tree of the project:
<file_tree>
${fileTree}
</file_tree>

2. The README file of the project:
<readme>
${readme}
</readme>

I want to create a wiki for this repository. Determine the most logical structure for a wiki based on the repository's content.

IMPORTANT: The wiki content will be generated in Japanese (日本語) language.

When designing the wiki structure, include pages that would benefit from visual diagrams, such as:
- Architecture overviews
- Data flow descriptions
- Component relationships
- Process workflows
- State machines
- Class hierarchies

Create a structured wiki with the following main sections:
- Overview (general information about the project)
- System Architecture (how the system is designed)
- Core Features (key functionality)
- Data Management/Flow: If applicable, how data is stored, processed, accessed, and managed (e.g., database schema, data pipelines, state management).
- Frontend Components (UI elements, if applicable.)
- Backend Systems (server-side components)
- Model Integration (AI model connections)
- Deployment/Infrastructure (how to deploy, what's the infrastructure like)
- Extensibility and Customization: If the project architecture supports it, explain how to extend or customize its functionality (e.g., plugins, theming, custom modules, hooks).

Each section should contain relevant pages. For example, the "Frontend Components" section might include pages for "Home Page", "Repository Wiki Page", "Ask Component", etc.

Generate the wiki structure in JSON format with the following structure:

{
  "title": "[Overall title for the wiki]",
  "description": "[Brief description of the repository]",
  "project_root": "${inputDir}",
  "sections": [
    {
      "id": "section-1",
      "title": "[Section title]",
      "pages": ["page-1", "page-2"],
      "subsections": ["section-2"]
    }
  ],
  "pages": [
    {
      "id": "page-1",
      "title": "[Page title]",
      "description": "[Brief description of what this page will cover]",
      "importance": "high|medium|low",
      "relevant_files": ["[Path to a relevant file]"],
      "related_pages": ["page-2"],
      "parent_section": "section-1"
    }
  ]
}

IMPORTANT: For comprehensive view:
- Include the "project_root" field with the value of `${inputDir}` (the input directory path, e.g., "repo/slack-agent")
- Include the "sections" array with all section information
- Include "subsections" array in each section for hierarchical structure
- Include "parent_section" property in each page object
- Ensure all section IDs and page IDs are unique
- Section references in "subsections" must reference valid section IDs
- Page references in section "pages" arrays must reference valid page IDs
- Parent section references in page "parent_section" must reference valid section IDs

IMPORTANT FORMATTING INSTRUCTIONS:
- Generate ONLY valid JSON structure as specified above
- DO NOT wrap the JSON in markdown code blocks (no \`\`\` or \`\`\`json)
- DO NOT include any explanation text before or after the JSON
- Ensure the JSON is properly formatted and valid (proper escaping, commas, brackets, etc.)
- Start directly with { and end with }
- All string values must be properly quoted
- All arrays must use square brackets []
- All objects must use curly braces {}

IMPORTANT:
1. Create 8-12 pages that would make a comprehensive wiki for this repository
2. Each page should focus on a specific aspect of the codebase (e.g., architecture, key features, setup)
3. The relevant_files should be actual files from the repository that would be used to generate that page
4. Generate ONLY valid JSON with the structure specified above, with no markdown code block delimiters
5. Ensure the JSON is valid and can be parsed by standard JSON parsers

## Step 5: JSONファイルの出力

1. 生成したJSON構造を `.wiki/wiki_structure.json` ファイルに書き込みます。
   - `write` ツールを使用して、生成したJSON構造を `.wiki/wiki_structure.json` に書き込みます
   - ファイルパスはワークスペースルートからの相対パス `.wiki/wiki_structure.json` を使用します
   - ファイル書き込みに失敗した場合は、エラーメッセージを報告し、処理を中断します

2. JSONファイルの検証を行います。
   - Dockerコンテナを使用してJSONファイルの構文検証を実行します
   - `docker run --rm -v $(pwd):/work -w /work python:3-alpine python -c "import json; json.load(open('.wiki/wiki_structure.json'))"` を実行してJSONファイルが有効なJSON構文に準拠していることを確認します
   - 検証に失敗した場合は、エラーメッセージを報告し、処理を中断します
   - 検証が成功した場合、JSONファイルが正しく生成されたことを確認します

3. 完了通知をユーザーに報告します。
   - 「Wiki構造が `.wiki/wiki_structure.json` に正常に出力されました。」というメッセージを報告します

</instructions>

## Tool Guidance

このコマンドの実行には以下のツールを使用します：

1. **run_terminal_cmd**: ディレクトリの削除（`rm -rf .wiki/`）と作成（`mkdir -p .wiki/`）、JSON検証（Dockerコンテナを使用）に使用
2. **list_dir**: 入力ディレクトリの存在確認とファイルツリーの取得に使用
3. **read_file**: READMEファイルの読み取りに使用
4. **write**: 生成したJSON構造を `.wiki/wiki_structure.json` に書き込むために使用

### 環境要件

- **Docker**: JSONファイルの検証にDockerコンテナ（`python:3-alpine`）を使用します
- **Python/Node.js**: ホスト環境にはインストール不要です（Dockerコンテナ内で使用）

## Output Description

このコマンドは以下の出力を生成します：

1. **`.wiki/wiki_structure.json`**: Wiki構造をJSON形式で表現したファイル
   - ワークスペースルートからの相対パス `.wiki/wiki_structure.json` に保存されます
   - 有効なJSON構文に準拠した形式で出力されます
   - `sections`配列とページの`parent_section`プロパティを含む完全なJSON構造

2. **完了通知**: 処理が正常に完了した場合、ユーザーに完了メッセージを報告します

## Safety & Fallback

### エラーハンドリング

1. **入力ディレクトリ不存在エラー**:
   - `${inputDir}`で指定されたディレクトリが存在しない場合、エラーメッセージを報告し、処理を中断します

2. **ディレクトリ操作エラー**:
   - `.wiki/`ディレクトリの削除または作成に失敗した場合、エラーメッセージを報告し、処理を中断します

3. **ファイル書き込みエラー**:
   - JSONファイルの書き込みに失敗した場合、エラーメッセージを報告し、処理を中断します

4. **JSON生成エラー**:
   - Wiki構造のJSON生成に失敗した場合、エラーメッセージを報告し、処理を中断します

5. **JSON検証エラー**:
   - Dockerコンテナを使用したJSON検証に失敗した場合、エラーメッセージを報告し、処理を中断します
   - Dockerがインストールされていない場合、またはDockerコンテナの実行に失敗した場合、エラーメッセージを報告し、処理を中断します

### フォールバック動作

- エラーが発生した場合、すべての処理を中断し、エラーメッセージをユーザーに報告します
- 部分的な出力（不完全なJSONファイルなど）が生成された場合、エラーメッセージを報告し、処理を中断します

