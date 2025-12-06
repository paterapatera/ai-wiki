<meta>
description: wiki_structure.jsonを読み込んで全ページのWikiコンテンツを生成する
argument-hint: [出力先ディレクトリ]
</meta>

# Wiki 生成

<background_information>
このコマンドは、`/ai-wiki/init`コマンドで生成された`wiki_structure.json`を読み込んで、定義された全ページのWikiコンテンツをMarkdown形式で生成するためのコマンドです。

## 主な機能
- `.wiki/wiki_structure.json`ファイルの読み込みと検証
- 出力先ディレクトリの指定（引数で指定可能、デフォルトは`wiki/`）
- `pages`配列内の各ページオブジェクトに対してWikiページを生成
- セクション構造の維持（フラット構造で実装）
- ページ間のリンク生成
- エラーハンドリングと進捗報告

## 処理フロー
1. 出力先ディレクトリの取得と確認
2. `wiki_structure.json`の読み込みと検証
3. セクション構造の解析
4. 各ページの生成ループ（既存のページ生成指示を使用）
5. ページ間リンクの生成
6. ファイル出力と完了報告

## 既存のページ生成指示との関係
このコマンドは、既存の`<instructions>`セクション内のページ生成指示を維持しつつ、JSON読み込み、ループ処理、出力先指定、セクション構造維持、リンク生成の機能を追加します。各ページの生成時には、既存の詳細なページ生成指示（Mermaidダイアグラム、テーブル、コードスニペット、ソース引用など）が使用されます。
</background_information>

<instructions>
## Step 1: 出力先ディレクトリの取得と確認

1. コマンド引数から出力先ディレクトリを取得します。
   - コマンド引数が指定されている場合: その値を出力先ディレクトリパスとして使用
   - コマンド引数が指定されていない場合: デフォルト値`wiki/`を使用
   - 取得したディレクトリパスを `${outputDir}` 変数として設定します
   - ワークスペースルートからの相対パスとして `${outputDir}` を解決します（例: `wiki/` → `./wiki/`）

2. `${outputDir}` で指定されたディレクトリの処理を実行します。
   - ディレクトリが存在する場合: ディレクトリ内の `.md` ファイルのみを削除します
     - `find ${outputDir} -type f -name "*.md" -delete` を実行してディレクトリ内の `.md` ファイルのみを削除します
     - エラーが発生した場合: エラーメッセージ「出力先ディレクトリ `${outputDir}` の `.md` ファイル削除に失敗しました。」を報告し、処理を中断します
   - ディレクトリが存在しない場合: `mkdir -p ${outputDir}` を実行してディレクトリを作成します
   - ディレクトリ作成に失敗した場合: エラーメッセージ「出力先ディレクトリ `${outputDir}` の作成に失敗しました。」を報告し、処理を中断します
   - ディレクトリが存在する、または作成に成功した場合: 次のステップに進みます

## Step 2: wiki_structure.jsonの読み込みと検証

1. `.wiki/wiki_structure.json` ファイルを読み込みます。
   - `read_file` ツールを使用して、`.wiki/wiki_structure.json` ファイルを読み込みます
   - ファイルパスはワークスペースルートからの相対パス `.wiki/wiki_structure.json` を使用します
   - ファイルが存在しない場合: エラーメッセージ「`.wiki/wiki_structure.json` ファイルが見つかりません。先に `/ai-wiki/init` コマンドを実行してWiki構造を生成してください。」を報告し、処理を中断します
   - ファイルが存在する場合: 読み込んだJSONデータを `${wikiStructure}` 変数として設定します

2. JSONファイルの構文検証を行います。
   - Dockerコンテナを使用してJSONファイルの構文検証を実行します
   - `docker run --rm -v $(pwd):/work -w /work python:3-alpine python -c "import json; json.load(open('.wiki/wiki_structure.json'))"` を実行してJSONファイルが有効なJSON構文に準拠していることを確認します
   - 検証に失敗した場合: エラーメッセージ「`.wiki/wiki_structure.json` のJSON構文が無効です。」を報告し、処理を中断します
   - 検証が成功した場合: JSONファイルが正しく読み込まれたことを確認します

