---
{"dg-publish":true,"permalink":"/claude/chats/2026-06-13-python-async-await-3d4cdf5d/","title":"Python async/await の内部動作メカニズム","tags":["claude","chat","cat/python","cat/js-ts"],"dg-note-properties":{"title":"Python async/await の内部動作メカニズム","date":"2026-06-13","created":"2026-06-13T10:50:09.671614Z","updated":"2026-06-13T11:04:37.777661Z","account":"権藤祥太郎 (shotaro.gondo@gmail.com)","conversation_id":"3d4cdf5d-085d-4119-b48f-60379adf6010","source":"claude-chat-export","tags":["claude","chat","cat/python","cat/js-ts"]}}
---

# Python async/await の内部動作メカニズム

## 🧑 User  
*2026-06-13T10:50:10.106939Z*

Python の async / await キーワードによる非同期処理は、内部的にどのように動作しますか？


---

## 🤖 Claude  
*2026-06-13T10:50:38.067156Z*

Pythonの非同期処理の内部動作を、段階的に説明します。

## 1. コルーチンとジェネレータの関係

`async def` で定義された関数は **コルーチンオブジェクト** を返します。これはPython 3.4以前の `yield`ベースのジェネレータコルーチンを洗練させたものです。

```python
async def fetch():
    await asyncio.sleep(1)
    return 42
```

これは内部的に、`__await__` メソッドを持つイテラブルオブジェクトになります。

---

## 2. `await` の意味

`await expr` は本質的に **「イテレータを一段階進める」** 操作です。

```python
# await は概念的にこう動く
result = yield from expr.__await__()
```

`await` できるオブジェクト（Awaitable）は3種類：
- **コルーチン** (`async def`関数の戻り値)
- **タスク / Future** (`asyncio.Task`, `asyncio.Future`)
- `__await__` を実装したオブジェクト

---

## 3. イベントループの役割

```
┌──────────────────────────────────────────┐
│              Event Loop                  │
│                                          │
│  ┌─────────┐    ready queue              │
│  │ Task A  │◄──────────────────┐         │
│  │ running │                   │         │
│  └────┬────┘                   │         │
│       │ await IO               │         │
│       ▼                        │         │
│  [I/O待ち登録]             IO完了時に     │
│  (selector/epoll)         readyへ戻す    │
│       │                        │         │
│  ┌─────────┐                   │         │
│  │ Task B  │ ← 次のタスクを実行 │         │
│  │ running │───────────────────┘         │
│  └─────────┘                             │
└──────────────────────────────────────────┘
```

イベントループは **シングルスレッド** で動き、タスクを協調的に切り替えます（プリエンプティブではない）。

---

## 4. 実行の流れ（具体例）

```python
import asyncio

async def task(name, delay):
    print(f"{name}: 開始")
    await asyncio.sleep(delay)   # ← ここで制御を返す
    print(f"{name}: 完了")

async def main():
    await asyncio.gather(
        task("A", 2),
        task("B", 1),
    )

asyncio.run(main())
```

```
task A: 開始
task B: 開始       ← A が sleep 中に B が実行される
task B: 完了       ← 1秒後
task A: 完了       ← 2秒後
# 合計2秒（逐次なら3秒）
```

ステップごとの内部動作：

1. `asyncio.run(main())` → イベントループ生成 → `main` をTaskとしてスケジュール
2. `gather()` → Task A, Task B を同時にスケジュール
3. Task A が `await asyncio.sleep(2)` に到達 → `Future`をループに渡して **サスペンド**
4. ループは即座に Task B を実行
5. Task B も `await asyncio.sleep(1)` でサスペンド
6. ループは `select()`/`epoll()` でI/Oと **タイマーを監視**
7. 1秒後 → Task B の Future が解決 → Task B を ready キューに追加 → 再開
8. 2秒後 → Task A 同様に再開

---

## 5. `asyncio.sleep` の最下層

`await asyncio.sleep(1)` の実装を追うと：

```python
# asyncio/tasks.py（簡略）
async def sleep(delay):
    loop = get_event_loop()
    future = loop.create_future()
    loop.call_later(delay, future.set_result, None)  # タイマー登録
    await future   # ← Future が解決されるまでサスペンド
```

最終的に `await future` は `yield` まで降りていき、制御をイベントループに返します。

---

## 6. スレッドとの違い

| | async/await | スレッド |
|---|---|---|
| 切り替え | 明示的（`await`時） | OSが任意のタイミングで |
| 並行数 | 数千〜数万 task | 数百スレッドが限界 |
| データ競合 | 原則なし | ロック必要 |
| 向いてる処理 | I/Oバウンド | CPUバウンド |

CPUバウンドな処理には `asyncio` は効果がなく、`ProcessPoolExecutor` や `run_in_executor()` との組み合わせが必要です。

---

**まとめると**：`async/await` はコルーチンの一時停止・再開をシンタックスシュガーで表現したもので、イベントループがシングルスレッド上でI/O完了通知（`epoll`/`kqueue`等）を使いタスクを協調的にスケジュールする仕組みです。


---

## 🧑 User  
*2026-06-13T11:00:59.367600Z*

