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
- インデックスはFernetを使用して暗号化保存
- shell=True 不使用、Atomic Write採用

脆弱性を発見した場合は、GitHubの「Private vulnerability reporting」から非公開で報告してください。

公開Issueには、脆弱性の詳細を投稿しないでください。

- 48時間以内に確認
- 修正まで詳細の非公開にご協力ください
- 協力いただいた方は謝辞に記載する場合があります
