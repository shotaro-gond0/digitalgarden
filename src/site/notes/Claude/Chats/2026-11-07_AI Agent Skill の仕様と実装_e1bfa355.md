---
{"dg-publish":true,"permalink":"/claude/chats/2026-11-07-ai-agent-skill-e1bfa355/","title":"AI Agent Skill の仕様と実装","tags":["claude","chat","cat/ml-dl","cat/python","cat/react","cat/js-ts","cat/claude","cat/md","cat/windows","cat/agentic-ai"],"dg-note-properties":{"title":"AI Agent Skill の仕様と実装","date":"2026-11-07","created":"2026-11-07T09:53:19.346617Z","updated":"2026-11-07T10:00:59.170610Z","account":"権藤祥太郎 (shotaro.gondo@gmail.com)","conversation_id":"e1bfa355-c868-4960-9921-b83032400fcf","source":"claude-chat-export","tags":["claude","chat","cat/ml-dl","cat/python","cat/react","cat/js-ts","cat/claude","cat/md","cat/windows","cat/agentic-ai"]}}
---

# AI Agent Skill の仕様と実装

## 🧑 User  
*2026-11-07T09:53:20.167837Z*

LLMの製品やプロバイダによらず一般に、規約または通例として定式化された AI Agent の Skill の情報格納形式、読み出し形式、動作プロセスといったものはありますか？


---

## 🤖 Claude  
*2026-11-07T09:53:46.009209Z*

現状を確認するため、Web検索で最新の標準化状況を調べます。

> [!note]- 🔧 Tool call: WebSearch
> ```json
> {
>   "mode": "standard",
>   "query": "Agent Skills open standard specification SKILL.md agentskills.io"
> }
> ```

> [!note]- 🔧 Tool call: WebSearch
> ```json
> {
>   "mode": "standard",
>   "query": "Agent Skills adopted by OpenAI Codex GitHub Copilot Cursor SKILL.md cross-vendor"
> }
> ```

> [!note]- ↩️ Tool result: tool
> ```
> agentskills.io
> https://agentskills.io/?trk=public_profile__posts-text
>
> what are agent skills and how to use them
> https://strapi.io/blog/what-are-agent-skills-and-how-to-use-them
>
> Agent Skills specification (agentskills%2Fagentskills)
> https://moltchat-agent-commons.onrender.com/wiki/Agent_Skills_specification_(agentskills%2Fagentskills).md
>
> Agent Skills specification (agentskills/agentskills)
> https://moltchat-agent-commons.onrender.com/wiki/Agent_Skills_specification_(agentskills%2Fagentskills)
>
> agentskills.io
> https://agentskills.io/
>
> agentskills.io
> https://agentskills.io/home
>
> agentskills.io
> https://agentskills.io/home?mode=reply
> ```

> [!note]- ↩️ Tool result: tool
> ```
> Rakesh Goyal
> https://velt.dev/blog/what-are-agent-skills
>
> DEV Community
> https://dev.to/nathanielc85523/skillmd-goes-multi-ecosystem-how-the-agent-skills-standard-jumped-from-anthropic-to-openai-and-3oeg
>
> agent skills skill md universal ai coding standard 2026
> https://noqta.tn/en/blog/agent-skills-skill-md-universal-ai-coding-standard-2026
>
> Codex CLI Agent Skills Install Guide
> https://itecsonline.com/post/codex-cli-agent-skills-guide-install-usage-cross-platform-resources-2026
>
> # Agent Skills
> https://thegtmdirectory.com/tools/agent-skills/md
>
> Cross-Agent Skills — Cursor, Codex, Cline, Antigravity, Gemini CLI, Mastra, Vercel: Portability in 2026
> https://mcp.directory/blog/cross-agent-skills-cursor-codex-cline-antigravity-gemini-mastra-portability
>
> Open StandardFree & Open Source
> https://claudeskills.info/agent-skills/
> ```

> [!note]- 🔧 Tool call: WebFetch
> ```json
> {
>   "prompt": "Summarize the full spec: directory structure, SKILL.md frontmatter fields (required/optional, constraints like name length/charset, license, compatibility, metadata, allowed-tools), body content guidance, progressive disclosure token budgets, file reference rules, validation.",
>   "url": "https://agentskills.io/specification"
> }
> ```