3. JSONデータから必要な情報を抽出します。
   - `${wikiStructure}` から `sections` 配列を取得し、`${sections}` 変数として設定します
   - `${wikiStructure}` から `pages` 配列を取得し、`${pages}` 変数として設定します
   - `${wikiStructure}` から `title` と `description` を取得します
   - `${wikiStructure}` から `project_root` を取得します（存在する場合）
   - `project_root` が存在しない場合は、`${pages}` 配列内の最初のページの `relevant_files` の最初のパスからプロジェクトルートを推測します
     - 例: `relevant_files` の最初のパスが `repo/slack-agent/README.md` の場合、`/` で分割して最後のファイル名を除いた部分（`repo/slack-agent`）をプロジェクトルートとして使用
     - 複数のパスがある場合は、共通のプレフィックスをプロジェクトルートとして使用
   - 取得したプロジェクトルートを `${projectRoot}` 変数として設定します（末尾に `/` がないことを確認）
   - セクションIDとページIDの一意性を確認します（重複がある場合はエラーメッセージを報告し、処理を中断します）

## Step 3: セクション構造の解析

1. セクション構造を解析します。
   - `${sections}` 配列をループ処理して、各セクションのIDとタイトルをマッピングします
   - セクションIDのマップを `${sectionMap}` 変数として設定します（キー: セクションID、値: セクションオブジェクト）
   - 各ページの `parent_section` プロパティが有効なセクションIDを参照していることを確認します
   - 無効な参照がある場合: エラーメッセージ「無効なセクションIDへの参照が見つかりました: ${invalidSectionId}」を報告し、処理を中断します

## Step 4: 各ページの生成ループ

1. `${pages}` 配列をループ処理して、各ページオブジェクトに対してWikiページを生成します。
   - ページ生成の成功/失敗を追跡するために、`${successCount}` と `${failureCount}` 変数を初期化します（初期値: 0）
   - 各ページオブジェクトを `${page}` 変数として設定します

2. 各ページの `relevant_files` のパスを変換し、ファイルを読み込みます。
   - `${page.relevant_files}` 配列をループ処理して、各ファイルパスを変換します
   - 各ファイルパス `${originalPath}` に対して以下の処理を実行します:
     - `${originalPath}` が `${projectRoot}` で始まる場合: `${projectRoot}` を削除し、先頭に `/` を付けて正規化します
       - 例: `${projectRoot}` が `repo/slack-agent` で、`${originalPath}` が `repo/slack-agent/README.md` の場合、変換後のパスは `/README.md`
       - 例: `${projectRoot}` が `repo/slack-agent` で、`${originalPath}` が `repo/slack-agent/src/main.py` の場合、変換後のパスは `/src/main.py`
     - `${originalPath}` が `${projectRoot}` で始まらない場合: そのまま使用します（既に正規化されている可能性がある）
   - 変換後のパスを `${normalizedPath}` 変数として設定します
   - 変換前のパス（`${originalPath}`）を使用して `read_file` ツールでファイルを読み込みます（ファイルシステム上の実際のパスは変換前のパスを使用）
   - ファイルが存在しない場合: エラーメッセージ「ファイル `${originalPath}` が見つかりません。このページの生成をスキップします。」を報告し、利用可能なファイルのみを使用するか、そのページの生成をスキップします
   - 読み込んだファイルの内容を `${relevantFilesContent}` 変数として設定します（キー: `${normalizedPath}`（変換後のパス）、値: ファイル内容）

3. 既存のページ生成指示を使用してMarkdownコンテンツを生成します。

You are an expert technical writer and software architect.
Your task is to generate a comprehensive and accurate technical wiki page in Markdown format about a specific feature, system, or module within a given software project.

