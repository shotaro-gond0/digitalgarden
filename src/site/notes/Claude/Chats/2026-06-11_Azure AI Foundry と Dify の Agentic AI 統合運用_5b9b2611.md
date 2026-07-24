---
{"dg-publish":true,"permalink":"/claude/chats/2026-06-11-azure-ai-foundry-dify-agentic-ai-5b9b2611/","tags":["cat/devops","cat/azure"],"dg-note-properties":{"tags":["cat/devops","cat/azure"]}}
---

---
title: Azure AI Foundry と Dify の Agentic AI 統合運用
date: '2026-06-11'
created: '2026-06-11T06:25:02.002637Z'
updated: '2026-06-11T06:29:00.620135Z'
account: 権藤祥太郎 (shotaro.gondo@gmail.com)
conversation_id: 5b9b2611-8d3f-4089-96ee-775862d6dd81
source: claude-chat-export
tags:
- claude
- chat
- cat/ml-dl
- cat/react
- cat/js-ts
- cat/devops
- cat/azure
---
# Azure AI Foundry と Dify の Agentic AI 統合運用

## 🧑 User  
*2026-06-11T06:25:02.364626Z*

Azure AI Foundry で開発するAgentic AIアプリケーションと、Dify上で開発するAgentic AIアプリケーションと、それらで開発する ChatOps フロントエンドと連携させて運用する前提で開発するAnsible Towerで稼働させるPlaybookのDevOpsを、GitHub Issues で要件・進捗管理するには、どのような連携構成がよいでしょうか？


---

## 🤖 Claude  
*2026-06-11T06:25:14.055912Z*

少し整理させてください。構成の前提を確認してから最適な連携方式を提案します。


---

## 🧑 User  
*2026-06-11T06:27:36.477203Z*

Q: ChatOpsフロントエンドはどのチャットツールを使いますか？
A: Chainlit and/or Gradio でカスタム開発するフロントエンド

Q: GitHub Issuesでの管理粒度はどうしますか？
A: Epic/Story/Task の階層管理

Q: 自動化の優先度はどこですか？（該当するものをすべて選択）
A: Playbook自動トリガーはServiceNow起票経由


---

## 🤖 Claude  
*2026-06-11T06:29:00.620135Z*

ありがとうございます。構成が明確になりました。整理して回答します。

全体像を図示してから詳細を解説します。

**🖼️ Diagram: devops_integration_architecture**

<svg width="100%" viewBox="0 0 680 720" role="img">
<title>DevOps連携アーキテクチャ全体図</title>
<desc>Azure AI Foundry・Dify・Chainlit/Gradio・Ansible Tower・GitHub Issues・ServiceNowの連携構成図</desc>
<defs>
  <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
    <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
  </marker>
</defs>
<!-- Layer 1: ChatOps Frontend (Chainlit/Gradio) -->
<g class="c-purple">
  <rect x="40" y="30" width="600" height="54" rx="10" stroke-width="0.5"/>
  <text class="th" x="340" y="50" text-anchor="middle" dominant-baseline="central">ChatOps フロントエンド</text>
  <text class="ts" x="340" y="69" text-anchor="middle" dominant-baseline="central">Chainlit / Gradio（カスタム開発）</text>
</g>
<!-- Layer 2: AI Agents -->
<g class="c-blue">
  <rect x="40" y="110" width="280" height="62" rx="8" stroke-width="0.5"/>
  <text class="th" x="180" y="133" text-anchor="middle" dominant-baseline="central">Azure AI Foundry</text>
  <text class="ts" x="180" y="153" text-anchor="middle" dominant-baseline="central">Agentic AI アプリケーション</text>
</g>
<g class="c-teal">
  <rect x="360" y="110" width="280" height="62" rx="8" stroke-width="0.5"/>
  <text class="th" x="500" y="133" text-anchor="middle" dominant-baseline="central">Dify</text>
  <text class="ts" x="500" y="153" text-anchor="middle" dominant-baseline="central">Agentic AI アプリケーション</text>
