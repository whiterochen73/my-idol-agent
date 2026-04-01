# my-idol-agent

話題に応じてキャラクターを自動選択し、そのキャラとして会話するClaude Codeスキル。

**担当アイドル（my_idol）を設定すると、いつもその子がお出迎えしてくれます。**

## 特徴

- 話題タグ自動抽出 → キャラマッチングで最適なキャラを選択
- 2段マッチング: ルールベース（高速・再現性）→ LLM tiebreak（僅差のみ）
- 担当アイドル（my_idol）設定で「迎える→紹介→戻る」フローを実現
- 会話スタイル・一人称・語尾を忠実に再現（HER Dual-layer thinking）
- キャラセットYAMLで自由にキャラをカスタマイズ可能

## インストール

```bash
# リポジトリをclone
git clone https://github.com/whiterochen73/my-idol-agent.git

# スキルディレクトリにコピー
mkdir -p ~/.claude/skills/persona-agent
cp -r my-idol-agent/* ~/.claude/skills/persona-agent/
```

## 使い方

Claude Code で以下のキーワードで起動:

- 「ペルソナエージェント」「キャラと話したい」
- 「アイドルと話したい」「推しと話したい」
- `persona agent`

### 初回起動フロー

1. キャラセットを自動読み込み（`character_sets/` 配下の `.yaml`）
2. 担当アイドル（my_idol）を設定するか聞かれます
3. 担当を設定すると、次回からその子がお出迎えします
4. スキップしてもそのまま会話できます

### 会話例

```
あなた: 「最近プロ野球が開幕したね」

→ スポーツ系キャラが自動選択:
  未央: 「スポーツの話なら加蓮ちゃんに任せよう！加蓮〜！」
  加蓮: 「野球かぁ。開幕したんだね。どのチームが好きなの？」
```

## キャラセットの作成

`templates/minimal_5chars.yaml` をコピーして `character_sets/` に保存してください。

```
character_sets/
└── my_characters.yaml   ← ここに保存
```

### 参照ファイル

| ファイル | 内容 |
|---------|------|
| `references/schema.md` | フィールド詳細・必須項目一覧 |
| `references/matching-logic.md` | マッチングアルゴリズム詳細 |
| `references/tag-dictionary.yaml` | 推奨タグ一覧 |
| `templates/minimal_5chars.yaml` | 最小構成テンプレート（5キャラ） |
| `templates/full_example.yaml` | フル構成テンプレート（10キャラ・全フィールド使用例） |

## サンプルキャラセット

`character_sets/cinderella_girls.yaml` — アイドルマスターシンデレラガールズ 10キャラ

| キャラ | 得意話題 |
|--------|---------|
| 本田未央 | エンタメ全般（デフォルト） |
| 島村卯月 | アイドル・応援・一生懸命系 |
| 渋谷凛 | ファッション・自然・読書 |
| 新田美波 | 相談・文学・知的な話 |
| 緒方智絵里 | 植物・動物・自然 |
| 十時愛梨 | スイーツ・カフェ・かわいいもの |
| 道明寺歌鈴 | クラシック音楽・和・歴史 |
| 荒木比奈 | マンガ・アニメ・ゲーム |
| 大石泉 | 読書・料理・論理系 |
| 北条加蓮 | スポーツ・水泳・ファッション |

## ディレクトリ構成

```
~/.claude/skills/persona-agent/
├── SKILL.md                       # スキル本体（Claude Codeが読む）
├── character_sets/
│   └── cinderella_girls.yaml      # サンプルキャラセット
├── references/
│   ├── schema.md                  # キャラセットスキーマ定義
│   ├── matching-logic.md          # マッチングロジック詳細
│   └── tag-dictionary.yaml        # 推奨タグ辞書
└── templates/
    ├── minimal_5chars.yaml        # 最小構成テンプレート
    └── full_example.yaml          # フル構成テンプレート
```
