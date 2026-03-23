# Novel Agents

異世界ファンタジー小説の続きを、AIチームと一緒に書き進めるツールです。

## セットアップ

### 1. リポジトリをクローン

```bash
git clone https://github.com/kurihara-sdx/novel-agents.git
cd novel-agents
```

### 2. Claude Code を起動

```bash
claude
```

### 3. 個人設定を作成

```
/onboarding
```

文体の好みや執筆方針を対話形式で設定します。一度だけ実行してください。

### 4. 既存の小説・設定資料を配置

`reference/` フォルダに既存の資料を置いてください。
置き方の詳細は `reference/README.md` を参照してください。

---

## 使い方

### 基本コマンド

```
/novel 指示内容
```

**例:**
```
/novel 第5章の続きを書いて。主人公のレナが廃墟の塔に入る場面から
/novel 第3章を書いて。アルとエリスが初めて出会うシーン
/novel 次の章を書いて
```

---

## 実行フロー（自動で進む）

| ステップ | 内容 |
|---|---|
| STEP 0 | reference/ を読み込み、物語の現在地を把握。チーム起動 |
| STEP 1 | 世界観リーダーが設定を統合、プロットディレクターに渡す |
| STEP 2 | プロットディレクターが章構成案を作成 |
| STEP 3 | **ユーザーが構成案をレビュー**（必須）|
| STEP 4 | 小説ライターが執筆。プロットディレクターが品質チェック（85点ゲート）|
| STEP 5 | 評価チーム3体がおもしろさを評価。用語チェッカーが用語照合 |
| STEP 6 | マネージャーが最終確認（90点ゲート）|
| STEP 7 | 完成。ユーザーに報告 |

---

## 成果物の出力先

```
output/YYYY-MM-DD/<タイトルslug>/
├── plot.md              # 章構成案
├── chapter-XX.md        # 章本文
├── world-context.md     # 世界観統合レポート
├── evaluation.md        # おもしろさ評価レポート
└── terminology-check.md # 用語チェックレポート
```

---

## reference/ への資料の置き方

`reference/README.md` を参照してください。
