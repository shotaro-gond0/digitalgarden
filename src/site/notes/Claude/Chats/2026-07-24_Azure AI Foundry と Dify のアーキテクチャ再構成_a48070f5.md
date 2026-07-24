---
{"dg-publish":true,"permalink":"/claude/chats/2026-07-24-azure-ai-foundry-dify-a48070f5/","dg-note-properties":{}}
---

---
title: Azure AI Foundry と Dify のアーキテクチャ再構成
date: '2026-07-24'
created: '2026-07-24T01:53:19.058246Z'
updated: '2026-07-24T06:03:55.762616Z'
account: 権藤祥太郎 (shotaro.gondo@gmail.com)
conversation_id: a48070f5-47c6-4fdf-8330-f4de1e37bd3a
source: claude-chat-export
tags:
- claude
- chat
- cat/ml-dl
- cat/python
- cat/react
- cat/js-ts
- cat/math
- cat/devops
- cat/css
- cat/claude
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

## 🧑 User  
*2026-07-24T05:44:01.893015Z*

上記のアスキーアートによる構成図はズレて適切に閲覧ができません。https://note.glidelow.com/claude/chats/2026-06-11-azure-ai-foundry-dify-agentic-ai-5b9b2611/ この記事中の図のように正しく表示できる形式で出力してください。


---

## 🤖 Claude  
*2026-07-24T05:50:10.398849Z*

> [!note]- Thinking
> The user needs the diagrams rendered as SVG like the example from that article, so let me retry fetching from the visualizer since the earlier 400 error might have been temporary.

> [!note]- 🔧 Tool call: visualize:read_me
> ```json
> {
>   "modules": [
>     "diagram"
>   ],
>   "platform": "desktop"
> }
> ```

