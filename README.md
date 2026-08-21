# FileSearchExplorer
### 高速・軽量なファイル検索エクスプローラー

### Python + SQLite FTS5 でファイル名・内容を高速にインデックス化し、リアルタイムに検索できます。

## 主な機能
・ ファイル名・中身の全文検索(SQLite FTS5)  
・ ドラッグ&ドロップ対応(tkinterdnd2)  
・ 画像プレビュー  
・ 暗号化設定の保存  
・ 通知機能(plyer)  
・ インストーラー版を提供  

## FileSearchExplorer_Setup.exe をダウンロード
1. 実行してインストール
2. スタートメニューから起動
3. インストール後、以下のファイルがアプリフォルダに配置されます。

FileSearchExplorer.exe  
README.txt  
LICENSE  
THIRD_PARTY_LICENSES.txt

## 使い方
1. 検索対象フォルダを指定
2. キーワードを入力
3. 結果をダブルクリックで開く / 右クリックで操作

## ライセンス
### 同梱ファイルについて
公開ZIP (FileSearchExplorer_vX.X.X.zip) と Inno Setup インストーラー   (FileSearchExplorer_Setup.exe) には、以下のライセンスファイルが同梱されています。

・ LICENSE : 自作コードのライセンス  
・ THIRD_PARTY_LICENSES.txt : 使用している第三者ライブラリのライセンス一覧  
インストール後、これらのファイルはアプリのインストール先フォルダ ({app}\LICENSE, {app}\THIRD_PARTY_LICENSES.txt) に配置されます。

### 自作コード
本プロジェクトの自作コード (FileSearchExplorer 本体) には MIT License を適用します。

Copyright (c) 2026 Unknown777

詳細は同梱の LICENSE ファイルを参照してください。作者名は LICENSE 内で変更可能です。

### 第三者ライブラリ
本ソフトは以下の第三者ライブラリを使用しています。各ライブラリには各固有のライセンスが適用されます。詳細は THIRD_PARTY_LICENSES.txt を参照してください。

・ cryptography - Apache-2.0 OR BSD-3-Clause
・ pyinstaller - GPL-2.0-only WITH Bootloader Exception (ビルドツールとして使用。生成物への同梱義務なし)
・ Pillow - HPND (Historical Permission Notice and Disclaimer) / MIT-CMU
・ plyer - MIT
・ tkinterdnd2 - MIT
・ SQLite / SQLite FTS5 - Public Domain

各ライブラリのバージョン、著作権表示、ライセンス本文は THIRD_PARTY_LICENSES.txt に記載しています。不明な情報は推測せず「要確認」として明示しています。

## 要件
・ Windows 10 / 11
・ Python 3.10+ (ソースから実行する場合)
requirements.txt 参照:

cryptography
Pillow
plyer
tkinterdnd2
## 作者
Unknown777
