---
{"dg-publish":true,"permalink":"/claude/chats/2026-07-24-azure-ai-foundry-dify-a48070f5/","dg-note-properties":{}}
---

---
title: Azure AI Foundry と Dify のアーキテクチャ再構成
date: '2026-07-24'
created: '2026-07-24T01:53:19.058246Z'
updated: '2026-07-24T02:10:14.056329Z'
account: 権藤祥太郎 (shotaro.gondo@gmail.com)
conversation_id: a48070f5-47c6-4fdf-8330-f4de1e37bd3a
source: claude-chat-export
tags:
- claude
- chat
- cat/ml-dl
- cat/python
- cat/react
- cat/math
- cat/devops
- cat/azure
- cat/md
- cat/linux
---
# Azure AI Foundry と Dify のアーキテクチャ再構成

## 🧑 User  
*2026-07-24T01:53:20.137713Z*

https://note.glidelow.com/claude/chats/2026-06-11-azure-ai-foundry-dify-agentic-ai-5b9b2611/

ここで提案されている　Azure AI Foundry, Dify, GitHub のスタックを、以下の条件にあうように再構成してください。なお、Dify とその他の基盤コンポーネントとの接続性の技術的フィージビリティの考慮も含めて構成を作成してください。

* Azure AI Foundry 及び Dify 、及び、それらを直接操作することができる Chainlit または Gradio は、ITインフラの運用タスク、デプロイタスクを実行するための自動化ワークフローを開発する基盤であって、プロダクション運用タスク、プロダクションデプロイタスクを起票発行することはない。
* Chainlit and/or Gradio は、Azure AI Foundry 及び Dify の開発基盤を操作するための開発者用の補助的ユーザーインターフェースであり、通常運用におけるプロダクション運用タスク、プロダクションデプロイタスクのためにエンドユーザーやビジネスユーザーが操作することはない。
* ただし、Chainlit/Gradio から、Azure AI Foundry または Dify 経由で、テスト環境に対してテスト目的で運用タスク、デプロイタスクを発行することはある
* GitHub は、Azure AI Foundry および Dify で開発される運用／デプロイの自動化ワークフローの開発資産を保持・バージョン管理する基盤である。
* GitHub Issues は、Azure AI Foundry および Dify での開発作業のみの Issue進捗管理に使われ、プロダクション運用タスク、プロダクションデプロイタスクには使われない。
* GitHub Actions は、Azure AI Foundry および Dify での開発作業のみのCI/CDに使われ、プロダクション運用タスク、プロダクションデプロイタスクのプロセスやワークフローには使われない。
* Azure AI Foundry で開発された自動化ワークフローの実行エンジンは、Azure Functions または AKS 上にデプロイされる。
* Dify で開発された自動化ワークフローは、Dify の基盤上で実行される。
* Azure AI Foundry 及び Dify で開発された自動化ワークフローは、プロダクション運用タスク、プロダクションデプロイタスクとして実行される場合は、ServiceNow のチケットからの実行命令にトリガーされて稼働する。
* IT運用全体の運用タスク、デプロイタスクは、すべて ServiceNow のチケットを経由して実行される。
* プロダクション運用タスク、プロダクションデプロイタスクは、ServiceNow がここで議論する開発基盤のスコープ外で接続・統合された、別のサービスオーダー用のポータルからServiceNowに対して起票され、そのServiceNowチケットを起点として実行される。
* Ansible Tower/AWX の Playbook は、ServiceNow から実行される場合と、Azure AI Foundry で開発され、Azure Functions または AKS にデプロイされた Pythonスクリプトから実行される場合の両方がある。
* ServiceNow と Azure Functions または AKS にデプロイされた Pythonスクリプト は、双方向のプロセス接続・トリガーが利用できる
* Dify から Ansible Tower/AWX への実行トリガーは、その技術的フィージビリティがある場合は利用される。
* ServiceNow から Dify、Dify から ServiceNow へのプロセス接続・トリガーは、各々、技術的フィージビリティがある場合は利用される。


---

## 🤖 Claude  
*2026-07-24T01:56:58.859898Z*

> [!note]- Thinking
> The user shared a URL I should try to fetch. Let me attempt to retrieve it from note.glidelow.com, and if that doesn't work, I'll rely on my memory of the architecture discussion from June 2026 about integrating Azure AI Foundry, Dify, Chainlit, Ansible Tower, and GitHub Issues. Then I'll reorganize and present the technical architecture.
>
> I should note the response needs to be in Japanese, so let me proceed with the fetch.

> [!note]- 🔧 Tool call: web_fetch
> ```json
> {
>   "url": "https://note.glidelow.com/claude/chats/2026-06-11-azure-ai-foundry-dify-agentic-ai-5b9b2611/"
> }
> ```

> [!note]- Thinking
> I'm restructuring the architecture to separate concerns: the development plane uses Azure AI Foundry, Dify, Chainlit, and Gradio for building workflows, while GitHub handles source control and development CI/CD only. ServiceNow becomes the single production control plane, triggered from an external service order portal, with execution split between Azure Functions/AKS for Foundry workflows and Dify's runtime for Dify workflows, plus Ansible Tower/AWX for orchestration.
>
> Now I'm evaluating the technical feasibility of connecting these systems. Dify can trigger Ansible Tower via HTTP requests to the AWX REST API with Bearer token authentication, though long-running jobs require polling or callbacks since Dify workflows have timeout constraints—and self-hosted Dify in Azure VNet is needed for private network access, since Dify Cloud can't reach private endpoints. ServiceNow can call Dify's Service API directly through Flow Designer REST steps, which is straightforward, though I need to account for Dify's blocking versus streaming modes and their timeout implications.
>
> For the reverse direction, Dify can push data to ServiceNow via HTTP requests to the Table or Import Set APIs using basic auth or OAuth2, but Dify lacks built-in OAuth2 client credentials flow so a token broker or basic auth is more practical. Dify itself doesn't have a generic inbound webhook trigger—the Service API is the only inbound path—so I should recommend a thin API gateway or Azure Functions shim to handle signature verification and webhook payload transformation rather than expecting Dify to listen for arbitrary events directly.
>
> On the implementation side, Dify 1.x has a plugin system and marketplace but no official ServiceNow or AWX connectors, so HTTP Request nodes are the way forward. Credentials should live in Key Vault via a shim rather than embedded, and since Dify's observability is limited, logs need to be pushed externally. GitHub can serve as the repository for Foundry code and Dify workflow definitions—Dify exports apps as DSL YAML that can be version controlled and re-imported via the console API.
>
> Environment separation is critical: dev/test/prod Dify instances, Foundry projects, and Function App slots need to be isolated. The test path can bypass ServiceNow through Chainlit or Gradio triggering Foundry tasks, but this must be hard-gated to test environments only using separate Tower organizations, inventories, credentials, and network segmentation. Production execution requires additional validation controls.
>
> The key safeguard is implementing a ticket-bound execution token that validates against a ServiceNow sys_id before allowing any workflow to run against production. Now I'm thinking through the overall architecture structure with the core planes: Development, Control, Execution, and Target environments.
>
> I'm sketching out a layered diagram showing how development tools feed into ServiceNow as the single source of truth for production requests, which then triggers the execution engines, which in turn orchestrate changes across target infrastructure through Ansible. I need to flesh out the remaining sections: component responsibilities, trigger paths, Dify's technical feasibility in detail, guardrails for prod/test separation, GitHub's role, and implementation notes. Let me start by pulling together the diagram module and then diving deeper into how Dify can actually connect to Ansible Tower—what the constraints and possibilities are.
>
> For the Dify-to-Tower integration specifically, I'm looking at using Dify's HTTP Request node to POST to Tower's job launch endpoint with OAuth2 authentication and extra variables in the body. The feasibility is high, but there's a key constraint: Tower's job launch is asynchronous and returns a 201 with a job ID, so we'd need polling to wait for completion. Dify has loop nodes but weak sleep capabilities, so the better pattern is fire-and-forget with Tower's Notification Webhook handling the result callback instead.
>
> There are also infrastructure considerations: Tower's execution timeout can be tuned via environment variables if self-hosted, network access requires Dify to be in the same VNet as Tower (ruling out Dify Cloud), and credential rotation is tricky with environment variables—short-lived tokens would be safer through a Functions layer. On the Tower side, the Job Template needs to have `ask_variables` enabled to accept the extra_vars parameter.
>
> For the reverse direction (ServiceNow calling Dify), I'd use Dify's workflow API with an App API Key, while ServiceNow handles it through Flow Designer or IntegrationHub. The main constraint is that blocking responses have a timeout—around 100 seconds on cloud, dependent on gunicorn settings for self-hosted—so streaming mode might be necessary.
>
> Dify doesn't have a generic webhook receiver, only the Service API endpoint, which means I'd need to add validation, idempotency keys, and retry logic myself. The practical solution is to put an API gateway or Azure Functions in between as a thin shim to handle these controls. For closed-network setups using ServiceNow's MID Server, I can route REST calls through that as well.
>
> Now looking at the outbound direction, where Dify calls ServiceNow using HTTP Request nodes with the Table API... The key constraint is that Dify can only connect to ServiceNow for returning results and triggering follow-up actions, not for initiating tickets. For the GitHub to Dify deployment pipeline, I'm considering that Dify apps can be exported and imported as YAML DSL files through the Console API, though this isn't officially stable across versions. The practical approach would be storing the DSL in a GitHub repository for PR review before promoting changes through dev, test, and production environments.
>
> For the Foundry side, I'm looking at Azure AI Foundry where Prompt Flow and Agent Service run on Azure Functions or AKS with Python as the execution engine, then using GitHub Actions to build container images and deploy them.
>
> Now I'm thinking through the ServiceNow integration — the Agent uses Tool Calling to reach Tower and ServiceNow, but production always originates from ServiceNow itself, so the Functions engine gets triggered either via HTTP calls from ServiceNow or through Service Bus/Event Grid. The bidirectional communication uses Flow Designer REST steps or IntegrationHub Spoke on the ServiceNow side (with Private Endpoint and APIM protecting Functions), while Functions calls back to ServiceNow through Table API or Scripted REST API to update work notes and handle state transitions.
>
> For long-running operations, Durable Functions with orchestration and external event waiting is ideal, or KEDA with queues if running on AKS. On the guardrails side, I'm separating Tower organizations into `ops-prod` and `ops-test` with completely isolated Inventory, Credentials, and Job Templates — the dev tools (Chainlit, Dify, Foundry) only connect to the test org. The execution engine requires an `execution_context` parameter, and when `env=prod`, it mandates the ServiceNow ticket ID and validates the callback back to ServiceNow. CI/CD パイプラインで Functions、AKS、Dify に反映させる。GitHub Issues は開発作業に限定し、ServiceNow チケットとの自動連携は避けるが、本番の不具合を backlog に取り込む際は手動で Issue を作成する。アーキテクチャの可視化のため図を描くことにしよう。

