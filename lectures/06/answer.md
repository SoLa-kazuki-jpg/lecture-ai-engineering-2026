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

- 取り組んだレベル：基礎 / 実装 / 発展
- やったこと：
- 結果：
- 気づき・疑問：