</g>
<!-- Layer 3: Issue / Ticket Management -->
<g class="c-amber">
  <rect x="40" y="206" width="280" height="62" rx="8" stroke-width="0.5"/>
  <text class="th" x="180" y="229" text-anchor="middle" dominant-baseline="central">GitHub Issues + Projects</text>
  <text class="ts" x="180" y="249" text-anchor="middle" dominant-baseline="central">Epic / Story / Task 階層管理</text>
</g>
<g class="c-coral">
  <rect x="360" y="206" width="280" height="62" rx="8" stroke-width="0.5"/>
  <text class="th" x="500" y="229" text-anchor="middle" dominant-baseline="central">ServiceNow</text>
  <text class="ts" x="500" y="249" text-anchor="middle" dominant-baseline="central">インシデント / 変更管理</text>
</g>
<!-- Layer 4: Automation Gateway -->
<g class="c-gray">
  <rect x="40" y="302" width="600" height="54" rx="8" stroke-width="0.5"/>
  <text class="th" x="340" y="322" text-anchor="middle" dominant-baseline="central">GitHub Actions / Webhook ブリッジ</text>
  <text class="ts" x="340" y="341" text-anchor="middle" dominant-baseline="central">ルーティング・認証・ペイロード変換</text>
</g>
<!-- Layer 5: Ansible Tower -->
<g class="c-green">
  <rect x="40" y="390" width="600" height="62" rx="10" stroke-width="0.5"/>
  <text class="th" x="340" y="413" text-anchor="middle" dominant-baseline="central">Ansible Tower (AWX)</text>
  <text class="ts" x="340" y="433" text-anchor="middle" dominant-baseline="central">Playbook 実行管理 / ジョブテンプレート / RBAC</text>
</g>
<!-- Layer 6: Infra -->
<g class="c-gray">
  <rect x="40" y="484" width="186" height="52" rx="8" stroke-width="0.5"/>
  <text class="th" x="133" y="507" text-anchor="middle" dominant-baseline="central">Azure 環境</text>
  <text class="ts" x="133" y="525" text-anchor="middle" dominant-baseline="central">VM / AKS / ネットワーク</text>
</g>
<g class="c-gray">
  <rect x="247" y="484" width="186" height="52" rx="8" stroke-width="0.5"/>
  <text class="th" x="340" y="507" text-anchor="middle" dominant-baseline="central">オンプレミス</text>
  <text class="ts" x="340" y="525" text-anchor="middle" dominant-baseline="central">サーバー / ネットワーク機器</text>
</g>
<g class="c-gray">
  <rect x="454" y="484" width="186" height="52" rx="8" stroke-width="0.5"/>
  <text class="th" x="547" y="507" text-anchor="middle" dominant-baseline="central">その他クラウド</text>
  <text class="ts" x="547" y="525" text-anchor="middle" dominant-baseline="central">AWS / GCP / SaaS</text>
