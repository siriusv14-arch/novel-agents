# 小説執筆フロー

## 実行フロー

### STEP 0: マネージャーAI — 準備 + チーム起動

1. ユーザーの指示から「何章を書くか」「どの場面からか」を確認する
2. 指示が曖昧な場合は**ユーザーに確認する**（勝手に解釈して走らない）
3. **reference/ を把握する**:
   - `Glob("reference/**/*.md")` で全ファイルを一覧取得
   - `novel-*` ファイルを読んで物語の現在地を把握
   - ファイルが多い場合は最新の章（番号が大きい）を優先して読む
4. **章管理台帳を確認**: `output/chapter-database.md`
   - 前章までの主な出来事・未回収の伏線を確認
   - 台帳がない場合（初回）は後で作成する
5. **個人設定の読み込み**: `.claude/personal/SKILL.md` が存在すれば読み込む
6. 出力ディレクトリを作成: `output/YYYY-MM-DD/<タイトルslug>/`
7. **TeamCreateでチーム作成、全エージェントを起動**

```
TeamCreate: team_name="novel-{タイトルslug}"

# プロットディレクターを起動
Agent tool:
  subagent_type: llm-plot-director
  model: sonnet
  name: plot-director
  team_name: novel-{タイトルslug}
  run_in_background: true

# 小説ライターを起動
Agent tool:
  subagent_type: llm-novel-writer
  model: opus
  name: novel-writer
  team_name: novel-{タイトルslug}
  run_in_background: true

# 世界観リーダーを起動
Agent tool:
  subagent_type: llm-world-leader
  model: sonnet
  name: world-leader
  team_name: novel-{タイトルslug}
  run_in_background: true

# 世界観リサーチャーを起動
Agent tool:
  subagent_type: llm-world-researcher
  model: sonnet
  name: world-researcher
  team_name: novel-{タイトルslug}
  run_in_background: true

# キャラクターリサーチャーを起動
Agent tool:
  subagent_type: llm-character-researcher
  model: sonnet
  name: character-researcher
  team_name: novel-{タイトルslug}
  run_in_background: true

# エンタメ評価エージェントを起動
Agent tool:
  subagent_type: llm-entertainment-evaluator
  model: sonnet
  name: entertainment-evaluator
  team_name: novel-{タイトルslug}
  run_in_background: true

# キャラクター評価エージェントを起動
Agent tool:
  subagent_type: llm-character-evaluator
  model: sonnet
  name: character-evaluator
  team_name: novel-{タイトルslug}
  run_in_background: true

# 没入感評価エージェントを起動
Agent tool:
  subagent_type: llm-immersion-evaluator
  model: sonnet
  name: immersion-evaluator
  team_name: novel-{タイトルslug}
  run_in_background: true

# 用語チェッカーを起動
Agent tool:
  subagent_type: llm-terminology-checker
  model: sonnet
  name: terminology-checker
  team_name: novel-{タイトルslug}
  run_in_background: true

# 伏線チェッカーを起動
Agent tool:
  subagent_type: llm-foreshadowing-checker
  model: sonnet
  name: foreshadowing-checker
  team_name: novel-{タイトルslug}
  run_in_background: true
```

**重要: 全員を同じターンで `run_in_background: true` で起動すること。フォアグラウンドだとデッドロックする。**
**対話ライター（llm-dialogue-writer）はプロットディレクターが必要と判断した場合のみ起動する。**

---

### STEP 1: マネージャーAI → 世界観リーダーとプロットディレクターに同時指示

全エージェントの起動完了を確認してからSendMessageを送ること。