You will be given:
1. The page object `${page}` with properties: `id`, `title`, `description`, `importance`, `relevant_files`, `related_pages`, `parent_section`
2. The relevant files content `${relevantFilesContent}` (key: file path, value: file content) that you MUST use as the sole basis for the content.

CRITICAL STARTING INSTRUCTION:
The very first thing on the page MUST be a \`<details>\` block listing ALL the \`[RELEVANT_SOURCE_FILES]\` you used to generate the content. There MUST be AT LEAST 5 source files listed - if fewer were provided, you MUST find additional related files to include.
Format it exactly like this:
<details>
<summary>Relevant source files</summary>

Remember, do not provide any acknowledgements, disclaimers, apologies, or any other preface before the \`<details>\` block. JUST START with the \`<details>\` block.
The following files were used as context for generating this wiki page:

${Object.keys(relevantFilesContent).map(path => `- [${path}](${generateFileUrl(path)})`).join('\n')}
<!-- Add additional relevant files if fewer than 5 were provided -->
</details>

IMPORTANT: The file paths listed in the \`<details>\` block MUST use the normalized paths (with project root removed and leading `/` added) from `${relevantFilesContent}` keys, NOT the original paths from `${page.relevant_files}`.

Immediately after the \`<details>\` block, the main title of the page should be a H1 Markdown heading: \`# ${page.title}\`.

Based ONLY on the content of the \`[RELEVANT_SOURCE_FILES]\`:

1.  **Introduction:** Start with a concise introduction (1-2 paragraphs) explaining the purpose, scope, and high-level overview of "${page.title}" within the context of the overall project. If relevant, and if information is available in the provided files, link to other potential wiki pages using the format \`[Link Text](#page-anchor-or-id)\`.

2.  **Detailed Sections:** Break down "${page.title}" into logical sections using H2 (\`##\`) and H3 (\`###\`) Markdown headings. For each section:
    *   Explain the architecture, components, data flow, or logic relevant to the section's focus, as evidenced in the source files.
    *   Identify key functions, classes, data structures, API endpoints, or configuration elements pertinent to that section.

3.  **Mermaid Diagrams:**
    *   EXTENSIVELY use Mermaid diagrams (e.g., \`flowchart TD\`, \`sequenceDiagram\`, \`classDiagram\`, \`erDiagram\`, \`graph TD\`) to visually represent architectures, flows, relationships, and schemas found in the source files.
    *   Ensure diagrams are accurate and directly derived from information in the \`[RELEVANT_SOURCE_FILES]\`.
    *   Provide a brief explanation before or after each diagram to give context.
    *   CRITICAL: All diagrams MUST follow strict vertical orientation:
       - Use "graph TD" (top-down) directive for flow diagrams
       - NEVER use "graph LR" (left-right)
       - Maximum node width should be 3-4 words
       - For sequence diagrams:
         - Start with "sequenceDiagram" directive on its own line
         - Define ALL participants at the beginning using "participant" keyword
         - Optionally specify participant types: actor, boundary, control, entity, database, collections, queue
         - Use descriptive but concise participant names, or use aliases: "participant A as Alice"
         - Use the correct Mermaid arrow syntax (8 types available):
           - -> solid line without arrow (rarely used)
           - --> dotted line without arrow (rarely used)
           - ->> solid line with arrowhead (most common for requests/calls)
           - -->> dotted line with arrowhead (most common for responses/returns)
           - ->x solid line with X at end (failed/error message)
           - -->x dotted line with X at end (failed/error response)
           - -) solid line with open arrow (async message, fire-and-forget)
           - --) dotted line with open arrow (async response)
           - Examples: A->>B: Request, B-->>A: Response, A->xB: Error, A-)B: Async event
         - Use +/- suffix for activation boxes: A->>+B: Start (activates B), B-->>-A: End (deactivates B)
         - Group related participants using "box": box GroupName ... end
         - Use structural elements for complex flows:
           - loop LoopText ... end (for iterations)
           - alt ConditionText ... else ... end (for conditionals)
           - opt OptionalText ... end (for optional flows)
           - par ParallelText ... and ... end (for parallel actions)
           - critical CriticalText ... option ... end (for critical regions)
           - break BreakText ... end (for breaking flows/exceptions)
         - Add notes for clarification: "Note over A,B: Description", "Note right of A: Detail"
         - Use autonumber directive to add sequence numbers to messages
         - NEVER use flowchart-style labels like A--|label|-->B. Always use a colon for labels: A->>B: My Label

4.  **Tables:**
    *   Use Markdown tables to summarize information such as:
        *   Key features or components and their descriptions.
        *   API endpoint parameters, types, and descriptions.
        *   Configuration options, their types, and default values.
        *   Data model fields, types, constraints, and descriptions.

5.  **Code Snippets (ENTIRELY OPTIONAL):**
    *   Include short, relevant code snippets (e.g., Python, Java, JavaScript, SQL, JSON, YAML) directly from the \`[RELEVANT_SOURCE_FILES]\` to illustrate key implementation details, data structures, or configurations.
    *   Ensure snippets are well-formatted within Markdown code blocks with appropriate language identifiers.

6.  **Source Citations (EXTREMELY IMPORTANT):**
    *   For EVERY piece of significant information, explanation, diagram, table entry, or code snippet, you MUST cite the specific source file(s) and relevant line numbers from which the information was derived.
    *   Place citations at the end of the paragraph, under the diagram/table, or after the code snippet.
    *   Use the exact format: \`Sources: [filename.ext:start_line-end_line]()\` for a range, or \`Sources: [filename.ext:line_number]()\` for a single line. Multiple files can be cited: \`Sources: [file1.ext:1-10](), [file2.ext:5](), [dir/file3.ext]()\` (if the whole file is relevant and line numbers are not applicable or too broad).
    *   If an entire section is overwhelmingly based on one or two files, you can cite them under the section heading in addition to more specific citations within the section.
    *   IMPORTANT: You MUST cite AT LEAST 5 different source files throughout the wiki page to ensure comprehensive coverage.

7.  **Technical Accuracy:** All information must be derived SOLELY from the \`[RELEVANT_SOURCE_FILES]\`. Do not infer, invent, or use external knowledge about similar systems or common practices unless it's directly supported by the provided code. If information is not present in the provided files, do not include it or explicitly state its absence if crucial to the topic.

8.  **Clarity and Conciseness:** Use clear, professional, and concise technical language suitable for other developers working on or learning about the project. Avoid unnecessary jargon, but use correct technical terms where appropriate.

9.  **Conclusion/Summary:** End with a brief summary paragraph if appropriate for "${page.title}", reiterating the key aspects covered and their significance within the project.

IMPORTANT: Generate the content in Japanese (日本語) language.

Remember:
- Ground every claim in the provided source files.
- Prioritize accuracy and direct representation of the code's functionality and structure.
- Structure the document logically for easy understanding by other developers.

4. 生成されたMarkdownコンテンツにページ間リンクを追加します。
   - `${page.related_pages}` 配列をループ処理して、各関連ページIDに対してリンクを生成します
   - ページIDからファイルパスへのマッピング: `${relatedPageId}` → `${relatedPageId}.md`
   - Markdownリンク構文（`[${relatedPageTitle}](${relatedPageId}.md)`）で生成します
   - リンク先のページタイトルを取得するために、`${pages}` 配列から該当するページオブジェクトを検索します
   - 生成されたリンクをMarkdownコンテンツの適切な位置（Introductionセクションの最後など）に追加します
   - リンク先のページが存在しない場合でも、リンクを生成します（警告は表示しません）

5. ファイル名を生成します。
   - ページIDをそのままファイル名として使用: `${page.id}.md`
   - 無効文字（`/`, `\`, `:`, `*`, `?`, `"`, `<`, `>`, `|`）が含まれる場合は`-`に置換します
   - 生成されたファイル名を `${fileName}` 変数として設定します

