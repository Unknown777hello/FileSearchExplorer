# File Search Explorer

Windows向けのデスクトップファイル検索・管理アプリケーションです。
高速なファイル名／内容検索、インデックス検索、重複ファイル検索などの機能を備えています。

現在のバージョン: **v0.8.5**

## 動作環境

- Windows 10 / 11（優先動作確認環境）
- 配布されているインストーラーからそのままインストールして利用できます（Python環境は不要です）
- 完全オフライン動作（外部への通信・テレメトリ・アナリティクスは一切行いません）

## 主な機能一覧

- ファイル名検索（部分一致・完全一致・正規表現・あいまい検索）
- ファイル内容検索（テキスト系ファイル、複数文字コード対応）
- インデックス検索（SQLite + FTS5による高速検索、暗号化保存）
- 複数フォルダの同時検索・お気に入り・検索履歴・検索条件の保存
- 重複ファイル検索（ハッシュ比較）
- 検索結果のプレビュー表示（テキスト／画像）・CSVエクスポート
- 大量結果に対応した仮想スクロール表示
- ライト／ダークテーマ、検索完了時の通知（音・デスクトップ通知・アプリ内通知）
- 検索速度履歴に基づく所要時間・件数の予測表示

## 起動方法

配布されているインストーラーを実行し、画面の指示に従ってインストールしてください。
インストール後はスタートメニューまたはデスクトップのショートカットから起動できます。

## データの保存場所

すべてのデータはローカル環境にのみ保存され、外部へ送信されることはありません。

```
%USERPROFILE%\.file_search_explorer\
    settings.json      … アプリ設定（検索条件・履歴・お気に入り等）
    session_state.json … 前回終了時の状態
    index.db.enc        … 暗号化されたインデックスDB
    index.key           … インデックス暗号化キー
    app_errors.log      … エラーログ
```

## 注意事項

- Windowsやアプリのシステムフォルダ（Windows、Program Files、ProgramData など）に対する
  削除操作は安全のためブロックされます。
- 完全削除はゴミ箱を経由しません。実行前に必ず確認ダイアログが表示されます。
- 実行ファイル・スクリプト系の拡張子を開く際は、誤操作防止のため確認ダイアログが表示されます。

## 主なキー操作

| キー | 動作 |
| --- | --- |
| Ctrl+T | 新しいタブを開く |
| Ctrl+W | 現在のタブを閉じる |
| Ctrl+L | 検索フォルダ欄にフォーカス |
| Ctrl+F | キーワード欄にフォーカス |
| F5 | 検索を実行 |
| Esc | 検索を停止 |

## 更新履歴（抜粋）

- **v0.8.5**: アプリ終了処理の安全性強化（検索中／インデックス作成中の終了処理、
  SQLite書き込み中の終了保護、after()コールバックの後始末、暗号化・VACUUM・DB書き込みの
  終了順序の明確化）
- **v0.8.4**: インデックスDBセキュリティ強化（設定/セッションファイルの権限制限、
  低メモリ環境向けフォールバック調整など）
- v0.8.0〜v0.8.3: インデックスDBセキュリティ・設定ファイル保護・検索エンジン安定性・
  仮想スクロール／大量データ対応の改善

## 不具合報告・要望

不具合報告や機能要望は GitHub Issues までお願いします。
https://github.com/Unknown777hello/FileSearchExplorer/issues

## ライセンス

MIT License

Copyright (c) Unknown777

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

作者: Unknown777
