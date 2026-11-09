---
{"dg-publish":true,"permalink":"/claude/chats/2026-11-08-pydantic-ai-lang-graph-lang-chain-51b9d603/","title":"Pydantic AI と LangGraph/LangChain","tags":["claude","chat","cat/ml-dl","cat/python","cat/github","cat/linux","cat/agentic-ai"],"dg-note-properties":{"title":"Pydantic AI と LangGraph/LangChain","date":"2026-11-08","created":"2026-11-08T05:29:20.872223Z","updated":"2026-11-08T08:54:37.381255Z","account":"権藤祥太郎 (shotaro.gondo@gmail.com)","conversation_id":"51b9d603-0fb9-4df9-a458-3715264fae2e","source":"claude-chat-export","tags":["claude","chat","cat/ml-dl","cat/python","cat/github","cat/linux","cat/agentic-ai"]}}
---

# Pydantic AI と LangGraph/LangChain

## 🧑 User  
*2026-11-08T05:29:21.791042Z*

Pydantic AI とはどのようなパッケージですか？


---

## 🤖 Claude  
*2026-11-08T05:29:33.778925Z*

> [!note]- 🔧 Tool call: WebSearch
> ```json
> {
>   "mode": "standard",
>   "query": "Pydantic AI agent framework"
> }
> ```

> [!note]- ↩️ Tool result: tool
> ```
> Pydantic AI
> https://www.promptlayer.com/glossary/pydantic-ai
>
> Capability · Framework — agents
> https://learn.engineering.vips.edu/frameworks/pydantic-ai
>
> Pydantic AI: Agent Framework
> https://medium.com/ai-agent-insider/pydantic-ai-agent-framework-02b138e8db71
>
> GAASAgentic AI as a Service
> https://gaas.it.com/overview-of-pydantic-ai.html
>
> pydantic-ai  by pydantic
> https://www.sourcepulse.org/projects/1833822
>
> pydantic ai
> https://www.respan.ai/market-map/pydantic-ai
>
> pydantic ai
> https://aiagentrank.io/glossary/pydantic-ai
> ```

Pydantic AI は、Pydantic の開発チームが作っている、型安全なエージェント構築用の Python フレームワークです。Pydantic をベースに、入力・出力・ツールのスキーマを構造化して扱えます。目指しているのは「LLM 向けの FastAPI」のような開発体験です。モデル非依存で、OpenAI、Anthropic、Google、Groq、Mistral、Ollama などで動きます。

## 主な特徴

- **Agent が中心の設計**: Agent に指示、ツール、構造化された出力型、依存関係の型、モデル設定を渡します。
- **構造化出力とバリデーション**: 期待する出力を Pydantic モデルで定義すると、フレームワークが自動で検証し、失敗した場合は再プロンプトします。
- **型付きツールと依存性注入**: ツールは型注釈付きの Python 関数として宣言します。RunContext 経由で依存オブジェクトを渡せるので、テストしやすい構成になります。
- **ストリーミング**: ストリーミング応答でも、受信しながら検証できます。
- **Pydantic Graph**: 複雑なエージェントのワークフローを定義するための機能です。
- **オブザーバビリティ**: 同じチームの Pydantic Logfire と密に連携し、エージェントの実行を可視化できます。

## 位置づけ

LangChain や LlamaIndex より小さく、Python らしい書き方に寄せた設計です。ライセンスは MIT です。2024 年末に始まり、2025 年に 1.0 が出ました。

LangGraph との比較では、Pydantic AI は小さく型安全で習得が早く、複雑なステートマシンや human-in-the-loop は LangGraph が得意とされています。

## 最小コード例

```python
from pydantic import BaseModel
from pydantic_ai import Agent

class Answer(BaseModel):
    city: str
    population: int

agent = Agent('anthropic:claude-sonnet-5-5', output_type=Answer)
result = agent.run_sync('インドの首都と人口は？')
print(result.output)
```

インストールは `pip install pydantic-ai` です。

LangGraph や CrewAI、MCP を使われているようなので、Pydantic AI も MCP クライアントとしてツール連携できます。LangGraph との使い分けや、既存の構成への組み込み方も整理できます。