6. 各ページをMarkdownファイルとして出力先ディレクトリに保存します。
   - `write` ツールを使用して、生成されたMarkdownコンテンツを `${outputDir}/${fileName}` に書き込みます
   - ファイル書き込みに失敗した場合: エラーメッセージ「ページ `${page.id}` のファイル書き込みに失敗しました。」を報告し、`${failureCount}` をインクリメントします（他のページの生成は継続します）
   - ファイル書き込みが成功した場合: `${successCount}` をインクリメントします

## Step 5: 完了報告

1. 処理の完了時に、生成されたページ数と失敗したページ数（該当する場合）を報告します。
   - 「Wikiページの生成が完了しました。生成されたページ数: ${successCount}、失敗したページ数: ${failureCount}」というメッセージを報告します
   - 生成されたファイルのパス（`${outputDir}/`）をユーザーに報告します

</instructions>

## Tool Guidance

このコマンドの実行には以下のツールを使用します：

1. **read_file**: JSONファイル（`.wiki/wiki_structure.json`）とソースファイル（`relevant_files`）の読み込みに使用
2. **write**: 生成されたMarkdownファイルを出力先ディレクトリに書き込むために使用
3. **run_terminal_cmd**: ディレクトリ操作（`mkdir -p`）とJSON検証（Dockerコンテナを使用）に使用
4. **list_dir**: ディレクトリ構造の取得に使用（必要に応じて）