> [!note]- 🔧 Tool call: visualize:read_me
> ```json
> {
>   "modules": [
>     "diagram"
>   ],
>   "platform": "desktop"
> }
> ```

> [!note]- ↩️ Tool result: visualize:read_me (error)
> ```
> Tool call failed: 400
> ```

> [!note]- Thinking
> The visualizer encountered an issue, so I'm trying again.

> [!note]- 🔧 Tool call: visualize:read_me
> ```json
> {
>   "modules": [
>     "diagram"
>   ]
> }
> ```

> [!note]- ↩️ Tool result: visualize:read_me (error) _(前回と同一内容のため省略)_

> [!note]- Thinking
> I'll structure the response in Japanese with a clear breakdown: starting with the core principles of the four-plane separation approach, then a text-based architecture diagram, followed by a component responsibility matrix and the trigger paths.
>
> Building the architecture diagram... Continuing the diagram to show how the execution engine and Dify runtime connect back to ServiceNow with bidirectional feedback loops, where results and progress updates flow back to the control plane depending on the Foundry setup.

元記事の構成は「GitHub Issues と ServiceNow を本番オペレーションの動線に組み込む」設計になっていましたが、ご提示の条件では**開発基盤と本番統制の動線を完全に切り離す**必要があります。その前提で再構成しました。

---

## 1. 再構成の基本方針 — 4プレーン分離

| プレーン | 構成要素 | 責務 | 本番タスクの起票／発行 |
|---|---|---|---|
| **開発プレーン** | Chainlit/Gradio、Azure AI Foundry、Dify(開発WS)、GitHub | 自動化ワークフローの開発・テスト・資産管理 | **しない** |
| **統制プレーン** | ServiceNow（+ スコープ外の発注ポータル） | 本番運用/デプロイタスクの唯一の起点・承認・状態管理・監査 | 起点 |
| **実行プレーン** | Azure Functions / AKS、Dify Runtime | ワークフローの実行 | 統制プレーンからのみ起動 |
| **作用プレーン** | Ansible Tower/AWX → 各対象環境 | Playbook 実行 | prod/test を Org 単位で分離 |

元記事との最大の差分は、**GitHub Actions を本番実行のトリガー経路から完全に外す**点です。GitHub は「開発資産の SoT（Source of Truth）」に純化し、本番の SoT は ServiceNow チケットになります。

---

## 2. 全体構成

### 開発プレーン（本番動線から隔離）

```
  開発者
    │ 操作（開発・デバッグ・テスト実行のみ）
    ▼
┌─────────────────────────────────────────────────────────┐
│ Chainlit / Gradio  … 開発者専用の補助UI                   │
│   ※ エンドユーザー／ビジネスユーザーは使用しない            │
└──────┬──────────────────────────┬───────────────────────┘
       ▼                          ▼
┌──────────────────┐      ┌──────────────────┐
│ Azure AI Foundry │      │ Dify (開発WS)     │
│ Agent / PromptFlow│      │ Workflow 開発     │
└──────┬───────────┘      └──────┬───────────┘
       │ コード/エージェント定義      │ DSL(YAML) export
       ▼                          ▼
┌─────────────────────────────────────────────────────────┐
│ GitHub                                                  │
│  Repo    : 開発資産のバージョン管理                        │
│  Issues  : 開発作業のみの Epic/Story/Task 進捗管理         │
│  Actions : 開発CI/CD（lint / test / eval / 各環境へ配備）  │
└─────────────────────────────────────────────────────────┘
       │ 配備（Actions = CD の終点はここまで）
       ▼
  実行エンジン（Functions/AKS）・Dify 実行WS へアーティファクト反映
```

### 統制・実行プレーン（本番動線）

```
【スコープ外】サービスオーダー用ポータル
        │ 起票
        ▼
╔═══════════════════════════════════════════════════════════╗
║ ServiceNow  … 本番運用/デプロイタスクの唯一の起点            ║
║   承認 → 実行命令 → 状態遷移 → クローズ / 監査証跡          ║
╚══╤═══════════════════╤══════════════════════╤═════════════╝
   │ ①直接実行          │ ②実行命令（双方向）    │ ③実行命令（FS次第・双方向）
   │                   │   ▲ 結果/進捗          │   ▲ 結果/進捗
   ▼                   ▼   │                   ▼   │
   │      ┌────────────────────────┐   ┌────────────────────┐
   │      │ Azure Functions / AKS   │   │ Dify Runtime        │
   │      │ Foundry製ワークフロー     │   │ Dify製ワークフロー   │
   │      │ の実行エンジン(Python)    │   │ （Dify基盤上で実行）  │
   │      └───────────┬────────────┘   └─────────┬──────────┘
   │                  │ ⑤                        │ ④（FS次第）
   ▼                  ▼                          ▼
┌──────────────────────────────────────────────────────────┐
│ Ansible Tower / AWX   ── Organization: ops-prod           │
└─────────────────────────┬────────────────────────────────┘
                          ▼
            本番環境（Azure / オンプレ / 他クラウド）
```

### テスト実行経路（開発プレーンから許可される唯一の実行）

