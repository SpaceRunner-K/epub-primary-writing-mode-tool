# Kindle 縦書き対策 EPUB Tool

**Kindle の縦書き表示不具合を回避するため、EPUB の `content.opf` に `primary-writing-mode` メタデータを追加・更新するブラウザ完結ツールです。**  
GitHub Pages で動作します。サーバーサイド処理は不要です。

🔗 **[ツールを開く](https://spacerunner-k.github.io/epub-primary-writing-mode-tool/)**

---

## 背景 — Kindle の縦書きバグとは

2026 年 4 月下旬の Kindle Paperwhite ファームウェアアップデート（5.19.3.0.1 確認）以降、Send to Kindle 経由で転送した日本語縦書き EPUB の表示が崩れるバグが発生しています。

具体的には画面の上下に不自然な余白（いわゆる **Death Margin**）が生じ、左右の余白がゼロになる症状です。Kindle ストアで購入した書籍では発症せず、自作・転送 EPUB のみに起こります。

### バグの推定メカニズム

日本語縦書き EPUB は、EPUB の構造仕様上 `page-progression-direction="rtl"` 属性を持ちます。日本語の縦書きは本来「右から左」にページが進行するため、電子書籍の論理構造として RTL と定義されるのが正しい挙動です。

しかし今回のファームウェアアップデート後、Kindle のレンダリングエンジンがこの RTL 属性を、**アラビア語・ヘブライ語などの横書き RTL 言語向けのレイアウト規則**として誤認識している可能性が高いと考えられます。横書き RTL 言語の組版では、行が右から左に流れる水平レイアウトが基本となるため、縦書き日本語に横書き RTL の余白・マージン規則が適用され、上下左右の余白が逆転したような表示になるのです。

### `primary-writing-mode` による回避

この誤認識を上書きするのが `content.opf` に付与する次のメタデータです。

```xml
<meta name="primary-writing-mode" content="horizontal-rl"/>
```

これは EPUB リーダーや Kindle のような読書プラットフォームに対し、「このコンテンツの主書字方向は `horizontal-rl`（縦書き・右から左方向）である」と明示する値です。`page-progression-direction` が論理的なページ送り方向を示すのに対し、`primary-writing-mode` はより具体的な**描画・組版上の書字方向**を指定するため、レンダリングエンジンが正しいレイアウト規則を選択しやすくなります。

Kindle が RTL をアラビア語系と混同する誤動作を、この明示的な書字方向指定によって正しい縦書きレイアウト規則へ誘導する — これが今回の回避策の原理です。

> **注意**: これは公式に確認されたバグ修正ではなく、症状と構造から導いた推定と回避策です。Kindle のファームウェアが修正された場合は不要になる可能性があります。

---

## 使い方

1. [ツールページ](https://spacerunner-k.github.io/epub-primary-writing-mode-tool/) を開く
2. `.epub` ファイルを選択
3. `content` 値を選択（縦書き日本語 EPUB なら **`horizontal-rl`** を推奨）
4. **保存前チェック** で OPF・mimetype・既存値・ファイルサイズ・保存方式を確認
5. **OPF を確認** で変更内容をプレビュー
6. **追加 / 更新して保存** で修正済み EPUB を保存

---

## content 値の意味

| 値 | 組版上の意味 | 用途例 |
|---|---|---|
| `horizontal-rl` | 縦書き・行は右から左 | **日本語縦書き（推奨）** |
| `horizontal-lr` | 縦書き・行は左から右 | モンゴル文字系 |
| `vertical-rl` | 横書き・右から左 | アラビア語・ヘブライ語 |
| `vertical-lr` | 横書き・左から右 | 一般的な横書き |

> ※ `primary-writing-mode` の `horizontal` / `vertical` はCSSの `writing-mode` とは軸の定義が異なります。日本語縦書きには `horizontal-rl` を選んでください。

---

## v0.2 の追加点

- 保存前チェック（OPF・mimetype・既存値・ファイルサイズ・保存方式）を追加
- JSZip の `generateAsync` 進捗バーを追加
- 対応ブラウザでは `showSaveFilePicker()` による直接保存に対応
- 非対応環境では通常ダウンロードへ自動フォールバック
- 大きい EPUB に対する注意表示を追加

---

## 仕組み

```
.epub (ZIP)
 └─ META-INF/container.xml  → package document のパスを取得
      └─ content.opf         → <metadata> に primary-writing-mode を追加/更新
                               → XMLSerializer で再シリアライズ
 → 再ZIP（mimetypeはSTORE/先頭を維持）→ 保存
```

- EPUB は ZIP 形式なので [JSZip](https://stuk.github.io/jszip/) で展開・再構築
- `container.xml` から `full-path` を読み OPF を特定
- `DOMParser` + `XMLSerializer` で XML を安全に編集
- `mimetype` は OCF 仕様に従い先頭・無圧縮（`STORE`）で格納し直す
- 既存の同名 `meta` がある場合は追加ではなく上書き（重複防止）

---

## 注意事項

- これは **Kindle の縦書き表示バグに対する回避策**であり、恒久的な修正ではありません
- 変更されるのは OPF のみです。本文 XHTML・CSS・画像には一切手を加えません
- `showSaveFilePicker()` は HTTPS の secure context と対応ブラウザが必要です
- 巨大な EPUB はブラウザのメモリ消費が増えます
- 仕上がり確認には [EPUBCheck](https://github.com/w3c/epubcheck/) の利用を推奨します

---

## 技術スタック

- 純粋な HTML + JavaScript（依存ライブラリ: JSZip CDN のみ）
- サーバー不要・GitHub Pages 対応
- ライト / ダークモード対応

---

## ライセンス

MIT License