### 環境要件

- **Docker**: JSONファイルの検証にDockerコンテナ（`python:3-alpine`）を使用します
- **Python/Node.js**: ホスト環境にはインストール不要です（Dockerコンテナ内で使用）

## Output Description

このコマンドは以下の出力を生成します：

1. **Markdownファイル**: 各ページに対応するMarkdown形式のWikiページファイル
   - 出力先ディレクトリ: `${outputDir}/`（デフォルト: `wiki/`）
   - ファイル名: `${page-id}.md`（フラット構造）
   - 形式: Markdown形式、UTF-8エンコーディング
   - 内容: ページタイトル、説明、Mermaidダイアグラム、テーブル、コードスニペット、ソース引用、ページ間リンクを含む

2. **完了報告**: 処理が正常に完了した場合、または一部のページの生成に失敗した場合、ユーザーに完了メッセージを報告します

## Safety & Fallback

### エラーハンドリング

1. **出力先ディレクトリ作成エラー**:
   - 出力先ディレクトリの作成に失敗した場合、エラーメッセージを報告し、処理を中断します

2. **ファイル不存在エラー**:
   - `.wiki/wiki_structure.json`が存在しない場合、エラーメッセージを報告し、処理を中断します
   - `relevant_files`に指定されたファイルが存在しない場合、エラーメッセージを報告し、そのページの生成をスキップするか、利用可能なファイルのみを使用します

3. **JSON構文エラー**:
   - JSONファイルの構文が無効な場合、エラーメッセージを報告し、処理を中断します
   - Dockerコンテナを使用したJSON検証に失敗した場合、エラーメッセージを報告し、処理を中断します
   - Dockerがインストールされていない場合、またはDockerコンテナの実行に失敗した場合、エラーメッセージを報告し、処理を中断します

4. **セクション構造エラー**:
   - 存在しないセクションIDへの参照がある場合、エラーメッセージを報告し、処理を中断します
   - セクションIDまたはページIDが重複している場合、エラーメッセージを報告し、処理を中断します

5. **ファイル書き込みエラー**:
   - Markdownファイルの書き込みに失敗した場合、エラーメッセージを報告し、そのページの生成をスキップします（他のページの生成は継続します）

### フォールバック動作

- 部分的な失敗時: 成功したページは保存し、失敗したページについてのみエラーを報告します
- エラーが発生した場合、すべてのエラーメッセージを日本語で明確に報告します
- 処理の完了時に、生成されたページ数と失敗したページ数（該当する場合）を報告します
