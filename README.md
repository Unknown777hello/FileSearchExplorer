# FileSearchExplorer v0.9.4

### 高速・軽量なWindows向けファイル検索アプリ

FileSearchExplorerは、Python + SQLite FTS5を使用した、ファイル名・ファイル内容を高速に検索できるWindows向けの完全オフライン検索アプリです。

* 完全オフラインで動作
* 外部へのデータ送信なし
* ファイル名・ファイル内容の全文検索
* SQLite FTS5による高速検索
* 暗号化インデックス対応
* Windows 10 / 11対応

**最新安定版：v0.9.4**
**v0.9.5 開発中**

---

## ダウンロード

[最新版 v0.9.4 をダウンロード](https://github.com/Unknown777hello/FileSearchExplorer/releases/tag/v0.9.4)

### 対応環境

* Windows 10 / 11
* 64bit
* インターネット接続不要
* インストーラー版あり
* Pythonソース版あり

---

## v0.9.4の変更点

* 起動時のモード選択画面を2段階選択に変更
* 検索モードの選択をホーム画面で行う方式に変更
* 検索画面から「検索モード：簡易 / 難しい」の切り替えボタンを廃止
* プレビュー機能の安定性を向上

---

## 検索モード

FileSearchExplorerには、大きく分けて2種類の検索モードがあります。

### 簡易モード

通常のファイル検索を簡単に行いたい場合に使用します。

簡易モードには、用途や検索量に応じて3つのモードがあります。

* Lite版
* Normal版
* Max版

それぞれ検索処理の負荷や検索範囲などが異なり、PC環境や用途に合わせて選択できます。

### 難しいモード

より高度な検索を行いたい場合に使用します。

複数の検索条件や高度な検索機能を利用できます。

---

## 主な機能

* ファイル名検索
* ファイル内容の全文検索
* SQLite FTS5による高速検索
* AND / OR / NOT検索
* 正規表現検索
* あいまい検索
* ドラッグ＆ドロップ対応
* 画像プレビュー
* 検索結果のプレビュー
* ファイル・フォルダ操作
* 暗号化インデックス
* 暗号化された設定の保存
* デスクトップ通知
* 検索モードの切り替え
* インストーラー版 / Pythonソース版の提供

---

## スクリーンショット

<img width="1919" height="1031" alt="FileSearchExplorer検索画面" src="https://github.com/user-attachments/assets/5f196c7e-0f21-4294-910d-e6f2c96a817e" />

---

## インストール方法

### インストーラー版

1. GitHub Releasesからインストーラーをダウンロード
2. インストーラーを実行
3. 画面の指示に従ってインストール
4. スタートメニューなどからFileSearchExplorerを起動

インストール後、アプリフォルダには以下のようなファイルが配置されます。

```text
FileSearchExplorer.exe
README.txt
LICENSE.md
THIRD-PARTY-LICENSES.txt
LICENSES/
app_icon.ico
```

---

### Python版

Python版を使用する場合は、必要なライブラリをインストールしてください。

```bash
pip install cryptography tkinterdnd2 plyer Pillow
```

その後、Pythonから起動します。

```bash
python file_search_explorer.py
```

Python版はソースコードの確認・学習・レビューなどにも利用できます。

---

## 使用方法

1. 検索対象のフォルダを指定
2. 検索モードを選択
3. キーワードを入力
4. 検索を実行
5. 検索結果を確認
6. 必要に応じて結果を開いたり操作したりする

用途に応じて「簡易」または「難しい」検索モードを選択できます。

---

## ライセンス

### 自作コード

本プロジェクトの自作コードには、独自ライセンスである「FileSearchExplorer License v1.3」を適用しています。

MIT Licenseではありません。

Copyright (c) 2026 Unknown777hello (aka Unknown777)

主な条件：

* 個人の非商用利用は無償・自由
* Python版は学習・レビュー目的の閲覧・実行を許可
* 改造版の再配布には事前許可が必要
* Forkは学習・PR目的に限り自由
* 商用利用には許可が必要
* 紹介動画・ブログなどでの紹介は、収益化されている場合でも許可
* 現状有姿で提供
* 法令で認められる範囲で免責

詳細については、必ず `LICENSE.md` を確認してください。

---

## 第三者ライブラリ

FileSearchExplorerでは、以下の第三者ライブラリを使用しています。

* cryptography - Apache-2.0 OR BSD-3-Clause
* PyInstaller - GPL-2.0-only WITH Bootloader Exception
* Pillow - HPND
* plyer - MIT
* tkinterdnd2 - MIT
* SQLite / SQLite FTS5 - Public Domain

各ライブラリのライセンスについては、`THIRD-PARTY-LICENSES.txt` および `LICENSES/` を確認してください。

---

## 配布物に含まれるライセンス

公開ZIPおよびInno Setupインストーラーには、必要なライセンス関連ファイルを同梱しています。

```text
LICENSE.md
THIRD-PARTY-LICENSES.txt
LICENSES/
README.txt
```

---

## 要件

### インストーラー版

* Windows 10 / 11
* 64bit
* Python不要
* インターネット接続不要

### Python版

* Windows 10 / 11
* 64bit
* Python 3.12.5（動作確認済み）
* 必要なPythonライブラリ

---

## プライバシー

FileSearchExplorerは完全オフラインで動作することを目的として設計されています。

アプリの通常動作において、ファイル検索などのデータを外部サービスへ送信することはありません。

詳しくは `PRIVACY.md` を確認してください。

---

## セキュリティ

セキュリティ上の問題を発見した場合は、`SECURITY.md` を確認してください。

---

## 注意事項

本ツールは、ご自身が所有・管理するPC、または所有者から明確な許可を得た範囲で使用してください。

---

## 作者

Unknown777hello
aka Unknown777

GitHub:
https://github.com/Unknown777hello/FileSearchExplorer