```
Chainlit / Gradio
   └→ Azure AI Foundry / Dify(開発WS)
        └→ 実行エンジン(dev/test インスタンス) / Dify(test)
             └→ AWX Organization: ops-test  →  テスト環境のみ
   ※ ServiceNow を経由しない。ops-prod へは到達不能（NW+RBAC で遮断）
```

---

## 3. トリガー経路の定義

| # | 起点 | 経路 | 対象 | 実現方式 |
|---|---|---|---|---|
| ① | ServiceNow | → AWX | 本番 | IntegrationHub Ansible Spoke / REST(MID Server) |
| ② | ServiceNow ⇄ Functions/AKS | 双方向 | 本番 | SN→: Flow Designer REST step + APIM<br>→SN: Table API / Scripted REST（work notes・state 更新） |
| ③ | ServiceNow ⇄ Dify | 双方向 | 本番 | SN→: Dify Service API `POST /v1/workflows/run`<br>→SN: Dify HTTP Request ノード（後述の制約あり） |
| ④ | Dify → AWX | 単方向 | 本番/テスト | HTTP Request ノード → `/api/v2/job_templates/{id}/launch/` |
| ⑤ | Functions/AKS → AWX | 単方向 | 本番/テスト | awxkit / requests（Durable Functions 推奨） |
| ⑥ | Chainlit/Gradio → Foundry/Dify | 単方向 | **テストのみ** | 実行コンテキストに `env=test` を強制 |
| ⑦ | GitHub Actions → Functions/AKS/Dify | 単方向 | 配備のみ | OIDC + Workload Identity Federation |

---

## 4. Dify の接続性 — 技術的フィージビリティ評価

ご要望の「フィージビリティ次第」の部分を個別に評価します。**結論として、Dify は self-hosted（AKS または Container Apps、AWX/ServiceNow と同一 VNet）を前提にしてください。** Dify Cloud では閉域到達性が確保できず、④③のいずれも成立しません。

### ④ Dify → Ansible Tower/AWX ： ◎ 実現可能

- **方式**: Workflow の HTTP Request ノードから `POST {AWX}/api/v2/job_templates/{id}/launch/`、`Authorization: Bearer <AWX OAuth2 token>`、body に `extra_vars`。
- **制約と対処**:
  - AWX の launch は **非同期**で 201 + job id を返すのみ。Dify のワークフローには実用的な sleep/待機ノードが無いため、ポーリングでの完了待ちは破綻しやすい → **fire-and-forget とし、完了通知は AWX の Notification Template（Webhook）から別経路（Functions → ServiceNow）で返す**設計が安定。
  - Dify のワークフロー実行時間上限（self-hosted は環境変数で調整可）に長時間 Playbook を合わせない。
  - `extra_vars` を渡すには Job Template 側で *Prompt on launch* を有効化する必要がある。任意変数の注入リスクがあるため、**Survey でホワイトリスト化**し、Dify 側では変数を自由記述させない。

### ③-a ServiceNow → Dify ： ○ 実現可能（シム推奨）

- **方式**: Dify Service API（App API Key、`POST /v1/workflows/run`）を ServiceNow の Flow Designer REST step または RESTMessageV2 から呼ぶ。
- **制約と対処**:
  - `response_mode: blocking` は同期タイムアウトの影響を受ける → **`streaming` か、即時 202 応答＋実行 ID 返却の非同期パターン**にする。ServiceNow 側で同期待ちしない。
  - Dify には**汎用 Webhook 受信エンドポイントが無く**、受け口は Service API のみ。署名検証・冪等キー・リトライ制御・レート制限の機構を持たない → **Azure API Management または Azure Functions を薄いシムとして前段に挟む**（元記事の「Dify はインバウンドが弱い」という指摘は、この統制機能の欠如として正しい）。
  - 閉域要件がある場合は ServiceNow MID Server 経由で APIM の Private Endpoint に到達させる。

### ③-b Dify → ServiceNow ： ○ 実現可能（用途を限定）

- **方式**: HTTP Request ノードから Table API `POST/PATCH /api/now/table/{table}` もしくは Scripted REST API。
- **制約と対処**:
  - Dify の HTTP ノードには **OAuth2 client credentials のトークン自動更新機構が無い**。Basic 認証は避けたいので、トークン取得ノードを前段に置くか、シム経由で ServiceNow を呼ぶ方が運用が堅牢。
  - **用途は「実行結果・進捗の既存チケットへの書き戻し（work notes / state 更新）」に限定**してください。新規チケットの起票はスコープ外ポータルの責務であり、Dify が起票主体になると統制モデルが崩れます。

### ⑦ GitHub → Dify のCD ： △ 部分的（半自動を許容）

- Dify アプリは **DSL(YAML) でエクスポート／インポート可能**なので、YAML をリポジトリの SoT にして PR レビューする運用は問題なく成立します。
- ただし import/export は **Console API（コンソール認証トークンが必要、公開安定 API ではない）**であり、バージョン間で仕様変更が起こり得ます。CI で完全自動化するとバージョンアップ時に壊れます。
- **推奨**: `dify/*.yaml` の差分レビューを GitHub Actions で必須化（自動）＋ 実行 WS への昇格はスクリプト実行を承認付き `workflow_dispatch` に(半自動)。Actions ランナーは閉域到達のため self-hosted。

### まとめ

| 接続 | 判定 | 主な前提 |
|---|---|---|
| Dify → AWX | ◎ | self-hosted Dify、非同期前提、Survey でパラメータ制約 |
| ServiceNow → Dify | ○ | APIM/Functions シム、非同期応答 |
| Dify → ServiceNow | ○ | 結果書き戻し用途に限定、トークン管理を工夫 |
| GitHub → Dify (CD) | △ | DSL は SoT 化可、反映は半自動を許容 |
| Dify の汎用 Webhook 受信 | ✕ | 存在しない。必ずシムを挟む |

---

## 5. 「本番／テスト」を構造的に強制するガードレール

条件の中で最も破られやすいのが「開発プレーンから本番タスクを発行しない」です。ポリシー文書ではなく**構造で不能にする**ことを推奨します。

1. **AWX の Organization 完全分離**  
   `ops-prod` / `ops-test` で Inventory・Credential・Job Template・Team を分離。Chainlit/Foundry(dev)/Dify(dev) に払い出すトークンは `ops-test` のみに紐づく。
2. **ticket-bound execution token**  
   実行エンジン（Functions/AKS）と Dify ワークフローの入力契約に `env` と `servicenow_ticket_sys_id` を必須化。`env=prod` の場合、**実行前に ServiceNow へ逆引きし、チケットが実在し承認済み state であることを検証**してから AWX を起動する。この検証を通らない限り prod Job Template は叩けない。
3. **ネットワーク分離**  
   `ops-prod` の AWX エンドポイントは開発 VNet / 開発用 Dify から到達不能にする。
4. **相関ID**  
   全実行に ServiceNow チケット番号（テスト時は run id）を correlation ID として貫通させ、AWX の `extra_vars` と job description に必ず埋め込む。

---

## 6. GitHub の位置づけ

```
repo/
  foundry/            Agent 定義 / Prompt Flow / 評価データセット
  engine/             Azure Functions / AKS 用 Python 実行エンジン
  dify/               Dify アプリの DSL(YAML)、プロンプト、テストケース
  contracts/          実行要求スキーマ(JSON Schema)、ServiceNow ペイロード契約
  ansible/            Playbook / Role（別リポジトリ分離も可）
  .github/workflows/  CI: lint/unit test/プロンプト評価
                      CD: Functions・AKS への配備、Dify DSL 反映
```

- **Issues**: 開発作業の Epic/Story/Task のみ。ServiceNow チケットとの自動連携は**張らない**。本番障害からの改善要求は、人手で Issue を起票する（意図的に手動の断絶を設ける）。
- **Actions**: 開発CI/CD専用。`repository_dispatch` による ServiceNow 連携は元記事から**削除**。
- **contracts/ が要**: 実行エンジンと Dify が受ける入力スキーマを一元定義しておくと、ServiceNow 側の Flow 実装と開発側の実装が独立して進められます。