</g>
<!-- Flow arrows: ChatOps → Agents -->
<line x1="200" y1="84" x2="200" y2="108" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.6"/>
<line x1="480" y1="84" x2="480" y2="108" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.6"/>
<!-- Agents → Issues/ServiceNow -->
<line x1="180" y1="172" x2="180" y2="204" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.6"/>
<line x1="500" y1="172" x2="500" y2="204" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.6"/>
<!-- ServiceNow → GitHub Issues (横矢印) -->
<path d="M360 238 L322 238" fill="none" stroke="#BA7517" stroke-width="1.2" marker-end="url(#arrow)" opacity="0.9"/>
<text class="ts" x="341" y="228" text-anchor="middle" opacity="0.85">自動起票</text>
<!-- Issues/ServiceNow → Actions Bridge -->
<line x1="180" y1="268" x2="260" y2="300" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.6"/>
<line x1="500" y1="268" x2="420" y2="300" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.6"/>
<!-- Bridge → Tower -->
<line x1="340" y1="356" x2="340" y2="388" stroke="var(--color-text-secondary)" stroke-width="1.2" marker-end="url(#arrow)" opacity="0.7"/>
<!-- Tower → Infra -->
<line x1="200" y1="452" x2="133" y2="482" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.5"/>
<line x1="340" y1="452" x2="340" y2="482" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.5"/>
<line x1="480" y1="452" x2="547" y2="482" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.5"/>
<!-- Feedback arrow: Tower results → GitHub Issues (右側に回す) -->
<path d="M640 421 Q670 421 670 237 Q670 60 630 60" fill="none" stroke="#3B8BD4" stroke-width="1.2" stroke-dasharray="5 3" marker-end="url(#arrow)" opacity="0.6"/>
<text class="ts" x="672" y="320" text-anchor="start" opacity="0.8" transform="rotate(90,672,320)">実行結果フィードバック</text>
<!-- Legend -->
<rect x="40" y="568" width="600" height="130" rx="8" fill="none" stroke="var(--color-border-tertiary)" stroke-width="0.5"/>
<text class="th" x="60" y="590" dominant-baseline="central">凡例</text>
<line x1="60" y1="610" x2="100" y2="610" stroke="var(--color-text-secondary)" stroke-width="1" marker-end="url(#arrow)" opacity="0.7"/>
<text class="ts" x="110" y="614">通常フロー（ユーザー操作・API連携）</text>
<line x1="60" y1="635" x2="100" y2="635" stroke="#3B8BD4" stroke-width="1.2" stroke-dasharray="5 3" marker-end="url(#arrow)" opacity="0.7"/>
<text class="ts" x="110" y="639">フィードバック / 非同期通知</text>
<line x1="60" y1="660" x2="100" y2="660" stroke="#BA7517" stroke-width="1.2" marker-end="url(#arrow)" opacity="0.9"/>
<text class="ts" x="110" y="664">ServiceNow → GitHub 自動起票</text>
<text class="ts" x="60" y="688">Playbook 実行トリガー: ServiceNow 起票 → GitHub Issues 連動 → Actions → Tower</text>
</svg>

全体構成を図示しました。各レイヤーの設計方針と実装ポイントを順に解説します。

---

## レイヤー別設計方針

### 1. ChatOps フロントエンド（Chainlit / Gradio）

Chainlit と Gradio はそれぞれ性格が異なるので役割分担が有効です。

| ツール | 向いている用途 |
|---|---|
| Chainlit | 会話履歴・マルチターン・ステップ表示（Playbookログの逐次表示） |
| Gradio | フォーム入力型オペレーション（パラメータ指定してジョブ実行など） |

どちらからも **GitHub Issues API** と **ServiceNow REST API** を直接叩くのではなく、後述の AI エージェント層を介すことで、自然言語→構造化パラメータへの変換を挟んでください。

---

### 2. AI エージェント層（Azure AI Foundry / Dify）の役割分担

**Azure AI Foundry** は Tool Calling 経由で以下を担います。

- GitHub Issues の Epic/Story/Task を読み書きする `gh_issue_tool`
- Ansible Tower の Job Template を起動する `tower_launch_tool`
- 実行ログを Issues へコメントバックする `gh_comment_tool`

**Dify** はノーコード/ローコード寄りのフローとして、ServiceNow ↔ GitHub 間の変換・ルーティングロジックに適しています。Dify の HTTP Request ノードで ServiceNow Webhook を受け取り、GitHub Issues へ起票する橋渡しフローを組むのが現実的です。

両者は排他ではなく、**Dify でインテグレーションフロー、Azure AI Foundry でエージェント推論** という役割で共存できます。

---

### 3. GitHub Issues の Epic / Story / Task 階層構成

GitHub Issues にはネイティブな階層機能がないため、以下のパターンで実現します。

```
Epic  → Issue with label:"epic"    + milestone でスプリント管理
Story → Issue with label:"story"   + 本文に "Epic: #N" 参照
Task  → Issue with label:"task"    + "Story: #N" 参照 + assignee
```

**GitHub Projects（v2）** のカスタムフィールドに `type: Epic|Story|Task` と `ServiceNow Change ID` を追加することで、ServiceNow の変更管理番号と紐付けたカンバン管理が可能になります。

**ラベル命名規則の例：**

```
epic / story / task              ← 階層
playbook:pending                 ← Playbookが実行待ち
playbook:running                 ← Tower実行中
playbook:success / :failed       ← 実行結果
sn:chg-XXXXXXX                  ← ServiceNow変更番号タグ
```

