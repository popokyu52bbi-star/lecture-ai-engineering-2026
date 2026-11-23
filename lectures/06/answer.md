# 第6回 解答

## 演習

- `--list-tools` で見えたツール名と、第3回（`--mcp` なし）との違い：
  - 第3回（`--mcp` なし）：`search_materials`
  - MCPあり：`glossary_search_glossary`、`glossary_get_lecture_summary`、`fetch_fetch`
  - 第3回は直接登録された関数ツールが1個だけ見えていたが、MCPではサーバー名をプレフィックスにしたツールが3個見えていた。
  - 公開されているツールの説明文は、直接登録の場合もMCP経由の場合も関数のdocstringまたはサーバー側の説明がモデルへ渡される形だった。
- `--mcp --trace` で見えたツール呼び出し（ツール名と引数）と、`requests` の回数：
  - 実行時にモデルが503で混み合っていたため、用語集の質問と第2回の質問では回答およびツール呼び出しを観測できなかった。
  - そのため、MCP経由と第3回の直接登録方式の`requests` 回数は比較できず、推測で埋めていない。
  - 問題が解消したら、同じ質問を再実行して`[tool call]`と`[usage]`の行を記録する。
- MCP サーバーに公開したツールと、その説明文：
  - `glossary_search_glossary`
    - 説明文：講座の用語集から用語を検索し、定義とどの回で扱うかを返す。
    - 引数：`query`（string）
  - `glossary_get_lecture_summary`
    - 説明文：指定した回（1始まりの整数）の講義の要点を返す。
    - 引数：`lecture`（integer）
- fetch サーバーを足したときのツール呼び出し（ツール名と引数）：
  - MCP一覧には`fetch_fetch`が表示され、引数は`url`、`max_length`、`start_index`、`raw`、`timeout_ms`だった。
  - fetchを使った実行では、`https://modelcontextprotocol.io/`へのアクセスがサーバー側で行われ、robots.txtの取得、リダイレクト、Markdown取得まで確認できた。
  - ただしモデルがその直後に503で失敗したため、エージェントの最終traceに表示される正確な`fetch_fetch`の引数は未観測だった。

## 宿題

- 取り組んだレベル：基礎
- やったこと：
  - MCPサーバーをstdioで起動する設定と、Streamable HTTPで起動する設定を確認した。
  - stdioではエージェントがサーバーをサブプロセスとして起動し、標準入出力で通信する。
  - HTTPでは `uv run python -m app.mcp_server --http` を別プロセスで起動し、`app/mcp_servers.json` の `glossary` を `"url": "http://127.0.0.1:8000/mcp"` に切り替えた。
  - `uv run python -m app.agent --mcp --list-tools`で、MCP経由で公開されたツールを確認した。
  - 実際のGemini APIが503を返したため、代わりに `uv run pytest -q tests/test_mcp_server.py` と `uv run pytest -q` を実行した。
- 結果：
  - `--list-tools`では `glossary_search_glossary`、`glossary_get_lecture_summary`、`fetch_fetch`が表示された。
  - MCPサーバーのツール公開、MCP経由の呼び出し、結果がエージェントへ戻ることをテストで確認した。
  - `tests/test_mcp_server.py`でMCPのテストを実行できる状態であり、全テストは `17 passed`だった。
  - Gemini APIが503を返したため、実際のツール呼び出しの引数と、モデル応答の`requests`回数は観測できなかった。
- 気づき・疑問：
  - stdioはクライアントがサーバーを起動するためポートが不要で、HTTPは利用者がサーバーを別プロセスとして起動し、クライアントは固定のURLへ接続する点が異なる。
  - MCPサーバーの配線は、実際のLLMを使わない`TestModel`でも確認できる。
  - 503が解消されたら、同じ質問をstdio版とHTTP版の両方で再実行し、ツール名、引数、回答、`requests`を比較すべきである。