---

## 7. 実装上の注意点

- **長時間実行**: Functions を使うなら **Durable Functions のオーケストレーター＋外部イベント待機**が最適解です（AWX 完了 Webhook を external event として受ける）。AKS なら KEDA + キューで同等の構成にします。
- **冪等性**: ServiceNow のチケット sys_id を冪等キーにし、実行エンジン側で「同一チケットの二重実行」を弾く。ServiceNow のリトライで Playbook が二度走る事故が最も起きやすい箇所です。
- **シークレット**: AWX / ServiceNow の資格情報は Key Vault に集約し、Functions/AKS は Managed Identity で取得。Dify は環境変数に置かざるを得ないため、**Dify に持たせる権限は最小（test org、または結果書き戻しのみ）**に寄せると全体のリスクが下がります。
- **可観測性**: Dify の実行ログは基盤内に閉じるため、重要な実行は Functions 経由にするか、Dify から Application Insights へログを送出する処理をワークフロー末尾に組み込んでください。

---

再構成の要点は、**「GitHub Actions を本番トリガーから外し、その役割を ServiceNow に集約したこと」** と **「Dify を self-hosted 前提にして、インバウンドは APIM/Functions のシム越し、アウトバウンドは非同期 fire-and-forget に統一したこと」** の2点です。どちらかについて、シーケンス図レベルまで具体化することもできます。


---

## 🧑 User  
*2026-07-24T01:58:35.687584Z*

Dify は社内サービスとして提供されているもので、社内ネットワーク閉域内に構築されたものか Dify Cloud を活用したものかの確認ができていません。閉域内に構築されたものだとすると、上記のフィージビリティは変わりますか？


---

## 🤖 Claude  
*2026-07-24T02:00:44.008982Z*

> [!note]- Thinking
> The distinction matters here—if Dify is a shared internal service that the team doesn't control, then network reachability isn't the constraint anymore. The real issue becomes governance: they can't modify environment variables or control the workflow configuration themselves, which fundamentally limits what they can do with the integration regardless of where it's hosted.
>
> And there's another critical layer: if ServiceNow is SaaS (which it typically is), a closed-network Dify likely can't reach it outbound without proxy allowlisting, and ServiceNow can't reach back into the closed network without a MID Server or exposed inbound path—neither of which a shared internal service would typically expose. So I need to reframe this as a checklist of questions to ask the Dify service owner about administrative access, network segmentation, and egress policies.
>
> For case A (closed network shared service), the Dify→AWX connection improves in reachability if both are in the same network, but internal segmentation and firewall rules between segments still matter. The HTTP node in Dify may be forced through a forward proxy, and the tenant can't control proxy configuration at the container level. Plus, many shared Dify deployments restrict outbound traffic to LLM endpoints only for SSRF protection—and Dify's default docker-compose actually includes an ssrf_proxy component that routes HTTP Request node traffic through it.
>
> For ServiceNow→Dify, the key is that ServiceNow is typically SaaS (external), so inbound to the closed network requires a MID Server. The MID Server sits inside the closed network and polls outbound, which means it can call internal REST endpoints. ServiceNow Flow Designer can execute REST steps "via MID Server," so the MID Server itself makes the HTTP call to Dify—this makes inbound feasible without exposing Dify externally, assuming a MID Server with network access to Dify exists. A reverse proxy or API gateway in the DMZ is an alternative approach.
>
> For the reverse direction, Dify→ServiceNow is much harder in a closed network. A shared internal Dify likely won't have a published external endpoint, and while it typically issues Service API keys per app for auth, the Service API layer might be disabled entirely in some internal deployments. The real blocker is egress: if the internal Dify has no internet access at all (common in closed builds), reaching external SaaS ServiceNow requires going through a proxy with allowlist rules and possibly the ssrf_proxy config. Without internet egress, Dify→ServiceNow becomes infeasible unless there's a relay in place.
>
> If the enterprise has ExpressRoute or VPN connectivity between the closed network and Azure, the relay pattern could route through an internal component or via Azure APIM/Functions. But this makes the relay mandatory rather than optional—it's no longer just a nice-to-have fallback.
>
> For GitHub→Dify CD, the challenge shifts: Console API access now requires admin or console-level credentials, which is significantly harder to provision in a closed environment.
>
> GitHub Actions runners are SaaS-hosted and can't reach the closed network directly, so a self-hosted runner inside the closed network becomes necessary. Plus, any egress from the closed network to GitHub.com needs to go through a proxy. The practical fallback is storing the DSL YAML in GitHub as the source of truth and doing manual imports via the UI—version control is preserved but automation is lost, which is acceptable.
>
> For a shared internal service, there's also the question of workspace isolation: can separate workspaces be created for dev, test, and production execution, or is there only one workspace? If it's just one, the separation between development and production execution environments becomes much harder to enforce.
>
> The biggest architectural risk is whether multiple workspaces with separate API keys and credential sets are even obtainable. If not, then dev and prod workflows coexist in the same space, and the ticket-bound token guardrail becomes the only real separation—at that point, moving prod execution to Functions or AKS might be necessary instead. There's also the operator upgrade problem: when Dify gets upgraded, DSL compatibility could break, so a change notification SLA is critical. And since the tenant can't control resource limits or timeout settings like `WORKFLOW_MAX_EXECUTION_TIME` or worker counts, fire-and-forget execution becomes mandatory rather than optional.
>
> Marketplace access is likely blocked in a closed network, so only built-in nodes are available—the HTTP Request node becomes the only integration path. Secrets are stored as workspace environment variables visible to all workspace members, with no Key Vault integration, so I'd minimize what privileges Dify gets (maybe just an AWX test org or a scoped token). The shared service probably doesn't have production-grade SLA, so if prod ops tasks depend on it, that's a real risk—better to route prod-critical paths through Functions or AKS and use Dify for lower-criticality or human-in-the-loop workflows. I also need to check whether the operator even provides execution logs back to tenants for audit purposes.
>
> My recommendation is to narrow Dify's role significantly: keep it for development, PoCs, and non-critical or test workflows, while making Functions or AKS the production execution engine. Any direct ServiceNow-to-Dify connection should be replaced with a relay like APIM or Functions. There's also an important distinction if this is Dify Cloud instead of self-hosted—Cloud can't reach internal AWX unless there's a published endpoint or reverse proxy, which flips the architecture: inbound from ServiceNow becomes easy, but outbound to internal infrastructure becomes impossible.
>
> Now I'm building a comparison table across the three deployment scenarios and laying out the specific technical constraints that emerge when Dify is a shared closed-network service run by another team, along with a checklist of questions to confirm with the service provider.
>
> I need to be precise about implementation details like the SSRF proxy configuration, MID Server integration, and workspace isolation — these are the concrete points that matter for feasibility. I should note that my knowledge reflects Dify's architecture as of early 2026, and given how quickly the platform evolves, some specifics around the console API and environment variables may have shifted since then.

> [!note]- 🔧 Tool call: web_search
> ```json
> {
>   "query": "Dify ssrf_proxy HTTP request node self-hosted 制限"
> }
> ```

> [!note]- Thinking
> This is a critical constraint I need to emphasize: Dify's SSRF proxy blocks requests to private IP ranges by default, which means even in a closed-network setup where Dify runs internally, it would still prevent direct communication with services like AWX on private IPs unless the operator explicitly configures NO_PROXY or updates the squid allowlist.
>
> The 120-second timeout on the SSRF proxy also reinforces that this approach is really only viable for fire-and-forget workflows.
>
> I should also check whether Dify now has a public API for app DSL import/export — that would be important for CI/CD integration.

> [!note]- 🔧 Tool call: web_search
> ```json
> {
>   "query": "Dify app DSL export import API console API CI/CD version control 2026"
> }
> ```

