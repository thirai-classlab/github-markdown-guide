# GitHub で PlantUML をプレビューする方法

## 目的
GitHub は標準で PlantUML をレンダリングしないため、`.md` に埋め込んだ図をプレビューさせるための実用的な回避策をまとめる。

## 背景
- GitHub の Markdown レンダラーは Mermaid のみネイティブサポート
- PlantUML コードブロックはそのままテキスト表示される
- 社内ドキュメントやアーキ図を GitHub 上で閲覧できるようにしたい

## やること
本 README に 3 つの方式のサンプルを記載:

1. [方式A: PlantUML サーバー経由の画像埋め込み](#方式a-plantuml-サーバー経由)
2. [方式B: GitHub Actions で `.puml` → SVG を自動生成](#方式b-github-actions-で自動レンダリング)
3. [方式C: Mermaid に移行（推奨）](#方式c-mermaid-に移行推奨)

---

## 方式A: PlantUML サーバー経由

`http://www.plantuml.com/plantuml/svg/<encoded>` の URL を Markdown の画像記法で埋め込む。

### 手順
1. PlantUML コードを書く
2. [PlantUML Text Encoder](https://kkeisuke.github.io/plantuml-online-encoder/) などで encode する
   - または `plantuml-encoder` (npm) をローカルで使う
3. 画像URLを Markdown に埋め込む

### サンプル

```
@startuml
actor User
participant "Next.js" as Next
database "microCMS" as CMS

User -> Next: リクエスト
Next -> CMS: 記事取得
CMS --> Next: JSON
Next --> User: HTML
@enduml
```

↑ を encode したURLを画像として埋め込む:

```markdown
![sequence](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuNBAJrBGjLDmpCbCJbMmKiX8pSd9vt98pKi1IW80)
```

実際の見え方:

![sequence](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuNBAJrBGjLDmpCbCJbMmKiX8pSd9vt98pKi1IW80)

### メリット / デメリット
- ✅ セットアップ不要、すぐ動く
- ❌ 外部サーバー依存（社内秘の図をパブリックサーバーに送るリスク）
- ❌ URL が長くなる・図を書き換えるたびに再 encode が必要

> **注意**: 機密情報を含む図は `plantuml.com` に送らないこと。自社でPlantUML サーバーをホストするか、方式B/Cを使う。

---

## 方式B: GitHub Actions で自動レンダリング

`.puml` ファイルを commit すると Actions が SVG を生成して同じディレクトリに commit し直す方式。

### ディレクトリ構成例
```
docs/
├── architecture.md
└── diagrams/
    ├── sequence.puml
    └── sequence.svg   ← Actions が自動生成
```

### `.github/workflows/plantuml.yml`

```yaml
name: Render PlantUML

on:
  push:
    paths:
      - '**/*.puml'

jobs:
  render:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Generate SVG from PlantUML
        uses: cloudbees/plantuml-github-action@master
        with:
          args: -v -tsvg docs/diagrams/*.puml

      - name: Commit generated SVG
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add docs/diagrams/*.svg
          git diff --staged --quiet || git commit -m "chore: auto-render PlantUML diagrams"
          git push
```

### Markdown での参照

```markdown
![architecture](./diagrams/sequence.svg)
```

### メリット / デメリット
- ✅ 機密情報が外部に出ない
- ✅ レビュー時に図の差分が SVG で見える
- ❌ セットアップが必要
- ❌ Actions 実行コストがかかる

---

## 方式C: Mermaid に移行（推奨）

GitHub がネイティブ対応しているため、**追加ツール不要**でレンダリングされる。

### サンプル: シーケンス図

````markdown
```mermaid
sequenceDiagram
    actor User
    participant Next as Next.js
    participant CMS as microCMS

    User->>Next: リクエスト
    Next->>CMS: 記事取得
    CMS-->>Next: JSON
    Next-->>User: HTML
```
````

実際の見え方:

```mermaid
sequenceDiagram
    actor User
    participant Next as Next.js
    participant CMS as microCMS

    User->>Next: リクエスト
    Next->>CMS: 記事取得
    CMS-->>Next: JSON
    Next-->>User: HTML
```

### サンプル: クラス図

````markdown
```mermaid
classDiagram
    class Article {
        +string id
        +string title
        +string body
        +publish()
    }
    class Category {
        +string name
    }
    Article --> Category
```
````

```mermaid
classDiagram
    class Article {
        +string id
        +string title
        +string body
        +publish()
    }
    class Category {
        +string name
    }
    Article --> Category
```

### サンプル: フローチャート

````markdown
```mermaid
flowchart LR
    A[記事作成] --> B{レビュー}
    B -->|OK| C[公開]
    B -->|NG| A
```
````

```mermaid
flowchart LR
    A[記事作成] --> B{レビュー}
    B -->|OK| C[公開]
    B -->|NG| A
```

### Mermaid でできること
- フローチャート、シーケンス図、クラス図、ER図、状態遷移図、ガントチャート、ユーザージャーニー、C4モデル など
- [公式ドキュメント](https://mermaid.js.org/intro/)

### メリット / デメリット
- ✅ 追加ツール・CI 不要
- ✅ 機密情報が外部に出ない
- ✅ PR レビューでそのまま見える
- ❌ PlantUML ほど表現力は高くない（特殊な図は描けない）

---

## 結論・推奨

| ユースケース | 推奨方式 |
|------|------|
| 新規で図を書き始める | **方式C (Mermaid)** |
| 既存の `.puml` 資産が大量にある | **方式B (Actions)** |
| PoC・個人メモ・公開しても問題ない図 | 方式A |

迷ったら **Mermaid** 一択。

## フォルダ構造
```
plantuml-github-sample/
└── README.md  ← このファイル
```
