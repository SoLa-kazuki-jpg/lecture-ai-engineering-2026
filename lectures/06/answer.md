# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：search_materials
- STEP 2 の [tool call]：glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：aws_aws___search_documentation({'topics': ['general'], 'search_phrase': 'Amazon S3 versioning overview'})
- 気づき・疑問：MCPサーバを介在させることで、ターミナルに入力するコマンド自体は変えずに、お題に応じてエージェント側で接続先を判定して正しい答えを持ってきていることが分かった（STEP4まで実行してmcp_servers.jsonの中身が書き換わっている状態でSTEP2を再度試した場合でも、glossary_search_glossary({'query': '3レベル'})を呼び出して正しい答えを導いていた。）。今回は分かりやすいお題だったのでこのような使い分けができたかと思うが、どのMCPサーバに接続すればよいのか微妙なお題を出題された際にどのような挙動や接続先決定の判断を行うかが気になった。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：実装
- やったこと：MCP サーバーに、第3回で作った make_quiz を、引数が範囲外のときに分かりやすいメッセージを返すようにコードを修正したうえで追加した。

  追加した make_quiz のコードは以下のとおり
  
　
def make_quiz(lecture: int) -> str:
    """指定した回の用語から1つを選び、確認問題と答えを返す。"""
    glossary = load_glossary()
    available_lectures = sorted(
        {term["lecture"] for term in glossary}
    )

    if lecture not in available_lectures:
        available = ", ".join(
            f"第{number}回" for number in available_lectures
        )
        return (
            f"第{lecture}回は指定できません。"
            f"利用可能なのは {available} です。"
        )

    terms = [
        term
        for term in glossary
        if term["lecture"] == lecture
    ]

    term = random.choice(terms)

    return json.dumps(
        {
            "question": f"「{term['term']}」の意味は？",
            "answer": term["definition"],
        },
        ensure_ascii=False,
    )

 
ツール追加後、ターミナルから以下のとおりコマンドを実行した。

①uv run python -m app.mcp_agent --trace \ "第6回の確認問題を作って"

②uv run python -c "from app.tools import make_quiz; print(make_quiz(99))"

③uv run python -m app.mcp_agent --trace \ "第99回の確認問題を作って"

- 結果：出力結果は以下のとおりだった。
- 
①
[server] glossary: コマンドで起動（uv run python -m app.mcp_server）
[server] fetch: コマンドで起動（uvx mcp-server-fetch==2026.11.11 --no-readability-js）
[server] aws: URL に接続（https://knowledge-mcp.global.api.aws）
[model] gemini-3.5-flash-lite
[tools] MCP サーバー（app/mcp_servers.json）
第6回の内容から確認問題を作成しました！

**【確認問題】**
「MCPサーバー」とは何をするプログラムでしょうか？その役割と仕組みを説明してください。

---
**（ヒント・考え方）**
MCP（Model Context Protocol）のアーキテクチャにおいて、クライアント（AIモデル側やホストアプリケーション）に対して「何を提供する側」のプログラムだったかを思い出してみましょう。ツールの名前や説明、引数のスキーマを公開し、要求に応じて実行結果を返すサーバーの役割を考えてみてくださいね！

（関係する回：第6回）

--- trace ---
[tool call]   glossary_make_quiz({'lecture': 6})
[tool return] glossary_make_quiz -> {"question": "「MCPサーバー」の意味は？", "answer": "MCPでツール（やリソース、プロンプト）を公開する側のプログラム。公開するのはツールの名前・説明文・引数のスキーマで、クライアントの要求に応じてツールを実行…
[usage] 入力 5,634 tokens / 出力 143 tokens / モデル応答 2 回

②
第99回は指定できません。利用可能なのは 第1回, 第2回, 第3回, 第4回, 第6回 です。

③
[server] glossary: コマンドで起動（uv run python -m app.mcp_server）
[server] fetch: コマンドで起動（uvx mcp-server-fetch==2026.11.11 --no-readability-js）
[server] aws: URL に接続（https://knowledge-mcp.global.api.aws）
[model] gemini-3.5-flash-lite
[tools] MCP サーバー（app/mcp_servers.json）
第99回は講座に存在しません。現在利用可能な回は、**第1回、第2回、第3回、第4回、第6回**です。

どの回の確認問題を作成しますか？該当する回を指定していただければ、問題を作成いたします。

--- trace ---
[tool call]   glossary_make_quiz({'lecture': 99})
[tool return] glossary_make_quiz -> 第99回は指定できません。利用可能なのは 第1回, 第2回, 第3回, 第4回, 第6回 です。…
[usage] 入力 5,570 tokens / 出力 78 tokens / モデル応答 2 回

- 気づき・疑問：

- app.tools.pyに自分で追加したツールをエージェントがMCPサーバから呼び出して使えることを確認できた。
  
- ②のように明示的にツールの場所を指定してMCPサーバを経由せずに実行することもできることが確認できた。
 
- app.tools.py側のコードを修正することで、第99回の確認問題を作成するといった引数が範囲外となる場合、ツールからエージェントには「glossary_make_quiz -> 第99回は指定できません。利用可能なのは 第1回, 第2回, 第3回, 第4回, 第6回 です。」といったメッセージが返却され、エージェントは、ツールの返却結果と、それを踏まえたユーザーへの確認（どの回の問題を作成するかの確認）を出力することが確認できた。