> [!note]- ↩️ Tool result: tool (error)
> ```
> {"error_type":"URL_NOT_ON_ALLOWLIST","source":"target","message":"This URL is not on the session's allowlist, so it was not fetched. Do not repeat the same fetch. Find the page with WebSearch and fetch a result link, or ask the user to send the URL."}
> ```

> [!note]- 🔧 Tool call: WebFetch
> ```json
> {
>   "prompt": "Summarize how a client should implement skills support: discovery scan locations, parsing, catalog disclosure, activation, trust/security, context management.",
>   "url": "https://agentskills.io/client-implementation/adding-skills-support"
> }
> ```

> [!note]- ↩️ Tool result: tool (error) _(前回と同一内容のため省略)_

> [!note]- 🔧 Tool call: WebSearch
> ```json
> {
>   "mode": "standard",
>   "query": "agentskills.io specification SKILL.md frontmatter name description license compatibility metadata allowed-tools"
> }
> ```

> [!note]- ↩️ Tool result: tool
> ```
> Agent Skills specification (SKILL.md format, verbatim)
> https://moltchat-agent-commons.onrender.com/wiki/Agent_Skills_specification_(SKILL.md_format%2C_verbatim)
>
> Agent Skills specification (SKILL.md format%2C verbatim)
> https://moltchat-agent-commons.onrender.com/wiki/Agent_Skills_specification_(SKILL.md_format%2C_verbatim).md
>
> agent skills specification
> https://moltchat-agent-commons.onrender.com/api/v1/pages/agent-skills-specification
>
> skill master
> https://smithery.ai/skills/itechmeat/skill-master
>
> agentskills.io
> https://agentskills.io/specification\
>
> > ## Documentation Index
> https://agentskills.io/specification.md
>
> plurigrid asi skill
> https://openskillindex.com/skills/plurigrid-asi-skill
> ```

はい、あります。2025年12月にAnthropicが公開した **Agent Skills** 仕様（agentskills.io）が、事実上の業界共通フォーマットになっています。Anthropicは2025年12月18日にAgent Skillsをオープン標準として公開し、仕様とSDKを公開しました。ただしこれは ISO や IETF のような標準化団体の規格ではなく、特定ベンダー発の仕様をエコシステムが採用している形です。

## 1. 格納形式

スキルの中核は `SKILL.md` を含むフォルダです。構成は次のとおりです。

```
skill-name/
├── SKILL.md      # 必須：YAMLフロントマター + Markdown本文
├── scripts/      # 任意：実行コード
├── references/   # 任意：参考ドキュメント
└── assets/       # 任意：テンプレート・リソース
```

フロントマターの仕様は次のとおりです。

| フィールド | 必須 | 制約 |
|---|---|---|
| `name` | 必須 | 1〜64文字。小文字英数字とハイフンのみで、先頭ハイフンと連続ハイフンは不可。ディレクトリ名と一致させる運用が一般的です |
| `description` | 必須 | 1〜1024文字。何をするか、いつ使うかの両方を書きます |
| `license` | 任意 | ライセンス名、または同梱ライセンスファイル名 |
| `compatibility` | 任意 | 指定する場合は1〜500文字。環境要件を書きます |
| `metadata` | 任意 | 文字列キーと文字列値のマップ（author、version など） |
| `allowed-tools` | 任意 | 事前承認ツールのスペース区切り文字列。仕様上 Experimental です |

## 2. 読み出し形式（段階的開示）

エージェントはスキルを段階的開示（progressive disclosure）で読み込みます。3段階です。

1. **Discovery**：起動時に各スキルの name と description だけを読み込みます。仕様の目安は1スキルあたり約100トークンです。
2. **Activation**：タスクがスキルの description に合致すると、エージェントが SKILL.md 全文をコンテキストに読み込みます。本文は5000トークン未満が推奨です。
3. **Execution**：エージェントは指示に従い、必要に応じて同梱コードを実行するか、参照ファイルを読み込みます。

