# File Search Explorer

Windows向けのファイル検索・管理デスクトップアプリです。フォルダを指定してファイル名・フォルダ名・ファイル内容をすばやく検索し、プレビュー・重複ファイル検出・インデックス検索などの機能を備えています。

**現在のバージョン: v0.8.0**

## 主な機能

- 通常検索（並列走査）・インデックス検索（SQLite / FTS5 trigram対応）の切り替え
- 部分一致・完全一致・正規表現・あいまい検索（誤字補正）の4種類の一致方法
- ファイル内容の全文検索（テキスト系拡張子、UTF-8 / UTF-16 / CP932対応）
- 拡張子フィルタ・除外フォルダ・サイズ範囲・更新日時／作成日時範囲での絞り込み
- 検索結果の簡易プレビュー（テキスト／画像）とハッシュ値計算（MD5 / SHA-1 / SHA-256）
- 重複ファイル検索（ハッシュ比較）
- インデックスDBの暗号化保存（終了時に暗号化、次回起動時に復号）
- ライト／ダークテーマ、検索条件のプリセット保存、検索速度履歴

## 動作環境

- Windows 10 / 11
- 追加のPython環境は不要です（インストーラーに同梱されています）

## インストール

配布されているインストーラーを実行し、画面の指示に従ってインストールしてください。ソースコードからのビルド・実行はサポート対象外です。

## バグ報告・機能要望

GitHub Issues にて受け付けています。

https://github.com/Unknown777hello/FileSearchExplorer/issues

このリポジトリはIssueトラッカーとしてのみ運用しており、ソースコードは公開していません。

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

## 作者

Unknown777
