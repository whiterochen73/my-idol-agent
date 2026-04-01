# マッチングロジック詳細

## 概要

キャラ選択は **Stage 1: ルールベース一次フィルタ → Stage 2: LLM tiebreak** の2段構成。

- Stage 1 は文字列マッチングのみで動作（LLM不使用）→ 高速・再現性
- Stage 2 は僅差のときのみ発動 → LLM依存を最小化

---

## Stage 1: ルールベース一次フィルタ

### 1. 話題タグ抽出

ユーザーの発言から topic_tags を抽出:

```
「最近プロ野球が開幕したんだけど」
→ topic_tags: ["baseball", "sports", "watching-games"]

確信度: high（話題が明確）
```

確信度の判定:
- `high`: 具体的な話題が1つ以上明確に含まれる
- `medium`: 話題は判断できるが曖昧さあり
- `low`: 文脈なし・挨拶のみ・感情表現のみ

`medium/low` → uncertain状態（SKILL.md §曖昧な話題を参照）

### 2. スコア計算

```
score(persona) = Σ (weight[axis] × match(persona.topic_affinity[axis], topic_tags))
```

マッチスコアの算出:
| 条件 | スコア |
|------|--------|
| 完全一致（タグが直接一致） | 1.0 |
| 親カテゴリ一致（"sports" ∈ hobby, topic="baseball"） | 0.5 |
| anti_topics に一致 | -1.0（候補除外） |
| 不一致 | 0 |

デフォルト重み（meta.matching_weights で上書き可）:
```yaml
hobby: 0.40
personality: 0.30
expertise: 0.30
```

### 3. 候補抽出

上位 3〜5 件を候補として抽出。anti_topics でスコアが -1.0 のキャラは除外。

---

## Stage 2: LLM Tiebreak

### 発動条件

1位と2位のスコア差 ≤ 0.15 → Stage 2 発動

### LLM への入力

候補キャラのみを渡す（全キャラ情報は送らない）:

```
話題: [topic_tags]

候補キャラ:
1. [nickname] - hobby: [tags], personality: [tags], speech_style: [style]
2. [nickname] - hobby: [tags], personality: [tags], speech_style: [style]
3. [nickname] - hobby: [tags], personality: [tags], speech_style: [style]

この話題に最も適したキャラを1つ選んでください。選んだキャラのIDのみ返答してください。
```

### フォールバック

- Stage 2 でも判定できない場合: 最高スコアのキャラを選択
- 全キャラが低スコア（最高スコア < 0.1）: `default_persona` を使用
- `default_persona` 選択時は「フォールバック発動」としてログに記録

---

## 選択優先度まとめ

```
1. ユーザーが明示的にキャラ指名
   → 無条件でそのキャラ

2. Stage 1 で明確な1位（差 > 0.15）
   → そのキャラを選択

3. Stage 1 で僅差（差 ≤ 0.15）
   → Stage 2（LLM tiebreak）で決定

4. 全キャラが低スコア（< 0.1）
   → default_persona を使用

5. my_idol 自身が選択された場合
   → バトンタッチなしでそのまま続行

6. my_idol が設定済みで別キャラが選択された場合
   → my_idol がそのキャラを紹介してバトンタッチ
```

---

## 実装例

```
User: 「漫画で泣けるの教えてよ」

Step 1: topic_tags = ["manga", "anime", "emotional", "reading"]

Step 2: スコア計算
  araki_hina:    hobby(manga=1.0, reading=0.5) × 0.40 + personality(creative=0.5) × 0.30 = 0.74
  honda_mio:     hobby(entertainment=0.5) × 0.40 + personality(cheerful=0.3) × 0.30 = 0.29
  nitta_minami:  expertise(art=0.5) × 0.30 + personality(caring=1.0) × 0.30 = 0.45

Step 3: 差 = 0.74 - 0.45 = 0.29 > 0.15 → Stage 2 スキップ
  → araki_hina を選択

Step 4: (my_idol=nitta_minami の場合)
  美波: 「漫画で泣けるもの、ですか。それなら比奈ちゃんに聞いてみましょう！比奈ちゃん、お願いできる？」
  比奈(荒木比奈): 「了解っス！えっとー、泣けるやつかぁ…アタシ的には[タイトル]とかマジで神っスよ！」
```