この方式でコンテキスト消費を抑えつつ、多数のスキルを保持できます。スキルを選ぶ判断は、仕様上は description だけが頼りです。そのため、どの場面で使うかを description に具体的に書くことが重要になります。

## 3. 採用状況と実運用上の差異

- **採用先**：Claude Code、OpenAI Codex CLI、GitHub Copilot、Cursor（2.4以降）、Gemini CLI、Google Antigravity、VS Code などです。2026年4月時点で30以上のAIコーディングツールが採用しているとする記事もありますが、集計の出どころが異なるので数は目安です。
- **配置パスは統一されていません**。Claude Code は `.claude/skills/`、クロスプラットフォーム用には `.agents/skills/` という使い分けが紹介されています。Codex CLI は専用パスではなく `.agents/skills` をプロジェクトパスに採用しており、クロスエージェントの慣例は `.agents/` に収束しつつあるという見方があります。Cursor は `.claude/skills/` と `.codex/skills/` も互換のため走査します。この点は仕様本体ではなく慣例の領域です。
- **仕様外の拡張が各社にあります**。たとえば `allowed-tools` の解釈や、ユーザーによる明示起動（スラッシュコマンド）などです。「SKILL.md を一度書けばどこでも動く」という触れ込みですが、実際はもう少し複雑という指摘もあるので、複数クライアントで試すのが安全です。

## 4. 関連する別の仕様

- **MCP**：スキルとは役割が違います。MCP は外部ツールやデータへの接続プロトコルで、スキルは手順知識と文脈のパッケージです。併用するのが一般的です。
- **AGENTS.md**、**CLAUDE.md**、**.cursorrules**：常時ロードされる指示ファイルです。オンデマンドで読み込むスキルとは別物です。

なお、LangChain/LangGraph、CrewAI、Dify など**フレームワーク側のスキル／ツール定義**には共通の標準は見当たりません。SKILL.md を読むローダーを自前で実装するか、各フレームワークの Tool 定義に変換する形になります。ご自身のIT運用自動化基盤に載せるなら、SKILL.md を正本とし、description を Discovery 用カタログとして使う設計が標準に沿った形です。