---

### 4. ServiceNow → GitHub → Ansible Tower トリガーフロー

これが本構成の核心です。

```
ServiceNow 変更承認
  → ServiceNow Business Rule / Flow Designer で Webhook 発火
    → GitHub Actions (repository_dispatch) でイベント受信
      → Actions ワークフローが Issues に Task を自動起票
        → Ansible Tower REST API で Job Template を起動
          → 実行結果を GitHub Issues にコメント・ラベル更新
            → ServiceNow の変更レコードを Close/Update
```

GitHub Actions 側の起点となるワークフローのポイント：

```yaml
on:
  repository_dispatch:
    types: [servicenow-change-approved]

jobs:
  trigger-playbook:
    runs-on: ubuntu-latest
    steps:
      - name: Create GitHub Issue (Task)
        uses: actions/github-script@v7
        with:
          script: |
            const issue = await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `[Task] ${{ github.event.client_payload.change_title }}`,
              labels: ['task', 'playbook:pending',
                       'sn:${{ github.event.client_payload.change_id }}'],
              body: `Story: #<parent_story_number>\nServiceNow: ${{ github.event.client_payload.change_id }}`
            });

      - name: Launch Ansible Tower Job
        run: |
          curl -s -X POST \
            -H "Authorization: Bearer ${{ secrets.TOWER_TOKEN }}" \
            -H "Content-Type: application/json" \
            -d '{"extra_vars": {"change_id": "${{ github.event.client_payload.change_id }}"}}' \
            ${{ secrets.TOWER_URL }}/api/v2/job_templates/<ID>/launch/
```

---

### 5. Playbook 実行結果の GitHub Issues フィードバック

Ansible Tower のジョブ完了通知は **Notification Template（Webhook タイプ）** を使い、GitHub Actions の `workflow_dispatch` または直接 Issues API に POST します。

```
Tower Notification Webhook
  → GitHub Actions (workflow_dispatch or direct API)
    → Issue のラベルを playbook:running → playbook:success/:failed に更新
    → Issue にコメント（ジョブID・実行時間・変更対象ホスト数・サマリ）
    → ServiceNow の変更レコードのステータスを更新
```

Issues コメントの構造テンプレート：

```markdown
## Playbook 実行結果 ✅

| 項目 | 値 |
|---|---|
| Job ID | 1234 |
| Template | deploy-webapp |
| 実行時間 | 4m 32s |
| 対象ホスト | 6台 |
| ServiceNow | CHG0012345 |

[Tower ジョブ詳細](<TOWER_URL>/jobs/playbook/1234)
```

---

### 6. AI エージェントが Issues を起票・更新するパターン

ChatOps フロントエンドからユーザーが「本番デプロイして」と発話した場合のフロー：

```
Chainlit → Azure AI Foundry Agent
  → gh_issue_tool: Story Issue 起票（デプロイ要件を本文に構造化）
  → 承認待ちラベルを付与 → 人間がレビュー
  → 承認ラベル付与 → Actions が検知
  → ServiceNow に変更申請を自動起票（Dify フロー）
  → ServiceNow 承認後 → Tower 起動（前述フロー）
```

これにより **「会話→Issue→変更管理→自動実行→フィードバック」** のループが一気通貫します。

---

### 7. 運用上の注意点

**冪等性の担保：** ServiceNow 起票と GitHub Issue 起票が重複しないよう、ServiceNow の Change ID をイシュータイトルや本文に含め、Actions 側で `gh issue list --search "sn:CHG0012345"` による重複チェックを必ず挟んでください。

**RBAC の整合：** Ansible Tower のジョブ実行権限と GitHub の Issue 操作権限は別系統です。Tower 側は Organization 単位で Service Account を分離し、GitHub Actions Secrets に格納したトークンのスコープを最小化（`repo` ではなく `issues:write` のみ）することを推奨します。

**Dify の制約：** Dify は外部 Webhook の受信（インバウンド）が標準では弱いため、ServiceNow からの Webhook 受信には Azure Functions や GitHub Actions を中継として挟み、Dify はアウトバウンド API コールに徹する設計が安定します。


---
