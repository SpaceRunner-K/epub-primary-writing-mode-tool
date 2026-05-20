# EPUB Primary Writing Mode Tool

**EPUB の `content.opf` に `primary-writing-mode` メタデータを追加・更新するブラウザ完結ツールです。**  
GitHub Pages で動作します。サーバーサイド処理は不要です。

🔗 **[ツールを開く](https://spacerunner-k.github.io/epub-primary-writing-mode-tool/)**

---

## 概要

EPUB の package document (`content.opf`) の `<metadata>` セクションに次の要素を追加または更新します。

```xml
<meta name="primary-writing-mode" content="horizontal-rl"/>
```

対応する `content` 値：

| 値 | 意味 |
|---|---|
| `horizontal-rl` | 横書き（左→右）|
| `horizontal-lr` | 横書き（左→右、RTL系）|
| `vertical-rl` | 縦書き（右→左方向）|
| `vertical-lr` | 縦書き（左→右方向）|

---

## 使い方

1. [ツールページ](https://spacerunner-k.github.io/epub-primary-writing-mode-tool/) を開く
2. `.epub` ファイルを選択
3. `content` 値をドロップダウンで選択（デフォルト: `horizontal-rl`）
4. **OPF を確認** で変更内容をプレビュー
5. **追加 / 更新して保存** で修正済み EPUB をダウンロード

---

## 仕組み

```
.epub (ZIP)
 └─ META-INF/container.xml  → package documentのパスを取得
      └─ content.opf         → <metadata> に meta 要素を追加/更新
                               → XMLSerializer で再シリアライズ
 → 再ZIP（mimetypeはSTORE/先頭を維持）→ ダウンロード
```

- EPUB は ZIP 形式なので [JSZip](https://stuk.github.io/jszip/) で展開・再構築
- `container.xml` から `full-path` を読み OPF を特定（パスがルート固定でない場合にも対応）
- `DOMParser` + `XMLSerializer` で XML を安全に編集
- `mimetype` は OCF 仕様に従い先頭・無圧縮（`STORE`）で格納し直す
- 既に同じ `name` の `meta` がある場合は追加ではなく上書き（重複防止）

---

## 注意事項

- `primary-writing-mode` は EPUB 3.3 の中核語彙として標準化されたメタデータではなく、一部リーダー・プラットフォームが参照する実装依存の値です
- 実際の書字方向は本文 XHTML/CSS の `writing-mode` プロパティに依存するため、この meta を追加しただけで表示が変わるとは限りません
- Apple Books では書字方向を CSS で設定するよう案内されています
- 変更されるのは OPF のみです。本文 XHTML/CSS/画像などには一切手を加えません

---

## 技術スタック

- 純粋な HTML + JavaScript（依存ライブラリ: JSZip CDN のみ）
- サーバー不要・静的ホスティング対応
- ライト/ダークモード対応

---

## ライセンス

MIT License