> [!note]- Thinking
> So the DSL import/export API remains internal to the console, and the community has open requests for a public version—that keeps CI/CD integration in the uncertain category. The real answer hinges on what my team can actually change within tenant permissions, not whether we're on a closed network or in the cloud. The critical discovery is that Dify's default SSRF proxy blocks private IP ranges, which means being inside a closed network alone doesn't automatically solve the problem.
>
> Now I need to structure the full answer around three shifting evaluation axes, then walk through how Dify Cloud versus a self-hosted closed-network setup each hit different constraints. The self-hosted path has its own pitfalls—the SSRF proxy, the 120-second timeout limit, plugin marketplace restrictions, and proxy configuration quirks. But the biggest constraint is actually that this is a shared service run by another department, which creates governance and change-request friction. I'll build a decision table for both scenarios, then list the specific questions to ask the service team and the no-regret design moves we can make right now without waiting for answers.

はい、変わります。ただし「閉域だから内部リソースに届く」という単純な改善方向ではなく、**判定軸そのものが移ります。**

---

## 1. 結論

| | Dify Cloud の場合 | 閉域内に構築されている場合 |
|---|---|---|
| Dify → AWX（④） | ほぼ不可 | **到達性は改善するが、Dify の SSRF 対策で既定ブロックの可能性あり** |
| ServiceNow → Dify（③-a） | 容易 | **MID Server 経由が必須級に** |
| Dify → ServiceNow（③-b）※SN が SaaS の場合 | 容易 | **プロキシ許可が必要。閉域完全遮断なら不可** |
| GitHub → Dify の CD（⑦） | 困難 | **さらに困難**（self-hosted runner 必須＋権限） |

そして最も重要なのは、**「閉域か否か」より「社内共有サービスであること（＝自チームに管理権限がないこと）」の方が制約として大きい**という点です。判定軸は以下の3つに移ります。

1. ネットワーク到達性（閉域なら改善）
2. **Dify 基盤側の設定変更権限**（共有サービスなら自チームにはない）
3. **テナント分離の粒度**（dev / test / 本番実行を分けられるか）

---

## 2. 閉域構築で新たに効いてくる技術的な落とし穴

### ⑴ Dify の SSRF 対策が内部 IP を既定でブロックする ← 最重要

Dify の公式仕様として、HTTP ノードや画像ダウンロードを含む Dify からの外向き HTTP リクエストはすべてプロキシ経由でルーティングされ、SSRF 攻撃を防ぐためにプライベート IP レンジ宛のリクエストがブロックされます。

つまり **Dify が閉域内にあっても、HTTP Request ノードから AWX（プライベート IP）を叩けるとは限りません。** 既定構成では逆に弾かれます。回避には基盤側で以下のいずれかが必要です。

- `docker-compose.yaml` の `api` / `worker` サービスに `NO_PROXY` を追加して AWX の宛先を除外する
- ssrf_proxy（squid）の ACL に AWX を許可登録する

いずれも**基盤運用者側の作業**であり、テナントである開発チームでは変更できません。これは前回の回答で「self-hosted なら到達可能」と書いた部分の重要な補正です。

### ⑵ ssrf_proxy のタイムアウト

HTTP Request ノードの通信は既定で内部の ssrf_proxy コンテナを経由し、そのタイムアウトは 120 秒程度と短く、調整用のパラメータも用意されていないという報告があります。回避策は前項同様 `NO_PROXY` 除外です。

→ **前回「推奨」とした fire-and-forget（Playbook 完了を Dify で待たない）設計は、閉域共有 Dify では事実上「必須」になります。**

### ⑶ プラグイン／Marketplace が使えない

閉域では `marketplace.dify.ai` に到達できないため、プラグインの追加インストールは（基盤側でプロキシ設定がされていない限り）不可です。結果として、**外部連携手段は組み込みの HTTP Request ノードのみ**になります。前回提案した「HTTP ノード中心の構成」はこの点でも妥当ですが、選択肢が無いことを前提に置いてください。

### ⑷ ServiceNow が SaaS なら、方向が逆転する

- **ServiceNow → Dify**：SaaS から閉域内への直接インバウンドは不可。**MID Server（閉域内に設置され、内部 REST を呼べる）経由**が現実解です。Flow Designer の REST step を「MID Server 経由で実行」に設定すれば、Dify を外部公開せずに Service API を叩けます。**MID Server が既にあるか、Dify セグメントに到達できるかの確認が必須。**
- **Dify → ServiceNow**：閉域から SaaS への外向き通信になるため、社内フォワードプロキシの許可＋ Dify 側のプロキシ設定が必要。閉域が完全遮断（インターネット egress なし）なら **不可**であり、この場合は Azure Functions / AKS 側を中継にせざるを得ません。

---

## 3. 「社内共有サービス」であることの制約（技術以前の問題）

閉域構築か Cloud かに関わらず、他部門が運用する共有 Dify である場合、以下が構成の成否を分けます。

| 論点 | 影響 |
|---|---|
| **ワークスペース／テナントを dev・test・本番実行用に分けられるか** | 分けられない場合、前回設計の「Dify(開発WS)」と「Dify Runtime(本番)」の分離が成立しない。→ **本番実行を Dify から Functions/AKS へ寄せる判断が必要** |
| Service API（App API Key）の払い出しが許可されているか | 不可なら ServiceNow → Dify は成立せず |
| `NO_PROXY` / ssrf_proxy ACL の個別許可申請ができるか | 不可なら Dify → AWX は成立せず |
| バージョンアップのタイミングと事前通知 SLA | 運用者都合の upgrade で DSL 互換性が壊れるリスク |
| 実行ログ・監査ログをテナントが取得できるか | 本番運用タスクの監査証跡要件を満たせるか |
| 可用性 SLA | 共有 PoC 基盤レベルなら、本番運用タスクの実行基盤に据えるのは危険 |
| シークレット管理 | ワークスペースの環境変数に平文で置く形になり、Key Vault 連携が無い。**Dify に渡す権限は最小化必須** |

### CI/CD についての補足

DSL のインポートは現状フロントエンド UI からのみ可能で、ヘッドレス／自動化用途のための公式なインポート API の追加は機能要望として起票されている段階です。実質的には Console API（`/console/api/login` でアクセストークンを取得して `/console/api/apps` を叩く）を使うしかなく、共有サービスでこの認証情報を得られる可能性は低いと考えるべきです。

また エクスポートされる DSL には、Tool ノードに設定済みのサードパーティ API キーなどの認証情報は含まれません。これは GitHub にシークレットが混入しない点で好都合ですが、**環境ごとの認証情報は別管理が必須**であることを意味します。

→ **現実解：`dify/*.yaml` を GitHub の SoT にして PR レビューは必須化。実環境への反映は UI からの手動インポート（承認記録付き）を許容する。** バージョン管理の要件（条件④）はこれで満たせます。

---

## 4. サービス提供元に確認すべき事項

構成を確定する前に、以下を Dify 基盤の運用部門に確認してください。上から重要度順です。

1. **デプロイ形態**：閉域 self-hosted か、Dify Cloud か、VPC 版か。バージョンは何か
2. **ワークスペース分離**：用途別（開発／テスト／本番実行）に独立したワークスペースを払い出せるか
3. **Service API**：App API Key の払い出しは許可されているか。エンドポイントは何か
4. **アウトバウンド許可**：HTTP Request ノードから社内プライベート IP（AWX）へ到達できるか。`NO_PROXY` / ssrf_proxy ACL の個別申請フローはあるか
5. **インターネット egress**：SaaS（ServiceNow）への外向き通信は許可されるか。プロキシ設定は入っているか
6. **タイムアウト**：ワークフロー実行時間上限、HTTP ノードのタイムアウト値
7. **アップグレード方針**：頻度、事前通知、DSL 互換性の扱い
8. **可用性 SLA と監査ログ**：本番オペレーションの実行基盤として耐えるか
9. **ServiceNow 側**：MID Server の有無と、Dify セグメントへの到達可否

---

## 5. 確認が取れるまでの推奨方針（no-regret な設計）

どちらの形態でも手戻りが出ない打ち手として、以下を先に固めておくことを勧めます。

