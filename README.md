# File Search Explorer

Windows向けのファイル検索・管理デスクトップアプリケーションです。Python (Tkinter) 製で、単体のWindowsインストーラーとして配布されており、別途Pythonをインストールする必要はありません。

現在のバージョン: **v0.7.8**

## 動作環境

- Windows 10 / Windows 11(64bit推奨)
- Python 本体のインストールは不要(単体のインストーラーとして配布)

## 主な機能一覧

- 高速なファイル検索(通常検索 / 事前インデックスを使った高速検索)
- 検索の並列化(`ParallelSearchWorker`)による大量ファイルの高速走査
- 部分一致・完全一致・正規表現・あいまい検索(誤字補正)などの一致方式
- ファイル内容検索(テキスト系ファイルの中身も検索対象に)
- サイズ・更新日時・作成日時での絞り込み
- 拡張子指定・除外拡張子・除外フォルダ指定
- 複数フォルダの同時検索
- 検索結果のCSVエクスポート(大量件数は自動でバックグラウンド処理)
- 検索結果のプレビュー表示(テキスト・画像)、ファイルハッシュ計算(MD5/SHA-1/SHA-256)
- 重複ファイル検索(ハッシュ比較)
- インデックス管理(作成・更新・VACUUM・整合性チェック・暗号化保存)
- 検索速度履歴の記録と、次回検索時の所要時間・件数の予測表示
- 検索条件のプリセット保存・お気に入りフォルダ・最近使ったフォルダ
- ライト/ダークテーマ切り替え
- 検索完了時の通知(通知音・デスクトップ通知・アプリ内ポップアップ)

## 起動方法

インストーラーでインストールした場合は、スタートメニューまたはデスクトップのショートカットから起動してください。

## データの保存場所

アプリのデータは、ユーザーのホームディレクトリ配下の `.file_search_explorer` フォルダに保存されます。

- `index.db` / `index.db.enc`: 検索インデックス(終了時に暗号化して保存)
- `index.key`: インデックス暗号化用の鍵ファイル
- `settings.json`: アプリの各種設定
- `session_state.json`: 前回終了時の状態(異常終了検知用)
- `app_errors.log`: エラーログ(トラブル時の調査用)

## 注意事項

- 本アプリはオフラインで動作し、外部への通信は一切行いません。
- ログファイル(`app_errors.log`)には、調査に必要な範囲でファイルパスや例外内容を記録しますが、検索キーワードやファイルの中身、パスワード等の秘密情報は記録しません。また、ユーザー名を含むホームディレクトリ部分は自動的にマスキングされます。
- Windows・Program Files・ProgramData などのシステム保護フォルダは、誤削除防止のため重複ファイル検索の対象から自動的に除外されます。
- 実行ファイル・スクリプト(.exe, .bat, .ps1 等)を開く際は、誤操作防止のため必ず確認ダイアログが表示されます。

## 主なキー操作

| キー | 動作 |
|---|---|
| Ctrl + T | 新しいタブを開く |
| Ctrl + W | 現在のタブを閉じる |
| Ctrl + L | 検索フォルダ欄にフォーカス |
| Ctrl + F | キーワード欄にフォーカス |
| F5 | 検索を実行 |
| Esc | 検索を停止 |

## v0.7.8での変更点(セキュリティ基礎強化)

- 外部通信・`shell=True`の使用有無を全ソースに対して監査し、いずれも該当箇所がないことを確認しました。
- ログ出力からユーザー名(ホームディレクトリ由来)を自動マスキングするようにしました。
- ファイルを開く際の外部コマンド呼び出しに、入力値検証による多重防御を追加しました。
- 上記に伴い、既存の検索・インデックス・UIの動作や操作感に変更はありません。

## ライセンス

MIT License

Copyright (c) Unknown777

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## 作者・連絡先

作者: Unknown777
リポジトリ: https://github.com/Unknown777hello/FileSearchExplorer
不具合報告・要望: GitHub Issues (https://github.com/Unknown777hello/FileSearchExplorer/issues)
