# File Search Explorer

Windows 10/11 向けの高速ファイル検索・管理デスクトップアプリです。

現在のバージョン: **v0.8.1**

## このリポジトリについて

このリポジトリは **Issue（不具合報告・機能要望）専用のトラッカー** です。
ソースコードそのものは公開していません。アプリ本体はビルド済みのインストーラーとして配布しています。

- 不具合を見つけた場合、要望がある場合は [Issues](https://github.com/Unknown777hello/FileSearchExplorer/issues) からご報告ください。
- 本リポジトリへの Pull Request は受け付けていません。

## 主な機能

- 高速なファイル名検索（部分一致・完全一致・正規表現・あいまい検索）
- ファイル内容検索（テキスト系ファイル対応）
- インデックス機能による高速検索（SQLite FTS5 使用、非対応環境では自動的に LIKE 検索へフォールバック）
- 検索結果の仮想スクロール表示（大量件数でも軽快に動作）
- 画像・テキストの簡易プレビュー
- 重複ファイル検索（ハッシュ比較による完全一致検出）
- 検索条件のブックマーク・プリセット保存
- ライト/ダークテーマ
- 完全オフライン動作（外部通信・テレメトリは一切行いません）

## 動作環境

- Windows 10 / 11（64bit）
- インストール不要な単一の実行ファイル、またはインストーラー形式で配布

> 本アプリは Windows 専用です。macOS・Linux には対応していません。

## インストール

配布ページ（Qiita 記事、または本リポジトリの Releases）からインストーラーをダウンロードし、
案内に従ってインストールしてください。Python 環境のセットアップは不要です。

## プライバシー・セキュリティについて

- 検索対象のファイル情報・検索キーワードなどが外部に送信されることはありません。
- インデックスデータベースは暗号化した状態でディスクに保存されます。
- 詳細な設計・セキュリティ監査の内容は Qiita 記事シリーズで解説しています。

## バージョン履歴（抜粋）

- **v0.8.1**: 設定ファイルの保存安全性・破損復旧・後方互換読込・入力値検証を強化
- v0.8.0: インデックスデータベースのセキュリティ強化（ファイル権限、暗号化まわりの堅牢化）
- v0.7.8 以前: 検索の高速化・並列化、例外処理の全面的な強化、ファイル操作の安全性向上 など

詳細は Qiita の連載記事をご覧ください。

## ライセンス

MIT License

Copyright (c) Unknown777 (KAW DAI)

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

**Unknown777**（KAW DAI）

- GitHub: [Unknown777hello](https://github.com/Unknown777hello)
- Issue報告・要望: [FileSearchExplorer Issues](https://github.com/Unknown777hello/FileSearchExplorer/issues)
