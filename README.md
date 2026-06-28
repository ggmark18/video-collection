# Video Gallery

動画リンクをグループ別にカード一覧表示するシンプルなHTMLギャラリーです。  
YouTube・Vimeo等の外部サービスURLや直接URLに対応しています。

## ファイル構成

```
hobby/
├── index.html       # メインページ
├── videos.json      # 動画リスト（グループ・アイテム定義）
├── urlformat.json   # URL・サムネイルのテンプレート定義
└── README.md
```

## 起動方法

`fetch` を使用しているため、ローカルサーバーが必要です。

```bash
python3 -m http.server 8080
# → http://localhost:8080 で確認
```

---

## videos.json

動画リストをグループ単位で定義します。

### 構造

```json
[
  {
    "group": "グループ名",
    "items": [ /* アイテムの配列 */ ]
  }
]
```

### アイテムのフィールド

| フィールド | 必須 | 説明 |
|---|---|---|
| `id` | 任意 | 任意の文字列ID。サムネイル未指定時のフォールバック画像シードに使用 |
| `title` | 必須 | カードに表示するタイトル |
| `urlformat` | 必須 | `urlformat.json` で定義したフォーマットキー（`"direct"` は直接指定） |
| `key` | 通常必須 | URLとサムネイル生成に使うサービス固有ID |
| `thumb_key` | 任意 | `key` と異なるサムネイル用IDを使いたい場合に指定 |
| `originalurl` | 任意 | オリジナルページのURL。指定するとカード右上にアイコンが表示され、クリックで別タブに開く |
| `category` | 任意 | カテゴリ名（カードにバッジ表示） |
| `person` | 任意 | 出演者・投稿者名（カードに表示） |

### urlformat ごとの書き方

**YouTube:**
```json
{
  "id": "music-001",
  "title": "Never Gonna Give You Up",
  "urlformat": "youtube",
  "key": "dQw4w9WgXcQ",
  "category": "Pop",
  "person": "Rick Astley"
}
```

**URLとサムネイルのキーが異なる場合（`thumb_key` を追加）:**
```json
{
  "id": "example-001",
  "title": "サンプル",
  "urlformat": "youtube",
  "key": "VIDEO_ID",
  "thumb_key": "THUMBNAIL_VIDEO_ID",
  "category": "Sample",
  "person": "Someone"
}
```

**オリジナルページへのリンクを追加（`originalurl` を指定）:**
```json
{
  "id": "music-001",
  "title": "Never Gonna Give You Up",
  "urlformat": "youtube",
  "key": "dQw4w9WgXcQ",
  "originalurl": "https://www.rickastley.co.uk",
  "person": "Rick Astley"
}
```

> `originalurl` を指定するとサムネイル右上に外部リンクアイコンが現れます。カード本体のクリックとは独立して動作します。  
> `urlformat.json` 側のテンプレートで固定URLを埋め込むこともできます（例: youtubeフォーマットで `"originalurl": "https://www.youtube.com/watch?v={key}"` と定義すれば、アイテム側に `originalurl` を書かなくても常にアイコンが表示されます）。

**direct（URLとサムネイルを直接指定）:**
```json
{
  "id": "direct-001",
  "title": "直接指定のサンプル",
  "urlformat": "direct",
  "url": "https://example.com/page",
  "thumbnail": "https://example.com/thumb.jpg",
  "category": "Other",
  "person": "Me"
}
```

---

## urlformat.json

URL・サムネイルのテンプレートをサービスごとに定義します。

### 構造

```json
{
  "フォーマットキー": {
    "url":         "URLテンプレート",
    "thumbnail":   "サムネイルURLテンプレート",
    "originalurl": "オリジナルURLテンプレート（省略可）"
  }
}
```

### テンプレート記法

- `{fieldname}` — アイテムの同名フィールドの値に置換されます
- `{a|b}` — `a` フィールドが存在すれば `a` を、なければ `b` を使うフォールバック記法
- `|` は何段階でも連結できます（例: `{thumb_key|fallback|key}`）

### 組み込みフォーマット

| キー | url | thumbnail | originalurl |
|---|---|---|---|
| `youtube` | `watch?v={key}` | `vi/{thumb_key\|key}/hqdefault.jpg` | `watch?v={key}`（固定） |
| `youtube_short` | `shorts/{key}` | `vi/{thumb_key\|key}/hqdefault.jpg` | `{originalurl}`（アイテムで指定） |
| `vimeo` | `vimeo.com/{key}` | `vumbnail.com/{thumb_key\|key}.jpg` | `{originalurl}`（アイテムで指定） |
| `direct` | `{url}` | `{thumbnail}` | `{originalurl}`（アイテムで指定） |

> `originalurl` テンプレートが空文字に解決されたアイテムにはアイコンは表示されません。

### 新しいサービスの追加例

```json
"nicovideo": {
  "url": "https://www.nicovideo.jp/watch/{key}",
  "thumbnail": "https://nicovideo.cdn.nimg.jp/thumbnails/{key}/{key}",
  "originalurl": "{originalurl}"
}
```

追加後、`videos.json` のアイテムで `"urlformat": "nicovideo"` と指定するだけで利用できます。

---

## 画面の操作

- **グループタブ**（ヘッダー）— タブをクリックするとそのグループのみ表示。「すべて」で全グループを表示。
- **カードクリック** — 対象URLを別タブで開きます。
- **右上アイコン**（マウスオーバーで表示）— `originalurl` が設定されているカードに表示される外部リンクアイコン。クリックでオリジナルページを別タブで開きます。カード本体のクリックとは独立しています。