Sources:
- [Pydantic AI (PromptLayer)](https://www.promptlayer.com/glossary/pydantic-ai)
- [Pydantic AI framework facts](https://learn.engineering.vips.edu/frameworks/pydantic-ai)
- [pydantic-ai by pydantic (SourcePulse)](https://www.sourcepulse.org/projects/1833822)
- [An Overview of Pydantic AI (GAAS)](https://gaas.it.com/overview-of-pydantic-ai.html)
- [pydantic ai (aiagentrank)](https://aiagentrank.io/glossary/pydantic-ai)


---

## 🧑 User  
*2026-11-08T05:30:13.798644Z*

Pydantic AI を LangGraph/LangChain と組み合わせて使うことができますか？


---

## 🤖 Claude  
*2026-11-08T05:30:29.331677Z*

> [!note]- 🔧 Tool call: WebSearch
> ```json
> {
>   "mode": "standard",
>   "query": "Pydantic AI with LangGraph integration LangChain tools"
> }
> ```

> [!note]- ↩️ Tool result: tool
> ```
> Pydantic AI vs LangChain
> https://composio.dev/content/pydantic-ai-vs-langchain
>
> Blog Pydantic AI vs LangGraph: Features, Integrations, and Pricing Compared
> https://zenml.io/blog/pydantic-ai-vs-langgraph
>
> Pydantic AI vs LangChain & LangGraph
> https://pydantic.dev/docs/ai/comparisons/vs-langchain-langgraph/
>
> The Ultimate Guide to Building AI Agents with PydanticAI and LangChain
> https://blog.dataengineerthings.org/the-ultimate-guide-to-building-ai-agents-with-pydanticai-and-langchain-95d1f8a7775c
>
> Pydantic AIとLangGraphで同じエージェントを書き比べてみる
> https://dev.classmethod.jp/articles/pydantic-ai-vs-langgraph/
>
> pydantic ai
> https://vantaige.io/es/ai-tool/pydantic-ai
>
> langgraph vs pydantic ai
> https://www.respan.ai/market-map/compare/langgraph-vs-pydantic-ai
> ```

はい、組み合わせられます。ただし、公式の「統合パッケージ」で繋ぐというより、**自分でノードやツールの境界に差し込む**形になります。検索で見つけた範囲では、公式の連携手順をまとめた資料は見つかりませんでした。以下のパターンは、両者の設計から考えたものです。

## 位置づけの違い

- **LangGraph**: グラフでフローを組む。Pydantic AI は「1つのエージェントの入出力を型で固める」フレームワークという整理があります。
- したがって、**LangGraph が全体の制御フロー、Pydantic AI が個々のノード内の型付きエージェント**、という分担が自然です。

## 組み合わせパターン

1. **LangGraph のノード内で Pydantic AI の Agent を呼ぶ**
   ノード関数の中で `agent.run_sync(...)` や `await agent.run(...)` を呼び、`result.output`（Pydantic モデル）を State に格納します。分岐、状態管理、human-in-the-loop は LangGraph に任せ、構造化出力の検証とリトライは Pydantic AI に任せられます。

2. **LangChain のツール資産を Pydantic AI で使う**
   Pydantic AI は型注釈付きの関数をツールにできます。LangChain ツールは、薄いラッパー関数で包めば呼べます。MCP サーバー経由でツールを共通化する方法もあります。例えばComposio は、LangChain 向けには provider パッケージ、Pydantic AI 向けには MCP ツールセットで、両方に同じツール基盤を提供しています。

3. **MCP を共通のツール層にする**
   ご自身の構成（ServiceNow、AWX、GitHub 連携など）なら、ツールを MCP サーバーとして公開すれば、LangGraph 側と Pydantic AI 側のどちらからも同じツールを使えます。フレームワークを混在させても、ツール実装が二重にならずに済みます。

## 注意点

- **Pydantic のバージョン**: 両者とも Pydantic v2 前提です。v1 と v2 が混在すると、特に LangChain の新しいモデルを使うときにクラッシュしやすいと報告されています。
- **古い記事のコードに注意**: 古い解説にある `from pydantic_ai import tool` のような書き方は動かない例が報告されています。Pydantic AI は `@agent.tool` / `@agent.tool_plain` が現在の書き方です。
- **メッセージ形式は別物**: LangGraph は State の `messages`、Pydantic AI は `result.all_messages()` を使います。会話履歴を受け渡す場合は、変換が必要です。
- **トレーシングが二系統になる**: LangSmith と Logfire が別々になります。OpenTelemetry に寄せるなど、方針を決めておくと運用が楽です。
- **役割が重なる部分**: Pydantic AI 側にもグラフ機能（Pydantic Graph）やマルチエージェント機能があります。LangGraph を全体制御に使うなら、重複部分は使わない、と決めておくと設計が混乱しません。

## 使い分けの目安

LangGraph が複雑な状態遷移や human-in-the-loop に強く、Pydantic AI は型安全性と軽さに強い、というのが共通した評価です。LangChain 1.0 は内部で LangGraph を使っているので、まず LangChain の簡単なインターフェースから始め、必要なときに LangGraph へ降りるという使い方もできます。

必要なら、LangGraph のノードから Pydantic AI の Agent を呼ぶ最小コード例を書きます。

Sources:
- [Pydantic AI vs LangChain (Composio)](https://composio.dev/content/pydantic-ai-vs-langchain)
- [Pydantic AI vs LangGraph (ZenML)](https://zenml.io/blog/pydantic-ai-vs-langgraph)
- [Pydantic AI vs LangChain & LangGraph (Pydantic docs)](https://pydantic.dev/docs/ai/comparisons/vs-langchain-langgraph/)
- [PydanticAI and LangChain guide (Data Engineer Things)](https://blog.dataengineerthings.org/the-ultimate-guide-to-building-ai-agents-with-pydanticai-and-langchain-95d1f8a7775c)
- [Pydantic AIとLangGraphで同じエージェントを書き比べてみる (Classmethod)](https://dev.classmethod.jp/articles/pydantic-ai-vs-langgraph/)
- [langgraph vs pydantic ai (Respan)](https://www.respan.ai/market-map/compare/langgraph-vs-pydantic-ai)


---

## 🧑 User  
*2026-11-08T05:39:39.732811Z*

LangGraph のノードから Pydantic AI の Agent を呼ぶ最小コード例を書いてください


---

## 🤖 Claude  
*2026-11-08T05:40:39.408974Z*

📁 添付（本文非取得）: `fbfc83fa-1641-565c-b2a9-e5579f11b54e`

📁 添付（本文非取得）: `10dde217-c9ea-4782-9c35-38f85f084135`

> [!note]- 🔧 Tool call: Write
> ```json
> {
>   "content": "\"\"\"LangGraph のノードから Pydantic AI の Agent を呼ぶ最小例。\n\npip install langgraph pydantic-ai\nexport ANTHROPIC_API_KEY=...   # 他プロバイダーなら MODEL を変更\n\"\"\"\nimport asyncio\nfrom typing import Literal, TypedDict\n\nfrom langgraph.graph import END, START, StateGraph\nfrom pydantic import BaseModel\nfrom pydantic_ai import Agent\n\nMODEL = \"anthropic:claude-sonnet-5-5\"\n\n\n# --- Pydantic AI 側: 出力を型で固めた Agent -------------------------------\nclass Triage(BaseModel):\n    category: Literal[\"incident\", \"request\", \"other\"]\n    priority: int  # 1(高)〜4(低)\n    summary: str\n\n\ntriage_agent = Agent(\n    MODEL,\n    output_type=Triage,\n    instructions=\"ITチケットの本文を分類し、優先度と要約を返してください。\",\n)\n\n\n# --- LangGraph 側: State とノード ------------------------------------------\nclass State(TypedDict, total=False):\n    ticket: str\n    triage: Triage\n    action: str\n\n\nasync def triage_node(state: State) -> State:\n    # ノード内で Pydantic AI を呼ぶ。result.output は検証済みの Triage\n    result = await triage_agent.run(state[\"ticket\"])\n    return {\"triage\": result.output}\n\n\ndef route(state: State) -> str:\n    # 型付きの出力をそのまま条件分岐に使える\n    t = state[\"triage\"]\n    return \"escalate\" if t.category == \"incident\" and t.priority <= 2 else \"queue\"\n\n\ndef escalate_node(state: State) -> State:\n    return {\"action\": f\"オンコールへエスカレーション: {state['triage'].summary}\"}\n\n\ndef queue_node(state: State) -> State:\n    return {\"action\": f\"通常キューへ登録: {state['triage'].summary}\"}\n\n\ngraph = StateGraph(State)\ngraph.add_node(\"triage\", triage_node)\ngraph.add_node(\"escalate\", escalate_node)\ngraph.add_node(\"queue\", queue_node)\ngraph.add_edge(START, \"triage\")\ngraph.add_conditional_edges(\"triage\", route, [\"escalate\", \"queue\"])\ngraph.add_edge(\"escalate\", END)\ngraph.add_edge(\"queue\", END)\napp = graph.compile()\n\n\nif __name__ == \"__main__\":\n    out = asyncio.run(\n        app.ainvoke({\"ticket\": \"本番のAWXジョブが全件失敗しています。至急対応願います。\"})\n    )\n    print(out[\"triage\"])\n    print(out[\"action\"])\n",
>   "file_path": "/home/claude/langgraph_pydantic_ai.py"
> }
> ```

> [!note]- ↩️ Tool result: tool
> ```
> File created successfully at: /home/claude/langgraph_pydantic_ai.py (file state is current in your context — no need to Read it back)
> ```

> [!note]- 🔧 Tool call: Bash
> ```
> cd /home/claude && pip install -q --break-system-packages langgraph pydantic-ai 2>&1 | tail -2; mkdir -p /tmp/t && cat > /tmp/t/run.py <<'EOF'
> import sys, asyncio
> sys.path.insert(0, "/home/claude")
> import os
> os.environ["ANTHROPIC_API_KEY"] = "dummy"
> import langgraph_pydantic_ai as m
> from pydantic_ai.models.test import TestModel
> with m.triage_agent.override(model=TestModel(custom_output_args={"category":"incident","priority":1,"summary":"AWX全件失敗"})):
>     out = asyncio.run(m.app.ainvoke({"ticket":"x"}))
> print(out["triage"]); print(out["action"])
> EOF
> python3 -I /tmp/t/run.py 2>&1 | tail -8
> ```

> [!note]- ↩️ Tool result: tool
> ```
> hint: The package was installed by debian. You should check if it can uninstall the package.
> Traceback (most recent call last):
>   File "/tmp/t/run.py", line 5, in <module>
>     import langgraph_pydantic_ai as m
>   File "/home/claude/langgraph_pydantic_ai.py", line 9, in <module>
>     from langgraph.graph import END, START, StateGraph
> ModuleNotFoundError: No module named 'langgraph'
> ```

> [!note]- 🔧 Tool call: Bash
> ```
> cd /tmp && pip install --break-system-packages langgraph pydantic-ai 2>&1 | tail -5
> ```

> [!note]- ↩️ Tool result: tool
> ```
> × Cannot uninstall argcomplete 3.1.4
> ╰─> The package's contents are unknown: no RECORD file was found for argcomplete.
>
> hint: The package was installed by debian. You should check if it can uninstall the package.
> Shell cwd was reset to /home/claude
> ```

> [!note]- 🔧 Tool call: Bash
> ```
> cd /tmp && python3 -m venv /tmp/venv && /tmp/venv/bin/pip install -q langgraph pydantic-ai 2>&1 | tail -3; /tmp/venv/bin/python -I /tmp/t/run.py 2>&1 | tail -8
> ```

> [!note]- ↩️ Tool result: tool
> ```
> /___.___\
>   /    |    \    observability: off — see every model and tool call live, with cost
> /      |      \    set it up free with Pydantic Logfire: uvx logfire-cli wizard --task pydantic-ai
> `---.._|_..---'    or use any OpenTelemetry backend: https://pydantic.dev/docs/ai/logfire/#otel
>
>                  goes away once observability is on — or PYDANTIC_AI_NO_BANNER=1
> category='incident' priority=1 summary='AWX全件失敗'
> オンコールへエスカレーション: AWX全件失敗
> Shell cwd was reset to /home/claude
> ```

> [!note]- 🔧 Tool call: SendUserFile
> ```json
> {
>   "caption": "LangGraph ノード内で Pydantic AI の Agent を呼ぶ最小例",
>   "display": "attach",
>   "files": [
>     "/home/claude/langgraph_pydantic_ai.py"
>   ],
>   "status": "normal"
> }
> ```

> [!note]- ↩️ Tool result: tool
> ```
> 1 file delivered to user.
>   /home/claude/langgraph_pydantic_ai.py → file_uuid: 10dde217-c9ea-4782-9c35-38f85f084135
> Each file shows as a card whose position relative to your text depends on the app the user is reading in, so refer to a file by its name, never as above or below.
> ```

> [!quote]- 📄 Artifact: langgraph_pydantic_ai.py
> Claude.ai 側のパス: `/home/claude/langgraph_pydantic_ai.py`
> このMac上には無し。記事内のツール呼び出し（作成＋編集）から復元した内容
>
> ```python
> """LangGraph のノードから Pydantic AI の Agent を呼ぶ最小例。
>
> pip install langgraph pydantic-ai
> export ANTHROPIC_API_KEY=...   # 他プロバイダーなら MODEL を変更
> """
> import asyncio
> from typing import Literal, TypedDict
>
> from langgraph.graph import END, START, StateGraph
> from pydantic import BaseModel
> from pydantic_ai import Agent
>
> MODEL = "anthropic:claude-sonnet-5-5"
>
>
> # --- Pydantic AI 側: 出力を型で固めた Agent -------------------------------
> class Triage(BaseModel):
>     category: Literal["incident", "request", "other"]
>     priority: int  # 1(高)〜4(低)
>     summary: str
>
>
> triage_agent = Agent(
>     MODEL,
>     output_type=Triage,
>     instructions="ITチケットの本文を分類し、優先度と要約を返してください。",
> )
>
>
> # --- LangGraph 側: State とノード ------------------------------------------
> class State(TypedDict, total=False):
>     ticket: str
>     triage: Triage
>     action: str
>
>
> async def triage_node(state: State) -> State:
>     # ノード内で Pydantic AI を呼ぶ。result.output は検証済みの Triage
>     result = await triage_agent.run(state["ticket"])
>     return {"triage": result.output}
>
>
> def route(state: State) -> str:
>     # 型付きの出力をそのまま条件分岐に使える
>     t = state["triage"]
>     return "escalate" if t.category == "incident" and t.priority <= 2 else "queue"
>
>
> def escalate_node(state: State) -> State:
>     return {"action": f"オンコールへエスカレーション: {state['triage'].summary}"}
>
>
> def queue_node(state: State) -> State:
>     return {"action": f"通常キューへ登録: {state['triage'].summary}"}
>
>
> graph = StateGraph(State)
> graph.add_node("triage", triage_node)
> graph.add_node("escalate", escalate_node)
> graph.add_node("queue", queue_node)
> graph.add_edge(START, "triage")
> graph.add_conditional_edges("triage", route, ["escalate", "queue"])
> graph.add_edge("escalate", END)
> graph.add_edge("queue", END)
> app = graph.compile()
>
>
> if __name__ == "__main__":
>     out = asyncio.run(
>         app.ainvoke({"ticket": "本番のAWXジョブが全件失敗しています。至急対応願います。"})
>     )
>     print(out["triage"])
>     print(out["action"])
> ```

`langgraph_pydantic_ai.py` を送りました。

**構成**
- Pydantic AI 側で `Triage` という Pydantic モデルを出力型にした `Agent` を定義しています。
- LangGraph 側では `triage_node` の中で `await triage_agent.run(...)` を呼び、検証済みの `result.output` を State に入れます。
- 次の条件分岐 `route` は、その型付きの値（`category` と `priority`）をそのまま見て `escalate` か `queue` に振り分けます。

**動作確認**
Pydantic AI の `TestModel` で LLM を差し替え、グラフ全体を実行しました。ノード呼び出し、型付き出力の受け渡し、分岐は意図どおり動きました。実際の LLM への接続は API キーがないため試していません。

**実行方法**
`pip install langgraph pydantic-ai` のあと、`ANTHROPIC_API_KEY` を設定して `python langgraph_pydantic_ai.py` を実行します。別のプロバイダーを使う場合は、冒頭の `MODEL` 文字列（例: `openai:...`）を変更してください。

次のステップとして、ノード内で Pydantic AI に MCP サーバーのツールを持たせる形にもできます。


---

## 🧑 User  
*2026-11-08T08:53:46.799678Z*

pydantic_ai.Agent に instructions 引数で与えたプロンプト文をシステムプロンプトとしてLLMに入力するコード、及び、pydantic_ai.Agent.tool_plain に引数としてあたえた Callable がLLMのツールとして動作するようにしているコードは、GitHub の何処にありますか？


---

## 🤖 Claude  
*2026-11-08T08:54:37.520838Z*

> [!note]- 🔧 Tool call: Bash
> ```
> mkdir -p /tmp/pai && cd /tmp/pai && git clone -q --depth 1 https://github.com/pydantic/pydantic-ai.git src 2>&1 | tail -2; cd src && git log -1 --format='%h %cd' && ls && grep -rn "def tool_plain" --include=*.py pydantic_ai_slim | head; grep -rn "instructions" --include=*.py pydantic_ai_slim/pydantic_ai/_agent_graph.py | head -30
> ```

> [!note]- ↩️ Tool result: tool
> ```
> 0f7648a Sun Nov 8 02:07:26 2026 +0000
> AGENTS.md
> CITATION.cff
> CLAUDE.md
> CONTRIBUTING.md
> LICENSE
> Makefile
> PLAN.md
> README.md
> THREAT_MODEL.md
> agent_docs
> clai
> docs
> examples
> pydantic_ai_slim
> pydantic_evals
> pydantic_graph
> pyproject.toml
> scripts
> src
> tests
> uv.lock
> pydantic_ai_slim/pydantic_ai/agent/__init__.py:3222:    def tool_plain(self, func: ToolFuncPlain[ToolParams], /) -> ToolFuncPlain[ToolParams]: ...
> pydantic_ai_slim/pydantic_ai/agent/__init__.py:3225:    def tool_plain(
> pydantic_ai_slim/pydantic_ai/agent/__init__.py:3248:    def tool_plain(
> pydantic_ai_slim/pydantic_ai/agent/__init__.py:3270:    def tool_plain(
> pydantic_ai_slim/pydantic_ai/toolsets/function.py:345:    def tool_plain(self, func: ToolFuncPlain[ToolParams], /) -> ToolFuncPlain[ToolParams]: ...
> pydantic_ai_slim/pydantic_ai/toolsets/function.py:348:    def tool_plain(
> pydantic_ai_slim/pydantic_ai/toolsets/function.py:371:    def tool_plain(
> pydantic_ai_slim/pydantic_ai/toolsets/function.py:393:    def tool_plain(
> pydantic_ai_slim/pydantic_ai/capabilities/capability.py:187:    def tool_plain(self, func: ToolFuncPlain[ToolParams], /) -> ToolFuncPlain[ToolParams]: ...
> pydantic_ai_slim/pydantic_ai/capabilities/capability.py:190:    def tool_plain(
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:46:    get_instructions as _get_history_instructions,
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:47:    get_instructions_source as _get_history_instructions_source,
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:104:from ._instructions import (
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:162:    resolve_loaded_capability_instructions,
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:164:from .toolsets._instruction_collection import collect_toolset_instructions
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:896:    get_instructions: Callable[[RunContext[DepsT]], Awaitable[list[_messages.InstructionPart] | None]]
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1085:    """The node that handles the user prompt and instructions."""
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1093:    instructions: str | None = None
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1094:    instructions_functions: list[_system_prompt.SystemPromptRunner[DepsT]] = dataclasses.field(
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1174:                        # of instructions.  Instructions will be applied by ModelRequestNode.run() on
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1177:                    instruction_parts = await _get_instructions(ctx, run_context)
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1179:                        # No pending tool calls and no instructions — nothing new to send to the model.
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1348:    """The request a run resuming from `request` without a new prompt sends, before its instructions are added."""
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1384:    parts on a fresh run and its instructions. It shares `UserPromptNode`'s history cleanup and
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1408:async def _get_instructions(
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1412:    """Combine base instructions (from agent/capabilities) with toolset instructions.
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1414:    Toolset instructions are fetched from the current tool manager's toolset,
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1419:    base = await ctx.deps.get_instructions(run_context)
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1423:    parts.extend(await collect_toolset_instructions(ctx.deps.tool_manager.toolset, run_context))
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1428:async def _get_loaded_instructions(
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1433:    """Resolve the instructions of the deferred capabilities in `capability_ids`, the ones the model has loaded.
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1443:            resolved[capability_id] = await resolve_loaded_capability_instructions(
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1490:    instructions the model never received.
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1492:    `None` means "unset" rather than "no instructions" — it's what makes `Model._get_instruction_parts`
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1493:    fall back to the request's own `instructions` — so it leaves the request alone.
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:1496:        request.instructions = _messages.InstructionPart.join(instruction_parts)
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:2941:        # Fetch instructions now that dynamic toolsets have been resolved by for_run_step.
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:2942:        instruction_parts = await _get_instructions(ctx, run_context)
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:2945:        self.request.instructions = _messages.InstructionPart.join(instruction_parts) if instruction_parts else None
> pydantic_ai_slim/pydantic_ai/_agent_graph.py:2950:        # Validate after instructions are resolved; self.request was appended above so [:-1] is prior history
> Shell cwd was reset to /home/claude
> ```

> [!note]- 🔧 Tool call: Bash
> ```
> cd /tmp/pai/src/pydantic_ai_slim/pydantic_ai && sed -n 3270,3330p agent/__init__.py; echo ----; sed -n 393,450p toolsets/function.py; echo ----; grep -n "def add_function\|def _function_schema\|function_schema(" toolsets/function.py _function_schema.py tools.py | head -20; echo ---; grep -n "_get_instruction_parts\|def _get_instructions\|InstructionPart" models/__init__.py | head -20; echo ---; grep -n "system=\|system_prompt\|_map_message\|tools=\|_get_tools\|def _map_tool_definition" models/anthropic.py | head -30
> ```

> [!note]- ↩️ Tool result: tool
> ````
> def tool_plain(
>         self,
>         func: ToolFuncPlain[ToolParams] | None = None,
>         /,
>         *,
>         name: str | None = None,
>         description: str | None = None,
>         retries: int | None = None,
>         prepare: ToolPrepareFunc[AgentDepsT] | None = None,
>         args_validator: ArgsValidatorFunc[AgentDepsT, ToolParams] | None = None,
>         docstring_format: DocstringFormat = 'auto',
>         require_parameter_descriptions: bool = False,
>         schema_generator: type[GenerateJsonSchema] = GenerateToolJsonSchema,
>         strict: bool | None = None,
>         sequential: bool = False,
>         requires_approval: bool = False,
>         metadata: dict[str, Any] | None = None,
>         tool_kind: _messages.ToolKindLike | None = None,
>         timeout: float | None = None,
>         defer_loading: bool = False,
>         include_return_schema: bool | None = None,
>     ) -> Any:
>         """Decorator to register a tool function which DOES NOT take `RunContext` as an argument.
>
>         Can decorate a sync or async functions.
>
>         The docstring is inspected to extract both the tool description and description of each parameter,
>         [learn more](../tools.md#function-tools-and-schema).
>
>         We can't add overloads for every possible signature of tool, since the return type is a recursive union
>         so the signature of functions decorated with `@agent.tool` is obscured.
>
>         Example:
>         ```python
>         from pydantic_ai import Agent
>
>         agent = Agent('test')
>
>         @agent.tool_plain
>         def foobar() -> int:
>             return 123
>
>         @agent.tool_plain(retries=2)
>         async def spam() -> float:
>             return 3.14
>
>         result = agent.run_sync('foobar')
>         print(result.output)
>         #> {"foobar":123,"spam":3.14}
>         ```
>
>         Args:
>             func: The tool function to register.
>             name: The name of the tool, defaults to the function name.
>             description: The description of the tool, defaults to the function docstring.
>             retries: The number of retries to allow for this tool, defaults to the agent's default retries,
>                 which defaults to 1.
>             prepare: custom method to prepare the tool definition for each step, return `None` to omit this
>                 tool from a given step. This is useful if you want to customise a tool at call time,
>                 or omit it completely from a step. See [`ToolPrepareFunc`][pydantic_ai.tools.ToolPrepareFunc].
>             args_validator: custom method to validate tool arguments after schema validation has passed,
> ----
>     def tool_plain(
>         self,
>         func: ToolFuncPlain[ToolParams] | None = None,
>         /,
>         *,
>         name: str | None = None,
>         description: str | None = None,
>         retries: int | None = None,
>         prepare: ToolPrepareFunc[AgentDepsT] | None = None,
>         args_validator: ArgsValidatorFunc[AgentDepsT, ToolParams] | None = None,
>         docstring_format: DocstringFormat | None = None,
>         require_parameter_descriptions: bool | None = None,
>         schema_generator: type[GenerateJsonSchema] | None = None,
>         strict: bool | None = None,
>         sequential: bool | None = None,
>         requires_approval: bool | None = None,
>         metadata: dict[str, Any] | None = None,
>         tool_kind: ToolKindLike | None = None,
>         timeout: float | None = None,
>         defer_loading: bool | None = None,
>         include_return_schema: bool | None = None,
>     ) -> Any:
>         """Decorator to register a tool function which DOES NOT take `RunContext` as an argument.
>
>         Can decorate a sync or async functions.
>
>         The docstring is inspected to extract both the tool description and description of each parameter,
>         [learn more](../tools.md#function-tools-and-schema).
>
>         We can't add overloads for every possible signature of tool, since the return type is a recursive union
>         so the signature of functions decorated with `@toolset.tool_plain` is obscured.
>
>         Example:
>         ```python
>         from pydantic_ai import Agent, FunctionToolset
>
>         toolset = FunctionToolset()
>
>         @toolset.tool_plain
>         def foobar(x: int) -> int:
>             return x + 1
>
>         @toolset.tool_plain(retries=2)
>         async def spam(y: float) -> float:
>             return y * 2.0
>
>         agent = Agent('test', toolsets=[toolset])
>         result = agent.run_sync('foobar')
>         print(result.output)
>         #> {"foobar":1,"spam":0.0}
>         ```
>
>         Args:
>             func: The tool function to register.
>             name: The name of the tool, defaults to the function name.
>             description: The description of the tool, defaults to the function docstring.
>             retries: The number of retries to allow for this tool, defaults to the toolset's default retries,
>                 which defaults to the agent's default.
> ----
> toolsets/function.py:614:    def add_function(
> tools.py:483:        self.function_schema = function_schema or _function_schema.function_schema(
> _function_schema.py:110:def function_schema(
> ---
> 119:    InstructionPart,
> 370:    instruction_parts: list[InstructionPart] | None = None
> 418:    def instruction_visibility_of(self, part: InstructionPart) -> InstructionVisibility:
> 1723:            parts = [*(params.instruction_parts or []), InstructionPart(content=output_instr)]
> 1724:            params = replace(params, instruction_parts=InstructionPart.sorted(parts))
> 1992:                    or self._get_instruction_parts(messages, model_request_parameters or ModelRequestParameters())
> 2208:    def _get_instruction_parts(
> 2210:    ) -> list[InstructionPart] | None:
> 2231:                    return [InstructionPart(content=message.instructions)]
> 2244:                return [InstructionPart(content=second.instructions)]
> 4079:    messages: Sequence[ModelMessage], *, inline_system: bool, prefix: list[InstructionPart] | None
> 4100:    rendered_prefix = InstructionPart.join(_render_append_parts(prefix)) if prefix else None
> 4145:            and instructions == InstructionPart.join(prefix)
> 4321:def _first_context_request(messages: list[ModelMessage], prefix: Sequence[InstructionPart] | None) -> int | None:
> 4339:        return any(isinstance(block, InstructionPart) and block.on_change == 'append' for block in part.instructions)
> ---
> ugrep: warning: models/anthropic.py: No such file or directory
> Shell cwd was reset to /home/claude
> ````

> [!note]- 🔧 Tool call: Bash
> ```
> cd /tmp/pai/src/pydantic_ai_slim/pydantic_ai && ls models | head -40; echo ---; awk 'NR>=3330 && NR<=3420' agent/__init__.py | grep -n "def tool_decorator\|add_function\|_function_toolset\|return" ; echo ---; grep -n "_instructions\b\|self._instructions" agent/__init__.py | head; echo ---; sed -n 1,60p _instructions.py | head -80; echo ---; sed -n 600,680p toolsets/function.py; echo ---; sed -n 440,520p tools.py | grep -n "class Tool\|function_schema\|tool_def\|ToolDefinition" ; grep -n "async def call\|class FunctionToolset\|def tool_def\|ToolDefinition(" toolsets/function.py | head
> ```

> [!note]- ↩️ Tool result: tool
> ```
> AGENTS.md
> CLAUDE.md
> __init__.py
> _abstract.py
> _anthropic_schema_limits.py
> _batch_codec.py
> _batch_model.py
> _batch_settings.py
> _continuation.py
> _decode_errors.py
> _google_batch.py
> _in_memory_batch.py
> _known_model_names.py
> _openai_batch.py
> _openai_responses_websocket.py
> _prompt_cache.py
> _reasoning_details.py
> _request_timeout.py
> _sdk_retries.py
> _speech.py
> _tool_choice.py
> _transport_errors.py
> anthropic
> batch.py
> bedrock.py
> bedrock_mantle.py
> cerebras.py
> cohere.py
> concurrency.py
> crusoe.py
> decision.py
> fallback.py
> function.py
> github_copilot.py
> google.py
> groq.py
> huggingface.py
> instrumented.py
> mcp_sampling.py
> mistral.py
> ---
> 21:                Its call and return parts are then promoted to the typed parts registered for the kind.
> 23:            timeout: Timeout in seconds for tool execution. If the tool takes longer, a retry prompt is returned to the model.
> 28:            include_return_schema: Whether to include the return schema in the tool definition sent to the model.
> 32:        def tool_decorator(func_: ToolFuncPlain[ToolParams]) -> ToolFuncPlain[ToolParams]:
> 34:            self._function_toolset.add_function(
> 52:                include_return_schema=include_return_schema,
> 54:            return func_
> 56:        return tool_decorator if func is None else tool_decorator(func)
> ---
> 53:    _instructions,
> 82:from .._instructions import AgentInstructions
> 146:from ..toolsets._deferred_capability_loader import capability_load_window, resolve_loaded_capability_instructions
> 151:from ..toolsets._instruction_collection import collect_toolset_instructions
> 464:def _agent_instruction_source(instruction: _instructions.AgentInstruction[Any]) -> _messages.InstructionSource | None:
> 475:def _run_instruction_source(instruction: _instructions.AgentInstruction[Any]) -> _messages.InstructionSource | None:
> 521:    instructions: list[_instructions.AgentInstruction[Any]]
> 591:    _instructions: list[_instructions.SourcedInstruction[AgentDepsT]] = dataclasses.field(repr=False)
> 812:        self._instructions = [
> 813:            _instructions.sourced_instruction(
> ---
> from __future__ import annotations
>
> import warnings
> from collections import Counter
> from collections.abc import Collection, Iterator, Mapping, Sequence
> from dataclasses import KW_ONLY, dataclass, field, replace
> from typing import Generic, TypeGuard
>
> from pydantic_ai._deferred_capabilities import LoadCapabilityReturnPart, LoadExchanges
> from pydantic_ai._run_context import AgentDepsT, RunContext
> from pydantic_ai._utils import dataclasses_no_defaults_repr, is_str_dict
> from pydantic_ai.exceptions import UserError
> from pydantic_ai.messages import (
>     _PYDANTIC_AI_METADATA_KEY,  # pyright: ignore[reportPrivateUsage]
>     _UNSTATED_LOADS_KEY,  # pyright: ignore[reportPrivateUsage]
>     CapabilityInstructionSource,
>     CompactionPart,
>     InstructionBaselinePart,
>     InstructionChangePolicy,
>     InstructionDeltaPart,
>     InstructionDigest,
>     InstructionId,
>     InstructionPart,
>     InstructionSource,
>     ModelMessage,
>     ModelRequest,
>     ModelRequestPart,
>     ModelResponsePart,
>     ModelSettingsDeltaPart,
>     _instruction_text_hash,  # pyright: ignore[reportPrivateUsage]
>     _post_compaction_window_start,  # pyright: ignore[reportPrivateUsage]
> )
> from pydantic_ai.template import TemplateStr
>
> from . import _system_prompt
>
> InstructionFunc = _system_prompt.PromptFunc[AgentDepsT, str | InstructionPart]
> """A function computing an instruction: it may or may not take `RunContext`, may or may not be async,
> and returns text, an [`InstructionPart`][pydantic_ai.messages.InstructionPart], or `None`.
>
> A returned `InstructionPart` declares its [`name`][pydantic_ai.messages.InstructionPart.name] and
> [`on_change`][pydantic_ai.messages.InstructionPart.on_change] policy from inside the function, the way
> a registering decorator would. Whatever it says about `dynamic`, a computed part counts as dynamic."""
>
> AgentInstruction = TemplateStr[AgentDepsT] | str | InstructionPart | InstructionFunc[AgentDepsT]
> """One instruction: literal text, a function computing it, or an `InstructionPart` declaring both the
> text and how it should be treated — its [`name`][pydantic_ai.messages.InstructionPart.name] (qualified
> into an id against whatever source contributes it), its
> [`on_change`][pydantic_ai.messages.InstructionPart.on_change] policy, and whether it counts as
> [`dynamic`][pydantic_ai.messages.InstructionPart.dynamic] for prompt caching. A function can return an
> `InstructionPart` to declare the same from inside it."""
>
> AgentInstructions = AgentInstruction[AgentDepsT] | Sequence[AgentInstruction[AgentDepsT]] | None
>
>
> def validate_instruction_id_segment(id: str, *, kind: str) -> None:
>     """Reject values that cannot be represented unambiguously in an instruction id."""
>     if ':' in id:
>         raise UserError(f'{kind} {id!r} cannot contain a colon because `:` is reserved as an instruction ID delimiter.')
>
> ---
>                 replacements when this block changes. Appending requires an addressable instruction
>                 identity; otherwise it warns and rewrites. The function still runs on every request.
>                 Use `'append'` for instructions that change mid-conversation, to keep the cached
>                 prefix; see [appending instruction changes](../agent.md#appending-instruction-changes).
>         """
>         if name is not None:
>             validate_instruction_name(name)
>
>         def decorator(func_: InstructionFunc[AgentDepsT]) -> InstructionFunc[AgentDepsT]:
>             self._instructions.append(SourcedInstruction(func_, name=name, dynamic=True, on_change=on_change))
>             return func_
>
>         return decorator if func is None else decorator(func)
>
>     def add_function(
>         self,
>         func: ToolFuncEither[AgentDepsT, ToolParams],
>         takes_ctx: bool | None = None,
>         name: str | None = None,
>         description: str | None = None,
>         retries: int | None = None,
>         prepare: ToolPrepareFunc[AgentDepsT] | None = None,
>         args_validator: ArgsValidatorFunc[AgentDepsT, ToolParams] | None = None,
>         docstring_format: DocstringFormat | None = None,
>         require_parameter_descriptions: bool | None = None,
>         schema_generator: type[GenerateJsonSchema] | None = None,
>         strict: bool | None = None,
>         sequential: bool | None = None,
>         requires_approval: bool | None = None,
>         defer_loading: bool | None = None,
>         metadata: dict[str, Any] | None = None,
>         tool_kind: ToolKindLike | None = None,
>         timeout: float | None = None,
>         include_return_schema: bool | None = None,
>     ) -> Tool[AgentDepsT]:
>         """Add a function as a tool to the toolset.
>
>         Can take a sync or async function.
>
>         The docstring is inspected to extract both the tool description and description of each parameter,
>         [learn more](../tools.md#function-tools-and-schema).
>
>         Args:
>             func: The tool function to register.
>             takes_ctx: Whether the function takes a [`RunContext`][pydantic_ai.tools.RunContext] as its first argument. If `None`, this is inferred from the function signature.
>             name: The name of the tool, defaults to the function name.
>             description: The description of the tool, defaults to the function docstring.
>             retries: The number of retries to allow for this tool, defaults to the agent's default retries,
>                 which defaults to 1.
>             prepare: custom method to prepare the tool definition for each step, return `None` to omit this
>                 tool from a given step. This is useful if you want to customise a tool at call time,
>                 or omit it completely from a step. See [`ToolPrepareFunc`][pydantic_ai.tools.ToolPrepareFunc].
>             args_validator: custom method to validate tool arguments after schema validation has passed,
>                 before execution. The validator receives the already-validated and type-converted parameters,
>                 with `RunContext` as the first argument.
>                 Raise [`ModelRetry`][pydantic_ai.exceptions.ModelRetry] to ask the model to correct the
>                 arguments and try again, or [`ToolFailed`][pydantic_ai.exceptions.ToolFailed] to report a
>                 terminal failure the model should adapt to instead of retrying. Return `None` on success.
>                 See [`ArgsValidatorFunc`][pydantic_ai.tools.ArgsValidatorFunc].
>             docstring_format: The format of the docstring, see [`DocstringFormat`][pydantic_ai.tools.DocstringFormat].
>                 If `None`, the default value is determined by the toolset.
>             require_parameter_descriptions: If True, raise an error if a parameter description is missing.
>                 If `None`, the default value is determined by the toolset.
>             schema_generator: The JSON schema generator class to use for this tool.
>                 If `None`, the default value is determined by the toolset.
>             strict: Whether to enforce (vendor-specific) strict schema adherence for tool calls (supported by OpenAI, Anthropic, Google, and Bedrock).
>                 See [`ToolDefinition`][pydantic_ai.tools.ToolDefinition] for more info.
>                 If `None`, the default value is determined by the toolset.
>             sequential: Whether this tool acts as a barrier that runs alone, not overlapping with other tool calls.
>                 See [`ToolDefinition`][pydantic_ai.tools.ToolDefinition] for more info. Defaults to False.
>                 If `None`, the default value is determined by the toolset.
>             requires_approval: Whether this tool requires human-in-the-loop approval. Defaults to False.
>                 See the [tools documentation](../deferred-tools.md#human-in-the-loop-tool-approval) for more info.
>                 If `None`, the default value is determined by the toolset.
>             defer_loading: Whether to hide this tool until it's revealed by tool search, `load_capability`,
>                 or another tool's `ToolReturn.tools`.
>                 See [Tool Search](../tools-advanced.md#tool-search) for more info.
>                 If `None`, the default value is determined by the toolset.
>             metadata: Optional metadata for the tool. This is not sent to the model but can be used for filtering and tool behavior customization.
>                 If `None`, the default value is determined by the toolset. If provided, it will be merged with the toolset's metadata.
>             tool_kind: What the tool is, independent of its name: a [`ToolKind`][pydantic_ai.messages.ToolKind], or the id of one.
> ---
> 22:                See [`ToolDefinition`][pydantic_ai.tools.ToolDefinition] for more info.
> 24:                See [`ToolDefinition`][pydantic_ai.tools.ToolDefinition] for more info. Defaults to False.
> 38:            function_schema: The function schema to use for the tool. If not provided, it will be generated.
> 44:        self.function_schema = function_schema or _function_schema.function_schema(
> 52:        self.takes_ctx = self.function_schema.takes_ctx
> 54:        self.description = description if description is not None else self.function_schema.description
> 42:class FunctionToolsetTool(ToolsetTool[AgentDepsT]):
> 59:class FunctionToolset(AbstractToolset[AgentDepsT]):
> 851:    async def call_tool(
> Shell cwd was reset to /home/claude
> ```

> [!note]- 🔧 Tool call: Bash
> ```
> cd /tmp/pai/src/pydantic_ai_slim/pydantic_ai && git -C /tmp/pai/src rev-parse HEAD; grep -n "^class Tool\b\|^class Tool(\|^class ToolDefinition" tools.py; grep -n "def function_schema\|call=\|json_schema\|validator\|async def call\|def call" _function_schema.py | head -20; grep -n "async def get_tools\|async def call_tool\|ToolDefinition(\|\.call(" toolsets/function.py | head; sed -n 805,825p agent/__init__.py; ls models/anthropic; grep -rn "system=\|instructions" models/anthropic/*.py | head -15; grep -n "ToolParam\|def _map_tool_definition\|tools=" models/anthropic/*.py | head -15; grep -n "_get_instruction_parts" -r models/*.py | head; grep -n "_ToolManager\|class ToolManager\|async def handle_call\|async def _call_tool\|def _validate" _tool_manager.py | head
> ```

> [!note]- ↩️ Tool result: tool
> ```
> 0f7648a1b0685389ea368ea4f7b754b393cea8c7
> 355:class Tool(Generic[ToolAgentDepsT]):
> 700:class ToolDefinition:
> 1:"""Used to build pydantic validators and JSON schemas from functions.
> 20:from pydantic.json_schema import GenerateJsonSchema
> 21:from pydantic.plugin._schema_validator import create_schema_validator
> 29:    check_object_json_schema,
> 51:    validator: SchemaValidator
> 52:    json_schema: ObjectJsonSchema
> 66:        Covers both model-like single args (via `single_arg_name`, which uses a wrap validator
> 76:        properties = self.json_schema.get('properties', {})
> 81:    async def call(self, args_dict: dict[str, Any], ctx: RunContext[Any]) -> Any:
> 110:def function_schema(
> 119:    """Build a Pydantic validator and JSON schema from a tool function.
> 122:        function: The function to build a validator and JSON schema for.
> 181:    schema_validator = create_schema_validator(
> 191:    schema_validator = cast(SchemaValidator, schema_validator)
> 192:    json_schema = schema_generator().generate(schema)
> 198:        single_arg_keys.update(json_schema.get('properties', {}))
> 206:        description = json_schema.pop('description', None)
> 209:    checked_json_schema = check_object_json_schema(json_schema)
> 215:        return_schema: ObjectJsonSchema = TypeAdapter(return_schema_type).json_schema(
> 230:        validator=schema_validator,
> 769:    async def get_tools(self, ctx: RunContext[AgentDepsT]) -> dict[str, ToolsetTool[AgentDepsT]]:
> 851:    async def call_tool(
>         self._deps_type = cast(type[AgentDepsT], deps_type)
>
>         self._output_schema = _output.OutputSchema[OutputDataT].build(output_type)
>         self._output_validators = []
>
>         # The agent's own literal instructions are one addressable part; instruction functions are
>         # only addressable if `@agent.instructions(name=...)` names them.
>         self._instructions = [
>             _instructions.sourced_instruction(
>                 instruction, _agent_instruction_source(instruction), name_source=_messages.AgentInstructionSource()
>             )
>             for instruction in _instructions.normalize_instructions(instructions)
>         ]
>
>         self._system_prompts = (system_prompt,) if isinstance(system_prompt, str) else tuple(system_prompt)
>         self._system_prompt_functions = []
>         self._system_prompt_dynamic_functions = {}
>
>         retry_overrides = _normalize_agent_retry_overrides(retries)
>         resolved_retries = _normalize_agent_retries(retry_overrides)
>         self._max_tool_retries = resolved_retries.tools
> AGENTS.md
> __init__.py
> _batch.py
> _bedrock_count_tokens.py
> _cache_control.py
> _citations.py
> _compaction.py
> _containers.py
> _errors.py
> _message_passes.py
> _native_tools.py
> _response.py
> _streaming.py
> _thinking_binding.py
> models/anthropic/__init__.py:289:        compaction_instructions,
> models/anthropic/__init__.py:377:    'anthropic_cache_instructions',
> models/anthropic/__init__.py:609:    anthropic_cache_instructions: bool | Literal['5m', '1h']
> models/anthropic/__init__.py:613:    allowing Anthropic to cache system instructions and reduce costs.
> models/anthropic/__init__.py:641:    This can be combined with explicit cache breakpoints (`anthropic_cache_instructions`,
> models/anthropic/__init__.py:1092:                settings.get('anthropic_cache_instructions'),
> models/anthropic/__init__.py:1304:        instructions: str | None = None,
> models/anthropic/__init__.py:1322:            instructions: Optional focus for the summary, such as what to keep. Anthropic replaces its whole
> models/anthropic/__init__.py:1323:                summarization prompt with the instructions it's given, so the focus is sent inside a full
> models/anthropic/__init__.py:1324:                summarization prompt rather than on its own. On OpenAI, `instructions` replaces the agent's
> models/anthropic/__init__.py:1325:                instructions for the compaction request instead.
> models/anthropic/__init__.py:1348:        if instructions and (focus := instructions.strip()):
> models/anthropic/__init__.py:1349:            compaction['instructions'] = compaction_instructions(focus)
> models/anthropic/__init__.py:2200:                    system=system_prompt or OMIT,
> models/anthropic/__init__.py:2220:                system=system_prompt or OMIT,
> models/anthropic/__init__.py:239:        BetaToolParam,
> models/anthropic/__init__.py:1146:                supported_native_tools=supported_native_tools,
> models/anthropic/__init__.py:1207:                    anthropic_count_tokens_accepts_server_tools=not isinstance(
> models/anthropic/__init__.py:1461:            supports_strict_tools=lambda: self.profile.get('supports_json_schema_output', False),
> models/anthropic/__init__.py:1462:            sent_tools=lambda settings, params: self._tool_choice_declarations(
> models/anthropic/__init__.py:1811:            tools=tools,
> models/anthropic/__init__.py:2110:            native_tools=[
> models/anthropic/__init__.py:2126:            native_tools=[
> models/anthropic/__init__.py:2205:                    tools=tools or OMIT,
> models/anthropic/__init__.py:2223:                tools=tools or OMIT,
> models/anthropic/__init__.py:2603:                        allowed_tools=tool.allowed_tools,
> models/anthropic/__init__.py:2703:            wire_native_tools=[
> models/anthropic/__init__.py:3861:    def _map_tool_definition(
> models/anthropic/__init__.py:3867:    ) -> BetaToolParam:
> models/anthropic/__init__.py:3868:        """Maps a `ToolDefinition` dataclass to an Anthropic `BetaToolParam` dictionary."""
> models/__init__.py:1992:                    or self._get_instruction_parts(messages, model_request_parameters or ModelRequestParameters())
> models/__init__.py:2208:    def _get_instruction_parts(
> models/cohere.py:457:        if instruction_parts := self._get_instruction_parts(messages, model_request_parameters):
> models/bedrock.py:1701:        if instruction_parts := self._get_instruction_parts(messages, model_request_parameters):
> models/groq.py:664:        if instruction_parts := self._get_instruction_parts(messages, model_request_parameters):
> models/mistral.py:725:        if instruction_parts := self._get_instruction_parts(messages, model_request_parameters):
> models/decision.py:699:        instruction_parts = self._get_instruction_parts(messages, model_request_parameters) or []
> models/google.py:1430:        if instruction_parts := self._get_instruction_parts(messages, model_request_parameters):
> models/huggingface.py:444:        if instruction_parts := self._get_instruction_parts(messages, model_request_parameters):
> models/openai.py:2632:        instruction_parts = self._get_instruction_parts(messages, model_request_parameters) or []
> ugrep: warning: _tool_manager.py: No such file or directory
> Shell cwd was reset to /home/claude
> ```

> [!note]- 🔧 Tool call: Bash
> ```
> cd /tmp/pai/src/pydantic_ai_slim/pydantic_ai && ls | grep -i "tool_manager\|toolsets"; grep -rn "class ToolManager" . | head -2; sed -n 769,790p toolsets/function.py; sed -n 851,870p toolsets/function.py; sed -n 2160,2200p models/anthropic/__init__.py | grep -n "system_prompt"; sed -n 3861,3880p models/anthropic/__init__.py; sed -n 2630,2640p models/openai.py; sed -n 81,92p _function_schema.py
> ```

> [!note]- ↩️ Tool result: tool
> ```
> tool_manager.py
> toolsets
> tool_manager.py:352:class ToolManager(Generic[AgentDepsT]):
>     async def get_tools(self, ctx: RunContext[AgentDepsT]) -> dict[str, ToolsetTool[AgentDepsT]]:
>         tools: dict[str, ToolsetTool[AgentDepsT]] = {}
>         for original_name, tool in self.tools.items():
>             max_retries = tool.max_retries if tool.max_retries is not None else self.max_retries
>             if max_retries is None:
>                 max_retries = ctx.max_retries
>             # `retries` is keyed by the name the model calls the tool by, computed here by applying
>             # the layers `PrefixedToolset` and `RenamedToolset` pushed, innermost first, the way they
>             # name the tools they list. That name is wrong when the tool's own `prepare` renames it,
>             # or when a wrapper other than `PrefixedToolset` and `RenamedToolset` renames tools,
>             # nested inside or around them, whether or not it calls `super().get_tools()`. If it
>             # then names an unrelated tool, `prepare` reads that tool's count.
>             exposed = original_name
>             for layer in reversed(ctx._tool_name_path):  # pyright: ignore[reportPrivateUsage]
>                 exposed = f'{layer}_{exposed}' if isinstance(layer, str) else layer.get(exposed) or exposed
>             run_context = replace(
>                 ctx,
>                 tool_name=original_name,
>                 retry=ctx.retries.get(exposed, 0),
>                 max_retries=max_retries,
>                 _tool_name_path=(),
>                 # A listing nested in a tool call must not inherit that call's tool manager name.
>     async def call_tool(
>         self, name: str, tool_args: dict[str, Any], ctx: RunContext[AgentDepsT], tool: ToolsetTool[AgentDepsT]
>     ) -> Any:
>         assert isinstance(tool, FunctionToolsetTool)
>
>         # Per-tool timeout takes precedence over toolset timeout. Read it off the tool definition, which a
>         # `prepare` function may have changed since `_tool_for` built the tool.
>         timeout = tool.tool_def.timeout if tool.tool_def.timeout is not None else self.timeout
>         if timeout is not None:
>             try:
>                 with anyio.fail_after(timeout), _utils.abandon_threads_on_cancel():
>                     return await tool.call_func(tool_args, ctx)
>             except TimeoutError:
>                 raise ModelRetry(f'Timed out after {timeout} seconds.') from None
>         else:
>             return await tool.call_func(tool_args, ctx)
> 2:        raise_earlier_breakpoint_ttls(system_prompt, anthropic_messages, tools, auto_cache_control)
> 41:                    system=system_prompt or OMIT,
>     def _map_tool_definition(
>         self,
>         f: ToolDefinition,
>         model_settings: AnthropicModelSettings,
>         *,
>         visibility: ToolVisibility,
>     ) -> BetaToolParam:
>         """Maps a `ToolDefinition` dataclass to an Anthropic `BetaToolParam` dictionary."""
>         tool_param: BetaToolParam = {
>             'name': f.name,
>             'description': f.description or '',
>             'input_schema': f.parameters_json_schema,
>         }
>         if f.strict and self.profile.get('supports_json_schema_output', False):
>             tool_param['strict'] = f.strict
>         if model_settings.get('anthropic_eager_input_streaming'):
>             tool_param['eager_input_streaming'] = True
>         if visibility == 'deferred':
>             tool_param['defer_loading'] = True
>         return tool_param
>         system_prompt_role = profile.get('openai_system_prompt_role', None) or 'system'
>         system_prompt_count = _leading_system_message_count(openai_messages, system_prompt_role)
>         instruction_parts = self._get_instruction_parts(messages, model_request_parameters) or []
>         if instruction_parts:
>             if system_prompt_role == 'developer':
>                 instruction_messages: list[chat.ChatCompletionMessageParam] = [
>                     chat.ChatCompletionDeveloperMessageParam(role='developer', content=part.content)
>                     for part in instruction_parts
>                 ]
>             elif system_prompt_role == 'user':
>                 instruction_messages = [
>     async def call(self, args_dict: dict[str, Any], ctx: RunContext[Any]) -> Any:
>         args, kwargs = self._call_args(args_dict, ctx)
>         if self.is_async:
>             function = cast(Callable[[Any], Awaitable[str]], self.function)
>             return await function(*args, **kwargs)
>         else:
>             # A plain `def` may still return an awaitable, which `run_in_executor` would leave un-awaited.
>             function = cast(Callable[[Any], str | Awaitable[str]], self.function)
>             return await await_maybe(await run_in_executor(function, *args, **kwargs))
>
>     def _call_args(
>         self,
> Shell cwd was reset to /home/claude
> ```

調査はほぼ終わっています。最後に、Anthropic 側でシステムプロンプトを組み立てている箇所の行番号を確認します。

> [!note]- 🔧 Tool call: Bash
> ```
> cd /tmp/pai/src/pydantic_ai_slim/pydantic_ai && grep -n "system_prompt" models/anthropic/__init__.py | head -12; grep -n "call_func\|function_schema.call" toolsets/function.py | head -5; grep -n "async def _call_tool\|async def handle_call\|toolset.call_tool\|\.call_tool(" tool_manager.py | head
> ```

> [!note]- ↩️ Tool result: tool
> ```
> 102:    _standing_system_prompt_count,  # pyright: ignore[reportPrivateUsage]
> 903:    system_prompt: str | list[BetaTextBlockParam]
> 1109:        for clients that don't support them (e.g. Bedrock, Vertex). `supports_inline_system_prompts` is
> 1129:        # Narrowed rather than handled in `_map_system_prompt_part` so `Model.prepare_messages` stays the only place
> 1132:        supports_inline_system_prompts = _profile.get('supports_inline_system_prompts', False) and not isinstance(
> 1148:                supports_inline_system_prompts=supports_inline_system_prompts,
> 1151:                supports_request_scoped_system_prompts=_profile.get('supports_request_scoped_system_prompts', False)
> 1152:                and supports_inline_system_prompts,
> 1712:            'system': prepared.system_prompt or OMIT,
> 1763:        system_prompt, anthropic_messages = await self._map_message(messages, model_request_parameters, model_settings)
> 1772:            system_prompt,
> 1778:        raise_earlier_breakpoint_ttls(system_prompt, anthropic_messages, tools, auto_cache_control)
> 45:    call_func: Callable[[dict[str, Any], RunContext[AgentDepsT]], Awaitable[Any]]
> 845:            call_func=tool.function_schema.call,
> 862:                    return await tool.call_func(tool_args, ctx)
> 866:            return await tool.call_func(tool_args, ctx)
> 1326:            tool_result = await self.toolset.call_tool(
> 1348:    async def handle_call(
> Shell cwd was reset to /home/claude
> ```

`pydantic/pydantic-ai` リポジトリのコードを clone して該当箇所を確認しました。確認したのは `main` の `0f7648a`（2026-11-08 時点）です。行番号はこのコミットのもので、更新されるとずれます。パスはすべて `pydantic_ai_slim/pydantic_ai/` 配下です。

## 1. `instructions` がシステムプロンプトになる流れ

| 段階 | 場所 | 内容 |
|---|---|---|
| 受け取り | `agent/__init__.py` 約 812 行（`Agent.__init__`） | `instructions` 引数を `_instructions.normalize_instructions(...)` で正規化し、`self._instructions` に保存します。 |
| 型と正規化 | `_instructions.py` | `AgentInstruction`（str、関数、`InstructionPart` など）の定義と、`normalize_instructions` があります。 |
| 実行時に解決 | `_agent_graph.py` の `_get_instructions`（約 1408 行） | エージェント側の instructions と、ツールセット側の instructions を結合します。 |
| リクエストに格納 | `_agent_graph.py` 約 2942〜2945 行 | `self.request.instructions = InstructionPart.join(instruction_parts)` で、`ModelRequest.instructions` に入れます。 |
| モデル共通の取り出し | `models/__init__.py` の `_get_instruction_parts`（約 2208 行） | 各プロバイダー実装が使う共通ヘルパーです。 |
| プロバイダーごとの変換 | Anthropic: `models/anthropic/__init__.py`（`_map_message` が `system_prompt` を作り、約 1712 行で `'system': ...` として送ります）。OpenAI: `models/openai.py` 約 2632 行 | OpenAI は `system` や `developer` ロールのメッセージにします。Google、Groq、Mistral などは各ファイルで `_get_instruction_parts` を呼びます。 |

つまり、`instructions` はメッセージ履歴の一部として保存されず、各リクエストごとにその時点の値から組み立てられます。この点が `system_prompt` 引数との違いです。

## 2. `tool_plain` に渡した関数が LLM のツールになる流れ

| 段階 | 場所 | 内容 |
|---|---|---|
| デコレータ | `agent/__init__.py` 3270 行付近（`Agent.tool_plain`） | 内部の `tool_decorator` が、`self._function_toolset.add_function(...)` を呼びます。 |
| 登録 | `toolsets/function.py` 614 行（`FunctionToolset.add_function`） | 関数を `Tool` オブジェクトにして登録します。 |
| スキーマ生成 | `tools.py` 355 行の `Tool` クラス。その中で `_function_schema.function_schema(...)` を呼びます。実体は `_function_schema.py` 110 行 | 型ヒントから Pydantic の検証器 `SchemaValidator` と JSON スキーマを作り、docstring から説明文を取り出します。 |
| LLM 向け定義 | `toolsets/function.py` 769 行（`get_tools`） | 各ツールから `ToolDefinition`（名前、説明、`parameters_json_schema`）を作ります。`ToolDefinition` の定義は `tools.py` 700 行です。 |
| プロバイダー形式へ変換 | Anthropic: `models/anthropic/__init__.py` 3861 行（`_map_tool_definition`） | `name`、`description`、`input_schema` を持つ `BetaToolParam` にします。 |
| ツール呼び出しの受付 | `tool_manager.py` 352 行（`ToolManager`）、`handle_call`（1348 行）、1326 行付近の `self.toolset.call_tool(...)` | LLM が返したツール呼び出しを受け、ツールセットに渡します。 |
| 実行 | `toolsets/function.py` 851 行（`call_tool`）→ `tool.call_func` → `_function_schema.py` 81 行（`call`） | 引数を検証したうえで、同期関数は `run_in_executor` で、非同期関数は `await` で実行します。 |

## 読む順番のおすすめ

1. 指示文: `_agent_graph.py` の `_get_instructions`、次に `models/anthropic/__init__.py` の `_map_message`。
2. ツール: `_function_schema.py` の `function_schema` が最も核心です。型ヒントがスキーマに変わり、同じ関数がそのまま呼ばれる仕組みがここにあります。

Sources:
- [pydantic/pydantic-ai (GitHub)](https://github.com/pydantic/pydantic-ai)


---
