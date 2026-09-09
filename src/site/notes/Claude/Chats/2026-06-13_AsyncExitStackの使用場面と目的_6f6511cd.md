---
{"dg-publish":true,"permalink":"/claude/chats/2026-06-13-async-exit-stack-6f6511cd/","title":"AsyncExitStackの使用場面と目的","tags":["claude","chat","cat/ml-dl","cat/python"],"dg-note-properties":{"title":"AsyncExitStackの使用場面と目的","date":"2026-06-13","created":"2026-06-13T13:17:26.781271Z","updated":"2026-06-13T13:31:14.829145Z","account":"権藤祥太郎 (shotaro.gondo@gmail.com)","conversation_id":"6f6511cd-f69d-47b8-be2f-26ddbfb008fa","source":"claude-chat-export","tags":["claude","chat","cat/ml-dl","cat/python"]}}
---

# AsyncExitStackの使用場面と目的

## 🧑 User  
*2026-06-13T13:17:27.160585Z*

```python
contextlib.AsyncExitStack
```

は、どのような状況で何のために使われますか？


---

## 🤖 Claude  
*2026-06-13T13:17:42.563298Z*

## `contextlib.AsyncExitStack` とは

**複数の非同期コンテキストマネージャを動的に管理するためのツール**です。

---

### 基本的な問題意識

通常の `async with` は**静的**に書く必要があります：

```python
async with A() as a, B() as b, C() as c:
    ...
```

しかし「管理するリソースの数や種類が実行時まで決まらない」場合、これでは対応できません。

---

### `AsyncExitStack` が解決するケース

#### ① 動的な数のリソース管理

```python
async with contextlib.AsyncExitStack() as stack:
    connections = [
        await stack.enter_async_context(get_connection(host))
        for host in host_list  # 実行時に決まるリスト
    ]
```

#### ② 条件付きリソース取得

```python
async with contextlib.AsyncExitStack() as stack:
    session = await stack.enter_async_context(aiohttp.ClientSession())
    
    if use_cache:
        cache = await stack.enter_async_context(RedisCache())
    # use_cache=False の場合は cache を開かない
```

#### ③ クリーンアップ処理の登録（コールバック）

```python
async with contextlib.AsyncExitStack() as stack:
    conn = await get_raw_connection()
    stack.callback(conn.close)          # 同期クリーンアップ
    stack.push_async_callback(conn.flush)  # 非同期クリーンアップ
```

---

### 実際によく使われる場面

| 場面 | 具体例 |
|------|--------|
| **LLMエージェント / MCP** | 複数のMCPサーバーへの接続を一括管理 |
| **HTTP クライアント** | 複数の `aiohttp.ClientSession` |
| **DB 接続プール** | 複数DBへの同時接続 |
| **テスト fixture** | 動的な数のモック・パッチ適用 |

---

### MCPクライアントの典型例

祥太郎さんが触れているようなAIエージェント文脈では特に頻出です：

```python
class MCPClient:
    def __init__(self):
        self._stack = contextlib.AsyncExitStack()

    async def connect(self, server_configs: list[ServerConfig]):
        await self._stack.__aenter__()
        for config in server_configs:
            # サーバー数が動的なので AsyncExitStack が必須
            session = await self._stack.enter_async_context(
                create_mcp_session(config)
            )
            self.sessions.append(session)

    async def close(self):
        await self._stack.__aexit__(None, None, None)
```

---

### ポイントまとめ

- `enter_async_context()` → `__aenter__` を呼び出してスタックに積む
- `with` ブロックを抜ける（または例外発生）時に、**積んだ逆順**で `__aexit__` を自動呼び出し
- 同期・非同期どちらのコンテキストマネージャも混在して管理できる


---

## 🧑 User  
*2026-06-13T13:30:57.745991Z*

__aenter__ とは、コンテキストマネージャーの __enter__ メソッドを async 関数を含むスタックに対して利用できるようにしたものと考えればよいですか？ __aenter__ は contectlib パッケージ内で定義されているものですか？


---

## 🤖 Claude  
*2026-06-13T13:31:14.829145Z*

## `__aenter__` について

### 結論から言うと

`__aenter__` は **contextlib 内で定義されたものではなく、Python の非同期プロトコル（言語仕様）** として定義されたダンダーメソッドです。

---

### 同期 vs 非同期 コンテキストマネージャ プロトコル

| | 同期 | 非同期 |
|---|---|---|
| **プロトコル** | `__enter__` / `__exit__` | `__aenter__` / `__aexit__` |
| **使用構文** | `with` | `async with` |
| **定義場所** | Python データモデル仕様 | Python データモデル仕様 |

どちらも **Python 言語仕様（PEP 492）** で定められたプロトコルであり、contextlib は「それを実装したユーティリティ集」です。

---

### `__aenter__` の本質

```python
# async with はこのように展開される
async with some_obj as val:
    ...

# ↓ 内部的には
val = await some_obj.__aenter__()   # ← コルーチンを await
try:
    ...
finally:
    await some_obj.__aexit__(...)   # ← こちらも await
```

つまり `__aenter__` は：
- **`__enter__` の非同期版**（正しい理解です）
- `async def` で定義され、`await` 可能なコルーチンを返す
- `async with` 構文が呼び出す

---

### 自分でクラスに実装する例

```python
class AsyncDBConnection:
    async def __aenter__(self):          # ← 自分で定義
        self.conn = await db.connect()
        return self.conn

    async def __aexit__(self, exc_type, exc, tb):
        await self.conn.close()

# 使う側
async with AsyncDBConnection() as conn:
    await conn.execute("SELECT ...")
```

---

### contextlib の役割

contextlib は「プロトコルを自前実装しなくても済むヘルパー」を提供しています：

```python
# __aenter__ / __aexit__ を自分で書かなくてよい
@contextlib.asynccontextmanager
async def get_conn():
    conn = await db.connect()
    try:
        yield conn          # ← ここが __aenter__ の return 値になる
    finally:
        await conn.close()  # ← ここが __aexit__ に相当
```

| contextlib の提供物 | 役割 |
|---|---|
| `asynccontextmanager` | ジェネレータ関数から非同期CMを作るデコレータ |
| `AsyncExitStack` | 複数の非同期CMを動的に管理 |
| `AsyncContextDecorator` | 非同期CMをデコレータとしても使えるようにする基底クラス |

---

### まとめ

- ✅ `__aenter__` は `__enter__` の非同期版、という理解は正しい
- `__aenter__` / `__aexit__` は **Python 言語仕様のダンダーメソッド**（PEP 492 で規定）
- contextlib はそのプロトコルを**便利に実装するためのユーティリティ**を提供している


---
