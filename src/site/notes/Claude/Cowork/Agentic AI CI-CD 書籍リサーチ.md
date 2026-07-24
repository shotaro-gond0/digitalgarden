---
{"dg-publish":true,"permalink":"/claude/cowork/agentic-ai-ci-cd/","tags":["cat/ml-dl","cat/python","cat/devops","cat/claude"],"dg-note-properties":{"tags":["cat/ml-dl","cat/python","cat/devops","cat/claude"]}}
---

---
title: Agentic AI・MCP開発とCI/CDパイプライン構築に関する書籍リサーチ
date: '2026-07-21'
source: cowork-manual-export
tags:
- claude
- cowork
- cat/ml-dl
- cat/python
- cat/devops
- cat/claude
---
# Agentic AI・MCP開発とCI/CDパイプライン構築に関する書籍リサーチ

> [!info] 概要
> LangGraph/LangChain/Chainlitを含むAgentic AIおよびMCPサーバー開発のためのCI/CDパイプライン構築に関する技術書調査。MS AI Foundryでの開発を踏まえた選定。

---

## 1. Agentic AI / LangGraph / MCP開発 書籍

### MCPサーバー開発大全 ⭐必読

- **出版社**: 技術評論社
- **ISBN**: 978-4-297-15327-4
- CI/CDパイプライン構築とテスト戦略を含む唯一の専門書
- MCPの基礎からサーバー実装、実例（天気予報サーバー・社内ドキュメントサーバー）まで網羅
- 著者考案の**「4層テスト戦略」**がMCP特有の課題に対応
- 自動テスト・CI/CD構築まで実装面をカバー
- 紙版・電子版あり
- リンク: [技術評論社](https://gihyo.jp/book/2025/978-4-297-15327-4) / [電子版](https://gihyo.jp/dp/ebook/2025/978-4-297-15328-1)

### LangChainとLangGraphによるRAG・AIエージェント［実践］入門 ⭐推奨

- 技術評論社（エンジニア選書）/ 著: 西見 公宏
- LangGraphを使ったAIエージェント設計パターン別実装
- AIエージェント開発の体系的なアプローチ、実践向けの設計思想
- リンク: [Amazon.co.jp](https://www.amazon.co.jp/LangChain%E3%81%A8LangGraph%E3%81%AB%E3%82%88%E3%82%8BRAG%E3%83%BBAI%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88%EF%BC%BB%E5%AE%9F%E8%B7%B5%EF%BC%BD%E5%85%A5%E9%96%80-%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2%E9%81%B8%E6%9B%B8-%E8%A5%BF%E8%A6%8B-%E5%85%AC%E5%AE%8F/dp/4297145308)

### 作って学ぶAIエージェント

- 技術評論社 / 2026年4月発行
- Microsoft Agent Framework 1.0対応
- TypeScript実装とGitHub統合による自動化ワークフロー、実装寄り・実践的
- リンク: [技術評論社](https://gihyo.jp/book/2026/978-4-297-15565-0)

### 実践 AIエージェント開発

- O'Reilly Japan / 2026年4月発行
- エージェント設計、アーキテクチャパターン、本番運用のベストプラクティス
- リンク: [オライリー・ジャパン](https://www.oreilly.co.jp/books/9784814401598/)

### LangChain完全入門

- インプレス / 著: 田村悠
- Chainlit統合の基礎、生成AIアプリケーション開発全般（補完的な基礎知識）
- リンク: [インプレスブックス](https://book.impress.co.jp/books/1123101047)

### MS AI Foundry関連リソース

- [Microsoft Foundry documentation (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/foundry/)
- [LangChain and LangGraph - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-frameworks/langchain-langgraph.html)
- 2026年新機能: Microsoft Agent Framework 1.0安定版、Foundry Hosted Agents（ハイパーバイザ隔離・Entra ID対応）、Adaptive Evaluations（自動テスト評価）

### 学習ロードマップ（第1回提案）

1. **基礎フェーズ**: LangChain完全入門 → LangChainとLangGraphによる実践入門
2. **MCP実装フェーズ**: MCPサーバー開発大全（テスト戦略を習得）
3. **本番設計フェーズ**: 実践 AIエージェント開発、作って学ぶAIエージェント
4. **MS Foundry対応**: 公式ドキュメント + 2026年ガイド記事

---

## 2. Python全般のCI/CDパイプライン（GitHub / GitHub Actions）書籍

### GitHub CI/CD実践ガイド――持続可能なソフトウェア開発を支えるGitHub Actionsの設計と運用 ⭐最推奨

- **著者**: 野村友規 / 技術評論社（エンジニア選書）
- MCPサーバー開発大全とセットで使用するのに最適
- GitHub Actionsの基本構文からハンズオン形式で段階的に進む
- **カバー範囲**:
  - テスト自動化（pytest含む）
  - 静的解析とコード品質チェック
  - リリース自動化
  - コンテナデプロイ
  - Dependabot・OpenID Connect
  - セキュリティベストプラクティス
  - GitHub Apps
- 「持続可能なソフトウェア開発」という視点がMCPサーバーの長期保守に適合
- リンク: [Amazon.co.jp](https://www.amazon.co.jp/GitHub-CI-CD%E5%AE%9F%E8%B7%B5%E3%82%AC%E3%82%A4%E3%83%89%E2%80%95%E2%80%95%E6%8C%81%E7%B6%9A%E5%8F%AF%E8%83%BD%E3%81%AA%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E9%96%8B%E7%99%BA%E3%82%92%E6%94%AF%E3%81%88%E3%82%8BGitHub-Actions%E3%81%AE%E8%A8%AD%E8%A8%88%E3%81%A8%E9%81%8B%E7%94%A8-%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2%E9%81%B8%E6%9B%B8/dp/4297141736) / [技術評論社](https://gihyo.jp/book/2024/978-4-297-14173-8)

### GitHub Actions 実践入門

- **著者**: 宮田淳平 / インプレス NextPublishing
- 基礎から実装までのバランスが良く、個別のCI/CDパターン解説が充実
- リンク: [インプレス NextPublishing](https://nextpublishing.jp/book/11908.html)

### オンラインガイド（補足）

- [現場で回る CI/CD パイプラインの設計術──よく踏む地雷と避け方 (Zenn, 2026年版)](https://zenn.dev/miyan/articles/agile-cicd-best-practices-2026) — テストピラミッド実装パターン、パイプラインセキュリティ対策
- [GitHub Actionsで実務で使えるCI/CDパイプライン設計パターン (Zenn)](https://zenn.dev/shineos/articles/github-actions-cicd-patterns)
- [Python のビルドとテスト - GitHub Actions（公式ドキュメント）](https://docs.github.com/en/actions/tutorials/build-and-test-code/python)
- [5分でできるGitHub Actionsを用いた自動テスト (Qiita)](https://qiita.com/Isaka-code/items/9c7fb54fc0bb2aaa5c83)
- [GitHub Actionsを用いた自動テストの実行と結果集計 (Qiita)](https://qiita.com/vZke/items/868eb9fd5289d2a25660)
- [CI/CDパイプライン設計のベストプラクティス完全ガイド（2026年版）(株式会社renue)](https://renue.co.jp/posts/cicd-pipeline-design-best-practices-github-actions-automation-guide)

### 学習ロードマップ（第2回提案）

```
【基礎】GitHub Actions 実践入門 → 基本構文・ワークフロー理解
    ↓
【実装】GitHub CI/CD実践ガイド（第3～5章）→ テスト自動化・デプロイパターン習得
    ↓
【実践設計】現場で回る CI/CD パイプライン設計術 → テスト戦略・セキュリティ対策
    ↓
【MCP連携】MCPサーバー開発大全 + GitHub CI/CD実践ガイド
    → MCP特有のテスト戦略と自動化の統合
```

### 2026年の重要な考慮事項

- **パイプラインセキュリティ**: コード品質だけでなく、パイプライン自体が攻撃対象化（2024〜2025年に急増）
- **ブランチ戦略とパイプラインの連動**: Git Flowなどの選択がパイプライン設計に大きく影響
- **テストピラミッドの実装**: 段階的テスト実行とコスト最適化

---

## まとめ：全体の書籍リスト

| 書籍名 | 分野 | 出版社 | 優先度 |
|---|---|---|---|
| MCPサーバー開発大全 | MCP + CI/CD/テスト | 技術評論社 | ⭐⭐⭐ |
| GitHub CI/CD実践ガイド | GitHub Actions全般 | 技術評論社 | ⭐⭐⭐ |
| LangChainとLangGraphによるRAG・AIエージェント実践入門 | LangGraph/エージェント | 技術評論社 | ⭐⭐⭐ |
| GitHub Actions 実践入門 | GitHub Actions基礎 | インプレス | ⭐⭐ |
| 実践 AIエージェント開発 | エージェント設計 | O'Reilly Japan | ⭐⭐ |
| 作って学ぶAIエージェント | 実装（TypeScript） | 技術評論社 | ⭐⭐ |
| LangChain完全入門 | LangChain + Chainlit基礎 | インプレス | ⭐ |

---

*このノートはClaude (Cowork) との会話から自動生成されました。*