1. **本番運用／デプロイタスクの実行主体は Azure Functions / AKS を第一候補にする。**  
   Dify を本番実行エンジンとして前提に置かない。Dify は「開発・PoC・テスト環境向けタスク・人手承認を挟む低クリティカリティの処理」に役割を寄せる。これは条件を満たしたまま、Dify 側の制約に構成全体が引きずられるのを防ぎます。

2. **ServiceNow ⇄ Dify は必ず中継（APIM / Azure Functions）を挟む前提で設計する。**  
   直結できる形態であっても、冪等性・署名検証・リトライ・監査を Dify に持たせられない以上、中継は無駄になりません。閉域構成なら MID Server → 中継 → Dify の経路になります。

3. **Dify に渡す権限を最小化する。**  
   具体的には AWX の `ops-test` Organization のみに紐づくトークン、または ServiceNow の work notes 追記のみのスコープ。閉域でもシークレットは環境変数に平文で載るため、ここは形態に関わらず効きます。

4. **`contracts/`（実行要求スキーマ）を先に確定する。**  
   Dify で実装するか Functions で実装するかが後から入れ替わっても、インターフェースが固定されていれば ServiceNow 側の Flow 実装は作り直しになりません。**これが最大の保険です。**

---

確認結果が判明した段階で、該当ケースに合わせて構成図とシーケンスを具体化できます。特に「ワークスペース分離が可能かどうか」で本番実行基盤の置き場所の判断が変わるため、そこを優先して確認されるとよいかと思います。


---

## 🧑 User  
*2026-07-24T02:07:20.347186Z*

「5. 確認が取れるまでの推奨方針（no-regret な設計）」で、現段階版の構成図とシーケンスを具体化してください。Dify は比較的簡易な業務フローユースケースについて補助的に活用して開発工数を節約する趣旨であり、主に Azure AI Foundry 上で開発される状況を想定しています。


---

## 🤖 Claude  
*2026-07-24T02:10:14.056329Z*

> [!note]- Thinking
> I'm clarifying the architecture where Azure AI Foundry serves as the primary development platform while Dify handles simpler workflows as a supplementary tool, with Azure Functions or AKS managing production execution. The constraint about Dify-developed workflows seems to be cut off, but I need to understand how Dify automation integrates back into the broader system.
>
> The key insight is creating a unified integration gateway (through APIM or Functions) as a single control point that routes requests to either Foundry-developed engines or Dify runtimes, with all callbacks flowing back through this gateway to ServiceNow. This makes Dify swappable without disrupting the overall architecture. I'm also thinking about a task catalog that maps ServiceNow items to their execution targets, which provides flexibility for the no-regret design.
>
> Now I'm planning out the main execution sequences to document: standard Foundry workflows from ServiceNow, simpler Dify-based workflows, test execution flows, and CI/CD pipelines from GitHub. For visualizing these, I'm considering whether to attempt the diagram tool again or stick with ASCII/Mermaid diagrams since the previous attempts had issues.
>
> I need to be thorough here — covering the component architecture, the gateway's responsibilities, task routing tables with examples, JSON Schema contracts, all four sequence diagrams, decision criteria for when to use Dify versus Foundry, and how the migration path works so ServiceNow never needs changes when moving workflows between platforms. The integration gateway itself needs careful design around Azure API Management and Azure Functions with Durable orchestration to handle routing, contract validation, and task execution. I'm debating whether to keep the gateway and executor as separate components or combine them for simplicity. I think the cleaner approach is to treat them as distinct logical roles—a gateway handling routing and control, and an executor running the Foundry workflows—but deploy them together initially within the same Function App or Kubernetes namespace. The contract between them stays internal. Now I'm starting to define the schema for how they communicate...
>
> Writing contract schema... The API returns a 202 status with an execution ID, and I'm defining task catalog entries that specify how each operation gets executed—whether through Foundry, Dify, or Ansible Tower—along with environment restrictions, approval requirements, and timeout settings.
>
> For Dify specifically, I'm outlining the adoption criteria to determine when it's suitable: operations that complete in under 60 seconds, don't spawn heavy Ansible workflows, have simple branching logic, rely primarily on LLM-based reasoning like classification or parameter extraction, and don't require strict idempotency or transactional guarantees. The key constraint is avoiding direct access to private internal systems without going through a gateway.
>
> The strongest pattern emerging is having Dify handle the judgment and transformation work, while delegating actual execution to a gateway or Functions layer. This means Dify returns structured instructions rather than directly triggering AWX—it sidesteps the SSRF proxy issues, centralizes audit trails, and makes future migration to Foundry cleaner. Even though direct Dify-to-AWX integration is technically feasible, deferring it in favor of this gateway pattern is the no-regret move. Generating sequence diagrams... Switching to a cleaner format with numbered protocol steps instead of ASCII sequence diagrams, since that'll be more readable and precise. I'll keep a simple component diagram for the overall picture, then list each interaction as a numbered step with the exact endpoints and authentication methods. I should also add a table for error scenarios like timeouts and failed responses, plus decision points where confirmation is needed.
>
> For the Foundry integration, I need to clarify how the Agent and Prompt Flow components map onto Functions or AKS — either containerizing the Prompt Flow and deploying it directly, or using Foundry's Agent Service as the runtime while Functions handles orchestration. Given the constraint that the executor runs on Functions or AKS, I'll document both approaches and note which fits better with the overall architecture.

Dify を「補助・簡易ユースケース限定」、Azure AI Foundry を主開発基盤とする前提で、現段階（Dify 基盤の仕様が未確認）でも手戻りが出ない構成に具体化します。

---

## 1. 現段階版の設計原則

| # | 原則 | 狙い |
|---|---|---|
| P1 | **ServiceNow は「実行したいこと（task_code）」だけを知る。実装がどこにあるかは知らない** | Dify ↔ Foundry の実装移設で ServiceNow 側を改修不要にする |
| P2 | **統合中継層（Integration Gateway）を単一の統制点にする** | 認証・冪等性・本番ガード・監査を Dify の外に置く |
| P3 | **Dify は「判断・変換」を担い、「作用（AWX 起動）」はさせない** | ssrf_proxy 制約・タイムアウト・監査欠落を回避。将来 Foundry へ移設しやすい |
| P4 | **AWX の起動口を Gateway に一本化する** | 実行主体が誰であれ監査証跡が1箇所に集まる |
| P5 | **長時間処理は必ず非同期（202 + コールバック）** | Dify のタイムアウト制約に全体を引きずられない |

P3 が現段階版の要です。前回整理した「Dify → AWX 直結（④）」は技術的には成立し得ますが、**フィージビリティ確認が取れるまでは採用せず、Dify は構造化された実行指示を Gateway に返すだけ**にします。確認が取れた後に直結へ切り替える場合も、変更箇所は Gateway のタスクカタログ設定のみで済みます。

---

## 2. 構成図（現段階版）

### 開発プレーン

```
  開発者
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│ Chainlit / Gradio（開発者専用UI・テスト実行のみ）             │
└───────┬──────────────────────────────┬───────────────────┘
        ▼                              ▼
┌────────────────────────┐   ┌────────────────────────┐
│ Azure AI Foundry【主】  │   │ Dify【従・簡易ケース】    │
│ Agent / Prompt Flow    │   │ Workflow（判断・変換）    │
│ 評価・プロンプト管理     │   │ ※簡易フローの工数削減用   │
└──────────┬─────────────┘   └──────────┬─────────────┘
           │ コード/定義                    │ DSL(YAML) 手動 export
           ▼                              ▼
┌──────────────────────────────────────────────────────────┐
│ GitHub                                                   │
│  Repo : foundry/ engine/ gateway/ dify/ contracts/       │
│  Issues : 開発作業のみ（ServiceNow とは連携しない）          │
│  Actions: CI（lint/test/eval/schema検証）                  │
│           CD（Functions・AKS へ配備／Dify は差分レビューのみ）│
└──────────────────────────────────────────────────────────┘
```

### 統制・実行プレーン

