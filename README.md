# FileSearchExplorer v0.9.3

### 高速・軽量なWindows向けファイル検索アプリ

Python + SQLite FTS5を使用した、
ファイル名・ファイル内容を高速に検索できるオフライン検索アプリです。
完全オフライン
インターネット接続を必要とせず、ファイル検索処理はローカルPC上で実行されます。

**最新バージョン：v0.9.3**

## ⬇️ ダウンロード

[最新版をダウンロード](https://github.com/Unknown777hello/FileSearchExplorer/releases/tag/v0.9.3)

- Windows 10 / 11対応
- インストーラー版あり
- オフラインで動作
- ファイル名・内容を全文検索
## 今回でのアップデートでの変更内容
- 「簡易モード」と「難しいモード」を選べるようにしました
- 「難しいモード」では検索機能を個別に選べます
- 機能の一覧は横スクロールで見られます
- 「実行」ボタンを新設しました
## 検索画面のスクリーンショット
<img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/5f196c7e-0f21-4294-910d-e6f2c96a817e" />

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
LICENSE.md  
THIRD-PARTY-LICENSES.txt
## 使い方
1. 検索対象フォルダを指定
2. キーワードを入力
3. 結果をダブルクリックで開く / 右クリックで操作
## ライセンス
### 同梱ファイルについて
公開ZIP (FileSearchExplorer_vX.X.X.zip) と Inno Setup インストーラー (FileSearchExplorer_Setup.exe) には、以下のライセンスファイルが同梱されています。
・ LICENSE.md : 自作コードのライセンス  
・ THIRD-PARTY-LICENSES.txt : 使用している第三者ライブラリのライセンス一覧  
インストール後、これらのファイルはアプリのインストール先フォルダ ({app}\LICENSE.md, {app}\THIRD-PARTY-LICENSES.txt) に配置されます。
### 自作コード
本プロジェクトの自作コード (FileSearchExplorer 本体) には、独自ライセンス「FileSearchExplorer License v1.1」を適用します（MIT License ではありません）。
Copyright (c) 2026 Unknown777
主な条件（詳細は同梱の LICENSE.md を必ず確認してください）:
・ 個人による非商用利用は無償・自由（第1条）
・ ソースコードは非公開。リバースエンジニアリング・逆コンパイル等は禁止（第2条）
・ 改造版の作成・再配布には、事前に GitHub Issue での許可が必要（第3条）
・ 企業・法人・業務利用等の商用利用にも別途許可が必要（第4条）
  （紹介・レビュー目的のYouTube動画やブログ記事等は、収益化を伴っても商用利用とみなしません）
・ 本ソフトは「現状有姿」で提供され、作者は原則として責任を負いません（第5条）
詳細・全文は同梱の LICENSE.md ファイルを参照してください。連絡先は GitHub Issues です。
### 第三者ライブラリ
本ソフトは以下の第三者ライブラリを使用しています。各ライブラリには各固有のライセンスが適用されます（第8条: 第三者ソフトウェア）。詳細は THIRD-PARTY-LICENSES.txt を参照してください。
・ cryptography - Apache-2.0 OR BSD-3-Clause
・ pyinstaller - GPL-2.0-only WITH Bootloader Exception (ビルドツールとして使用。生成物への同梱義務なし)
・ Pillow - HPND (Historical Permission Notice and Disclaimer) / MIT-CMU
・ plyer - MIT
・ tkinterdnd2 - MIT
・ SQLite / SQLite FTS5 - Public Domain
各ライブラリのバージョン、著作権表示、ライセンス本文は THIRD-PARTY-LICENSES.txt に記載しています。不明な情報は推測せず「要確認」として明示しています。
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
