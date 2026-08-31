# FileSearchExplorer v0.9.4

### 高速・軽量なWindows向けファイル検索アプリ

Python + SQLite FTS5を使用した、ファイル名・ファイル内容を高速に検索できる完全オフライン検索アプリです。

- ✅ 完全オフライン / 外部送信なし
- ✅ ファイル名・中身の全文検索 (SQLite FTS5)
- ✅ 暗号化インデックス対応

**最新安定版：v0.9.4** | **v0.9.5 開発中（ローカルで洗練中）**

## ⬇ ダウンロード

[最新版 v0.9.4 をダウンロード](https://github.com/Unknown777hello/FileSearchExplorer/releases/tag/v0.9.4)

- Windows 10 / 11 (64bit) 対応
- インストーラー版あり
- インターネット接続不要

## 今回のアップデート (v0.9.4)
- 起動時のモード選択画面を2段階選択に変更
- 検索画面の「検索モード：簡易/難しい」切り替えボタンを廃止し、ホーム画面で選択する方式に変更
- プレビュー機能の安定性向上

## 検索画面のスクリーンショット
<img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/5f196c7e-0f21-4294-910d-e6f2c96a817e" />

## 主な機能
- ファイル名・中身の全文検索 (SQLite FTS5)
- ドラッグ&ドロップ対応 (tkinterdnd2)
- 画像プレビュー (Pillow)
- 暗号化設定の保存 (cryptography)
- デスクトップ通知 (plyer)
- インストーラー版 / Pythonソース版の2形態で提供
- 3モード選択：軽量 / フル / 難しいモード(機能選択)

## インストール方法
### インストーラー版
1. `FileSearchExplorer_Setup.exe` をダウンロードして実行
2. 画面に従ってインストール
3. スタートメニューから起動

インストール後、以下のファイルがアプリフォルダに配置されます：
FileSearchExplorer.exe
README.txt
LICENSE.md
THIRD-PARTY-LICENSES.txt
LICENSES/
app_icon.ico

### Python版 (v0.9.4-python以降)

pip install cryptography tkinterdnd2 plyer Pillow
python file_search_explorer.py


## 使い方
1. 検索対象フォルダを指定
2. キーワードを入力して検索
3. 結果をダブルクリックで開く / 右クリックで操作
4. モード選択で用途に応じて切り替え

## ライセンス

### 同梱ファイルについて
公開ZIP (FileSearchExplorer_vX.X.X.zip) と Inno Setup インストーラー (FileSearchExplorer_Setup.exe) には、以下のライセンスファイルが同梱されています。
- LICENSE.md : 自作コードのライセンス (FileSearchExplorer License v1.3)
- THIRD-PARTY-LICENSES.txt : 第三者ライブラリのライセンス一覧
- LICENSES/ : 第三者ライセンス原文
- README.txt

### 自作コード
本プロジェクトの自作コードには、独自ライセンス「FileSearchExplorer License v1.3」を適用します（MITではありません）。
Copyright (c) 2026 Unknown777hello (aka Unknown777)

主な条件（詳細は LICENSE.md を必ず確認）:
- 個人の非商用利用は無償・自由
- Python版は学習・レビュー目的の閲覧・実行を許可
- 改造版の再配布には事前許可が必要。Forkは学習・PR目的に限り自由
- 商用利用は要許可（紹介動画・ブログは収益化しても自由）
- 現状有姿で提供、法令の範囲内で免責

### 第三者ライブラリ
詳細は THIRD-PARTY-LICENSES.txt を参照
- cryptography - Apache-2.0 OR BSD-3-Clause
- pyinstaller - GPL-2.0-only WITH Bootloader Exception
- Pillow - HPND
- plyer - MIT
- tkinterdnd2 - MIT
- SQLite / SQLite FTS5 - Public Domain

## 要件
- Windows 10 / 11 (64bit)
- Python 3.10+ (ソースから実行する場合)

## プライバシー
本アプリは完全オフラインで動作し、外部へのデータ送信は一切行いません。詳細は PRIVACY.md をご覧ください。

注意：本ツールはご自身が所有・管理するPC、または所有者から明確な許可を得た範囲でのみご使用ください。

## 作者
Unknown777hello (aka Unknown777)
Repository: https://github.com/Unknown777hello/FileSearchExplorer
