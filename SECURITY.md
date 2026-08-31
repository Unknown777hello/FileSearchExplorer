# Security Policy

## Supported Versions

セキュリティアップデートは、常に最新のリリースバージョンのみを対象とします。

| Version | Supported |
| ------- | --------- |
| latest  | :white_check_mark: |
| others  | :x: |

最新版へのアップデートを推奨します。

## 本アプリのセキュリティ設計

- 完全オフライン動作、外部送信なし
- インデックスは暗号化保存 (Fernet/AES128)
- shell=True 不使用、Atomic Write採用

## Reporting a Vulnerability

脆弱性を発見した場合は、公開Issueではなく以下へ：

**連絡先:** https://github.com/Unknown777hello/FileSearchExplorer/issues
タイトルに `[SECURITY]` を付けて作成してください。

- 48時間以内に確認
- 修正まで詳細の非公開にご協力ください
- 協力いただいた方は謝辞に記載する場合があります