> [!note]- ↩️ Tool result: visualize:read_me
> ````
> # Imagine — Visual Creation Suite
>
> ## Modules
> Call read_me again with the modules parameter to load detailed guidance:
> - `diagram` — SVG flowcharts, structural diagrams, illustrative diagrams
> - `mockup` — UI mockups, forms, cards, dashboards
> - `interactive` — interactive explainers with controls
> - `chart` — charts, data analysis, geographic maps (Chart.js, D3 choropleth)
> - `art` — illustration and generative art
> Pick the closest fit. The module includes all relevant design guidance.
>
> **Complexity budget — hard limits:**
> - Box subtitles: ≤5 words. Detail goes in click-through (`sendPrompt`) or the prose below — not the box.
> - Colors: ≤2 ramps per diagram. If colors encode meaning (states, tiers), add a 1-line legend. Otherwise use one neutral ramp.
> - Horizontal tier: ≤4 boxes at full width (~140px each). 5+ boxes → shrink to ≤110px OR wrap to 2 rows OR split into overview + detail diagrams.
>
> If you catch yourself writing "click to learn more" in prose, the diagram itself must ACTUALLY be sparse. Don't promise brevity then front-load everything.
>
> **Accessibility:** For HTML widgets, begin with a visually-hidden `<h2 class="sr-only">` containing a one-sentence summary of the visualization for screen-reader users. (SVG widgets use `role="img"` with `<title>` and `<desc>` instead — see SVG setup.)
>
> You create rich visual content — SVG diagrams/illustrations and HTML interactive widgets — that renders inline in conversation. The best output feels like a natural extension of the chat.
>
> ## Core Design System
>
> These rules apply to ALL use cases.
>
> ### Philosophy
> - **Seamless**: Users shouldn't notice where claude.ai ends and your widget begins.
> - **Flat**: No gradients, mesh backgrounds, noise textures, or decorative effects. Clean flat surfaces.
> - **Compact**: Show the essential inline. Explain the rest in text.
> - **Text goes in your response, visuals go in the tool** — All explanatory text, descriptions, introductions, and summaries must be written as normal response text OUTSIDE the tool call. The tool output should contain ONLY the visual element (diagram, chart, interactive widget). Never put paragraphs of explanation, section headings, or descriptive prose inside the HTML/SVG. If the user asks "explain X", write the explanation in your response and use the tool only for the visual that accompanies it. The user's font settings only apply to your response text, not to text inside the widget.
>
> ### Streaming
> Output streams token-by-token. Structure code so useful content appears early.
> - **HTML**: `<style>` (short) → content HTML → `<script>` last.
> - **SVG**: `<defs>` (markers) → visual elements immediately.
> - Prefer inline `style="..."` over `<style>` blocks — inputs/controls must look correct mid-stream.
> - Keep `<style>` under ~15 lines. Interactive widgets with inputs and sliders need more style rules — that's fine, but don't bloat with decorative CSS.
> - Gradients, shadows, and blur flash during streaming DOM diffs. Use solid flat fills instead.
>
> ### Rules
> - No `<!-- comments -->` or `/* comments */` (waste tokens, break streaming)
> - No font-size below 11px
> - No emoji. Icons = Tabler **outline** webfont (5800+, already loaded): `<i class="ti ti-home"></i>`. Outline only — never use `-filled` suffixes (`ti-heart-filled` etc. are not loaded and will render blank). Inherits color + font-size from parent. Decorative icons get `aria-hidden="true"`; icon-only buttons get `aria-label`. Common: ti-home ti-settings ti-user ti-search ti-x ti-check ti-plus ti-trash ti-edit ti-download ti-upload ti-file ti-folder ti-chart-bar ti-calendar ti-clock ti-arrow-right ti-arrow-left ti-chevron-down ti-external-link ti-copy ti-refresh ti-player-play ti-player-pause ti-heart ti-star ti-bell ti-mail ti-lock ti-eye ti-menu-2. Don't hand-draw icon SVG paths.
> - No gradients, drop shadows, blur, glow, or neon effects
> - No dark/colored backgrounds on outer containers (transparent only — host provides the bg)
> - **Typography**: The default font is Anthropic Sans. For the rare editorial/blockquote moment, use `font-family: var(--font-voice)`.
> - **Headings**: h1 = 22px, h2 = 18px, h3 = 16px — all `font-weight: 500`. Heading color is pre-set to `var(--text-primary)` — don't override it. Body text = 16px, weight 400, `line-height: 1.7`. **Two weights only: 400 regular, 500 bold.** Never use 600 or 700 — they look heavy against the host UI.
> - **Sentence case** always. Never Title Case, never ALL CAPS. This applies everywhere including SVG text labels and diagram headings.
> - **No mid-sentence bolding**, including in your response text around the tool call. Entity names, class names, function names go in `code style` not **bold**. Bold is for headings and labels only.
> - The widget container is `display: block; width: 100%`. Your HTML fills it naturally — no wrapper div needed. Just start with your content directly. If you want vertical breathing room, add `padding: 1rem 0` on your first element.
> - Never use `position: fixed` — the iframe viewport sizes itself to your in-flow content height, so fixed-positioned elements (modals, overlays, tooltips) collapse it to `min-height: 100px`. For modal/overlay mockups: wrap everything in a normal-flow `<div style="min-height: 400px; background: rgba(0,0,0,0.45); display: flex; align-items: center; justify-content: center;">` and put the modal inside — it's a faux viewport that actually contributes layout height.
> - No DOCTYPE, `<html>`, `<head>`, or `<body>` — just content fragments.
> - When placing text on a colored background (badges, pills, cards, tags), use the darkest shade from that same color family for the text — never plain black or generic gray.
> - **Corners**: use `border-radius: var(--radius)` for controls, `12px` for cards. In SVG, `rx="4"` is the default — larger values make pills, use only when you mean a pill.
> - **No rounded corners on single-sided borders** — if using `border-left` or `border-top` accents, set `border-radius: 0`. Rounded corners only work with full borders on all sides.
> - **No titles or prose inside the tool output** — see Philosophy above.
> - **Icon sizing**: Tabler `<i class="ti …">` sizes with `font-size` — 16–20px inline, 24px max decorative. For one-off inline SVG icons, set `width`/`height` explicitly (same limits).
> - No tabs, carousels, or `display: none` sections during streaming — hidden content streams invisibly. Show all content stacked vertically. (Post-streaming JS-driven steppers are fine — see Illustrative/Interactive sections.)
> - No nested scrolling — auto-fit height.
> - Scripts execute after streaming — load libraries via `<script src="https://cdnjs.cloudflare.com/ajax/libs/...">` (UMD globals), then use the global in a plain `<script>` that follows. The library `<script src>` tag must come BEFORE any inline script that uses its global — never call a library from code that appears above its `<script src>` tag.
> - **CDN allowlist (CSP-enforced)**: external resources may ONLY load from `cdnjs.cloudflare.com`, `esm.sh`, `cdn.jsdelivr.net`, `unpkg.com`, `fonts.googleapis.com`, `fonts.gstatic.com`. All other origins are blocked by the sandbox — the request silently fails.
>
> ### CSS Variables
> **Surfaces**: `--surface-2` (white), `--surface-1` (card), `--surface-0` (page bg); role tints `--bg-{accent,danger,success,warning}`
> **Text**: `--text-primary` (black), `--text-secondary` (muted), `--text-muted` (hints); role `--text-{accent,danger,success,warning}`
> **Borders**: `--border` (default hairline), `--border-strong` (hover), `--border-stronger`; role `--border-{accent,danger,success,warning}`
> **Typography**: `--font-sans`, `--font-voice` (serif), `--font-mono`
> **Layout**: `--radius` (8px), `--pad-{sm,md,lg,xl}`, `--gap-{xs,sm,md,lg,xl}`; for larger corners use literal `12px`/`16px`
> All auto-adapt to light/dark mode. For custom colors in HTML, use CSS variables.
>
> **Dark mode is mandatory** — every color must work in both modes:
> - In SVG: use the pre-built color classes (`c-blue`, `c-teal`, `c-amber`, etc.) for colored nodes — they handle light/dark mode automatically. Never write `<style>` blocks for colors.
> - In SVG: every `<text>` element needs a class (`t`, `ts`, `th`) — never omit fill or use `fill="inherit"`. Inside a `c-{color}` parent, text classes auto-adjust to the ramp.
> - In HTML: always use CSS variables (--text-primary, --text-secondary) for text. Never hardcode colors like color: #333 — invisible in dark mode.
> - Mental test: if the background were near-black, would every text element still be readable?
>
> ### sendPrompt(text)
> A global function that sends a message to chat as if the user typed it. Use it when the user's next step benefits from Claude thinking. Handle filtering, sorting, toggling, and calculations in JS instead.
>
> ### Links
> `<a href="https://...">` just works — clicks are intercepted and open the host's link-confirmation dialog. Or call `openLink(url)` directly.
>
> ## When nothing fits
> Pick the closest use case below and adapt. When nothing fits cleanly:
> - Default to editorial layout if the content is explanatory
> - Default to card layout if the content is a bounded object
> - All core design system rules still apply
> - Use `sendPrompt()` for any action that benefits from Claude thinking
>
>
> ## Color palette
>
> 9 color ramps, each with 7 stops from lightest to darkest. 50 = lightest fill, 100-200 = light fills, 400 = mid tones, 600 = strong/border, 800-900 = text on light fills.
>
> | Class | Ramp | 50 (lightest) | 100 | 200 | 400 | 600 | 800 | 900 (darkest) |
> |-------|------|------|-----|-----|-----|-----|-----|------|
> | `c-purple` | Purple | #EEEDFE | #CECBF6 | #AFA9EC | #7F77DD | #534AB7 | #3C3489 | #26215C |
> | `c-teal` | Teal | #E1F5EE | #9FE1CB | #5DCAA5 | #1D9E75 | #0F6E56 | #085041 | #04342C |
> | `c-coral` | Coral | #FAECE7 | #F5C4B3 | #F0997B | #D85A30 | #993C1D | #712B13 | #4A1B0C |
> | `c-pink` | Pink | #FBEAF0 | #F4C0D1 | #ED93B1 | #D4537E | #993556 | #72243E | #4B1528 |
> | `c-gray` | Gray | #F1EFE8 | #D3D1C7 | #B4B2A9 | #888780 | #5F5E5A | #444441 | #2C2C2A |
> | `c-blue` | Blue | #E6F1FB | #B5D4F4 | #85B7EB | #378ADD | #185FA5 | #0C447C | #042C53 |
> | `c-green` | Green | #EAF3DE | #C0DD97 | #97C459 | #639922 | #3B6D11 | #27500A | #173404 |
> | `c-amber` | Amber | #FAEEDA | #FAC775 | #EF9F27 | #BA7517 | #854F0B | #633806 | #412402 |
> | `c-red` | Red | #FCEBEB | #F7C1C1 | #F09595 | #E24B4A | #A32D2D | #791F1F | #501313 |
>
> **How to assign colors**: Color should encode meaning, not sequence. Don't cycle through colors like a rainbow (step 1 = blue, step 2 = amber, step 3 = red...). Instead:
> - Group nodes by **category** — all nodes of the same type share one color. E.g. in a vaccine diagram: all immune cells = purple, all pathogens = coral, all outcomes = teal.
> - For illustrative diagrams, map colors to **physical properties** — warm ramps for heat/energy, cool for cold/calm, green for organic, gray for structural/inert.
> - Use **gray for neutral/structural** nodes (start, end, generic steps).
> - Use **2-3 colors per diagram**, not 6+. More colors = more visual noise. A diagram with gray + purple + teal is cleaner than one using every ramp.
> - **Prefer purple, teal, coral, pink** for general diagram categories. Reserve blue, green, amber, and red for cases where the node genuinely represents an informational, success, warning, or error concept — those colors carry strong semantic connotations from UI conventions. (Exception: illustrative diagrams may use blue/amber/red freely when they map to physical properties like temperature or pressure.)
>
> **Text on colored backgrounds:** Always use the 800 or 900 stop from the same ramp as the fill. Never use black, gray, or --text-primary on colored fills. **When a box has both a title and a subtitle, they must be two different stops** — title darker (800 in light mode, 100 in dark), subtitle lighter (600 in light, 200 in dark). Same stop for both reads flat; the weight difference alone isn't enough. For example, text on Blue 50 (#E6F1FB) must use Blue 800 (#0C447C) or 900 (#042C53), not black. This applies to SVG text elements inside colored rects, and to HTML badges, pills, and labels with colored backgrounds.
>
> **Light/dark mode quick pick** — use only stops from the table, never off-table hex values:
> - **Light mode**: 50 fill + 600 stroke + **800 title / 600 subtitle**
> - **Dark mode**: 800 fill + 200 stroke + **100 title / 200 subtitle**
> - Apply `c-{ramp}` to a `<g>` wrapping shape+text, or directly to a `<rect>`/`<circle>`/`<ellipse>`. Never to `<path>` — paths don't get ramp fill. For colored connector strokes use inline `stroke="#..."` (any mid-ramp hex works in both modes). Dark mode is automatic for ramp classes. Available: c-gray, c-blue, c-red, c-amber, c-green, c-teal, c-purple, c-coral, c-pink.
>
> For status/semantic meaning in UI (success, warning, danger) use CSS variables. For categorical coloring in both diagrams and UI, use these ramps.
>
>
> ## SVG setup
>
> **ViewBox safety checklist** — before finalizing any SVG, verify:
> 1. Find your lowest element: max(y + height) across all rects, max(y) across all text baselines.
> 2. Set viewBox height = that value + 40px buffer.
> 3. Find your rightmost element: max(x + width) across all rects. All content must stay within x=0 to x=680.
> 4. For text with text-anchor="end", the text extends LEFT from x. If x=118 and text is 200px wide, it starts at x=-82 — outside the viewBox. Increase x or use text-anchor="start".
> 5. Never use negative x or y coordinates. The viewBox starts at 0,0.
> 6. **No unintentional overlaps.** For every pair of elements that aren't meant to layer (label-on-label, label-on-arrow, box-on-box, callout-on-shape), check their bounding boxes do not intersect. The only allowed overlaps are deliberate: a label centered inside its own box, an arrowhead touching the box it points to, a highlight rect behind the thing it highlights. If two unrelated elements would collide, move one — shorten the label, shift the y, add a row. A diagram with crossed labels reads as broken regardless of how good the content is.
> 7. Flowcharts/structural only: for every pair of boxes in the same row, check that the left box's (x + width) is less than the right box's x by at least 20px. If four 160px boxes plus three 20px gaps sum to more than 640px, the row doesn't fit — shrink the boxes or cut the subtitles, don't let them overlap.
>
> **SVG setup**: `<svg width="100%" viewBox="0 0 680 H" role="img"><title>…</title><desc>…</desc>…` — 680px wide, flexible height. The root `<svg>` MUST carry `role="img"` with `<title>` and `<desc>` as its first children so screen readers can announce what the diagram shows. Set H to fit content tightly — the last element's bottom edge + 40px padding. Don't leave excess empty space below the content. Safe area: x=40 to x=640, y=40 to y=(H-40). Background transparent. **Do not wrap the SVG in a container `<div>` with a background color** — the widget host already provides the card container and background. Output the raw `<svg>` element directly.
>
> **The 680 in viewBox is load-bearing — do not change it.** It matches the widget container width so SVG coordinate units render 1:1 with CSS pixels. With `width="100%"`, the browser scales the entire coordinate space to fit the container: `viewBox="0 0 476 H"` in a 680px container scales everything by 680/476 = 1.43×, so your `class="th"` 14px text renders at ~20px. The font calibration table below and all "text fits in box" math assume 1:1. If your diagram content is naturally narrow, **keep viewBox width at 680 and center the content** (e.g. content spans x=240..440) — do not shrink the viewBox to hug the content. This applies equally to inline SVGs inside HTML steppers and widgets: same `viewBox="0 0 680 H"`, same 1:1 guarantee.
>
> **viewBox height:** After layout, find max_y (bottom-most point of any shape, including text baselines + 4px descent). Set viewBox height = max_y + 20. Don't guess.
>
> **text-anchor='end' at x<60 is risky** — the longest label will extend left past x=0. Use text-anchor='start' and right-align the column instead, or check: label_chars × 8 < anchor_x.
>
> **One SVG per tool call** — each call must contain exactly one <svg> element. Never leave an abandoned or partial SVG in the output. If your first attempt has problems, replace it entirely — do not append a corrected version after the broken one.
>
> **Style rules for all diagrams**:
> - Every `<text>` element must carry one of the pre-built classes (`t`, `ts`, `th`). An unclassed `<text>` inherits the default sans font, which is the tell that you forgot the class.
> - Use only two font sizes: 14px for node/region labels (class="t" or "th"), 12px for subtitles, descriptions, and arrow labels (class="ts"). No other sizes.
> - No decorative step numbers, large numbering, or oversized headings outside boxes.
> - No icons or illustrations inside boxes — text only. (Exception: illustrative diagrams may use simple shape-based indicators inside drawn objects — see below.)
> - Sentence case on all labels.
>
> **Font size calibration for diagram text labels** - Here's csv table to give you better sense of the Anthropic Sans font rendering width:
> ```csv
> text, chars length, font-weight, font-size, rendered width
> Authentication Service, chars: 22, font-weight: 500, font-size: 14px, width: 167px
> Background Job Processor, chars: 24, font-weight: 500, font-size: 14px, width: 201px
> Detects and validates incoming tokens, chars: 37, font-weight: 400, font-size: 14px, width: 279px
> forwards request to, chars: 19, font-weight: 400, font-size: 12px, width: 123px
> データベースサーバー接続, chars: 12, font-weight: 400, font-size: 14px, width: 181px
> ```
>
> Before placing text in a box, check: does (text width + 2×padding) fit the container?
>
> **SVG `<text>` never auto-wraps.** Every line break needs an explicit `<tspan x="..." dy="1.2em">`. If your subtitle is long enough to need wrapping, it's too long — shorten it (see complexity budget).
>
> **Example check**: You want to put "Glucose (C₆H₁₂O₆)" in a rounded rect. The text is 20 characters at 14px ≈ 180px wide. Add 2×24px padding = 228px minimum box width. If your rect is only 160px wide, the text WILL overflow — either shorten the label (e.g. just "Glucose") or widen the box. Subscript characters like ₆ and ₁₂ still take horizontal space — count them.
>
> **Pre-built classes** (already loaded in SVG widget):
> - `class="t"` = sans 14px primary, `class="ts"` = sans 12px secondary, `class="th"` = sans 14px medium (500)
> - `class="box"` = neutral rect (`--surface-1` fill, `--border-strong` stroke)
> - `class="node"` = clickable group with hover effect (cursor pointer, slight dim on hover)
> - `class="arr"` = arrow line (1.5px, open chevron head)
> - `class="leader"` = dashed leader line (tertiary stroke, 0.5px, dashed)
> - `class="c-{ramp}"` = colored node (c-blue, c-teal, c-amber, c-green, c-red, c-purple, c-coral, c-pink, c-gray). Apply to `<g>` or shape element (rect/circle/ellipse), NOT to paths. Sets fill+stroke on shapes, auto-adjusts child `t`/`ts`/`th`, dark mode automatic.
>
> **c-{ramp} nesting:** These classes use direct-child selectors (`>`). Nest a `<g>` inside a `<g class="c-blue">` and the inner shapes become grandchildren — they lose the fill and render BLACK (SVG default). Put `c-*` on the innermost group holding the shapes, or on the shapes directly. If you need click handlers, put `onclick` on the `c-*` group itself, not a wrapper.
>
> - Short aliases: `var(--p)`, `var(--s)`, `var(--t)`, `var(--bg2)`, `var(--b)`
> - Arrow marker: always include this `<defs>` at the start of every SVG:
>   `<defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>`
>   Then use `marker-end="url(#arrow)"` on lines. The head uses `context-stroke`, so it inherits the colour of whichever line it sits on — a dashed green line gets a green head, a grey line gets a grey head. Never a colour mismatch. Do not add filters or extra markers to `<defs>`. `<pattern>` fills are allowed when used as a secondary encoding for categorical data — keep them subtle (thin hatching, sparse dots). Never rely on color alone to distinguish categories; pair each color with a secondary visual cue (hatching, dash pattern, or shape). Illustrative diagrams may add a single `<clipPath>` or `<linearGradient>` (see Illustrative section).
>
> **Minimize standalone labels.** Every `<text>` element must be inside a box (title or ≤5-word subtitle) or in the legend. Arrow labels are usually unnecessary — if the arrow's meaning isn't obvious from its source + target, put it in the box subtitle or in prose below. Labels floating in space collide with things and are ambiguous.
>
> **Stroke width:** Use 0.5px strokes for diagram borders and edges — not 1px or 2px. Thin strokes feel more refined.
>
> **Connector paths need `fill="none"`.** SVG defaults to `fill: black` — a curved connector without `fill="none"` renders as a huge black shape instead of a clean line. Every `<path>` or `<polyline>` used as a connector/arrow MUST have `fill="none"`. Only set fill on shapes meant to be filled (rects, circles, polygons).
>
> **Rect rounding:** `rx="4"` for subtle corners. `rx="8"` max for emphasized rounding. `rx` ≥ half the height = pill shape — deliberate only.
>
> **Schematic containers use dashed rects with a label.** Don't draw literal shapes (organelle ovals, cloud outlines, server tower icons) — the diagram is a schema, not an illustration. A dashed `<rect>` labeled "Reactor vessel" reads cleaner than an `<ellipse>` that clips content.
>
> **Lines stop at component edges.** When a line meets a component (wire into a bulb, edge into a node), draw it as segments that stop at the boundary — never draw through and rely on a fill to hide the line. The background color is not guaranteed; any occluding fill is a coupling. Compute the stop/start coordinates from the component's position and size.
>
> **Physical-color scenes (sky, water, grass, skin, materials):** Use ALL hardcoded hex — never mix with `c-*` theme classes. The scene should not invert in dark mode. If you need a dark variant, provide it explicitly with `@media (prefers-color-scheme: dark)` — this is the one place that's allowed. Mixing hardcoded backgrounds with theme-responsive `c-*` foreground breaks: half inverts, half doesn't.
>
> **No rotated text**. `<defs>` may contain the arrow marker, a `<clipPath>`, subtle `<pattern>` fills used as a secondary visual cue alongside color for categorical data, and — in illustrative diagrams only — a single `<linearGradient>`. Nothing else: no filters, no extra markers.
>
>
> ## Diagram types
> *"Explain how compound interest works" / "How does a process scheduler work"*
>
> **Two rules that cause most diagram failures — check these before writing each arrow and each box:**
> 1. **Arrow intersection check**: before writing any `<line>` or `<path>`, trace its coordinates against every box you've already placed. If the line crosses any rect's interior (not just its source/target), it will visibly slash through that box — use an L-shaped `<path>` detour instead. This applies to arrows crossing labels too.
> 2. **Box width from longest label**: before writing a `<rect>`, find its longest child text (usually the subtitle). `rect_width = max(title_chars × 8, subtitle_chars × 7) + 24`. A 100px-wide box holds at most a 10-char subtitle. If your subtitle is "Files, APIs, streams" (20 chars), the box needs 164px minimum — 100px will visibly overflow.
>
> **Tier packing:** Compute total width BEFORE placing. Example — 4 pub/sub consumer boxes:
> - WRONG: x=40,160,260,360 w=160 → 40-60px overlaps (4×160=640 > 480 available)
> - RIGHT: x=50,200,350,500 w=130 gap=20 → fits (4×130 + 3×20 = 580 ≤ 590 safe width; right edge at 630 ≤ 640)
> Work bottom-up for trees: size leaf tier first, parent width ≥ sum of children.
>
> **Diagrams are the hardest use case** — they have the highest failure rate due to precise coordinate math. Common mistakes: viewBox too small (content clipped), arrows through unrelated boxes, labels on arrow lines, text past viewBox edges. For illustrative diagrams, also watch for: shapes extending outside the viewBox, overlapping labels that obscure the drawing, and color choices that don't map intuitively to the physical properties being shown. Double-check coordinates before finalizing.
>
> Use SVG for diagrams. The widget automatically wraps SVG output in a card.
>
> **Pick the right diagram type.** The decision is about *intent*, not subject matter. Ask: is the user trying to *document* this, or *understand* it?
>
> **Reference diagrams** — the user wants a map they can point at. Precision matters more than feeling. Boxes, labels, arrows, containment. These are the diagrams you'd find in documentation.
> - **Flowchart** — steps in sequence, decisions branching, data transforming. Good for: approval workflows, request lifecycles, build pipelines, "what happens when I click submit". Trigger phrases: *"walk me through the process"*, *"what are the steps"*, *"what's the flow"*.
> - **Structural diagram** — things inside other things. Good for: file systems (blocks in inodes in partitions), VPC/subnet/instance, "what's inside a cell". Trigger phrases: *"what's the architecture"*, *"how is this organised"*, *"where does X live"*.
>
> **Intuition diagrams** — the user wants to *feel* how something works. The goal isn't a correct map, it's the right mental model. These should look nothing like a flowchart. The subject doesn't need a physical form — it needs a *visual metaphor*.
> - **Illustrative diagram** — draw the mechanism. Physical things get cross-sections (water heaters, engines, lungs). Abstract things get spatial metaphors: an LLM is a stack of layers with tokens lighting up as attention weights, gradient descent is a ball rolling down a loss surface, a hash table is a row of buckets with items falling into them, TCP is two people passing numbered envelopes. Good for: ML concepts (transformers, attention, backprop, embeddings), physics intuition, CS fundamentals (pointers, recursion, the call stack), anything where the breakthrough is *seeing* it rather than *reading* it. Trigger phrases: *"how does X actually work"*, *"explain X"*, *"I don't get X"*, *"give me an intuition for X"*.
>
> **Route on the verb, not the noun.** Same subject, different diagram depending on what was asked:
>
> | User says | Type | What to draw |
> |---|---|---|
> | "how do LLMs work" | **Illustrative** | Token row, stacked layer slabs, attention threads glowing warm between tokens. Go interactive if you can. |
> | "transformer architecture" | Structural | Labelled boxes: embedding, attention heads, FFN, layer norm. |
> | "how does attention work" | **Illustrative** | One query token, a fan of lines to every key, line opacity = weight. |
> | "how does gradient descent work" | **Illustrative** | Contour surface, a ball, a trail of steps. Slider for learning rate. |
> | "what are the training steps" | Flowchart | Forward → loss → backward → update. Boxes and arrows. |
> | "how does TCP work" | **Illustrative** | Two endpoints, numbered packets in flight, an ACK returning. |
> | "TCP handshake sequence" | Flowchart | SYN → SYN-ACK → ACK. Three boxes. |
> | "explain the Krebs cycle" / "how does the event loop work" | **HTML stepper** | Click through stages. Never a ring. |
> | "how does a hash map work" | **Illustrative** | Key falling through a funnel into one of N buckets. |
> | "draw the database schema" / "show me the ERD" | **mermaid.js** | `erDiagram` syntax. Not SVG. |
>
> The illustrative route is the default for *"how does X work"* with no further qualification. It is the more ambitious choice — don't chicken out into a flowchart because it feels safer. Claude draws these well.
>
> Don't mix families in one diagram. If you need both, draw the intuition version first (build the mental model), then the reference version (fill in the precise labels) as a second tool call with prose between.
>
> **For complex topics, use multiple SVG calls** — break the explanation into a series of smaller diagrams rather than one dense diagram. Each SVG streams in with its own animation and card, creating a visual narrative the user can follow step by step.
>
> **Always add prose between diagrams** — never stack multiple SVG calls back-to-back without text. Between each SVG, write a short paragraph (in your normal response text, outside the tool call) that explains what the next diagram shows and connects it to the previous one.
>
> **Promise only what you deliver** — if your response text says "here are three diagrams", you must include all three tool calls. Never promise a follow-up diagram and omit it. If you can only fit one diagram, adjust your text to match. One complete diagram is better than three promised and one delivered.
>
> #### Flowchart
>
> For sequential processes, cause-and-effect, decision trees.
>
> **Planning**: Size boxes to fit their text generously. At 14px sans-serif, each character is ~8px wide — a label like "Load Balancer" (13 chars) needs a rect at least 140px wide. When in doubt, make boxes wider and leave more space between them. Cramped diagrams are the most common failure mode.
>
> **Special characters are wider**: Chemical formulas (C₆H₁₂O₆), math notation (∑, ∫, √), subscripts/superscripts via <tspan> with dy/baseline-shift, and Unicode symbols all render wider than plain Latin characters. For labels containing formulas or special notation, add 30-50% extra width to your estimate. When in doubt, make the box wider — overflow looks worse than extra padding.
>
> **Spacing**: 60px minimum between boxes, 24px padding inside boxes, 12px between text and edges. Leave 10px gap between arrowheads and box edges. Two-line boxes (title + subtitle) need at least 56px height with 22px between the lines.
>
> **Vertical text placement**: Every `<text>` inside a box needs `dominant-baseline="central"`, with y set to the *centre* of the slot it sits in. Without it SVG treats y as the baseline, the glyph body sits ~4px higher than you intended, and the descenders land on the line below. Formula: for text centred in a rect at (x, y, w, h), use `<text x={x+w/2} y={y+h/2} text-anchor="middle" dominant-baseline="central">`. For a row inside a multi-row box, y is the centre of *that row*, not of the whole box.
>
> **Layout**: Prefer single-direction flows (all top-down or all left-right). Keep diagrams simple — max 4-5 nodes per diagram. The widget is narrow (~680px) so complex layouts break.
>
> **When the prompt itself is over budget**: if the user lists 6+ components ("draw me auth, products, orders, payments, gateway, queue"), don't draw all of them in one pass — you'll get overlapping boxes and arrows through text, every time. Decompose: (1) a stripped overview with the boxes only and at most one or two arrows showing the main flow — no fan-outs, no N-to-N meshes; (2) then one diagram per interesting sub-flow ("here's what happens when an order is placed", "here's the auth handshake"), each with 3-4 nodes and room to breathe. Count the nouns before you draw. The user asked for completeness — give it to them across several diagrams, not crammed into one.
>
> **Cycles don't get drawn as rings.** If the last stage feeds back into the first (Krebs cycle, event loop, GC mark-and-sweep, TCP retransmit), your instinct is to place the stages around a circle. Don't. Every spacing rule in this spec is Cartesian — there is no collision check for "input box orbits outside stage box on a ring". You will get satellite boxes overlapping the stages they feed, labels sitting on the dashed circle, and tangential arrows that point nowhere. The ring is decoration; the loop is conveyed by the return arrow.
>
> Build a stepper in HTML. One panel per stage, dots or pills showing position (● ○ ○), Next wraps from the last stage back to the first — that's the loop. Each panel owns its inputs and products: an event loop's pending callbacks live *inside* the Poll panel, not floating next to a box on a ring. Nothing collides because nothing shares the canvas. Only fall back to a linear SVG (stages in a row, curved `<path>` return arrow) when there's one input and one output total and no per-stage detail to show.
>
> **Feedback loops in linear flows:** Don't draw a physical arrow traversing the layout (it fights the flow direction and clips edges). Instead:
> - Small `↻` glyph + text near the cycle point: `<text>↻ returns to start</text>`
> - Or restructure the whole diagram as a circle if the cycle IS the point
>
> **Arrows:** A line from A to B must not cross any other box or label. If the direct path crosses something, route around with an L-bend: `<path d="M x1 y1 L x1 ymid L x2 ymid L x2 y2"/>`. Place arrow labels in clear space, not on the midpoint.
>
> Keep all nodes the same height when they have the same content type (e.g. all single-line boxes = 44px, all two-line boxes = 56px).
>
> **Flowchart components** — use these patterns consistently:
>
> *Single-line node* (44px tall): title only. The `c-blue` class sets fill, stroke, and text colors for both light and dark mode automatically — no `<style>` block needed.
> ```svg
> <g class="node c-blue" onclick="sendPrompt('Tell me more about T-cells')">
>   <rect x="100" y="20" width="180" height="44" rx="8" stroke-width="0.5"/>
>   <text class="th" x="190" y="42" text-anchor="middle" dominant-baseline="central">T-cells</text>
> </g>
> ```
>
> *Two-line node* (56px tall): bold title + muted subtitle.
> ```svg
> <g class="node c-blue" onclick="sendPrompt('Tell me more about dendritic cells')">
>   <rect x="100" y="20" width="200" height="56" rx="8" stroke-width="0.5"/>
>   <text class="th" x="200" y="38" text-anchor="middle" dominant-baseline="central">Dendritic cells</text>
>   <text class="ts" x="200" y="56" text-anchor="middle" dominant-baseline="central">Detect foreign antigens</text>
> </g>
> ```
>
> *Connector* (no label — meaning is clear from source + target):
> ```svg
> <line x1="200" y1="76" x2="200" y2="120" class="arr" marker-end="url(#arrow)"/>
> ```
>
> *Neutral node* (gray, for start/end/generic steps): use `class="box"` for auto-themed fill/stroke, and default text classes.
>
> Make all nodes clickable by default — wrap in `<g class="node" onclick="sendPrompt('...')">`. The hover effect is built in.
>
> #### Structural diagram
>
> For concepts where physical or logical containment matters — things inside other things.
>
> **When to use**: The explanation depends on *where* processes happen. Examples: how a cell works (organelles inside a cell), how a file system works (blocks inside inodes inside partitions), how a building's HVAC works (ducts inside floors inside a building), how a CPU cache hierarchy works (L1 inside core, L2 shared).
>
> **Core idea**: Large rounded rects are containers. Smaller rects inside them are regions or sub-structures. Text labels describe what happens in each region. Arrows show flow between regions or from external inputs/outputs.
>
> **Container rules**:
> - Outermost container: large rounded rect, rx=20-24, lightest fill (50 stop), 0.5px stroke (600 stop). Label at top-left inside, 14px bold.
> - Inner regions: medium rounded rects, rx=8-12, next shade fill (100-200 stop). Use a different color ramp if the region is semantically different from its parent.
> - 20px minimum padding inside every container — text and inner regions must not touch the container edges.
> - Max 2-3 nesting levels. Deeper nesting gets unreadable at 680px width.
>
> **Layout**:
> - Place inner regions side by side within the container, with 16px+ gap between them.
> - External inputs (sunlight, water, data, requests) sit outside the container with arrows pointing in.
> - External outputs sit outside with arrows pointing out.
> - Keep external labels short — one word or a short phrase. Details go in the prose between diagrams.
>
> **What goes inside regions**: Text only — the region name (14px bold) and a short description of what happens there (12px). Don't put flowchart-style boxes inside regions. Don't draw illustrations or icons inside.
>
> **Structural container example** (library branch with two side-by-side regions, an internal labeled arrow, and an external input). ViewBox 700x320, horizontal layout, color classes handle both light and dark mode — no `<style>` block:
> ```svg
> <defs>
>   <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
>     <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
>   </marker>
> </defs>
> <!-- Outer container -->
> <g class="c-green">
>   <rect x="120" y="30" width="560" height="260" rx="20" stroke-width="0.5"/>
>   <text class="th" x="400" y="62" text-anchor="middle">Library branch</text>
>   <text class="ts" x="400" y="80" text-anchor="middle">Main floor</text>
> </g>
> <!-- Inner: Circulation desk -->
> <g class="c-teal">
>   <rect x="150" y="100" width="220" height="160" rx="12" stroke-width="0.5"/>
>   <text class="th" x="260" y="130" text-anchor="middle">Circulation desk</text>
>   <text class="ts" x="260" y="148" text-anchor="middle">Checkouts, returns</text>
> </g>
> <!-- Inner: Reading room -->
> <g class="c-amber">
>   <rect x="450" y="100" width="210" height="160" rx="12" stroke-width="0.5"/>
>   <text class="th" x="555" y="130" text-anchor="middle">Reading room</text>
>   <text class="ts" x="555" y="148" text-anchor="middle">Seating, reference</text>
> </g>
> <!-- Arrow between inner boxes with label -->
> <text class="ts" x="410" y="175" text-anchor="middle">Books</text>
> <line x1="370" y1="185" x2="448" y2="185" class="arr" marker-end="url(#arrow)"/>
> <!-- External input: New acq. — text vertically aligned with arrow -->
> <text class="ts" x="40" y="185" text-anchor="middle">New acq.</text>
> <line x1="75" y1="185" x2="118" y2="185" class="arr" marker-end="url(#arrow)"/>
> ```
>
> **Color in structural diagrams**: Nested regions need distinct ramps — `c-{ramp}` classes resolve to fixed fill/stroke stops, so the same class on parent and child gives identical fills and flattens the hierarchy. Pick a *related* ramp for inner structures (e.g. Green for the library envelope, Teal for the circulation desk inside it) and a *contrasting* ramp for a region that does something functionally different (e.g. Amber for the reading room). This keeps the diagram scannable — you can see at a glance which parts are related.
>
> **Database schemas / ERDs — use mermaid.js, not SVG.** A schema table is a header plus N field rows plus typed columns plus crow's-foot connectors. That is a text-layout problem and hand-placing it in SVG fails the same way every time. mermaid.js `erDiagram` does layout, cardinality, and connector routing for free. ERDs only; everything else stays in SVG.
>
> ```
> erDiagram
>   USERS ||--o{ POSTS : writes
>   POSTS ||--o{ COMMENTS : has
>   USERS {
>     uuid id PK
>     string email
>     timestamp created_at
>   }
>   POSTS {
>     uuid id PK
>     uuid user_id FK
>     string title
>   }
> ```
>
> Use HTML for ERDs. Import and initialize in a `<script type="module">`. The host CSS re-styles mermaid's output to match the design system — keep the init block exactly as shown (fontFamily + fontSize are used for layout measurement; deviate and text clips). After rendering, replace sharp-cornered entity `<path>` elements with rounded `<rect rx="8">` to match the design system, and strip borders from attribute rows (only the outer container and header row keep visible borders — alternating fill colors separate the rows):
> ```html
> <style>
> #erd svg.erDiagram .divider path { stroke-opacity: 0.5; }
> #erd svg.erDiagram .row-rect-odd path,
> #erd svg.erDiagram .row-rect-odd rect,
> #erd svg.erDiagram .row-rect-even path,
> #erd svg.erDiagram .row-rect-even rect { stroke: none !important; }
> </style>
> <div id="erd"></div>
> <script type="module">
> import mermaid from 'https://esm.sh/mermaid@11/dist/mermaid.esm.min.mjs';
> const dark = matchMedia('(prefers-color-scheme: dark)').matches;
> await document.fonts.ready;
> mermaid.initialize({
>   startOnLoad: false,
>   theme: 'base',
>   fontFamily: '"Anthropic Sans", sans-serif',
>   themeVariables: {
>     darkMode: dark,
>     fontSize: '13px',
>     fontFamily: '"Anthropic Sans", sans-serif',
>     lineColor: dark ? '#9c9a92' : '#73726c',
>     textColor: dark ? '#c2c0b6' : '#3d3d3a',
>   },
> });
> const { svg } = await mermaid.render('erd-svg', `erDiagram
>   USERS ||--o{ POSTS : writes
>   POSTS ||--o{ COMMENTS : has`);
> document.getElementById('erd').innerHTML = svg;
>
> // Round only the outermost entity box corners (not internal row stripes)
> document.querySelectorAll('#erd svg.erDiagram .node').forEach(node => {
>   const firstPath = node.querySelector('path[d]');
>   if (!firstPath) return;
>   const d = firstPath.getAttribute('d');
>   const nums = d.match(/-?[\d.]+/g)?.map(Number);
>   if (!nums || nums.length < 8) return;
>   const xs = [nums[0], nums[2], nums[4], nums[6]];
>   const ys = [nums[1], nums[3], nums[5], nums[7]];
>   const x = Math.min(...xs), y = Math.min(...ys);
>   const w = Math.max(...xs) - x, h = Math.max(...ys) - y;
>   const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
>   rect.setAttribute('x', x); rect.setAttribute('y', y);
>   rect.setAttribute('width', w); rect.setAttribute('height', h);
>   rect.setAttribute('rx', '8');
>   for (const a of ['fill', 'stroke', 'stroke-width', 'class', 'style']) {
>     if (firstPath.hasAttribute(a)) rect.setAttribute(a, firstPath.getAttribute(a));
>   }
>   firstPath.replaceWith(rect);
> });
>
> // Strip borders from attribute rows (mermaid v11: .row-rect-odd / .row-rect-even)
> document.querySelectorAll('#erd svg.erDiagram .row-rect-odd path, #erd svg.erDiagram .row-rect-even path').forEach(p => {
>   p.setAttribute('stroke', 'none');
> });
> </script>
> ```
>
> Works identically for `classDiagram` — swap the diagram source; init stays the same.
>
> #### Illustrative diagram
>
> For building *intuition*. The subject might be physical (an engine, a lung) or completely abstract (attention, recursion, gradient descent) — what matters is that a spatial drawing conveys the mechanism better than labelled boxes would. These are the diagrams that make someone go "oh, *that's* what it's doing."
>
> **Two flavours, same rules:**
> - **Physical subjects** get drawn as simplified versions of themselves. Cross-sections, cutaways, schematics. A water heater is a tank with a burner underneath. A lung is a branching tree in a cavity. You're drawing *the thing*, stylised.
> - **Abstract subjects** get drawn as *spatial metaphors*. You're inventing a shape for something that doesn't have one — but the shape should make the mechanism obvious. A transformer is a stack of horizontal slabs with a bright thread of attention connecting tokens across layers. A hash function is a funnel scattering items into a row of buckets. The call stack is literally a stack of frames growing and shrinking. Embeddings are dots clustering in space. The metaphor *is* the explanation.
>
> This is the most ambitious diagram type and the one Claude is best at. Lean into it. Use colour for intensity (a hot attention weight glows amber, a cold one stays gray). Use repetition for scale (many small circles = many parameters).
>
> **Prefer interactive over static.** A static cross-section is a good answer; a cross-section you can *operate* is a great one. The decision rule: if the real-world system has a control, give the diagram that control. A water heater has a thermostat — so give the user a slider that shifts the hot/cold boundary, a toggle that fires the burner and animates convection currents. An LLM has input tokens — let the user click one and watch the attention weights re-fan. A cache has a hit rate — let them drag it and watch latency change. Reach for HTML with inline SVG first; only fall back to static SVG when there's genuinely nothing to twiddle.
>
> **When NOT to use**: The user is asking for a *reference*, not an *intuition*. "What are the components of a transformer" wants labelled boxes — that's a structural diagram. "Walk me through our CI pipeline" wants sequential steps — that's a flowchart. Also skip this when the metaphor would be arbitrary rather than revealing: drawing "the cloud" as a cloud shape or "microservices" as little houses doesn't teach anything about how they work. If the drawing doesn't make the *mechanism* clearer, don't draw it.
>
> **Fidelity ceiling**: These are schematics, not illustrations. Every shape should read at a glance. If a `<path>` needs more than ~6 segments to draw, simplify it. A tank is a rounded rect, not a Bézier portrait of a tank. A flame is three triangles, not a fire. Recognisable silhouette beats accurate contour every time — if you find yourself carefully tracing an outline, you're overshooting.
>
> **Core principle**: Draw the mechanism, not a diagram *about* the mechanism. Spatial arrangement carries the meaning; labels annotate. A good illustrative diagram works with the labels removed.
>
> **What changes from flowchart/structural rules**:
>
> - **Shapes are freeform.** Use `<path>`, `<ellipse>`, `<circle>`, `<polygon>`, and curved lines to represent real forms. A water tank is a tall rect with rounded bottom. A heart valve is a pair of curved paths. A circuit trace is a thin polyline. You are not limited to rounded rects.
> - **Layout follows the subject's geometry**, not a grid. If the thing is tall and narrow (a water heater, a thermometer), the diagram is tall and narrow. If it's wide and flat (a PCB, a geological cross-section), the diagram is wide. Let the subject dictate proportions within the 680px viewBox width.
> - **Color encodes intensity**, not category. For physical subjects: warm ramps (amber, coral, red) = heat/energy/pressure, cool ramps (blue, teal) = cold/calm, gray = inert structure. For abstract subjects: warm = active/high-weight/attended-to, cool or gray = dormant/low-weight/ignored. A user should be able to glance at the diagram and see *where the action is* without reading a single label.
> - **Layering and overlap are encouraged — for shapes.** Unlike flowcharts where boxes must never overlap, illustrative diagrams can layer shapes for depth — a pipe entering a tank, attention lines fanning through layers, insulation wrapping a chamber. Use z-ordering (later in source = on top) deliberately.
> - **Text is the exception — never let a stroke cross it.** The overlap permission is for shapes only. Every label needs 8px of clear air between its baseline/cap-height and the nearest stroke. Don't solve this with a background rect — solve it by *placing the text somewhere else*. Labels go in the quiet regions: above the drawing, below it, in the margin with a leader line, or in the gap between two fans of lines. If there is no quiet region, the drawing is too dense — remove something or split into two diagrams.
> - **Small shape-based indicators are allowed** when they communicate physical state. Triangles for flames. Circles for bubbles or particles. Wavy lines for steam or heat radiation. Parallel lines for vibration. These aren't decoration — they tell the user what's happening physically. Keep them simple: basic SVG primitives, not detailed illustrations.
> - **One gradient per diagram is permitted** — the only exception to the global no-gradients rule — and only to show a *continuous* physical property across a region (temperature stratification in a tank, pressure drop along a pipe, concentration in a solution). It must be a single `<linearGradient>` between exactly two stops from the same colour ramp. No radial gradients, no multi-stop fades, no gradient-as-aesthetic. If two stacked flat-fill rects communicate the same thing, do that instead.
> - **Animation is permitted for interactive HTML versions.** Use CSS `@keyframes` animating only `transform` and `opacity`. Keep loops under ~2s, and wrap every animation in `@media (prefers-reduced-motion: no-preference)` so it's opt-out by default. Animations should show how the system *behaves* — convection current, rotation, flow — not just move for the sake of moving. No physics engines or heavy libraries.
>
> All core rules still apply (viewBox 680px, dark mode mandatory, 14/12px text, pre-built classes, arrow marker, clickable nodes).
>
> **Label placement**:
> - Place labels *outside* the drawn object when possible, with a thin leader line (0.5px dashed, `var(--t)` stroke) pointing to the relevant part. This keeps the illustration uncluttered.
> - For large internal zones (like temperature regions in a tank), labels can sit inside if there's ample clear space — minimum 20px from any edge.
> - External labels sit in the margin area or above/below the object. **Pick one side for labels and put them all there** — at 680px wide you don't have room for a drawing *and* label columns on both sides. Reserve at least 140px of horizontal margin on the label side. Labels on the left are the ones that clip: `text-anchor="end"` extends leftward from x, and with multi-line callouts it's very easy to blow past x=0 without noticing. Default to right-side labels with `text-anchor="start"` unless the subject's geometry forces otherwise. Use `class="ts"` (12px) for callouts, `class="th"` (14px medium) for major component names.
>
> **Composition approach**:
> 1. Start with the main object's silhouette — the largest shape, centered in the viewBox.
> 2. Add internal structure: chambers, pipes, membranes, mechanical parts.
> 3. Add external connections: pipes entering/exiting, arrows showing flow direction, labels for inputs and outputs.
> 4. Add state indicators last: color fills showing temperature/pressure/concentration, small animated elements showing movement or energy.
> 5. Leave generous whitespace around the object for labels — don't crowd annotations against the viewBox edges.
>
> **Static vs interactive**: Static cutaways and cross-sections work best as pure SVG. If the diagram benefits from controls — a slider that changes a temperature zone, buttons toggling between operating states, live readouts — use HTML with inline SVG for the drawing and HTML controls around it.
>
> **Illustrative diagram example** — interactive water heater cross-section with vivid physical-realism colors, animated convection currents, and controls. Uses HTML with inline SVG: a thermostat slider shifts the hot/cold gradient boundary, a heating toggle animates flames on/off and transitions convection to paused. viewBox is 680×560; tank occupies x=180..440, leaving 140px+ of right margin for labels. Smooth convection paths use `stroke-dasharray:5 5` at ~1.6s for a gentle flow feel. A warm-glow overlay on the hot zone pulses subtly when heating is on. Flame shapes use warm gradient fills and clean opacity transitions. Labels sit along the right margin with leader lines.
> ```html
> <style>
>   @keyframes conv { to { stroke-dashoffset: -20; } }
>   @keyframes flicker { 0%,100%{opacity:1} 50%{opacity:.82} }
>   @keyframes glow { 0%,100%{opacity:.3} 50%{opacity:.6} }
>   .conv { stroke-dasharray:5 5; animation: conv var(--dur,1.6s) linear infinite; transition: opacity .5s; }
>   .conv.off { opacity:0; animation-play-state:paused; }
>   #flames path { transition: opacity .5s; }
>   #flames.off path { opacity:0; animation:none; }
>   #flames path:nth-child(odd)  { animation: flicker .6s ease-in-out infinite; }
>   #flames path:nth-child(even) { animation: flicker .8s ease-in-out infinite .15s; }
>   #warm-glow { animation: glow 3s ease-in-out infinite; transition: opacity .5s; }
>   #warm-glow.off { opacity:0; animation:none; }
>   .toggle-track { position:relative;width:32px;height:18px;background:var(--border-strong);border-radius:9px;transition:background .2s;display:inline-block; }
>   .toggle-track:has(input:checked) { background:var(--text-accent); }
>   #heat-toggle:checked + span { transform:translateX(14px); }
> </style>
> <svg width="100%" viewBox="0 0 680 560">
>   <defs>
>     <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker>
>     <linearGradient id="tg" x1="0" y1="0" x2="0" y2="1">
>       <stop id="gh" offset="40%" stop-color="#E8593C" stop-opacity="0.45"/>
>       <stop id="gc" offset="40%" stop-color="#3B8BD4" stop-opacity="0.4"/>
>     </linearGradient>
>     <linearGradient id="fg1" x1="0" y1="1" x2="0" y2="0"><stop offset="0%" stop-color="#E85D24"/><stop offset="60%" stop-color="#F2A623"/><stop offset="100%" stop-color="#FCDE5A"/></linearGradient>
>     <linearGradient id="fg2" x1="0" y1="1" x2="0" y2="0"><stop offset="0%" stop-color="#D14520"/><stop offset="50%" stop-color="#EF8B2C"/><stop offset="100%" stop-color="#F9CB42"/></linearGradient>
>     <linearGradient id="pipe-h" x1="0" y1="0" x2="0" y2="1"><stop offset="0%" stop-color="#D05538" stop-opacity=".25"/><stop offset="100%" stop-color="#D05538" stop-opacity=".08"/></linearGradient>
>     <linearGradient id="pipe-c" x1="0" y1="0" x2="0" y2="1"><stop offset="0%" stop-color="#3B8BD4" stop-opacity=".25"/><stop offset="100%" stop-color="#3B8BD4" stop-opacity=".08"/></linearGradient>
>     <clipPath id="tc"><rect x="180" y="55" width="260" height="390" rx="14"/></clipPath>
>   </defs>
>   <!-- Tank fill -->
>   <g clip-path="url(#tc)"><rect x="180" y="55" width="260" height="390" fill="url(#tg)"/></g>
>   <!-- Warm glow overlay (pulses when heating) -->
>   <g clip-path="url(#tc)"><rect id="warm-glow" x="180" y="55" width="260" height="160" fill="#E8593C" opacity=".3"/></g>
>   <!-- Tank shell (double stroke for solidity) -->
>   <rect x="180" y="55" width="260" height="390" rx="14" fill="none" stroke="var(--t)" stroke-width="2.5" opacity=".25"/>
>   <rect x="180" y="55" width="260" height="390" rx="14" fill="none" stroke="var(--t)" stroke-width="1"/>
>   <!-- Hot pipe out (top right) -->
>   <rect x="370" y="14" width="16" height="50" rx="4" fill="url(#pipe-h)"/>
>   <path d="M378 14V55" stroke="var(--t)" stroke-width="3" stroke-linecap="round" fill="none"/>
>   <!-- Cold pipe in + dip tube (top left) -->
>   <rect x="234" y="14" width="16" height="50" rx="4" fill="url(#pipe-c)"/>
>   <path d="M242 14V55" stroke="var(--t)" stroke-width="3" stroke-linecap="round" fill="none"/>
>   <path d="M242 55V395" stroke="var(--t)" stroke-width="2.5" stroke-linecap="round" fill="none" opacity=".5"/>
>   <!-- Convection currents (curved paths at different speeds) -->
>   <path class="conv" style="--dur:1.6s" fill="none" stroke="#D05538" stroke-width="1" opacity=".5" d="M350 380C355 320,365 240,358 140Q355 110,340 100"/>
>   <path class="conv" style="--dur:2.1s" fill="none" stroke="#C04828" stroke-width=".8" opacity=".35" d="M300 390C308 340,320 260,315 170Q312 130,298 115"/>
>   <path class="conv" style="--dur:2.6s" fill="none" stroke="#B05535" stroke-width=".7" opacity=".3" d="M380 370C382 310,388 230,382 150Q378 120,365 110"/>
>   <!-- Burner bar -->
>   <rect x="188" y="454" width="244" height="5" rx="2" fill="var(--t)" opacity=".6"/>
>   <rect x="220" y="462" width="180" height="6" rx="3" fill="var(--t)" opacity=".3"/>
>   <!-- Flames (gradient-filled organic shapes) -->
>   <g id="flames">
>     <path d="M240,454Q248,430 252,438Q256,424 260,454Z" fill="url(#fg1)"/>
>     <path d="M278,454Q285,426 290,434Q295,418 300,454Z" fill="url(#fg2)"/>
>     <path d="M320,454Q328,428 333,436Q338,420 342,454Z" fill="url(#fg1)"/>
>     <path d="M360,454Q367,430 371,438Q375,422 380,454Z" fill="url(#fg2)"/>
>     <path d="M398,454Q404,434 408,440Q412,428 416,454Z" fill="url(#fg1)"/>
>   </g>
>   <!-- Labels (right margin) -->
>   <g class="node" onclick="sendPrompt('How does hot water exit the tank?')">
>     <line class="leader" x1="386" y1="34" x2="468" y2="70"/><circle cx="386" cy="34" r="2" fill="var(--t)"/>
>     <text class="ts" x="474" y="74">Hot water outlet</text></g>
>   <g class="node" onclick="sendPrompt('How does the cold water inlet work?')">
>     <line class="leader" x1="250" y1="34" x2="468" y2="140"/><circle cx="250" cy="34" r="2" fill="var(--t)"/>
>     <text class="ts" x="474" y="144">Cold water inlet</text></g>
>   <g class="node" onclick="sendPrompt('What does the dip tube do?')">
>     <line class="leader" x1="250" y1="260" x2="468" y2="220"/><circle cx="250" cy="260" r="2" fill="var(--t)"/>
>     <text class="ts" x="474" y="224">Dip tube</text></g>
>   <g class="node" onclick="sendPrompt('What does the thermostat control?')">
>     <line class="leader" x1="440" y1="250" x2="468" y2="300"/><circle cx="440" cy="250" r="2" fill="var(--t)"/>
>     <text class="ts" x="474" y="304">Thermostat</text></g>
>   <g class="node" onclick="sendPrompt('What material is the tank made of?')">
>     <line class="leader" x1="440" y1="380" x2="468" y2="380"/><circle cx="440" cy="380" r="2" fill="var(--t)"/>
>     <text class="ts" x="474" y="384">Tank wall</text></g>
>   <g class="node" onclick="sendPrompt('How does the gas burner heat water?')">
>     <line class="leader" x1="432" y1="454" x2="468" y2="454"/><circle cx="432" cy="454" r="2" fill="var(--t)"/>
>     <text class="ts" x="474" y="458">Heating element</text></g>
> </svg>
> <div style="display:flex;align-items:center;gap:16px;margin:12px 0 0;font-size:13px;color:var(--text-secondary)">
>   <label style="display:flex;align-items:center;gap:6px;cursor:pointer;user-select:none">
>     <span class="toggle-track">
>       <input type="checkbox" id="heat-toggle" checked onchange="toggleHeat(this.checked)" style="position:absolute;opacity:0;width:100%;height:100%;cursor:pointer;margin:0">
>       <span style="position:absolute;top:2px;left:2px;width:14px;height:14px;background:#fff;border-radius:50%;transition:transform .2s;pointer-events:none"></span>
>     </span>
>     Heating
>   </label>
>   <span>Thermostat</span>
>   <input type="range" id="temp-slider" min="10" max="90" value="40" style="flex:1" oninput="setTemp(this.value)">
>   <span id="temp-label" style="min-width:36px;text-align:right">40%</span>
> </div>
> <script>
> function setTemp(v) {
>   document.getElementById('gh').setAttribute('offset', v+'%');
>   document.getElementById('gc').setAttribute('offset', v+'%');
>   document.getElementById('temp-label').textContent = v+'%';
> }
> function toggleHeat(on) {
>   document.getElementById('flames').classList.toggle('off', !on);
>   document.getElementById('warm-glow').classList.toggle('off', !on);
>   document.querySelectorAll('.conv').forEach(p => p.classList.toggle('off', !on));
> }
> </script>
> ```
>
> **Illustrative example — abstract subject** (attention in a transformer). Same rules, no physical object. A row of tokens at the bottom, one query token highlighted, weight-scaled lines fanning to every other token. Caption sits below the fan — clear of every stroke — not inside it.
> ```svg
> <rect class="c-purple" x="60" y="40"  width="560" height="26" rx="6" stroke-width="0.5"/>
> <rect class="c-purple" x="60" y="80"  width="560" height="26" rx="6" stroke-width="0.5"/>
> <rect class="c-purple" x="60" y="120" width="560" height="26" rx="6" stroke-width="0.5"/>
> <text class="ts" x="72" y="57" >Layer 3</text>
> <text class="ts" x="72" y="97" >Layer 2</text>
> <text class="ts" x="72" y="137">Layer 1</text>
>
> <line stroke="#EF9F27" stroke-linecap="round" x1="340" y1="230" x2="116" y2="146" stroke-width="1"   opacity="0.25"/>
> <line stroke="#EF9F27" stroke-linecap="round" x1="340" y1="230" x2="228" y2="146" stroke-width="1.5" opacity="0.4"/>
> <line stroke="#EF9F27" stroke-linecap="round" x1="340" y1="230" x2="340" y2="146" stroke-width="4"   opacity="1.0"/>
> <line stroke="#EF9F27" stroke-linecap="round" x1="340" y1="230" x2="452" y2="146" stroke-width="2.5" opacity="0.7"/>
> <line stroke="#EF9F27" stroke-linecap="round" x1="340" y1="230" x2="564" y2="146" stroke-width="1"   opacity="0.2"/>
>
> <g class="node" onclick="sendPrompt('What do the attention weights mean?')">
>   <rect class="c-gray"  x="80"  y="230" width="72" height="36" rx="6" stroke-width="0.5"/>
>   <rect class="c-gray"  x="192" y="230" width="72" height="36" rx="6" stroke-width="0.5"/>
>   <rect class="c-amber" x="304" y="230" width="72" height="36" rx="6" stroke-width="1"/>
>   <rect class="c-gray"  x="416" y="230" width="72" height="36" rx="6" stroke-width="0.5"/>
>
> ...[1307 文字省略（原文が長すぎるため切り詰め）]...
> ````

> [!note]- Thinking
> The visualizer is working, so now I need to render the diagrams properly instead of using ASCII art. I have six diagrams to create: the development plane architecture, the control and execution plane architecture, the test execution flow, and three different sequences for production Foundry, Dify, and test environments.
>
> Given the complexity constraints of fitting 4-5 nodes per diagram with no more than 4 boxes per horizontal tier, I'll split this into multiple SVG diagrams with explanatory text between them. Rather than attempting sequence diagrams in SVG, I'll use vertical flowcharts which render more cleanly. My plan is to create four key diagrams: an overview showing the three-plane separation, a main production execution flow combining sequences A and B, a test execution path, and a comparison of the Foundry versus Dify differences. For the first diagram showing all three planes, I'm designing a layered vertical layout where the development plane sits at the top as a container with Chainlit/Gradio, Azure AI Foundry, and Dify as separate components, with GitHub also included, then the control plane below with ServiceNow, and the execution plane at the bottom with the gateway, execution engine, and Dify runtime.
>
> Laying out the development plane container...
>
> I'm reconsidering the overall structure—maybe it makes sense to separate this into multiple diagrams: the development plane on its own, then the control and execution planes together as the main flow, followed by a production execution sequence diagram. Let me think through how to organize these visually. I'm settling on four diagrams to replace the ASCII art: the development plane, the control/execution plane, the production execution sequence with branching logic for Foundry versus Dify, and the test execution path. The sequences will stay as clean numbered lists without ASCII alignment, alongside tables. Now I'm designing the first diagram for the development plane, laying out the Chainlit/Gradio components at the top with a dashed container structure. Adding the secondary boxes and verifying spacing... I'm setting the viewBox height to 360 and positioning the Japanese title "開発プレーン" centered at the top, with the subtitle below it, making sure it doesn't overlap with the Chainlit box that starts at y=76. be light, so I'll use a plain dashed rect for the container with the box class. For the color scheme, I'm going with Chainlit and GitHub as gray, Foundry as purple, and Dify as teal—that's three color ramps plus gray, which fits within the acceptable range.
>
> Now moving to the second diagram covering the control and execution plane, which is the larger one with elements like ServiceNow, the Integration Gateway, and the execution engine.
>
> For the layout, I'm positioning the portal box at the top as a dashed outline to indicate it's out of scope. The text "サービスオーダーポータル" is about 12 characters, which fits better than the longer version. I need to make the box wide enough—around 240 pixels—to accommodate the text comfortably, and I'm considering whether to add "スコープ外" as a subtitle or keep it as a single-line box.
>
> Now I'm working on the arrows between components. The guidance suggests minimizing standalone labels, but arrow labels are acceptable when they're brief and meaningful. I'll add short labels like "起票", "実行命令", and "結果書き戻し" to clarify the flow. The ServiceNow box comes next at y=104, with the title and a subtitle explaining it's the single source of truth for production tasks. The arrow flowing down from there will carry the "実行命令" label.
>
> For the Gateway section, I'm laying out the Integration Gateway box with multiple lines of information. The title combines Japanese and English, and I need to fit the subtitle and additional details about authentication, idempotency, production guards, and audit capabilities within the width constraints.
>
> Now I'm working through the three execution engine branches below the gateway — the Runtime implementations and direct AWX integration. I'm reconsidering whether to show all three as separate boxes or simplify by showing just two implementation boxes with a direct passthrough arrow to AWX. back to the Gateway. I'll use bidirectional arrows with marker-start and marker-end to keep things clean and avoid clutter. Same approach for the ServiceNow ↔ Gateway connection. Now I'm positioning the AWX box at the bottom with its title and subtitle.
>
> I'm realizing the flow needs adjustment—the Gateway should be the sole launcher to AWX per the design, but both the execution engine and Dify return to Gateway first before it routes to AWX. This means I need to reconsider the layout so the arrow from Gateway to AWX has a clear path. Maybe restructuring with Gateway as a central spine would work better, or repositioning the components vertically so the routing logic is visually clearer.
>
> Refining the layout now: placing the execution engine and Dify Runtime side by side below Gateway with bidirectional arrows between them, then routing an L-shaped path from Gateway's left edge down the left margin to AWX to avoid overlapping with the other components. This keeps the constraint that Gateway is the only direct launcher to AWX while maintaining clean visual separation. I'm hitting a layout collision with the execution engine box at y=340, so I'll move the "起動" label down to y=378 where there's clear space. Actually, it's cleaner to just skip the label and explain it in the prose instead. For the production environment box, I'm expanding it to show the target environments—Azure, on-premises, and other cloud options—with a title and subtitle layout, and drawing an arrow from AWX down to connect it. Verifying the vertical spacing and arrow paths — the left path descends at x=60 without collision since the Gateway starts at x=100, and I'm noting that the gap between the implementation boxes and AWX is intentional since there's no direct arrow between them, which I'll clarify in the prose explanation.
>
> Now I'm working through the color scheme: ServiceNow gets coral for control, and I need to decide on Gateway's color — it's the central hub so it should stand out, maybe purple or amber depending on the visual hierarchy I want.
>
> Actually, let me organize this by function instead. Control functions like ServiceNow and the portal use coral, the relay and execution layer like Gateway and the execution engine use purple, Dify components use teal, and the actual target environments use gray. That gives me a clean four-color palette.
>
> The portal is out of scope anyway, so I'll mark it with a dashed box to signal that visually. Now moving on to the next diagram — a vertical sequence flow showing the production execution path with the Dify/Foundry branch, starting with ServiceNow's approval and execution command flowing into Gateway.
>
> I'm mapping out the full execution sequence: ServiceNow triggers the approval, Gateway validates the schema and idempotency, then branches to either Foundry or Dify Runtime, launches AWX, receives the completion webhook, and writes the result back to ServiceNow. That's seven steps, which exceeds the typical 4-5 node budget, but a vertical linear flow with narrow boxes should stay readable. I'll compress it to six key steps and keep each box single-line, using a 680-pixel width with centered boxes. Adding the final workflow steps and connection logic... Laying out the test execution diagram with five boxes in a vertical column, spacing them evenly from y=30 down to y=470, and positioning the main content around x=80 with a width of 300 pixels. I'll add the blocked production path branching off to the right from the Gateway box. Adding the dashed line with a blocking label and finalizing the diagram dimensions. I'm planning to keep the textual step lists alongside the four rendered diagrams, covering the main sequences and branches the user asked for.
>
> Writing SVG code... Widening the portal box to 240px and shifting the layout down to accommodate a subtitle, then positioning the ServiceNow box below with proper arrow spacing and labels. fits 206), subtitle "Azure Functions / AKS" (21 latin @12px = 137px, fits)
>   - Dify Runtime: x=360,y=308,w=230,h=56 → 360..590. title "Dify Runtime", subtitle "判断・変換のみ" (7 JP = 91px)
>     Gap between 320 and 360 = 40. Good.
>   - AWX: x=140,y=404,w=400,h=56 → 404..460. title "Ansible Tower / AWX", subtitle "ops-prod / ops-test 分離"
>   - 対象環境: x=190,y=500,w=300,h=56 → 500
>
> Positioning the execution engine and Dify Runtime side by side with proper spacing, then placing AWX below them and the target environment at the bottom of the diagram.
>
> Now I'm checking the connection paths between components — the left path from Gateway curves down and across to AWX without overlapping the execution engine, while the right path goes from AWX up to Gateway, passing beside Dify Runtime. The vertical line from AWX connects down to the target environment box.
>
> I'm considering whether to add labels for these paths to clarify the bidirectional flow, particularly for the left path showing AWX startup through Gateway. The space is tight between components, so I'm weighing whether explicit labels would help or clutter the diagram. Now checking the layout constraints for Diagram 3—verifying that the box widths and text labels fit properly within their allocated spaces, including the ServiceNow component, Gateway validation layers, and the execution engine with its subtitle about Foundry workflows. Calculating vertical spacing for the diagram elements with 8-pixel gaps between boxes, then mapping out the arrow paths that connect them—checking that the main column width accommodates all the text labels properly. Checking the spacing and dimensions — the 12px gap works, and the arrows align at x=230. The text fits within the boxes once I account for character widths in both Latin and Japanese. Now I'm assigning colors: gray for the UI and test environment boxes, purple for the Foundry/Dify gateway, teal for the AWX test instance, and red for the blocked production system.
>
> I'm concerned about the response length with four diagrams, so I'll keep the prose between them minimal. I should also dial back mid-sentence bolding to be safe, though headings can stay bold. Time to start writing the response with a brief intro, then move into the first diagram. Writing SVG markup...

ASCII の図を SVG に描き直します。内容は前回と同一で、表現のみ変更します。

まず開発プレーンです。

**🖼️ Diagram: development_plane_foundry_primary_dify_secondary**

<svg width="100%" viewBox="0 0 680 356" role="img" xmlns="http://www.w3.org/2000/svg"><title>開発プレーンの構成</title><desc>Chainlit/Gradio から Azure AI Foundry と Dify を操作し、開発資産を GitHub で管理する構成</desc>
<defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
<rect x="40" y="30" width="600" height="306" rx="16" fill="none" stroke="var(--b)" stroke-width="0.5" stroke-dasharray="5 5"/>
<text class="th" x="340" y="52" text-anchor="middle" dominant-baseline="central">開発プレーン</text>
<g class="node c-gray" onclick="sendPrompt('Chainlit と Gradio の役割分担を詳しく')">
<rect x="210" y="76" width="260" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="96" text-anchor="middle" dominant-baseline="central">Chainlit / Gradio</text>
<text class="ts" x="340" y="114" text-anchor="middle" dominant-baseline="central">開発者専用UI</text>
</g>
<path d="M340 132 V149 H205 V158" class="arr" fill="none" marker-end="url(#arrow)"/>
<path d="M340 132 V149 H475 V158" class="arr" fill="none" marker-end="url(#arrow)"/>
<g class="node c-purple" onclick="sendPrompt('Azure AI Foundry 側の開発成果物の内訳を詳しく')">
<rect x="80" y="166" width="250" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="205" y="186" text-anchor="middle" dominant-baseline="central">Azure AI Foundry</text>
<text class="ts" x="205" y="204" text-anchor="middle" dominant-baseline="central">主開発基盤</text>
</g>
<g class="node c-teal" onclick="sendPrompt('Dify に任せる簡易ユースケースの選定基準は')">
<rect x="350" y="166" width="250" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="475" y="186" text-anchor="middle" dominant-baseline="central">Dify</text>
<text class="ts" x="475" y="204" text-anchor="middle" dominant-baseline="central">簡易ユースケース補助</text>
</g>
<path d="M205 222 V239 H340 V248" class="arr" fill="none" marker-end="url(#arrow)"/>
<path d="M475 222 V239 H340 V248" class="arr" fill="none" marker-end="url(#arrow)"/>
<g class="node c-gray" onclick="sendPrompt('GitHub リポジトリ構成と CI の内容を詳しく')">
<rect x="80" y="256" width="520" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="276" text-anchor="middle" dominant-baseline="central">GitHub</text>
<text class="ts" x="340" y="294" text-anchor="middle" dominant-baseline="central">開発資産の SoT・開発Issue・開発CI/CD</text>
</g>
</svg>

> [!note]- Thinking
> I'm moving on to describe the second diagram in prose form.

Foundry が主、Dify は簡易ユースケース補助という比重で、両者の成果物はいずれも GitHub に集約されます。GitHub のスコープは開発作業のみで、本番の動線には一切登場しません。

次が中核となる統制・実行プレーンです。

**🖼️ Diagram: control_and_execution_plane_gateway_centric**

<svg width="100%" viewBox="0 0 680 576" role="img" xmlns="http://www.w3.org/2000/svg"><title>統制プレーンと実行プレーンの構成</title><desc>ServiceNow を起点に統合中継層 Gateway が実行エンジンと Dify Runtime を呼び分け、AWX の起動を Gateway に一本化する構成</desc>
<defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
<g class="node" onclick="sendPrompt('サービスオーダーポータルとの接続はどう扱う')">
<rect x="220" y="30" width="240" height="56" rx="8" fill="none" stroke="var(--b)" stroke-width="0.5" stroke-dasharray="5 5"/>
<text class="th" x="340" y="50" text-anchor="middle" dominant-baseline="central">サービスオーダーポータル</text>
<text class="ts" x="340" y="68" text-anchor="middle" dominant-baseline="central">スコープ外</text>
</g>
<line x1="340" y1="86" x2="340" y2="108" class="arr" marker-end="url(#arrow)"/>
<text class="ts" x="356" y="99" dominant-baseline="central">起票</text>
<g class="node c-coral" onclick="sendPrompt('ServiceNow 側の Flow Designer 実装を詳しく')">
<rect x="140" y="116" width="400" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="136" text-anchor="middle" dominant-baseline="central">ServiceNow</text>
<text class="ts" x="340" y="154" text-anchor="middle" dominant-baseline="central">本番タスクの唯一の起点</text>
</g>
<line x1="340" y1="172" x2="340" y2="212" class="arr" marker-start="url(#arrow)" marker-end="url(#arrow)"/>
<text class="ts" x="356" y="192" dominant-baseline="central">実行命令 / 結果</text>
<g class="node c-purple" onclick="sendPrompt('Integration Gateway の実装構成を詳しく')">
<rect x="100" y="212" width="480" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="232" text-anchor="middle" dominant-baseline="central">統合中継層 Integration Gateway</text>
<text class="ts" x="340" y="250" text-anchor="middle" dominant-baseline="central">認証・冪等性・本番ガード・監査</text>
</g>
<line x1="205" y1="268" x2="205" y2="308" class="arr" marker-start="url(#arrow)" marker-end="url(#arrow)"/>
<line x1="475" y1="268" x2="475" y2="308" class="arr" marker-start="url(#arrow)" marker-end="url(#arrow)"/>
<g class="node c-purple" onclick="sendPrompt('実行エンジンの Durable Functions 実装を詳しく')">
<rect x="90" y="308" width="230" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="205" y="328" text-anchor="middle" dominant-baseline="central">実行エンジン</text>
<text class="ts" x="205" y="346" text-anchor="middle" dominant-baseline="central">Azure Functions / AKS</text>
</g>
<g class="node c-teal" onclick="sendPrompt('Dify Runtime に作用を持たせない理由を詳しく')">
<rect x="360" y="308" width="230" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="475" y="328" text-anchor="middle" dominant-baseline="central">Dify Runtime</text>
<text class="ts" x="475" y="346" text-anchor="middle" dominant-baseline="central">判断・変換のみ</text>
</g>
<path d="M100 240 H60 V432 H132" class="arr" fill="none" marker-end="url(#arrow)"/>
<text class="ts" x="96" y="420" text-anchor="middle" dominant-baseline="central">起動</text>
<path d="M540 432 H620 V240 H588" class="arr" fill="none" marker-end="url(#arrow)"/>
<text class="ts" x="612" y="384" text-anchor="end" dominant-baseline="central">完了通知</text>
<g class="node c-gray" onclick="sendPrompt('AWX の Organization 分離設計を詳しく')">
<rect x="140" y="404" width="400" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="424" text-anchor="middle" dominant-baseline="central">Ansible Tower / AWX</text>
<text class="ts" x="340" y="442" text-anchor="middle" dominant-baseline="central">ops-prod / ops-test 分離</text>
</g>
<line x1="340" y1="460" x2="340" y2="492" class="arr" marker-end="url(#arrow)"/>
<g class="node c-gray" onclick="sendPrompt('対象環境ごとの認証情報の持たせ方は')">
<rect x="190" y="500" width="300" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="520" text-anchor="middle" dominant-baseline="central">対象環境</text>
<text class="ts" x="340" y="538" text-anchor="middle" dominant-baseline="central">Azure / オンプレ / 他クラウド</text>
</g>
</svg>

> [!note]- Thinking
> The 実行エンジン and Dify Runtime intentionally don't connect directly to AWX in this architecture. Now I'm moving on to diagram 3.

この図で注目していただきたいのは、実行エンジンからも Dify Runtime からも AWX へ直接の線が引かれていないことです。両者は Gateway と双方向にやり取りするだけで、AWX を起動するのは Gateway のみ（図の左側を回り込む線）。これが原則 P3 と P4 の実体で、次の3つを同時に満たします。

- Dify が内部システムに触れないため、ssrf_proxy によるプライベート IP ブロックや 120 秒タイムアウトの影響を受けない
- AWX の起動口が1箇所なので、実行主体が Foundry でも Dify でも監査証跡が同じ場所に集まる
- 実装を Dify から Foundry へ移しても、AWX 起動部分は書き換え不要

AWX の完了通知（右側を回り込む線）も Gateway が受けるため、ServiceNow への書き戻しは常に Gateway が担います。

次に、本番タスクが流れる際のシーケンスです。

**🖼️ Diagram: production_execution_sequence_with_impl_branch**

<svg width="100%" viewBox="0 0 680 682" role="img" xmlns="http://www.w3.org/2000/svg"><title>本番運用タスクの実行シーケンス</title><desc>ServiceNow の実行命令から Gateway の検証と本番ガードを経て、Foundry 実行エンジンまたは Dify Runtime に分岐し、Gateway が AWX を起動して ServiceNow へ書き戻すまでの流れ</desc>
<defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
<g class="node c-coral" onclick="sendPrompt('ServiceNow の承認状態と task_code の対応付けを詳しく')">
<rect x="170" y="30" width="340" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="50" text-anchor="middle" dominant-baseline="central">ServiceNow</text>
<text class="ts" x="340" y="68" text-anchor="middle" dominant-baseline="central">承認後に実行命令を発行</text>
</g>
<line x1="340" y1="86" x2="340" y2="118" class="arr" marker-end="url(#arrow)"/>
<g class="node c-purple" onclick="sendPrompt('スキーマ検証と冪等性チェックの実装を詳しく')">
<rect x="170" y="126" width="340" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="146" text-anchor="middle" dominant-baseline="central">Gateway 受付</text>
<text class="ts" x="340" y="164" text-anchor="middle" dominant-baseline="central">スキーマ検証・冪等性チェック</text>
</g>
<line x1="340" y1="182" x2="340" y2="214" class="arr" marker-end="url(#arrow)"/>
<g class="node c-purple" onclick="sendPrompt('本番ガードのチケット逆引き検証を詳しく')">
<rect x="170" y="222" width="340" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="242" text-anchor="middle" dominant-baseline="central">Gateway 本番ガード</text>
<text class="ts" x="340" y="260" text-anchor="middle" dominant-baseline="central">チケット逆引きで状態を検証</text>
</g>
<path d="M340 278 V298 H195 V310" class="arr" fill="none" marker-end="url(#arrow)"/>
<path d="M340 278 V298 H485 V310" class="arr" fill="none" marker-end="url(#arrow)"/>
<g class="node c-purple" onclick="sendPrompt('Foundry 製ワークフローの実行エンジン側の処理を詳しく')">
<rect x="60" y="318" width="270" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="195" y="338" text-anchor="middle" dominant-baseline="central">実行エンジン</text>
<text class="ts" x="195" y="356" text-anchor="middle" dominant-baseline="central">Foundry 製ワークフロー</text>
</g>
<g class="node c-teal" onclick="sendPrompt('Dify の返却 JSON の出力スキーマ検証を詳しく')">
<rect x="350" y="318" width="270" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="485" y="338" text-anchor="middle" dominant-baseline="central">Dify Runtime</text>
<text class="ts" x="485" y="356" text-anchor="middle" dominant-baseline="central">判断・変換のみ・作用なし</text>
</g>
<path d="M195 374 V394 H340 V406" class="arr" fill="none" marker-end="url(#arrow)"/>
<path d="M485 374 V394 H340 V406" class="arr" fill="none" marker-end="url(#arrow)"/>
<g class="node c-purple" onclick="sendPrompt('extra_vars への相関ID埋め込みの設計を詳しく')">
<rect x="170" y="414" width="340" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="434" text-anchor="middle" dominant-baseline="central">Gateway が AWX 起動</text>
<text class="ts" x="340" y="452" text-anchor="middle" dominant-baseline="central">extra_vars に相関IDを埋込</text>
</g>
<line x1="340" y1="470" x2="340" y2="502" class="arr" marker-end="url(#arrow)"/>
<g class="node c-gray" onclick="sendPrompt('AWX の完了 Webhook と外部イベント待機の実装を詳しく')">
<rect x="170" y="510" width="340" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="530" text-anchor="middle" dominant-baseline="central">AWX が対象環境へ実行</text>
<text class="ts" x="340" y="548" text-anchor="middle" dominant-baseline="central">完了は Webhook で Gateway へ</text>
</g>
<line x1="340" y1="566" x2="340" y2="598" class="arr" marker-end="url(#arrow)"/>
<g class="node c-coral" onclick="sendPrompt('ServiceNow への書き戻し内容と state 遷移を詳しく')">
<rect x="170" y="606" width="340" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="340" y="626" text-anchor="middle" dominant-baseline="central">Gateway が ServiceNow へ書き戻し</text>
<text class="ts" x="340" y="644" text-anchor="middle" dominant-baseline="central">work notes・state 更新</text>
</g>
</svg>

> [!note]- Thinking
> Now I'm laying out the steps in a clean numbered format, followed by a diagram, and then I'll create a compact comparison table showing the differences between branches.

分岐している中段が、シーケンス A（Foundry 製）と B（Dify 製）の唯一の違いです。ServiceNow から見ればどちらも同じ `POST /v1/executions` であり、`task_code` の `impl` 値によって Gateway が呼び分けます。

| 段階 | シーケンス A（Foundry 製） | シーケンス B（Dify 製） |
|---|---|---|
| 呼び出し先 | Function App / AKS の実行エンジン | Dify Service API `POST /v1/workflows/run` |
| 待機方式 | Durable Functions の外部イベント待機 | `streaming` で `max_wait_sec` 以内に完結 |
| 返却物 | AWX 実行指示（パラメータ確定済み） | 構造化 JSON（判断結果・生成テキスト） |
| 検証 | 契約スキーマで検証 | 出力スキーマで検証。不適合なら AWX を起動せず中断 |
| 失敗時の副作用 | なし（起動前に停止） | なし（起動前に停止） |

Dify の失敗が対象環境に届く前に必ず Gateway で止まる点が、Dify を本番動線に載せるうえでの安全余裕になっています。

最後にテスト実行経路です。

**🖼️ Diagram: test_only_execution_path_with_prod_blocked**

<svg width="100%" viewBox="0 0 680 490" role="img" xmlns="http://www.w3.org/2000/svg"><title>テスト実行経路</title><desc>Chainlit/Gradio から Foundry または Dify を経由し Gateway が env=test のみ受理して AWX の ops-test に到達する経路。ops-prod へはスキーマと RBAC で遮断される</desc>
<defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
<g class="node c-gray" onclick="sendPrompt('開発者UIからの認証はどう通す')">
<rect x="80" y="30" width="300" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="230" y="50" text-anchor="middle" dominant-baseline="central">Chainlit / Gradio</text>
<text class="ts" x="230" y="68" text-anchor="middle" dominant-baseline="central">開発者専用UI</text>
</g>
<line x1="230" y1="86" x2="230" y2="118" class="arr" marker-end="url(#arrow)"/>
<g class="node c-gray" onclick="sendPrompt('開発ワークスペースと実行ワークスペースの分離を詳しく')">
<rect x="80" y="126" width="300" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="230" y="146" text-anchor="middle" dominant-baseline="central">Foundry / Dify</text>
<text class="ts" x="230" y="164" text-anchor="middle" dominant-baseline="central">開発ワークスペース</text>
</g>
<line x1="230" y1="182" x2="230" y2="214" class="arr" marker-end="url(#arrow)"/>
<g class="node c-purple" onclick="sendPrompt('env=prod をスキーマで弾く仕組みを詳しく')">
<rect x="80" y="222" width="300" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="230" y="242" text-anchor="middle" dominant-baseline="central">Integration Gateway</text>
<text class="ts" x="230" y="260" text-anchor="middle" dominant-baseline="central">env: test のみ受理</text>
</g>
<line x1="380" y1="250" x2="424" y2="250" stroke="var(--t)" stroke-width="0.5" stroke-dasharray="4 4" fill="none"/>
<text class="ts" x="402" y="236" text-anchor="middle" dominant-baseline="central">遮断</text>
<g class="c-red">
<rect x="430" y="222" width="190" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="525" y="242" text-anchor="middle" dominant-baseline="central">AWX  ops-prod</text>
<text class="ts" x="525" y="260" text-anchor="middle" dominant-baseline="central">到達不能</text>
</g>
<line x1="230" y1="278" x2="230" y2="310" class="arr" marker-end="url(#arrow)"/>
<g class="node c-teal" onclick="sendPrompt('AWX の Organization 単位の権限分離を詳しく')">
<rect x="80" y="318" width="300" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="230" y="338" text-anchor="middle" dominant-baseline="central">AWX  ops-test</text>
<text class="ts" x="230" y="356" text-anchor="middle" dominant-baseline="central">テスト用 Organization</text>
</g>
<line x1="230" y1="374" x2="230" y2="406" class="arr" marker-end="url(#arrow)"/>
<g class="node c-gray" onclick="sendPrompt('テスト環境での実行結果の返し方を詳しく')">
<rect x="80" y="414" width="300" height="56" rx="8" stroke-width="0.5"/>
<text class="th" x="230" y="434" text-anchor="middle" dominant-baseline="central">テスト環境</text>
<text class="ts" x="230" y="452" text-anchor="middle" dominant-baseline="central">ServiceNow への書き戻しなし</text>
</g>
</svg>

右側の遮断は二重で担保します。実行要求スキーマの `allOf` 制約により `channel: devui` では `env: prod` が構造的に指定できず、加えて test 用マネージド ID に `ops-prod` の権限を一切付与しないため、Gateway の実装バグがあっても本番には到達しません。

なお前回提示した `contracts/execution-request.schema.json`、`contracts/task-catalog.yaml`、異常系の扱い、確認結果に応じた変更箇所の対応表は、いずれもコードブロックと表なので表示上の問題はないはずです。そのまま有効な内容としてご参照ください。

次に詰めるとすれば、Gateway の Durable Functions オーケストレーターのコードか、ServiceNow の Flow Designer のステップ構成のいずれかになります。


---

## 🤖 Claude  
*2026-07-24T05:52:18.348391Z*

> [!note]- Thinking
> The user needs the diagrams rendered properly like the SVG from the referenced article. The visualizer tool had a 400 error earlier, so I'll try the read_me function again, but if that fails I can fall back to creating a Mermaid diagram or SVG file in the outputs folder instead.
>
> Let me attempt the visualizer first.

> [!note]- 🔧 Tool call: visualize:read_me
> ```json
> {
>   "modules": [
>     "diagram"
>   ],
>   "platform": "desktop"
> }
> ```

> [!note]- ↩️ Tool result: visualize:read_me _(前回と同一内容のため省略)_

> [!note]- Thinking
> The visualizer is working, so now I need to convert those ASCII diagrams into SVG format. Given the complexity of the architecture, I'll split this into 3-4 separate diagrams with explanatory text between them—keeping each diagram to around 4-5 nodes maximum. My plan covers the overall system structure, the production execution flow for sequences A and B, the test execution path, and the development CI/CD pipeline, though I'm leaning toward condensing this to just 3 diagrams.
>
> For the first diagram showing the overall architecture, I'm thinking through the layout carefully—I need to represent the development plane with its tools (Chainlit/Gradio, Azure AI Foundry, Dify), GitHub, ServiceNow, the integration gateway, the execution engine, AWX, and the environments. That's quite a few components, so I'll need to group related elements together and use containers to show relationships rather than trying to fit everything as individual boxes.
>
> I'm settling on four diagrams total: one for the development plane, one for the control and execution plane, one for the production execution sequence showing the flow between components, and one for the test execution path. This gives a complete picture of both the architecture and how data flows through the system.
>
> For the sequence diagram, I'll use a vertical flowchart approach with actor labels on each step rather than a traditional sequence diagram, since I'm keeping everything in SVG. I'll make sure to highlight the guard step and lay out the coordinates carefully for clarity.
>
> Writing SVG diagrams... Setting up the arrow paths from the top box down to the two boxes below, using L-shaped connectors that branch at a midpoint, then positioning the bottom box to receive arrows from both branches. Trimming the Japanese subtitle to just the core concept, then moving on to the color scheme — keeping the development UI gray, Foundry purple, Dify teal, and GitHub gray. Now I'm laying out the second diagram for the control and execution plane, starting with a grayed-out service order portal box marked as out of scope, followed by ServiceNow in coral as the single source of truth for production tasks.
>
> From there, I'm adding a bidirectional connection to an Integration Gateway in purple that handles API management and durable functions, which then splits into two execution paths — one for the execution engine in purple handling functions and AKS, and another for Dify Runtime in teal focused on decision-making and transformation. The Dify component feeds back to the gateway, and both paths eventually converge toward Ansible Tower for orchestration. Continuing the arrow routing — I need to handle the bidirectional connection between Dify and Gateway using markers on both ends, then work out the path from 実行エンジン down to AWX, and figure out how to route the direct Gateway-to-AWX connection without it crossing through the 実行エンジン box. Gateway also receives a completion webhook from AWX on the right side, routing up from AWX's right edge at x=635 with 25px clearance from Dify's adjusted bounds (x=380 w=230). I'm also adding the connection from AWX down to 環境.
>
> Now I'm refining the arrow paths between the execution engine and Dify—splitting the flow so each gets its own target with proper coordinates, and checking that the Gateway-to-AWX left path doesn't cross any boxes.
>
> The feedback path from AWX back to Gateway curves around the right side, and I'm verifying it clears the Dify box horizontally. For the overall canvas height, I'm calculating that the 環境 box needs about 620 pixels to accommodate all elements with proper spacing, which is taller but acceptable. I still need to figure out the labels for the arrows, keeping them minimal as the guidance suggests. I'm debating whether to add a legend for the color categories—purple for main routes, teal for Dify, coral for ServiceNow, gray for external systems. The guidance suggests including one if colors encode meaning, so I'll add a brief legend line at the bottom of the diagram rather than leaving it to prose explanation.
>
> Now for the execution sequence diagram, I'm laying out the flow as a vertical series of steps showing the interaction between ServiceNow, the Gateway, the execution engine, and external systems like AWX. I'll keep it to around 6-7 steps maximum, with each step labeled by the actor initiating it—starting from ServiceNow sending the execution command through the Gateway's validation checks, the ticket verification guard, workflow execution, job launch to AWX, completion notification, and finally writing results back to ServiceNow.
>
> For the visual layout, I'm calculating dimensions for the state transition boxes—each box needs to accommodate the actor labels and action descriptions, which can be quite long when mixing Latin characters with Japanese text. I'm considering a two-line format where the top line shows the actor transition and the bottom line shows the action, which should fit comfortably within the available width while keeping the overall diagram compact.
>
> For the color scheme, I'm thinking about using distinct colors to highlight different types of steps: ServiceNow interactions in coral, Gateway operations in purple, the guard step in amber, and other steps in gray—though I'm reconsidering whether to stick with just two color ramps to keep the design cleaner.
>
> Now moving to the test execution path diagram, I'm setting up the Chainlit/Gradio component as the starting point. I'm reconsidering the diagram structure—instead of trying to show a blocked arrow, I'll make the Gateway a decision point with two separate paths: one flowing to ops-test and the test environment, and another showing ops-prod as unreachable with a dashed line labeled 「遮断」off to the side.
>
> Now I'm laying out the specific boxes with their positions and labels: starting with the developer tools at the top, then the Foundry/Dify workspace, followed by the Gateway with its dual security layers, then ops-test flowing down to the test environment, and finally ops-prod positioned separately on the right as inaccessible. Adding a blocked indicator...
>
> I'm setting the stroke color to a dark red and calculating the vertical positions for the remaining elements. Now I need to ensure every text element has the proper font class, add the arrow marker definitions, and include accessibility attributes like role and title. Let me start writing out the first SVG diagram.
>
> Writing SVG code... Writing SVG diagram... Shortening the subtitle to just "開発資産の SoT" fits better within the box constraints. The second diagram will use a viewBox of 680 by 620, and I'm setting up the box positions starting with the Port element. Writing SVG diagram... Checking the label placements for collisions with the diagram elements — the direct launch label needs a 10px gap from the vertical line, so I'll shift it slightly left. The completion notification label clears the Dify box vertically. Setting the total height to 615 pixels. For the sequence diagram, I'm laying out 7 boxes with an 80-pixel step between them starting at y=40. I'm refining the step titles to keep them consistent with the actor/action format. For step 3, I'm adjusting the Gateway → ServiceNow validation step to fit the pixel constraints while clearly indicating that mismatched ticket states return a 403 error. Then continuing with steps 4, 5, and 6 covering the workflow execution through Dify, job launch to AWX, and the completion webhook callback. Continuing to define the diagram elements with the test workflow box, integration gateway with dual production safeguards, test organization box, and starting the final element. Finalizing the SVG dimensions — E's bottom edge lands at 492, so I'll set the overall height to 520 for proper spacing. I need to write the response in Japanese with concise prose between the four diagrams, avoiding any explanatory text within the tool calls and keeping mid-sentence formatting clean.

先ほどのアスキーアートを、図として描画し直します。構成図3点とシーケンス図1点に分けて出力します。

まず開発プレーンです。Azure AI Foundry を主開発基盤、Dify を簡易ユースケース限定の補助として配置し、両者の成果物を GitHub に集約します。

**🖼️ Diagram: dev_plane_foundry_dify_github**

<svg width="100%" viewBox="0 0 680 340" role="img"><title>開発プレーンの構成</title><desc>Chainlit と Gradio から Azure AI Foundry と Dify を操作し、開発資産を GitHub で管理する構成図</desc><defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs><g class="c-gray"><rect x="240" y="40" width="200" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="60" text-anchor="middle" dominant-baseline="central">Chainlit / Gradio</text><text class="ts" x="340" y="78" text-anchor="middle" dominant-baseline="central">開発者専用UI</text></g><path class="arr" fill="none" d="M340 96 L340 124 L185 124 L185 148" marker-end="url(#arrow)"/><path class="arr" fill="none" d="M340 96 L340 124 L495 124 L495 148" marker-end="url(#arrow)"/><g class="c-purple"><rect x="60" y="150" width="250" height="56" rx="8" stroke-width="0.5"/><text class="th" x="185" y="170" text-anchor="middle" dominant-baseline="central">Azure AI Foundry</text><text class="ts" x="185" y="188" text-anchor="middle" dominant-baseline="central">主開発基盤</text></g><g class="c-teal"><rect x="370" y="150" width="250" height="56" rx="8" stroke-width="0.5"/><text class="th" x="495" y="170" text-anchor="middle" dominant-baseline="central">Dify</text><text class="ts" x="495" y="188" text-anchor="middle" dominant-baseline="central">従・簡易ユースケース</text></g><path class="arr" fill="none" d="M185 206 L185 234 L260 234 L260 258" marker-end="url(#arrow)"/><path class="arr" fill="none" d="M495 206 L495 234 L420 234 L420 258" marker-end="url(#arrow)"/><g class="c-gray"><rect x="190" y="260" width="300" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="280" text-anchor="middle" dominant-baseline="central">GitHub</text><text class="ts" x="340" y="298" text-anchor="middle" dominant-baseline="central">開発資産の SoT</text></g></svg>

> [!note]- ↩️ Tool result: visualize:show_widget (error)
> ```
> No result received from the Claude Desktop app after waiting 4 minutes. The local MCP server providing this tool may be unresponsive, crashed, or not running. Further calls to this tool are likely to time out the same way; consider using an alternative approach or ask the user to restart their local MCP servers.
> ```

> [!note]- Thinking
> The visualizer timed out, so I'll create an HTML file with all four diagrams embedded together instead—that way they render as a single artifact with proper layout and headings.
>
> I should check the frontend-design skill since I'm building an HTML file, even though it's borderline for a static diagram page. Let me do a quick view to be safe, then build the HTML with self-contained SVG diagrams and CSS that supports light and dark modes using color scheme preferences. Since I can't rely on the visualizer's built-in classes, I'll need to define my own styling from scratch.
>
> Writing HTML structure... Writing SVG styles... I'll add h2 headings between the diagrams to make the file self-contained and clear. Before I start writing, let me quickly check the frontend-design skill documentation to make sure I'm following the right guidelines.

> [!note]- 🔧 Tool call: view
> ```json
> {
>   "description": "Check frontend-design skill before creating the HTML diagram file",
>   "path": "/mnt/skills/public/frontend-design/SKILL.md"
> }
> ```

> [!note]- ↩️ Tool result: view
> ```
> 1	---
>      2	name: frontend-design
>      3	description: Guidance for distinctive, intentional visual design when building new UI or reshaping an existing one. Helps with aesthetic direction, typography, and making choices that don't read as templated defaults.
>      4	license: Complete terms in LICENSE.txt
>      5	---
>      6	
>      7	# Frontend Design
>      8	
>      9	Approach this as the design lead at a small studio known for giving every client a visual identity that could not be mistaken for anyone else's. This client has already rejected proposals that felt templated, and is paying for a distinctive point of view: make deliberate, opinionated choices about palette, typography, and layout that are specific to this brief, and take one real aesthetic risk you can justify.
>     10	
>     11	## Ground it in the subject
>     12	
>     13	If the brief does not pin down what the product or subject is, pin it yourself before designing: name one concrete subject, its audience, and the page's single job, and state your choice. If there's any information in your memory about the human's preferences, context about what they're building, or designs you've made before – use that as a hint. The subject's own world, its materials, instruments, artifacts, and vernacular, is where distinctive choices come from. Build with the brief's real content and subject matter throughout.
>     14	
>     15	## Design principles
>     16	
>     17	For web designs, the hero is a thesis. Open with the most characteristic thing in the subject's world, in whatever form makes sense for it: a headline, an image, an animation, a live demo, an interactive moment. Be deliberate with your choice: a big number with a small label, supporting stats, and a gradient accent is the template answer, only use if that's truly the best option.
>     18	
>     19	Typography carries the personality of the page. Pair the display and body faces deliberately, not the same families you would reach for on any other project, and set a clear type scale with intentional weights, widths, and spacing. Make the type treatment itself a memorable part of the design, not a neutral delivery vehicle for the content.
>     20	
>     21	Structure is information. Structural devices, numbering, eyebrows, dividers, labels, should encode something true about the content, not decorate it. Many generic designs use numbered markers (01 / 02 / 03), but that's only appropriate if the content actually is a sequence - like a real process or a typed timeline where order carries information the reader needs. Question if choices like numbered markers actually make sense before incorporating them.
>     22	
>     23	Leverage motion deliberately. Think about where and if animation can serve the subject: a page-load sequence, a scroll-triggered reveal, hover micro-interactions, ambient atmosphere. An orchestrated moment usually lands harder than scattered effects; choose what the direction calls for. However, sometimes less is more, and extra animation contributes to the feeling that the design is AI-generated.
>     24	
>     25	Match complexity to the vision. Maximalist directions need elaborate execution; minimal directions need precision in spacing, type, and detail. Elegance is executing the chosen vision well.
>     26	
>     27	Consider written content carefully. Often a design brief may not contain real content, and it's up to you to come up with copy. Copy can make a design feel as templated as the design itself. See the below section on writing for more guidance.
>     28	
>     29	## Process: brainstorm, explore, plan, critique, build, critique again
>     30	
>     31	For calibration: AI-generated design right now clusters around three looks: (1) a warm cream background (near #F4F1EA) with a high-contrast serif display and a terracotta or warm-clay accent (often near #D97757 — Anthropic's own Claude-interaction accent, so on a user's brief it reads as a tell); (2) a near-black background with a single bright acid-green or vermilion accent; (3) a broadsheet-style layout with hairline rules, zero border-radius, and dense newspaper-like columns. All three are legitimate for some briefs, but they are defaults rather than choices, and they appear regardless of subject. Where the brief pins down a visual direction, follow it exactly — the brief's own words always win, including when it asks for one of these looks. Where it leaves an axis free, don't spend that freedom on one of these defaults. Just like a human designer who's hired, there's often a careful balance between doing what you're good at and taking each project as a chance to experiment and learn.
>     32	
>     33	Work in two passes. First, brainstorm a short design plan based on the human's design brief: create a compact token system with color, type, layout, and signature. Color: describe the palette as 4–6 named hex values. Type: the typefaces for 2+ roles (a characterful display face that's used with restraint, a complementary body face, and a utility face for captions or data if needed). Layout: a layout concept, using one-sentence prose descriptions and ASCII wireframes to ideate and compare. Signature: the single unique element this page will be remembered by that embodies the brief in an appropriate way.
>     34	
>     35	Then review that plan against the brief before building: if any part of it reads like the generic default you would produce for any similar page (work through a similar prompt to see if you arrive somewhere similar) rather than a choice made for this specific brief — revise that part, say what you changed and why. Only after you've confirmed the relative uniqueness of your design plan should you start to write the code, following the revised plan exactly and deriving every color and type decision from it.
>     36	
>     37	When writing the code, be careful of structuring your CSS selector specificities. It's easy to generate CSS classes that cancel each other out (especially with a type-based selector like .section and a element-based selector like .cta). This can happen often with paddings/margins between sections.
>     38	
>     39	Try to do a lot of this planning and iteration in your thinking, and only show ideas to the user when you have higher confidence it'll delight them.
>     40	
>     41	## Restraint and self-critique
>     42	
>     43	Spend your boldness in one place. Let the signature element be the one memorable thing, keep everything around it quiet and disciplined, and cut any decoration that does not serve the brief. Not taking a risk can be a risk itself! Build to a quality floor without announcing it: responsive down to mobile, visible keyboard focus, reduced motion respected. Critique your own work as you build, taking screenshots if your environment supports it – a picture is worth 1000 tokens. Consider Chanel's advice: before leaving the house, take a look in the mirror and remove one accessory. Human creators have memory and always try to do something new, so if you have a space to quickly jot down notes about what you've tried, it can help you in future passes.
>     44	
>     45	## More on writing in design
>     46	
>     47	Words appear in a design for one reason: to make it easier to understand, and therefore easier to use. They are design material, not decoration. Bring the same intentionality to copy that you would bring to spacing and color. Before writing anything, ask what the design needs to say, and how it can best be said to help the person navigate the experience.
>     48	
>     49	Write from the end user's side of the screen. Name things by what people control and recognize, never by how the system is built. A person manages notifications, not webhook config. Describe what something does in plain terms rather than selling it. Being specific is always better than being clever.
>     50	
>     51	Use active voice as default. A control should say exactly what happens when it's used: "Save changes," not "Submit." An action keeps the same name through the whole flow, so the button that says "Publish" produces a toast that says "Published." The vocabulary of an interface is the signposting for someone navigating the product. Cohesion and consistency are how people learn their way around.
>     52	
>     53	Treat failure and emptiness as moments for direction, not mood. Explain what went wrong and how to fix it, in the interface's voice rather than a person's. Errors don't apologize, and they are never vague about what happened. An empty screen is an invitation to act.
>     54	
>     55	Keep the register conversational and tuned: plain verbs, sentence case, no filler, with tone matched to the brand and the audience. Let each element do exactly one job. A label labels, an example demonstrates, and nothing quietly does double duty.
>     56
> ```

> [!note]- Thinking
> I'm creating an HTML file with four technical architecture diagrams, keeping the design clean and functional without unnecessary embellishment.

Visualizer が応答しないため、同等の SVG 図をファイルとして出力します。

> [!note]- 🔧 Tool call: create_file
> ```json
> {
>   "description": "4つの構成図・シーケンス図をSVGで描画したHTMLファイルを作成",
>   "path": "/mnt/user-data/outputs/architecture-diagrams.html",
>   "file_text": "<meta charset=\"utf-8\">\n<meta name=\"viewport\" content=\"width=device-width,initial-scale=1\">\n<title>現段階版 構成図・シーケンス図</title>\n<style>\n:root{\n  --tp:#26251f; --ts:#5f5e5a; --line:#888780; --page:#faf9f5; --card:#ffffff; --edge:#e5e3db;\n}\n@media (prefers-color-scheme: dark){\n  :root{ --tp:#e8e6df; --ts:#a3a199; --line:#7b7a74; --page:#1c1c1a; --card:#242422; --edge:#383834; }\n}\nbody{margin:0;padding:32px 20px 56px;background:var(--page);color:var(--tp);\n  font-family:\"Helvetica Neue\",Arial,\"Hiragino Kaku Gothic ProN\",\"Yu Gothic\",Meiryo,sans-serif;\n  font-size:16px;line-height:1.7;-webkit-font-smoothing:antialiased}\nmain{max-width:760px;margin:0 auto}\nh1{font-size:22px;font-weight:500;margin:0 0 4px}\nh2{font-size:18px;font-weight:500;margin:0 0 8px}\np.lead{color:var(--ts);font-size:14px;margin:0 0 32px}\nsection{background:var(--card);border:1px solid var(--edge);border-radius:12px;padding:24px;margin-bottom:24px}\nsection p{color:var(--ts);font-size:14px;margin:0 0 16px}\nsvg{display:block;width:100%;height:auto}\n.legend{display:flex;flex-wrap:wrap;gap:16px;margin-top:16px;font-size:12px;color:var(--ts)}\n.legend span{display:flex;align-items:center;gap:6px}\n.sw{width:12px;height:12px;border-radius:3px;border:1px solid}\ntext.t{font-size:14px;font-weight:400}\ntext.th{font-size:14px;font-weight:500}\ntext.ts{font-size:12px;font-weight:400}\ntext{font-family:inherit}\n.n>rect{fill:var(--f);stroke:var(--s);stroke-width:.5}\n.n>text.th{fill:var(--tt)}\n.n>text.ts{fill:var(--st)}\n.arr{stroke:var(--line);stroke-width:1.5;fill:none;stroke-linecap:round}\n.c-purple{--f:#EEEDFE;--s:#534AB7;--tt:#3C3489;--st:#534AB7;--sw:#7F77DD}\n.c-teal{--f:#E1F5EE;--s:#0F6E56;--tt:#085041;--st:#0F6E56;--sw:#1D9E75}\n.c-coral{--f:#FAECE7;--s:#993C1D;--tt:#712B13;--st:#993C1D;--sw:#D85A30}\n.c-gray{--f:#F1EFE8;--s:#5F5E5A;--tt:#2C2C2A;--st:#5F5E5A;--sw:#888780}\n.c-amber{--f:#FAEEDA;--s:#854F0B;--tt:#633806;--st:#854F0B;--sw:#BA7517}\n.c-red{--f:#FCEBEB;--s:#A32D2D;--tt:#791F1F;--st:#A32D2D;--sw:#E24B4A}\n@media (prefers-color-scheme: dark){\n  .c-purple{--f:#3C3489;--s:#AFA9EC;--tt:#CECBF6;--st:#AFA9EC}\n  .c-teal{--f:#085041;--s:#5DCAA5;--tt:#9FE1CB;--st:#5DCAA5}\n  .c-coral{--f:#712B13;--s:#F0997B;--tt:#F5C4B3;--st:#F0997B}\n  .c-gray{--f:#444441;--s:#B4B2A9;--tt:#D3D1C7;--st:#B4B2A9}\n  .c-amber{--f:#633806;--s:#EF9F27;--tt:#FAC775;--st:#EF9F27}\n  .c-red{--f:#791F1F;--s:#F09595;--tt:#F7C1C1;--st:#F09595}\n}\n</style>\n<main>\n<h1>現段階版の構成図・シーケンス図</h1>\n<p class=\"lead\">Azure AI Foundry を主開発基盤、Dify を簡易ユースケース限定の補助として位置づけた構成</p>\n\n<section>\n<h2>図1　開発プレーン</h2>\n<p>本番タスクを起票・発行しない領域。開発成果物はすべて GitHub に集約する。</p>\n<svg viewBox=\"0 0 680 340\" role=\"img\"><title>開発プレーンの構成</title><desc>Chainlit と Gradio から Azure AI Foundry と Dify を操作し、開発資産を GitHub で管理する構成</desc>\n<defs><marker id=\"a1\" viewBox=\"0 0 10 10\" refX=\"8\" refY=\"5\" markerWidth=\"6\" markerHeight=\"6\" orient=\"auto-start-reverse\"><path d=\"M2 1L8 5L2 9\" fill=\"none\" stroke=\"context-stroke\" stroke-width=\"1.5\" stroke-linecap=\"round\" stroke-linejoin=\"round\"/></marker></defs>\n<g class=\"n c-gray\"><rect x=\"240\" y=\"40\" width=\"200\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"60\" text-anchor=\"middle\" dominant-baseline=\"central\">Chainlit / Gradio</text><text class=\"ts\" x=\"340\" y=\"78\" text-anchor=\"middle\" dominant-baseline=\"central\">開発者専用UI</text></g>\n<path class=\"arr\" d=\"M340 96 L340 124 L185 124 L185 148\" marker-end=\"url(#a1)\"/>\n<path class=\"arr\" d=\"M340 96 L340 124 L495 124 L495 148\" marker-end=\"url(#a1)\"/>\n<g class=\"n c-purple\"><rect x=\"60\" y=\"150\" width=\"250\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"185\" y=\"170\" text-anchor=\"middle\" dominant-baseline=\"central\">Azure AI Foundry</text><text class=\"ts\" x=\"185\" y=\"188\" text-anchor=\"middle\" dominant-baseline=\"central\">主開発基盤</text></g>\n<g class=\"n c-teal\"><rect x=\"370\" y=\"150\" width=\"250\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"495\" y=\"170\" text-anchor=\"middle\" dominant-baseline=\"central\">Dify</text><text class=\"ts\" x=\"495\" y=\"188\" text-anchor=\"middle\" dominant-baseline=\"central\">従・簡易ユースケース</text></g>\n<path class=\"arr\" d=\"M185 206 L185 234 L262 234 L262 258\" marker-end=\"url(#a1)\"/>\n<path class=\"arr\" d=\"M495 206 L495 234 L418 234 L418 258\" marker-end=\"url(#a1)\"/>\n<g class=\"n c-gray\"><rect x=\"190\" y=\"260\" width=\"300\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"280\" text-anchor=\"middle\" dominant-baseline=\"central\">GitHub</text><text class=\"ts\" x=\"340\" y=\"298\" text-anchor=\"middle\" dominant-baseline=\"central\">開発資産の SoT</text></g>\n</svg>\n</section>\n\n<section>\n<h2>図2　統制・実行プレーン</h2>\n<p>本番運用／デプロイタスクの動線。ServiceNow が唯一の起点となり、AWX への起動は Gateway に一本化する。</p>\n<svg viewBox=\"0 0 680 615\" role=\"img\"><title>統制・実行プレーンの構成</title><desc>ポータルから ServiceNow を経由し、Integration Gateway が実行エンジンと Dify Runtime を制御して AWX を起動する構成</desc>\n<defs><marker id=\"a2\" viewBox=\"0 0 10 10\" refX=\"8\" refY=\"5\" markerWidth=\"6\" markerHeight=\"6\" orient=\"auto-start-reverse\"><path d=\"M2 1L8 5L2 9\" fill=\"none\" stroke=\"context-stroke\" stroke-width=\"1.5\" stroke-linecap=\"round\" stroke-linejoin=\"round\"/></marker></defs>\n<g class=\"n c-gray\"><rect x=\"190\" y=\"40\" width=\"300\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"60\" text-anchor=\"middle\" dominant-baseline=\"central\">サービスオーダー用ポータル</text><text class=\"ts\" x=\"340\" y=\"78\" text-anchor=\"middle\" dominant-baseline=\"central\">本スコープ外</text></g>\n<path class=\"arr\" d=\"M340 96 L340 132\" marker-end=\"url(#a2)\"/>\n<g class=\"n c-coral\"><rect x=\"190\" y=\"136\" width=\"300\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"156\" text-anchor=\"middle\" dominant-baseline=\"central\">ServiceNow</text><text class=\"ts\" x=\"340\" y=\"174\" text-anchor=\"middle\" dominant-baseline=\"central\">本番タスクの唯一の起点</text></g>\n<path class=\"arr\" d=\"M340 196 L340 228\" marker-start=\"url(#a2)\" marker-end=\"url(#a2)\"/>\n<g class=\"n c-purple\"><rect x=\"150\" y=\"232\" width=\"380\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"252\" text-anchor=\"middle\" dominant-baseline=\"central\">Integration Gateway</text><text class=\"ts\" x=\"340\" y=\"270\" text-anchor=\"middle\" dominant-baseline=\"central\">APIM + Durable Functions</text></g>\n<path class=\"arr\" d=\"M340 288 L340 312 L190 312 L190 336\" marker-end=\"url(#a2)\"/>\n<path class=\"arr\" d=\"M340 288 L340 312 L495 312 L495 336\" marker-start=\"url(#a2)\" marker-end=\"url(#a2)\"/>\n<g class=\"n c-purple\"><rect x=\"70\" y=\"340\" width=\"240\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"190\" y=\"360\" text-anchor=\"middle\" dominant-baseline=\"central\">実行エンジン</text><text class=\"ts\" x=\"190\" y=\"378\" text-anchor=\"middle\" dominant-baseline=\"central\">Functions / AKS</text></g>\n<g class=\"n c-teal\"><rect x=\"380\" y=\"340\" width=\"230\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"495\" y=\"360\" text-anchor=\"middle\" dominant-baseline=\"central\">Dify Runtime</text><text class=\"ts\" x=\"495\" y=\"378\" text-anchor=\"middle\" dominant-baseline=\"central\">判断・変換のみ</text></g>\n<path class=\"arr\" d=\"M190 396 L190 422 L262 422 L262 444\" marker-end=\"url(#a2)\"/>\n<path class=\"arr\" d=\"M150 260 L45 260 L45 476 L146 476\" marker-end=\"url(#a2)\"/>\n<text class=\"ts\" x=\"52\" y=\"436\" fill=\"var(--ts)\">直接起動</text>\n<path class=\"arr\" d=\"M530 476 L635 476 L635 260 L534 260\" marker-end=\"url(#a2)\"/>\n<text class=\"ts\" x=\"625\" y=\"436\" text-anchor=\"end\" fill=\"var(--ts)\">完了通知</text>\n<g class=\"n c-gray\"><rect x=\"150\" y=\"448\" width=\"380\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"468\" text-anchor=\"middle\" dominant-baseline=\"central\">Ansible Tower / AWX</text><text class=\"ts\" x=\"340\" y=\"486\" text-anchor=\"middle\" dominant-baseline=\"central\">ops-prod / ops-test を分離</text></g>\n<path class=\"arr\" d=\"M340 504 L340 540\" marker-end=\"url(#a2)\"/>\n<g class=\"n c-gray\"><rect x=\"230\" y=\"544\" width=\"220\" height=\"44\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"566\" text-anchor=\"middle\" dominant-baseline=\"central\">本番環境</text></g>\n</svg>\n<div class=\"legend\">\n<span><i class=\"sw c-purple\" style=\"background:var(--f);border-color:var(--s)\"></i>主経路（Foundry 系）</span>\n<span><i class=\"sw c-teal\" style=\"background:var(--f);border-color:var(--s)\"></i>Dify</span>\n<span><i class=\"sw c-coral\" style=\"background:var(--f);border-color:var(--s)\"></i>統制（ServiceNow）</span>\n<span><i class=\"sw c-gray\" style=\"background:var(--f);border-color:var(--s)\"></i>共通・外部</span>\n</div>\n</section>\n\n<section>\n<h2>図3　本番実行シーケンス</h2>\n<p>手順4で Dify を呼ぶか実行エンジンを呼ぶかが分かれるが、ServiceNow から見た手順は変わらない。</p>\n<svg viewBox=\"0 0 680 600\" role=\"img\"><title>本番運用タスクの実行シーケンス</title><desc>ServiceNow からの実行命令を Gateway が検証し、実行後に結果を書き戻すまでの7手順</desc>\n<defs><marker id=\"a3\" viewBox=\"0 0 10 10\" refX=\"8\" refY=\"5\" markerWidth=\"6\" markerHeight=\"6\" orient=\"auto-start-reverse\"><path d=\"M2 1L8 5L2 9\" fill=\"none\" stroke=\"context-stroke\" stroke-width=\"1.5\" stroke-linecap=\"round\" stroke-linejoin=\"round\"/></marker></defs>\n<g class=\"n c-gray\"><rect x=\"140\" y=\"40\" width=\"400\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"60\" text-anchor=\"middle\" dominant-baseline=\"central\">ServiceNow → Gateway</text><text class=\"ts\" x=\"340\" y=\"78\" text-anchor=\"middle\" dominant-baseline=\"central\">実行命令 POST /v1/executions</text></g>\n<path class=\"arr\" d=\"M340 96 L340 116\" marker-end=\"url(#a3)\"/>\n<g class=\"n c-gray\"><rect x=\"140\" y=\"120\" width=\"400\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"140\" text-anchor=\"middle\" dominant-baseline=\"central\">Gateway</text><text class=\"ts\" x=\"340\" y=\"158\" text-anchor=\"middle\" dominant-baseline=\"central\">スキーマ検証・冪等チェック</text></g>\n<path class=\"arr\" d=\"M340 176 L340 196\" marker-end=\"url(#a3)\"/>\n<g class=\"n c-amber\"><rect x=\"140\" y=\"200\" width=\"400\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"220\" text-anchor=\"middle\" dominant-baseline=\"central\">Gateway → ServiceNow（本番ガード）</text><text class=\"ts\" x=\"340\" y=\"238\" text-anchor=\"middle\" dominant-baseline=\"central\">チケットを逆引き検証し不一致は 403</text></g>\n<path class=\"arr\" d=\"M340 256 L340 276\" marker-end=\"url(#a3)\"/>\n<g class=\"n c-gray\"><rect x=\"140\" y=\"280\" width=\"400\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"300\" text-anchor=\"middle\" dominant-baseline=\"central\">Gateway → 実行エンジン / Dify</text><text class=\"ts\" x=\"340\" y=\"318\" text-anchor=\"middle\" dominant-baseline=\"central\">ワークフロー実行・パラメータ確定</text></g>\n<path class=\"arr\" d=\"M340 336 L340 356\" marker-end=\"url(#a3)\"/>\n<g class=\"n c-gray\"><rect x=\"140\" y=\"360\" width=\"400\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"380\" text-anchor=\"middle\" dominant-baseline=\"central\">Gateway → AWX</text><text class=\"ts\" x=\"340\" y=\"398\" text-anchor=\"middle\" dominant-baseline=\"central\">ジョブ起動（作用は Gateway が実施）</text></g>\n<path class=\"arr\" d=\"M340 416 L340 436\" marker-end=\"url(#a3)\"/>\n<g class=\"n c-gray\"><rect x=\"140\" y=\"440\" width=\"400\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"460\" text-anchor=\"middle\" dominant-baseline=\"central\">AWX → Gateway</text><text class=\"ts\" x=\"340\" y=\"478\" text-anchor=\"middle\" dominant-baseline=\"central\">完了 Webhook を外部イベントで受信</text></g>\n<path class=\"arr\" d=\"M340 496 L340 516\" marker-end=\"url(#a3)\"/>\n<g class=\"n c-gray\"><rect x=\"140\" y=\"520\" width=\"400\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"340\" y=\"540\" text-anchor=\"middle\" dominant-baseline=\"central\">Gateway → ServiceNow</text><text class=\"ts\" x=\"340\" y=\"558\" text-anchor=\"middle\" dominant-baseline=\"central\">work notes 記録・state 遷移</text></g>\n</svg>\n</section>\n\n<section>\n<h2>図4　テスト実行経路</h2>\n<p>スキーマ制約と RBAC の二重で本番到達を封じる。開発プレーンからは ops-test にしか到達できない。</p>\n<svg viewBox=\"0 0 680 520\" role=\"img\"><title>テスト実行経路と本番ガード</title><desc>Chainlit と Gradio からの実行はテスト用 Organization にのみ到達し、本番用 Organization は遮断される構成</desc>\n<defs><marker id=\"a4\" viewBox=\"0 0 10 10\" refX=\"8\" refY=\"5\" markerWidth=\"6\" markerHeight=\"6\" orient=\"auto-start-reverse\"><path d=\"M2 1L8 5L2 9\" fill=\"none\" stroke=\"context-stroke\" stroke-width=\"1.5\" stroke-linecap=\"round\" stroke-linejoin=\"round\"/></marker></defs>\n<g class=\"n c-gray\"><rect x=\"120\" y=\"40\" width=\"260\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"250\" y=\"60\" text-anchor=\"middle\" dominant-baseline=\"central\">Chainlit / Gradio</text><text class=\"ts\" x=\"250\" y=\"78\" text-anchor=\"middle\" dominant-baseline=\"central\">env=test 固定</text></g>\n<path class=\"arr\" d=\"M250 96 L250 132\" marker-end=\"url(#a4)\"/>\n<g class=\"n c-purple\"><rect x=\"120\" y=\"136\" width=\"260\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"250\" y=\"156\" text-anchor=\"middle\" dominant-baseline=\"central\">Foundry / Dify 開発WS</text><text class=\"ts\" x=\"250\" y=\"174\" text-anchor=\"middle\" dominant-baseline=\"central\">テスト用ワークフロー</text></g>\n<path class=\"arr\" d=\"M250 192 L250 228\" marker-end=\"url(#a4)\"/>\n<g class=\"n c-purple\"><rect x=\"90\" y=\"232\" width=\"320\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"250\" y=\"252\" text-anchor=\"middle\" dominant-baseline=\"central\">Integration Gateway</text><text class=\"ts\" x=\"250\" y=\"270\" text-anchor=\"middle\" dominant-baseline=\"central\">二重の本番ガード</text></g>\n<path class=\"arr\" d=\"M250 288 L250 348\" marker-end=\"url(#a4)\"/>\n<path d=\"M410 260 L535 260 L535 342\" fill=\"none\" stroke=\"#A32D2D\" stroke-width=\"1\" stroke-dasharray=\"4 4\"/>\n<path d=\"M522 344 L548 344\" stroke=\"#A32D2D\" stroke-width=\"2\" stroke-linecap=\"round\"/>\n<text class=\"ts\" x=\"546\" y=\"300\" fill=\"#A32D2D\">遮断</text>\n<g class=\"n c-gray\"><rect x=\"120\" y=\"352\" width=\"260\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"250\" y=\"372\" text-anchor=\"middle\" dominant-baseline=\"central\">AWX ops-test</text><text class=\"ts\" x=\"250\" y=\"390\" text-anchor=\"middle\" dominant-baseline=\"central\">テスト用 Organization</text></g>\n<g class=\"n c-red\"><rect x=\"440\" y=\"352\" width=\"190\" height=\"56\" rx=\"8\"/><text class=\"th\" x=\"535\" y=\"372\" text-anchor=\"middle\" dominant-baseline=\"central\">AWX ops-prod</text><text class=\"ts\" x=\"535\" y=\"390\" text-anchor=\"middle\" dominant-baseline=\"central\">到達不能</text></g>\n<path class=\"arr\" d=\"M250 408 L250 444\" marker-end=\"url(#a4)\"/>\n<g class=\"n c-gray\"><rect x=\"170\" y=\"448\" width=\"160\" height=\"44\" rx=\"8\"/><text class=\"th\" x=\"250\" y=\"470\" text-anchor=\"middle\" dominant-baseline=\"central\">テスト環境</text></g>\n</svg>\n</section>\n</main>\n"
> }
> ```

> [!note]- ↩️ Tool result: create_file
> ```
> File created successfully: /mnt/user-data/outputs/architecture-diagrams.html
> ```

> [!note]- 🔧 Tool call: present_files
> ```json
> {
>   "filepaths": [
>     "/mnt/user-data/outputs/architecture-diagrams.html"
>   ]
> }
> ```

4点の図を SVG で描画しました。等幅フォント依存のアスキーアートと違い、環境によらず崩れずに表示されます。

各図の要点は以下の通りです。

**図1（開発プレーン）** — Chainlit/Gradio は Foundry と Dify の両方を操作しますが、この層から本番タスクは出ません。成果物は Foundry がコード、Dify が DSL(YAML) という形で GitHub に集約されます。

**図2（統制・実行プレーン）** — 左側の「直接起動」が `impl: awx-direct`、右側の「完了通知」が AWX の Notification Webhook です。Dify Runtime だけが AWX に線を持たず、Gateway との双方向矢印で終わっている点が P3（Dify に作用させない）の表現です。

**図3（本番実行シーケンス）** — 琥珀色の手順3が唯一の本番ガードです。ここを通らない限り AWX には到達しません。手順4で Dify を経由する場合も、返却 JSON の検証に失敗すれば手順5に進まないため、副作用が出る前に止まります。

**図4（テスト実行経路）** — 赤の破線が「Gateway のスキーマ制約と RBAC で遮断される経路」です。図2と図4で AWX が別の Organization として描かれているのが分離の実体になります。

ファイルはブラウザで開くほか、各 SVG を切り出して設計書に貼り込むこともできます。図の粒度や表記（日本語／英語、ボックスの分割単位など）を調整したい場合はお知らせください。


---

## 🧑 User  
*2026-07-24T06:02:34.950410Z*

Architecture diagrams をこのチャットで回答として表示してください。


---

## 🤖 Claude  
*2026-07-24T06:03:55.762616Z*

> [!note]- Thinking
> The visualizer tool timed out before, but let me try running it again—it might succeed this time. If it doesn't work, I'll need to find an alternative way to display the diagrams inline in the chat rather than as a file.

**🖼️ Diagram: dev_plane_foundry_dify_github**

<svg width="100%" viewBox="0 0 680 340" role="img"><title>開発プレーンの構成</title><desc>Chainlit と Gradio から Azure AI Foundry と Dify を操作し、開発資産を GitHub で管理する構成</desc><defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs><g class="c-gray"><rect x="240" y="40" width="200" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="60" text-anchor="middle" dominant-baseline="central">Chainlit / Gradio</text><text class="ts" x="340" y="78" text-anchor="middle" dominant-baseline="central">開発者専用UI</text></g><path class="arr" fill="none" d="M340 96 L340 124 L185 124 L185 148" marker-end="url(#arrow)"/><path class="arr" fill="none" d="M340 96 L340 124 L495 124 L495 148" marker-end="url(#arrow)"/><g class="c-purple"><rect x="60" y="150" width="250" height="56" rx="8" stroke-width="0.5"/><text class="th" x="185" y="170" text-anchor="middle" dominant-baseline="central">Azure AI Foundry</text><text class="ts" x="185" y="188" text-anchor="middle" dominant-baseline="central">主開発基盤</text></g><g class="c-teal"><rect x="370" y="150" width="250" height="56" rx="8" stroke-width="0.5"/><text class="th" x="495" y="170" text-anchor="middle" dominant-baseline="central">Dify</text><text class="ts" x="495" y="188" text-anchor="middle" dominant-baseline="central">従・簡易ユースケース</text></g><path class="arr" fill="none" d="M185 206 L185 234 L262 234 L262 258" marker-end="url(#arrow)"/><path class="arr" fill="none" d="M495 206 L495 234 L418 234 L418 258" marker-end="url(#arrow)"/><g class="c-gray"><rect x="190" y="260" width="300" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="280" text-anchor="middle" dominant-baseline="central">GitHub</text><text class="ts" x="340" y="298" text-anchor="middle" dominant-baseline="central">開発資産の SoT</text></g></svg>

> [!note]- Thinking
> Now I'll work through the remaining three diagrams with explanatory text between them.

図1は開発プレーンです。この層から本番タスクは出ず、成果物は Foundry がコード、Dify が DSL(YAML) という形で GitHub に集約されます。

次が統制・実行プレーンです。ServiceNow が唯一の起点となり、AWX への起動口は Gateway に一本化されています。

**🖼️ Diagram: control_execution_plane_servicenow_gateway_awx**

<svg width="100%" viewBox="0 0 680 615" role="img"><title>統制・実行プレーンの構成</title><desc>ポータルから ServiceNow を経由し、Integration Gateway が実行エンジンと Dify Runtime を制御して AWX を起動する構成</desc><defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs><g class="c-gray"><rect x="190" y="40" width="300" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="60" text-anchor="middle" dominant-baseline="central">サービスオーダー用ポータル</text><text class="ts" x="340" y="78" text-anchor="middle" dominant-baseline="central">本スコープ外</text></g><path class="arr" fill="none" d="M340 96 L340 132" marker-end="url(#arrow)"/><g class="c-coral"><rect x="190" y="136" width="300" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="156" text-anchor="middle" dominant-baseline="central">ServiceNow</text><text class="ts" x="340" y="174" text-anchor="middle" dominant-baseline="central">本番タスクの唯一の起点</text></g><path class="arr" fill="none" d="M340 196 L340 228" marker-start="url(#arrow)" marker-end="url(#arrow)"/><g class="c-purple"><rect x="150" y="232" width="380" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="252" text-anchor="middle" dominant-baseline="central">Integration Gateway</text><text class="ts" x="340" y="270" text-anchor="middle" dominant-baseline="central">APIM + Durable Functions</text></g><path class="arr" fill="none" d="M340 288 L340 312 L190 312 L190 336" marker-end="url(#arrow)"/><path class="arr" fill="none" d="M340 288 L340 312 L495 312 L495 336" marker-start="url(#arrow)" marker-end="url(#arrow)"/><g class="c-purple"><rect x="70" y="340" width="240" height="56" rx="8" stroke-width="0.5"/><text class="th" x="190" y="360" text-anchor="middle" dominant-baseline="central">実行エンジン</text><text class="ts" x="190" y="378" text-anchor="middle" dominant-baseline="central">Functions / AKS</text></g><g class="c-teal"><rect x="380" y="340" width="230" height="56" rx="8" stroke-width="0.5"/><text class="th" x="495" y="360" text-anchor="middle" dominant-baseline="central">Dify Runtime</text><text class="ts" x="495" y="378" text-anchor="middle" dominant-baseline="central">判断・変換のみ</text></g><path class="arr" fill="none" d="M190 396 L190 422 L262 422 L262 444" marker-end="url(#arrow)"/><path class="arr" fill="none" d="M150 260 L45 260 L45 476 L146 476" marker-end="url(#arrow)"/><text class="ts" x="52" y="436">直接起動</text><path class="arr" fill="none" d="M530 476 L635 476 L635 260 L534 260" marker-end="url(#arrow)"/><text class="ts" x="625" y="436" text-anchor="end">完了通知</text><g class="c-gray"><rect x="150" y="448" width="380" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="468" text-anchor="middle" dominant-baseline="central">Ansible Tower / AWX</text><text class="ts" x="340" y="486" text-anchor="middle" dominant-baseline="central">ops-prod / ops-test を分離</text></g><path class="arr" fill="none" d="M340 504 L340 540" marker-end="url(#arrow)"/><g class="c-gray"><rect x="230" y="544" width="220" height="44" rx="8" stroke-width="0.5"/><text class="th" x="340" y="566" text-anchor="middle" dominant-baseline="central">本番環境</text></g></svg>

左端の「直接起動」が `impl: awx-direct`、右端の「完了通知」が AWX の Notification Webhook です。Dify Runtime だけが AWX に線を持たず Gateway との双方向で終わっている点が、原則 P3（Dify に作用させない）の表現になっています。紫が Foundry 系の主経路、青緑が Dify、朱色が統制、灰色が共通・外部です。

続いて、その本番動線を時系列に展開したシーケンスです。

**🖼️ Diagram: prod_execution_sequence_servicenow_to_awx**

<svg width="100%" viewBox="0 0 680 600" role="img"><title>本番運用タスクの実行シーケンス</title><desc>ServiceNow からの実行命令を Gateway が検証し、AWX で実行して結果を書き戻すまでの7手順</desc><defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs><g class="c-gray"><rect x="140" y="40" width="400" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="60" text-anchor="middle" dominant-baseline="central">ServiceNow → Gateway</text><text class="ts" x="340" y="78" text-anchor="middle" dominant-baseline="central">実行命令 POST /v1/executions</text></g><path class="arr" fill="none" d="M340 96 L340 116" marker-end="url(#arrow)"/><g class="c-gray"><rect x="140" y="120" width="400" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="140" text-anchor="middle" dominant-baseline="central">Gateway</text><text class="ts" x="340" y="158" text-anchor="middle" dominant-baseline="central">スキーマ検証・冪等チェック</text></g><path class="arr" fill="none" d="M340 176 L340 196" marker-end="url(#arrow)"/><g class="c-amber"><rect x="140" y="200" width="400" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="220" text-anchor="middle" dominant-baseline="central">Gateway → ServiceNow（本番ガード）</text><text class="ts" x="340" y="238" text-anchor="middle" dominant-baseline="central">チケットを逆引き検証し不一致は 403</text></g><path class="arr" fill="none" d="M340 256 L340 276" marker-end="url(#arrow)"/><g class="c-gray"><rect x="140" y="280" width="400" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="300" text-anchor="middle" dominant-baseline="central">Gateway → 実行エンジン / Dify</text><text class="ts" x="340" y="318" text-anchor="middle" dominant-baseline="central">ワークフロー実行・パラメータ確定</text></g><path class="arr" fill="none" d="M340 336 L340 356" marker-end="url(#arrow)"/><g class="c-gray"><rect x="140" y="360" width="400" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="380" text-anchor="middle" dominant-baseline="central">Gateway → AWX</text><text class="ts" x="340" y="398" text-anchor="middle" dominant-baseline="central">ジョブ起動（作用は Gateway が実施）</text></g><path class="arr" fill="none" d="M340 416 L340 436" marker-end="url(#arrow)"/><g class="c-gray"><rect x="140" y="440" width="400" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="460" text-anchor="middle" dominant-baseline="central">AWX → Gateway</text><text class="ts" x="340" y="478" text-anchor="middle" dominant-baseline="central">完了 Webhook を外部イベントで受信</text></g><path class="arr" fill="none" d="M340 496 L340 516" marker-end="url(#arrow)"/><g class="c-gray"><rect x="140" y="520" width="400" height="56" rx="8" stroke-width="0.5"/><text class="th" x="340" y="540" text-anchor="middle" dominant-baseline="central">Gateway → ServiceNow</text><text class="ts" x="340" y="558" text-anchor="middle" dominant-baseline="central">work notes 記録・state 遷移</text></g></svg>

琥珀色の手順3が唯一の本番ガードです。ここを通らない限り AWX には到達しません。手順4で Dify を経由する場合も、返却 JSON の検証に失敗すれば手順5に進まないため、副作用が出る前に止まります。

最後がテスト実行経路です。

**🖼️ Diagram: test_execution_path_ops_test_isolation**

<svg width="100%" viewBox="0 0 680 520" role="img"><title>テスト実行経路と本番ガード</title><desc>Chainlit と Gradio からの実行はテスト用 Organization にのみ到達し、本番用 Organization は遮断される構成</desc><defs><marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs><g class="c-gray"><rect x="120" y="40" width="260" height="56" rx="8" stroke-width="0.5"/><text class="th" x="250" y="60" text-anchor="middle" dominant-baseline="central">Chainlit / Gradio</text><text class="ts" x="250" y="78" text-anchor="middle" dominant-baseline="central">env=test 固定</text></g><path class="arr" fill="none" d="M250 96 L250 132" marker-end="url(#arrow)"/><g class="c-purple"><rect x="120" y="136" width="260" height="56" rx="8" stroke-width="0.5"/><text class="th" x="250" y="156" text-anchor="middle" dominant-baseline="central">Foundry / Dify 開発WS</text><text class="ts" x="250" y="174" text-anchor="middle" dominant-baseline="central">テスト用ワークフロー</text></g><path class="arr" fill="none" d="M250 192 L250 228" marker-end="url(#arrow)"/><g class="c-purple"><rect x="90" y="232" width="320" height="56" rx="8" stroke-width="0.5"/><text class="th" x="250" y="252" text-anchor="middle" dominant-baseline="central">Integration Gateway</text><text class="ts" x="250" y="270" text-anchor="middle" dominant-baseline="central">二重の本番ガード</text></g><path class="arr" fill="none" d="M250 288 L250 348" marker-end="url(#arrow)"/><path fill="none" stroke="#A32D2D" stroke-width="1" stroke-dasharray="4 4" d="M410 260 L535 260 L535 342"/><path fill="none" stroke="#A32D2D" stroke-width="2" stroke-linecap="round" d="M522 344 L548 344"/><text class="ts" x="546" y="300">遮断</text><g class="c-gray"><rect x="120" y="352" width="260" height="56" rx="8" stroke-width="0.5"/><text class="th" x="250" y="372" text-anchor="middle" dominant-baseline="central">AWX ops-test</text><text class="ts" x="250" y="390" text-anchor="middle" dominant-baseline="central">テスト用 Organization</text></g><g class="c-red"><rect x="440" y="352" width="190" height="56" rx="8" stroke-width="0.5"/><text class="th" x="535" y="372" text-anchor="middle" dominant-baseline="central">AWX ops-prod</text><text class="ts" x="535" y="390" text-anchor="middle" dominant-baseline="central">到達不能</text></g><path class="arr" fill="none" d="M250 408 L250 444" marker-end="url(#arrow)"/><g class="c-gray"><rect x="170" y="448" width="160" height="44" rx="8" stroke-width="0.5"/><text class="th" x="250" y="470" text-anchor="middle" dominant-baseline="central">テスト環境</text></g></svg>

赤い破線が、Gateway のスキーマ制約と RBAC の二重で遮断される経路です。図2と図4で AWX が別の Organization として描かれているのが、本番とテストの分離の実体になります。

図の粒度や表記（ボックスの分割単位、日本語／英語の混在具合など）を調整したい場合はお知らせください。


---