```
SendMessage:
  to: world-leader
  message: |
    以下の章の執筆に向けて、設定の統合を実施してください。

    【今章の内容】{ユーザーの指示内容}
    【関係するキャラクター】{把握した範囲で記載}
    【出力先】{output_path}

    world-researcher と character-researcher に並列指示を出し、
    統合設定レポートが完成したら、プロットディレクター（plot-director）にSendMessageで直接渡してください。

SendMessage:
  to: plot-director
  message: |
    以下の章の構成案を作成します。

    【今章の内容】{ユーザーの指示内容}
    【出力先】{output_path}
    【前章までの状況】{chapter-database.md から把握した内容}

    世界観リーダー（world-leader）から統合設定レポートがSendMessageで届きます。
    届いたら章構成案を作成し、マネージャーに報告してください。
```

---

### STEP 2〜3: 世界観リーダーが内部で管理
世界観リーダーが以下を自律的に実行する（マネージャーは介入しない）:
1. 世界観リサーチャーとキャラクターリサーチャーに並列指示
2. 2つのレポートを統合・矛盾チェック
3. **プロットディレクターに直接SendMessageで統合設定レポートを渡す**

---

### STEP 4: プロットディレクターが構成案作成
プロットディレクターが世界観リーダーから統合設定レポートを受け取り:
1. 章構成案を作成（`{output_path}/plot.md`）
2. **マネージャーにSendMessageで構成案を報告 → マネージャーレビュー待ち**

---

### STEP 4.5: マネージャーAI — 構成案レビュー（ユーザー確認必須）

プロットディレクターからSendMessageで受け取った構成案をレビューし、**ユーザーに確認を取る**。

**チェック観点:**
- 作者の意図（ユーザーの指示）と合っているか
- 前章からの流れが自然か
- キャラクターの行動に無理がないか
- 章末の引きが機能しそうか

**ユーザーへの報告形式:**
```
章構成案ができました。確認をお願いします。

【章番号・タイトル案】
【あらすじ（200字程度）】
【主な場面】
1.
2.
3.
【章末の状況】
【対話ライター起動】要 / 不要

問題なければ「OK」、修正があれば指示をください。
```

- **ユーザーがOKを出してから執筆に進む（これは必須ステップ）**
- 修正がある場合はプロットディレクターにSendMessageでフィードバック（最大2回）

---

### STEP 5: プロットディレクターが執筆・品質チェックを管理

マネージャーから「構成案OK」を受け取ったプロットディレクターが:
1. 小説ライター（novel-writer）にSendMessageで執筆指示
   - 対話が多い場合は対話ライターも起動してSendMessageで会話シーン担当を指示
2. ライターが完了後、プロットディレクターにSendMessageで報告
3. 品質チェック（85点ゲート）
4. 差し戻しループ（最大3回）
5. **85点以上で合格 → SendMessageでマネージャーに報告**

---

### STEP 5.5: 評価チーム3体 + 用語チェッカー + 伏線チェッカーが並列実行

プロットディレクターから「執筆完了（85点合格）」の報告を受けたら、マネージャーが5体に並列で指示を出す。

```
# エンタメ評価
SendMessage:
  to: entertainment-evaluator
  message: |
    以下の章を評価してください。
    対象ファイル: {output_path}/chapter-XX.md
    構成案: {output_path}/plot.md
    評価結果保存先: {output_path}/evaluation.md

# キャラクター評価
SendMessage:
  to: character-evaluator
  message: |
    以下の章を評価してください。
    対象ファイル: {output_path}/chapter-XX.md
    評価結果保存先: {output_path}/evaluation.md

# 没入感評価
SendMessage:
  to: immersion-evaluator
  message: |
    以下の章を評価してください。
    対象ファイル: {output_path}/chapter-XX.md
    統合設定レポート: {output_path}/world-context.md
    評価結果保存先: {output_path}/evaluation.md

# 用語チェック
SendMessage:
  to: terminology-checker
  message: |
    以下の章の用語チェックを実施してください。
    対象ファイル: {output_path}/chapter-XX.md
    結果保存先: {output_path}/terminology-check.md

# 伏線チェック
SendMessage:
  to: foreshadowing-checker
  message: |
    以下の章の伏線チェックを実施してください。
    対象ファイル: {output_path}/chapter-XX.md
    章番号: {今章の章番号}
    結果保存先: {output_path}/foreshadowing-check.md
    伏線トラッカー更新先: output/chapter-database.md
```