```
【スコープ外】サービスオーダー用ポータル
        │ 起票
        ▼
╔═════════════════════════════════════════════════════════╗
║ ServiceNow  … 本番タスクの唯一の起点／状態の SoT           ║
║   知っているのは task_code とパラメータのみ                 ║
╚═════╤═══════════════════════════════════▲════════════════╝
      │ ①実行命令 POST /v1/executions      │ 進捗・結果の書き戻し
      │ （OAuth2 CC、必要なら MID Server）   │ （work notes / state）
      ▼                                    │
┌──────────────────────────────────────────┴────────────────┐
│ 統合中継層 Integration Gateway                             │
│  Azure API Management ＋ Durable Functions               │
│  ・認証/認可  ・スキーマ検証  ・冪等性（request_id）         │
│  ・本番ガード（チケット逆引き検証）                          │
│  ・タスクカタログによるルーティング                          │
│  ・実行状態管理  ・コールバック集約  ・監査ログ               │
└───┬────────────────────┬──────────────────────┬──────────┘
    │ impl=foundry        │ impl=dify             │ impl=awx-direct
    ▼                     ▼                        │
┌────────────────────┐  ┌──────────────────────┐   │
│ 実行エンジン         │  │ Dify Runtime          │   │
│ Azure Functions/AKS│  │ Service API 経由で起動  │   │
│ （Foundry製・主）    │  │ 判断/変換のみ・作用なし │   │
│  └→ Foundry の推論   │  │  └→ 実行指示(JSON)を   │   │
│     エンドポイント参照 │  │     Gateway へ返却 ───┼───┤
└─────────┬──────────┘  └──────────────────────┘   │
          │                                         │
          ▼                                         ▼
┌───────────────────────────────────────────────────────────┐
│ Ansible Tower / AWX                                       │
│   Organization: ops-prod ／ ops-test（完全分離）             │
└───────────────────────┬───────────────────────────────────┘
                        ▼            ▲ 完了 Notification Webhook
             本番環境／テスト環境       └→ Gateway が受信し ServiceNow へ
```

### テスト実行経路

```
Chainlit / Gradio
  └→ Foundry / Dify（開発WS）
       └→ Gateway  POST /v1/executions  { env: "test" }
            └→ 実行エンジン(test) → AWX ops-test → テスト環境
  ※ env="test" の資格情報では ops-prod のジョブテンプレートに到達不能
```

---

## 3. 中核となる2つの契約（これを先に固定する）

### ⑴ 実行要求スキーマ `contracts/execution-request.schema.json`

```json
{
  "type": "object",
  "required": ["request_id", "task_code", "env", "source", "parameters"],
  "properties": {
    "request_id":   { "type": "string", "format": "uuid" },
    "task_code":    { "type": "string", "pattern": "^[A-Z]+-[A-Z0-9-]+$" },
    "task_version": { "type": "string" },
    "env":          { "enum": ["prod", "test"] },
    "source": {
      "type": "object",
      "required": ["channel"],
      "properties": {
        "channel":        { "enum": ["servicenow", "devui"] },
        "ticket_number":  { "type": "string" },
        "ticket_sys_id":  { "type": "string" },
        "ticket_table":   { "type": "string" },
        "requested_by":   { "type": "string" }
      }
    },
    "parameters":     { "type": "object" },
    "correlation_id": { "type": "string" }
  },
  "allOf": [{
    "if":   { "properties": { "env": { "const": "prod" } } },
    "then": { "properties": { "source": {
                "required": ["ticket_sys_id", "ticket_table"],
                "properties": { "channel": { "const": "servicenow" } } } } }
  }]
}
```

最後の `allOf` が**「本番タスクは ServiceNow 起点でしか受け付けない」をスキーマレベルで強制する部分**です。Chainlit/Gradio からの要求は `channel: devui` になるため、`env: prod` を指定した時点でスキーマ検証で落ちます。

応答は常に非同期です。

```json
HTTP 202 Accepted
{ "execution_id": "exec-01J...", "status": "accepted",
  "status_url": "/v1/executions/exec-01J..." }
```

### ⑵ タスクカタログ `contracts/task-catalog.yaml`

```yaml
tasks:
  - task_code: OPS-VM-RESTART            # 標準ケース：Foundry製
    version: "1.2.0"
    impl: foundry
    executor: { function: exec-ops-vm-restart }
    awx: { organization: "ops-{env}", job_template: "vm-restart" }
    allowed_env: [prod, test]
    approval_state_required: ["Scheduled", "Implement"]
    timeout_min: 30

  - task_code: OPS-REQ-TRIAGE            # 簡易ケース：Dify製（判断のみ）
    version: "1.0.1"
    impl: dify
    dify:
      app_id: "xxxxxxxx"
      api_key_ref: "kv://dify-app-req-triage"
      response_mode: streaming
      max_wait_sec: 60                   # ssrf_proxy 制約への安全マージン
    post_action:                         # 作用は Gateway 側が実施
      type: awx_launch
      awx: { organization: "ops-{env}", job_template: "{result.job_template}" }
    allowed_env: [prod, test]
    timeout_min: 5

  - task_code: OPS-PATCH-APPLY           # 素通し：ServiceNow→AWX 直
    impl: awx-direct
    awx: { organization: "ops-{env}", job_template: "patch-apply" }
    allowed_env: [prod, test]
    timeout_min: 120
```

**`impl` の値を書き換えるだけで Dify → Foundry の移設が完了する**構造にしておくことが、no-regret の実体です。ServiceNow 側は `task_code` しか知らないため無改修で済みます。

---

## 4. シーケンス

### シーケンス A：本番運用/デプロイタスク（Foundry製・標準ケース）

```
① Portal → ServiceNow        チケット起票（スコープ外）
② ServiceNow                 承認 → state 遷移 → Flow Designer 発火
③ ServiceNow → Gateway       POST /v1/executions
                             { request_id, task_code:"OPS-VM-RESTART",
                               env:"prod", source:{channel:"servicenow",
                               ticket_sys_id, ticket_table}, parameters }
                             認証: OAuth2 client credentials（閉域なら MID Server 経由）

④ Gateway                    スキーマ検証 → 失敗なら 400 即返却
⑤ Gateway                    冪等チェック（request_id が既存なら既存 execution_id を返す）
⑥ Gateway → ServiceNow       GET /api/now/table/{table}/{sys_id}      ★本番ガード
                             ・チケットが実在するか
                             ・state が approval_state_required に含まれるか
                             ・task_code とチケットのカタログ項目が一致するか
                             → 不一致なら 403 + work notes に拒否理由を記録して終了
⑦ Gateway → ServiceNow       202 { execution_id }／work notes に受付を記録
⑧ Gateway                    Durable Orchestrator 起動（correlation_id = チケット番号）

⑨ Orchestrator → Executor    Foundry製ワークフローの実行エンジンを起動
⑩ Executor → Foundry         必要に応じて推論エンドポイントを呼び、パラメータを確定
⑪ Executor → Gateway         確定した AWX 実行指示を返却（作用はここでは行わない）
⑫ Gateway → AWX              POST /api/v2/job_templates/{id}/launch/
                             extra_vars に correlation_id / execution_id を必ず埋め込む
                             → 201 job_id を記録
⑬ Orchestrator               WaitForExternalEvent("awx-completed")  ← ポーリングしない

⑭ AWX → 本番環境              Playbook 実行
⑮ AWX → Gateway              Notification Template（Webhook）で完了通知
⑯ Gateway                    job_id → execution_id を突合（照合できない通知は破棄）
⑰ Gateway → ServiceNow       PATCH /api/now/table/{table}/{sys_id}
                             ・work notes に結果サマリ（job_id / 所要時間 / 対象ホスト数）
                             ・state を Review / Closed へ遷移
⑱ Gateway                    監査ログ確定（correlation_id で全ホップを追跡可能）
```

**⑬ が長時間 Playbook に耐える鍵**です。Durable Functions の外部イベント待機を使い、Gateway 側もポーリングしません。⑫で `extra_vars` に ID を埋めているため、⑯の突合が成立します。

### シーケンス B：本番運用タスク（Dify製・簡易ケース）