Sources:
- [agentskills.io](https://agentskills.io/)
- [agentskills.io 仕様](https://agentskills.io/specification.md)
- [Strapi: What are Agent Skills](https://strapi.io/blog/what-are-agent-skills-and-how-to-use-them)
- [noqta: SKILL.md universal standard](https://noqta.tn/en/blog/agent-skills-skill-md-universal-ai-coding-standard-2026)
- [mcp.directory: Cross-Agent Skills portability](https://mcp.directory/blog/cross-agent-skills-cursor-codex-cline-antigravity-gemini-mastra-portability)


---

## 🧑 User  
*2026-11-07T09:56:26.986592Z*

LangChain/LangGraphをベースとするカスタムSDKでAnthropic の仕様による Skillを使えるように環境を整備したい場合、SDKにはどのような実装が必要ですか？


---

## 🤖 Claude  
*2026-11-07T09:57:01.964154Z*

> [!note]- 🔧 Tool call: WebSearch
> ```json
> {
>   "mode": "standard",
>   "query": "LangChain deepagents skills SKILL.md middleware progressive disclosure LangGraph"
> }
> ```

> [!note]- 🔧 Tool call: WebSearch
> ```json
> {
>   "mode": "standard",
>   "query": "agentskills.io client implementation adding skills support discovery catalog activation trust"
> }
> ```

> [!note]- ↩️ Tool result: tool
> ```
> docs.langchain.com
> https://docs.langchain.com/oss/javascript/deepagents/skills/index.html
>
> deepagents skills
> https://skills.cat/skills/christian-bromann/langchain-skills/deepagents-skills
>
> docs.langchain.com
> https://docs.langchain.com/oss/python/deepagents/skills
>
> > ## Documentation Index
> https://docs.langchain.com/oss/javascript/deepagents/skills.md
>
> docs.langchain.com
> https://docs.langchain.com/oss/python/deepagents/skills/index.html
>
> > ## Documentation Index
> https://docs.langchain.com/oss/python/deepagents/skills.md
>
> Now liveThe Skillselion MCP - thousands of ranked skills, loaded into your agent mid-task. No install.Get it →
> https://skillselion.com/skills/langchain-ai/langchain-skills/deep-agents-core
> ```

> [!note]- ↩️ Tool result: tool
> ```
> github.com
> https://github.com/different-ai/openwork/pull/2965
>
> client implementation
> https://agentskills.io/client-implementation
>
> adding skills support
> https://agentskills.io/client-implementation/adding-skills-support
>
> > ## Documentation Index
> https://agentskills.io/client-implementation/adding-skills-support.md
>
> integrate skills
> https://agentskills.io/integrate-skills
>
> llms full
> https://agentskills.io/llms-full.txt
>
> Adding Agent Skills Support to Your AI Agent: A Practical Guide
> https://www.neura.market/ai-agents/resources/guides/agent-skills-adding-support-to-your-agent
> ```

結論から言うと、Agent Skills は「ファイル形式の仕様」と「クライアント実装ガイド」だけで、実行ランタイムは定義していません。そのため SDK 側が実装するのは、**スキルを見つけて catalog 化し、必要時に本文を読ませるローダー**と、**スクリプトを安全に実行する仕組み**です。

## 先に検討すべきこと：自前実装か deepagents の流用か

LangChain の Deep Agents には、Agent Skills 仕様に従う `SkillsMiddleware` が既にあります。起動時に設定されたスキルパスを走査し、各 SKILL.md のフロントマターを解析して、name と description をシステムプロンプトに注入します。タスクが合致すると、エージェントは `read_file` で SKILL.md を読み込みます。カスタム SDK がこの仕組みに乗れるなら、まず deepagents のミドルウェアを再利用するか、その実装を参考にするのが最短です。独自のステート管理や承認フローがあって乗れない場合は、以下を自前で実装します。

## SDK に必要な実装（agentskills.io のクライアント実装ガイドに準拠）

| # | コンポーネント | 内容 |
|---|---|---|
| 1 | **Discovery（探索）** | 複数スコープを走査します。ガイドでは、プロジェクトの `.<client>/skills/` と `.agents/skills/`、ユーザーの `~/.<client>/skills/` と `~/.agents/skills/` が示されています。クラウドやサンドボックス上で動かす場合はローカル FS を走査できないので、API、リモートレジストリ、同梱アセットから取得します。 |
| 2 | **Parser / Validator** | SKILL.md から YAML フロントマターと本文を分離します。ガイドは寛容な検証を勧めています。name がディレクトリ名と不一致、または64文字超過の場合は、警告を出しつつ読み込みます。YAML が壊れている場合（コロンを含む description など）は、値を引用符で囲んで再試行するフォールバックも推奨されています。 |
| 3 | **Registry** | name をキーとするインメモリ map です。各スキルのレコードには最低3項目（name、description、location）が必要です。本文は探索時に保持しても、活性化時にファイルから読んでも構いません。 |
| 4 | **Catalog 注入（Tier 1）** | name と description（任意で location）をモデルに見せます。1スキルあたり約50〜100トークンで、システムプロンプトに入れる方法と、専用の活性化ツールの description に埋め込む方法があります。併せて「いつスキルを使うか」の短い指示文も添えます。 |
| 5 | **Activation（Tier 2）** | 本文をコンテキストに読み込む手段です。FS を読めるモデルなら既存の read ツールで足ります。専用の `activate_skill` ツールを作る場合は、本文とスキルのディレクトリパスを返し、相対パス解決の基点を与えます。 |
| 6 | **Resource / Script 実行（Tier 3）** | `references/` や `assets/` の読み込みと、`scripts/` の実行です。ファイル読み取りはスキルディレクトリ配下に制限し、スクリプト実行は専用のサンドボックスに隔離します。 |
| 7 | **Trust / 優先順位** | 同名衝突時の決定的な優先順位です。ガイドは project が user を上書きする慣例を示しています。プロジェクト由来のスキルは信頼できないリポジトリ由来の可能性があるため、フォルダが信頼済みと明示された場合のみ読み込むゲートを検討するよう勧められています。衝突時は警告ログを出します。 |
| 8 | **Context 管理** | 活性化したスキル本文が、メッセージのトリミングや要約で失われないようにします。同一スキルの重複活性化も防ぎます。 |

`allowed-tools` は仕様上 Experimental です。実装するなら、承認フロー（interrupt）との対応を自前で決める必要があります。

## LangGraph への載せ方（骨子）

下は私が書いた最小のスケッチで、動作確認はしていません。

```python
from dataclasses import dataclass
from pathlib import Path
import re, yaml
from langchain_core.tools import tool

FM = re.compile(r"\A---\s*\n(.*?)\n---\s*\n?(.*)\Z", re.S)

@dataclass(frozen=True)
class Skill:
    name: str
    description: str
    location: Path          # SKILL.md の絶対パス

def parse_skill(path: Path) -> tuple[Skill, str]:
    m = FM.match(path.read_text(encoding="utf-8"))
    if not m:
        raise ValueError(f"no frontmatter: {path}")
    fm = yaml.safe_load(m.group(1))   # 失敗時は引用符で囲んで再試行するフォールバックを推奨
    if not fm.get("name") or not fm.get("description"):
        raise ValueError(f"name/description required: {path}")
    return Skill(fm["name"], fm["description"], path.resolve()), m.group(2)

def discover(roots: list[Path]) -> dict[str, Skill]:
    reg: dict[str, Skill] = {}
    for root in roots:                 # 優先度の低い順に渡す（後勝ちで project > user）
        for p in root.glob("*/SKILL.md"):
            try:
                s, _ = parse_skill(p)
                reg[s.name] = s
            except Exception as e:
                print(f"skip {p}: {e}")
    return reg

def catalog_prompt(reg: dict[str, Skill]) -> str:
    items = "\n".join(
        f'<skill><name>{s.name}</name><description>{s.description}</description>'
        f'<location>{s.location}</location></skill>' for s in reg.values())
    return ("利用可能なスキル。タスクが description に合致したら "
            "activate_skill を呼び、指示に従うこと。\n"
            f"<available_skills>\n{items}\n</available_skills>")

def make_activate_tool(reg: dict[str, Skill]):
    @tool
    def activate_skill(name: str) -> str:
        """指定スキルの SKILL.md 全文を読み込む。"""
        s = reg.get(name)
        if not s:
            return f"unknown skill: {name}. available: {list(reg)}"
        _, body = parse_skill(s.location)
        return (f'<skill_content name="{s.name}">\n{body}\n\n'
                f"Skill directory: {s.location.parent}\n</skill_content>")
    return activate_skill
```

組み込みの要点は次のとおりです。

- **Catalog 注入**：LangChain 1.x の `AgentMiddleware`（`wrap_model_call` など）でシステムプロンプトに追記するのが素直です。`create_react_agent` 系なら、`prompt` を関数にして毎回 catalog を差し込みます。
- **ツール**：`activate_skill` のほか、スキルディレクトリに制限した `read_skill_file` と、サンドボックス化した `run_skill_script` をツールとして登録します。
- **ステート**：活性化済みスキル名のリストを graph state に持たせると、重複活性化の抑止やトリム時の保護がしやすくなります。チェックポインタ使用時もステートとして永続化されます。
- **ストレージ抽象**：IT運用基盤のように複数ワーカーで動かす場合は、deepagents の `StoreBackend` のように、スキルを FS ではなく LangGraph Store や Git 由来のストレージから読む抽象を挟むと、ワーカー間でスキルが一貫します。

## 設計上の注意

- **セキュリティ**：スクリプトを同梱できる以上、実行は常にサンドボックス内で行い、ネットワークと資格情報を制限します。ServiceNow や AWX の資格情報をスキル経由で触らせる場合は、専用の承認ステップ（HITL interrupt）を必ず挟む設計をお勧めします。
- **検証**：仕様の命名・形式に従っているかは、`skills-ref` 検証ツールで確認できます。CI に組み込んでおくと、スキルの品質を機械的に担保できます。
- **互換性テスト**：Anthropic の公式スキル（skill-creator など）を実際に読ませて動かすと、実装の互換性をすぐ確認できます。

必要なら、deepagents を使う版と自前ミドルウェア版のどちらかについて、動くサンプルまで作ります。

Sources:
- [agentskills.io: How to add skills support to your agent](https://agentskills.io/client-implementation/adding-skills-support)
- [agentskills.io: client implementation](https://agentskills.io/client-implementation)
- [agentskills.io: integrate skills](https://agentskills.io/integrate-skills)
- [agentskills.io: llms-full.txt](https://agentskills.io/llms-full.txt)
- [LangChain Docs: Deep Agents Skills (Python)](https://docs.langchain.com/oss/python/deepagents/skills)
- [LangChain Docs: Deep Agents Skills (Python, index)](https://docs.langchain.com/oss/python/deepagents/skills/index.html)


---

## 🧑 User  
*2026-11-07T10:00:43.435407Z*

skill は、日本語で記述しても十分意図通りに動作しますか？


---

## 🤖 Claude  
*2026-11-07T10:00:59.265459Z*

はい、実用上はほぼ問題なく動きます。ただし、仕様が日本語を保証しているわけではなく、いくつか注意点があります。以下は仕様の読み取りと LLM の一般的な挙動に基づく見解で、日本語スキルの精度を比較した公開ベンチマークは確認していません。

## 問題になりにくい部分

- **本文（SKILL.md の指示）**：日本語で書いて問題ありません。仕様は本文の言語を規定していません。現行の主要モデルは日本語の手順書を十分に理解して実行できます。
- **description**：仕様は1〜1024文字という文字数制限だけで、言語は問いません。モデルが意味的に合致を判断するので、日本語の依頼と日本語の description の組み合わせなら、通常は問題なくマッチします。

## 注意が必要な部分

1. **`name` は ASCII にするのが安全です。** 仕様では name に使えるのは小文字英数字とハイフンのみで、先頭ハイフンと連続ハイフンは不可とされています。同じ箇所に「unicode lowercase alphanumeric」という記述もあり、日本語の文字が許されるかは曖昧です。かなや漢字には大文字小文字の区別がないため、検証ツールやクライアントによって扱いが分かれる可能性があります。ディレクトリ名との一致も求められるので、`incident-triage` のような英語名にし、日本語名は `metadata` などに持たせるのが無難です。
2. **トークン消費が増えます。** 日本語は英語より多くのトークンを使う傾向があります（モデルのトークナイザによって差があります）。仕様の目安は、catalog が1スキルあたり約100トークン、本文が5000トークン未満です。日本語だとこの枠に収まりにくくなるため、description は簡潔にし、長い資料は `references/` に逃がします。
3. **言語をまたぐマッチは精度が落ちる可能性があります。** description が日本語で依頼が英語（またはその逆）の場合、多くは動きますが、境界的なケースで発動を取りこぼすことがあります。対策として、description に主要な英語キーワード（「ServiceNow インシデント一次切り分け（incident triage）」のような併記）を入れておくと安定します。
4. **YAML まわり：**
   - ファイルは UTF-8 で保存します。
   - description に ASCII の `: `（コロンと空白）が含まれると YAML の解析に失敗することがあります。引用符で囲むか、ブロックスカラー（`>`）を使います。
   - 実装ガイドが勧める「引用符で囲んで再試行する」フォールバックは、日本語の description でも有効です。
5. **スクリプトの入出力：** `scripts/` が出力するメッセージやファイルパスに日本語が混じると、環境によって文字化けします（特に Windows）。`PYTHONUTF8=1` などで UTF-8 を明示すると回避できます。

## 実務上のお勧め

- name は英語、description は日本語の要点に英語キーワードを併記、本文は日本語、という構成が最も無難です。
- 利用者が日本語で依頼する運用が中心なら、description を日本語主体にして構いません。
- 発動率はスキルごとに差が出ます。代表的な依頼文を10〜20件用意し、意図したスキルが発動するかを確認する簡易評価を、自前 SDK の CI に入れておくことをお勧めします。モデルを変更したときの回帰確認にも使えます。


---