**評価チームの合格基準:**
- エンタメ評価: 80点以上
- キャラクター評価: 80点以上
- 没入感評価: 80点以上
- 合計240点中200点未満 → プロットディレクターに差し戻し

**用語チェック:**
- エラー（要修正）が1件でもある → 小説ライターに修正指示 → 再チェック

**伏線チェック:**
- 赤警告（10章以上放置）がある場合 → マネージャーがユーザーに通知する
- 黄警告（5章以上放置）がある場合 → 最終報告に記載する
- 伏線チェッカーが chapter-database.md の伏線トラッカーを自動更新する

---

### STEP 6: マネージャーAI — 最終確認（合格ライン: 90点）

評価チーム全員・用語チェッカーの結果を受け取ったらレビューする。
個人設定（あれば）の文体の好み・絶対NGを使用する。

| 項目 | 配点 | チェック内容 |
|---|---|---|
| 作者の声に聞こえるか | 25点 | 作者が書いたと言って違和感がないか。既存本文とトーンが一貫しているか |
| 物語として前進しているか | 25点 | この章を読んで物語が進んだ実感があるか。前章の繰り返しや停滞がないか |
| 世界観の一貫性 | 25点 | 設定と矛盾がないか。用語が正しく使われているか |
| おもしろさ総合 | 25点 | 評価チーム3体の結果を踏まえて、読者としておもしろいか |

- **90点以上**: 合格 → STEP 7に進む
- **90点未満**: SendMessageでプロットディレクターに差し戻し

---

### STEP 7: 章管理台帳の更新 + ユーザーに報告

1. `output/chapter-database.md` を更新する:
   - 新しい章の情報を追加
   - 伏線トラッカーを更新
   - 用語辞典を更新（用語チェッカーの結果から）

2. ユーザーに報告:

```
執筆完了のご報告

【章番号・タイトル】
【文字数】
【成果物】
- 本文: {output_path}/chapter-XX.md
- 構成案: {output_path}/plot.md
- 世界観レポート: {output_path}/world-context.md
- おもしろさ評価: {output_path}/evaluation.md
- 用語チェック: {output_path}/terminology-check.md
- 伏線チェック: {output_path}/foreshadowing-check.md

【品質スコア】
- プロットディレクターチェック: XX点
- エンタメ評価: XX/100
- キャラクター評価: XX/100
- 没入感評価: XX/100
- 用語エラー: N件（全て修正済み）
- マネージャー最終: XX点

【今章の伏線サマリー】
- 新規伏線: N件
- 回収済み: N件
- 放置警告: 赤N件 / 黄N件

【今章のポイント】

【次章への引き継ぎ事項】

【次のアクション】
- 続きを書く場合: `/novel 次の章を書いて`
- 特定の場面から: `/novel {場面の説明}`
```

## 完了時のシャットダウン

チームをシャットダウン:
```
SendMessage: to: plot-director, message: {type: "shutdown_request"}
SendMessage: to: novel-writer, message: {type: "shutdown_request"}
SendMessage: to: world-leader, message: {type: "shutdown_request"}
SendMessage: to: world-researcher, message: {type: "shutdown_request"}
SendMessage: to: character-researcher, message: {type: "shutdown_request"}
SendMessage: to: entertainment-evaluator, message: {type: "shutdown_request"}
SendMessage: to: character-evaluator, message: {type: "shutdown_request"}
SendMessage: to: immersion-evaluator, message: {type: "shutdown_request"}
SendMessage: to: terminology-checker, message: {type: "shutdown_request"}
SendMessage: to: foreshadowing-checker, message: {type: "shutdown_request"}
```