```
①〜⑧  シーケンス A と完全に同一（ServiceNow から見て違いは無い）

⑨  Orchestrator → Dify   POST {DIFY}/v1/workflows/run
                         Authorization: Bearer <App API Key>（Key Vault から取得）
                         response_mode: "streaming"
                         inputs: { correlation_id, parameters }
⑩  Dify                  ワークフロー実行（LLM による分類・パラメータ抽出・
                         定型文生成など。外部システムへの作用は行わない）
⑪  Dify → Orchestrator   構造化された実行指示 JSON を返却
                         例: { job_template:"vm-restart", hosts:[...], reason:"..." }
⑫  Orchestrator          返却 JSON を出力スキーマで検証        ★重要
                         → 検証失敗／max_wait_sec 超過なら中断し
                            ServiceNow へ「要人手対応」で書き戻して終了
⑬  Gateway → AWX         post_action に従い job_template を起動（＝作用は Gateway）

⑭〜⑱  シーケンス A と同一
```

Dify の役割が **⑩〜⑪の「判断・変換」だけ**に閉じています。この形なら以下がすべて回避できます。

- ssrf_proxy によるプライベート IP ブロック（Dify は内部システムに触らない）
- ssrf_proxy の 120 秒タイムアウト（Dify の実行は 60 秒以内で完結）
- Dify に AWX 資格情報を置く必要（不要になる）
- Dify 内で実行ログが閉じてしまう監査上の問題（Gateway に集約）

そして条件「Dify で開発されたワークフローは Dify 基盤上で実行される」は満たしたままです。

### シーケンス C：テスト実行（Chainlit / Gradio 起点）

```
① 開発者 → Chainlit/Gradio   自然言語または フォームで指示
② UI → Foundry / Dify        開発WS のワークフローを起動
③ Foundry/Dify → Gateway     POST /v1/executions
                             { env:"test", source:{ channel:"devui",
                               requested_by:"<Entra ID の oid>" } }
                             認証: 開発者の Entra ID トークン（オンビハーフ）
④ Gateway                    スキーマ検証
                             → env:"prod" は allOf 制約で 400（構造的に不能）
⑤ Gateway                    task_code の allowed_env に "test" が含まれるか確認
⑥ Gateway → AWX(ops-test)    テスト用 Organization のジョブテンプレートのみ起動可能
                             ※ Gateway の test 用マネージド ID には
                                ops-prod への権限を一切付与しない（二重防御）
⑦ AWX → テスト環境            Playbook 実行
⑧ AWX → Gateway → UI         結果を開発者へ返却（ServiceNow への書き戻しは行わない）
```

**④のスキーマ制約と⑥の RBAC の二重で本番到達を封じます。** ④だけだと Gateway の実装バグで破れるため、権限側でも切っておく点が重要です。

### シーケンス D：開発CI/CD（GitHub）

```
【Foundry 系・完全自動】
① 開発者          feature ブランチで engine/ gateway/ foundry/ を変更 → PR
② GitHub Actions  CI: lint / unit test / contracts のスキーマ検証
                      / タスクカタログの整合チェック（impl と executor の存在確認）
                      / Foundry の評価データセットによるプロンプト回帰テスト
③ レビュー承認 → main マージ
④ GitHub Actions  CD: OIDC (Workload Identity Federation) で Azure 認証
                      → test 環境の Function App / AKS へ配備
⑤ 受け入れテスト（シーケンス C）
⑥ GitHub Actions  CD: environments の承認ゲート付きで prod スロットへ配備
                      → スロットスワップ

【Dify 系・半自動】
① 開発者          Dify UI でワークフローを編集 → DSL(YAML) を export
② 開発者          dify/<app>.yaml を差し替えて PR
③ GitHub Actions  CI: YAML 構文 / 出力スキーマ定義との整合チェック
                      / シークレット混入検査（DSL には認証情報が含まれない前提の検証）
④ レビュー承認 → main マージ（★ここでバージョン管理の要件を満たす）
⑤ 手動            実行用ワークスペースへ UI からインポート
                  → 実施記録を PR にコメントで残す（承認証跡）
```

Dify のインポート API が公開されていない現状では⑤が手動になりますが、**「GitHub の YAML が正、Dify 上の実体は写像」というルールを CI で担保**しておけば、後日 API が利用可能になった時点で⑤を Actions に置き換えるだけで済みます。

---

## 5. 異常系の扱い

| 事象 | Gateway の挙動 | ServiceNow への反映 |
|---|---|---|
| スキーマ検証エラー | 400 即返却、実行しない | Flow 側でエラー記録、state 変更なし |
| チケット状態が不正（未承認等） | 403、実行しない | work notes に拒否理由 |
| 同一 request_id の再送 | 既存 execution_id を 202 で返す（再実行しない） | 変更なし |
| Dify が `max_wait_sec` 超過 | 実行中断、`impl` を切り替えず人手対応へ | work notes に「Dify 応答なし・要人手」＋ state: On Hold |
| Dify の返却 JSON がスキーマ不適合 | AWX を起動しない | 同上（**作用前に落ちるので副作用なし**） |
| AWX ジョブ失敗 | 完了 Webhook を受信し失敗として確定 | work notes に失敗ログ抜粋 ＋ state: Review |
| AWX 完了 Webhook が届かない | `timeout_min` 経過で Orchestrator がタイムアウト判定 | 「実行状態不明・要確認」で人手エスカレーション |
| ServiceNow への書き戻し失敗 | 指数バックオフでリトライ、超過で DLQ | 運用チームへアラート（実行自体は完了済み） |

「Dify の失敗が本番環境への副作用に繋がらない」ことが、この配置の最大の利点です。

---

## 6. 確認結果に応じた変更箇所

前回挙げた確認事項が判明した際に、**どこを変えれば済むか**を対応付けておきます。

| 確認結果 | 変更箇所 | 影響範囲 |
|---|---|---|
| Dify → AWX が到達可能（NO_PROXY 許可が取れた） | タスクカタログの `post_action` を Dify 内実行に変更 | Gateway 設定のみ。ServiceNow・Foundry は無改修 |
| Dify のワークスペース分離が不可 | 該当タスクの `impl` を `foundry` に変更 | Gateway 設定＋ engine/ の実装追加。ServiceNow 無改修 |
| Dify の Service API 払い出し不可 | 同上（Dify を開発時のプロトタイピング専用に降格） | 同上 |
| ServiceNow → Gateway に MID Server が必要 | ServiceNow の Flow 設定のみ | Gateway 以降は無改修 |
| Dify の SLA が本番要件に満たない | `impl: dify` のタスクを test 環境限定に（`allowed_env: [test]`） | タスクカタログのみ |
| Dify のインポート API が使用可能に | シーケンス D ⑤を Actions 化 | CI/CD のみ |

いずれのケースでも **ServiceNow 側の実装（Flow Designer / カタログ項目）は改修不要**です。ここが確定しているので、Dify の確認を待たずに ServiceNow 連携と Gateway の実装を先行着手できます。

---

## 7. 着手順の推奨

1. `contracts/`（実行要求スキーマ・タスクカタログの定義）を確定 ← **最優先。他の全作業の前提**
2. Gateway（APIM ＋ Durable Functions）の骨格と本番ガードを実装
3. ServiceNow の Flow Designer 実装 ＋ 1タスクでの疎通（`impl: awx-direct` の素通しが最短）
4. Foundry 製ワークフロー1件を `impl: foundry` で実装し、シーケンス A を通す
5. Chainlit/Gradio からのテスト経路（シーケンス C）を整備
6. Dify の確認結果が出た段階で、簡易ユースケース1件を `impl: dify` で追加

3 で AWX 素通しを先に通しておくと、Gateway・ServiceNow・AWX の3者間の認証とコールバック経路の検証が Foundry の実装完了を待たずに済みます。

---

Gateway の Durable Functions のオーケストレーターコードや、ServiceNow 側の Flow Designer の具体的なステップ構成まで落とし込むこともできます。どちらを先に詰めますか。


---
