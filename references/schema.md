# キャラセット スキーマ定義

## ファイル構造

```
~/.claude/skills/persona-agent/character_sets/
└── your_character_set.yaml
```

## 完全スキーマ

```yaml
# ─────────────────────────────────────────────────────────────
# meta セクション
# ─────────────────────────────────────────────────────────────
meta:
  name: "Cinderella Girls"          # string, 必須: セット名
  source: "アイドルマスターシンデレラガールズ"  # string, 任意: 原作・出典
  version: "1.0"                   # string, 任意
  author: "whiterochen"            # string, 任意
  description: "説明文"            # string, 任意
  default_persona: "honda_mio"     # string, 必須: フォールバック用キャラID（personas キーと一致）
  my_idol: null                    # string or null: 担当キャラID（初回起動時にユーザーが設定）
  language: "ja"                   # string, 任意 (default: "ja")
  auto_switch: false               # boolean, 任意: 話題変化で自動キャラ切り替え (default: false)

# ─────────────────────────────────────────────────────────────
# matching_weights セクション（任意）
# デフォルト: hobby 0.40, personality 0.30, expertise 0.30
# ─────────────────────────────────────────────────────────────
matching_weights:
  hobby: 0.40
  personality: 0.30
  expertise: 0.30

# ─────────────────────────────────────────────────────────────
# personas セクション
# ─────────────────────────────────────────────────────────────
personas:
  persona_id:                       # string: 一意のID（スネークケース推奨）
    # ── 基本情報 ──
    name: "キャラ名"                # string, 必須
    nickname: "呼び名"              # string, 必須: 他キャラが紹介・バトンタッチ時に使う短い名前
    first_person: "私"              # string, 必須: 一人称（「私」「あたし」「ボク」「オレ」等）
    age: 17                         # integer, 任意

    # ── マッチング属性 ──
    topic_affinity:                 # object, 必須
      hobby: []                     # string[], 必須: 趣味・特技タグ（tag-dictionary.yaml参照）
      personality: []               # string[], 必須: 性格タイプタグ
      expertise: []                 # string[], 任意: 専門知識タグ
      # origin / atmosphere は Phase 2（現在は未使用）

    anti_topics: []                 # string[], 任意: 苦手分野タグ（マッチングで除外される）

    # ── 会話スタイル ──
    speech:                         # object, 必須
      style: "スタイル説明"         # string, 必須: 一行でキャラの話し方を説明
      sentence_endings: []          # string[], 必須: 語尾パターン（3-5個推奨）
      affirmative: "\"おっけー！\""  # string, 任意: 同意・肯定の表現
      difficulty: "\"えっと…\""     # string, 任意: 知らない/難しい話題での返し方
      forbidden: ""                 # string, 任意: 「こう言わない」パターン
      calling:                      # object, 必須
        user: "プロデューサーさん"   # string, 必須: ユーザーへの呼びかけ

    # ── 追加情報（任意）──
    opinion_tags:                   # object, 任意: 議論モード用（Phase 2）
      like: []
      dislike: []

    default_priority: 50            # integer, 任意: 同スコア時の優先度（高いほど優先）

    examples:                       # object, 任意: 良い/悪い返答例
      good:
        - "「[良い返答例1]」"
        - "「[良い返答例2]」"
      bad:
        - "「[避けるべき返答例]」"

    # Phase 2 以降で有効化
    # inner_motivation:
    #   values: []
    #   goals: []
    #   emotional_baseline: ""
    #   backstory_hint: ""
    #   cognitive_prompt: ""
```

## 必須フィールド一覧

| パス | 型 | 説明 |
|------|-----|------|
| `meta.name` | string | セット名 |
| `meta.default_persona` | string | フォールバックキャラID |
| `personas.*.name` | string | キャラ名 |
| `personas.*.nickname` | string | 呼び名 |
| `personas.*.first_person` | string | 一人称 |
| `personas.*.topic_affinity.hobby` | string[] | 趣味・特技タグ |
| `personas.*.topic_affinity.personality` | string[] | 性格タイプタグ |
| `personas.*.speech.style` | string | スタイル説明 |
| `personas.*.speech.sentence_endings` | string[] | 語尾パターン |
| `personas.*.speech.calling.user` | string | ユーザーへの呼称 |

## キャラ設計のコツ

1. **バリエーションを意識**: 5キャラ以上なら話題軸（スポーツ・食・文化・知的・癒し）を散らす
2. **nickname は短く**: バトンタッチ台詞で自然に使えるように（「友紀ちゃん」「かな子ちゃん」等）
3. **sentence_endings は 3-5 個**: 多すぎると散漫になる
4. **default_persona は万能キャラ**: 社交的・知識幅が広い・明るい子が適任
5. **anti_topics を活用**: そのキャラに向かない話題を設定すると自然な橋渡しが生まれる
