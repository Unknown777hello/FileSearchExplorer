# FileSearchExplorer
### 高速・軽量なファイル検索エクスプローラー

### Python + SQLite FTS5 でファイル名・内容を高速にインデックス化し、リアルタイムに検索できます。

Python License Platform

## 主な機能
ファイル名・中身の全文検索 (SQLite FTS5)  
ドラッグ&ドロップ対応 (tkinterdnd2)  
画像プレビュー (Pillow)  
暗号化設定の保存 (cryptography)  
通知機能 (plyer)  
ポータブル版 (ZIP) とインストーラー版の両方を提供  
ダウンロード / インストール  
1. 公開ZIP版 (ポータブル)
FileSearchExplorer_vX.X.X.zip をダウンロード
解凍
FileSearchExplorer.exe を実行
2. インストーラー版
FileSearchExplorer_Setup.exe をダウンロード
実行してインストール
スタートメニューから起動
インストール後、以下のファイルがアプリフォルダに配置されます。

FileSearchExplorer.exe
README.txt
LICENSE
THIRD_PARTY_LICENSES.txt
使い方
検索対象フォルダを指定
キーワードを入力
結果をダブルクリックで開く / 右クリックで操作
ビルド方法
bat
build_all.bat
実行すると以下が行われます。

PyInstallerで dist\FileSearchExplorer\ にビルド
LICENSE と THIRD_PARTY_LICENSES.txt を dist\FileSearchExplorer\ に自動コピー
公開ZIP FileSearchExplorer_vX.X.X.zip を作成 (ZIPには両ライセンスファイルが含まれる)
Inno Setupでインストーラーを作成 (installer.iss の [Files] に両ファイルが含まれる)
ライセンス
同梱ファイルについて
公開ZIP (FileSearchExplorer_vX.X.X.zip) と Inno Setup インストーラー (FileSearchExplorer_Setup.exe) には、以下のライセンスファイルが同梱されています。

LICENSE : 自作コードのライセンス
THIRD_PARTY_LICENSES.txt : 使用している第三者ライブラリのライセンス一覧
インストール後、これらのファイルはアプリのインストール先フォルダ ({app}\LICENSE, {app}\THIRD_PARTY_LICENSES.txt) に配置されます。

自作コード
本プロジェクトの自作コード (FileSearchExplorer 本体) には MIT License を適用します。

Copyright (c) 2026 Unknown777

詳細は同梱の LICENSE ファイルを参照してください。作者名は LICENSE 内で変更可能です。

第三者ライブラリ
本ソフトは以下の第三者ライブラリを使用しています。各ライブラリには各固有のライセンスが適用されます。詳細は THIRD_PARTY_LICENSES.txt を参照してください。

cryptography - Apache-2.0 OR BSD-3-Clause
pyinstaller - GPL-2.0-only WITH Bootloader Exception (ビルドツールとして使用。生成物への同梱義務なし)
Pillow - HPND (Historical Permission Notice and Disclaimer) / MIT-CMU
plyer - MIT
tkinterdnd2 - MIT
SQLite / SQLite FTS5 - Public Domain
各ライブラリのバージョン、著作権表示、ライセンス本文は THIRD_PARTY_LICENSES.txt に記載しています。不明な情報は推測せず「要確認」として明示しています。

要件
Windows 10 / 11
Python 3.10+ (ソースから実行する場合)
requirements.txt 参照:

cryptography
Pillow
plyer
tkinterdnd2
作者
Unknown777

更新履歴
v1.0.0: 初回公開、OSSライセンス表示対応