CPUバウンドな処理とは、torch.Tensor の計算のようにロード・状態遷移させるデバイスを固定して実行するプログラムのことですか？


---

## 🤖 Claude  
*2026-06-13T11:01:18.770371Z*

少し違います。CPUバウンドとデバイス固定は直交する概念です。

## CPUバウンドの正確な定義

**「処理速度のボトルネックがCPUの演算能力にある」** 状態のことです。

```
CPUバウンド：CPU が常に100%稼働 → CPUを増やせば速くなる
I/Oバウンド：CPUはほぼ待機  → 待ち時間を非同期化すれば速くなる
```

---

## `torch.Tensor` 計算の位置づけ

```python
# CPU テンソル → CPUバウンド
x = torch.randn(10000, 10000)          # CPU上
y = torch.matmul(x, x)                # CPUの演算器をフル使用

# CUDA テンソル → GPUバウンド（別の話）
x = torch.randn(10000, 10000).cuda()   # GPU上
y = torch.matmul(x, x)                # GPUの演算器をフル使用
```

`torch.Tensor` の計算は「どのデバイスで動くか」によってボトルネックが変わります。「CPUバウンド」はあくまでCPUがボトルネックの場合の表現です。

---

## ボトルネック別の分類

| ボトルネック | 具体例 | `asyncio` で解決？ |
|---|---|---|
| **CPUバウンド** | 数値計算、暗号化、圧縮、CPUテンソル演算 | ❌ |
| **GPUバウンド** | CUDA上の行列演算、深層学習の順伝播 | ❌ |
| **I/Oバウンド（ディスク）** | ファイル読み書き、DBクエリ | ✅ |
| **I/Oバウンド（ネットワーク）** | HTTP通信、APIコール | ✅ |
| **メモリバウンド** | キャッシュに乗らない大規模データのスキャン | ❌ |

---

## `asyncio` が効かない理由（CPU/GPUバウンド）

```python
# ❌ これは意味がない
async def bad():
    x = torch.randn(10000, 10000)
    y = torch.matmul(x, x)  # await できない・ここでCPUが100%占有
    return y

# await できる箇所がないため、他のタスクに制御が渡らない
```

`await` は **「待ち時間が発生する箇所」** で制御を返す仕組みです。純粋な演算には待ち時間がなく、CPUが占有され続けるため `asyncio` の恩恵がありません。

---

## 深層学習での実際のボトルネック

```
DataLoader (CPUで前処理)  →  CPUバウンドになりやすい
      ↓
GPU転送 (.to(device))     →  PCIeバス帯域がボトルネックになることも
      ↓
forward / backward        →  GPUバウンド（理想的な状態）
      ↓
optimizer.step()          →  GPU or CPUバウンド
```

訓練ループで `nvidia-smi` を見たときにGPU使用率が低い場合、DataLoaderのCPU前処理がボトルネックになっている（CPUバウンド）ことが多く、`num_workers` を増やして解消するのはそのためです。


---

## 🧑 User  
*2026-06-13T11:04:25.985083Z*

>CPUバウンドな処理には `asyncio` は効果がなく、`ProcessPoolExecutor` や `run_in_executor()` との組み合わせが必要です。

つまりこの説明は、非同期処理でパフォーマンス向上の効果があるのは、CPU/メモリでの内部演算に時間を要することがボトルネックになっている場合と、I/O経由で外部処理の戻りを待つ時間が発生することがボトルネックになっている場合でasync/awaitで解決するかマルチスレッド化で解決するかを選択することが有効だ、といっているのですね？


---

## 🤖 Claude  
*2026-06-13T11:04:37.777661Z*

少し整理が必要です。まとめ方が逆になっています。

## 正しい対応関係

| ボトルネック | 有効な解決策 |
|---|---|
| **I/Oバウンド**（ネットワーク・ディスク待ち） | `async/await` ✅ |
| **CPUバウンド**（重い演算） | `ProcessPoolExecutor`（マルチプロセス）✅ |

---

## なぜ逆になりがちか

```
# I/Oバウンド → async/await が有効
await httpx.get("https://...")  # 待ち時間の間、他のタスクを実行できる

# CPUバウンド → async/await は無効、マルチプロセスが有効
result = torch.matmul(x, x)    # awaitできない、CPUを占有し続ける
```

`async/await` は **「待っている間に別のことをする」** 仕組みなので、そもそも待ち時間が発生しないCPUバウンドには効きません。

---

## マルチスレッドの位置づけ（補足）

スレッドはどちらにも使えそうに見えますが、Pythonには **GIL（Global Interpreter Lock）** があるため：

```
Pythonのマルチスレッド
├── I/Oバウンド → GILが一時解放されるので有効（ただしasyncの方が効率的）
└── CPUバウンド → GILが解放されないので無効（コア並列化できない）
```

CPUバウンドに対してマルチコアを活かすには、GILを回避できる **マルチプロセス**（`ProcessPoolExecutor`）が必要です。

---

## 整理すると

- `async/await` → **I/Oバウンド**の待ち時間を重ねて隠す
- `ProcessPoolExecutor` → **CPUバウンド**の演算を複数コアに分散する
- マルチスレッドはPythonでは中途半端（I/OにはasyncIOが上位互換、CPUにはGILが邪魔）


---
