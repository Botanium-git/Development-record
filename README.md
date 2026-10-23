# Development Record

Scarlet X と関連開発（SCARLET-X-Probe、公式X解析）の**横断的な開発記録**。

## ここから読む

- [開発の全体像・進捗](SCARLET-X/STATUS.md)
- [開発年表](SCARLET-X/TIMELINE.md)
- [決定事項と設計上の制約](SCARLET-X/DECISIONS.md)
- [これからの作業と未解決事項](SCARLET-X/NEXT.md)

## 記録のルール

1. **事実・推測・未検証**を分離する。GitHub Actions成功と実機での動作確認を混同しない。
2. 日付・対象リポジトリ・ブランチ・コミット・変更意図・結果・未解決事項を、確認できる限り残す。
3. アプリのコード・解析証拠の正本は各開発リポジトリ側に置く。ここは**索引と意思決定・進捗の横断的な記録**。
4. 初期記録は**これまでのチャット／公開済みGitHub記録から確認できる範囲をまとめたもの**。過去の全コミットを完全網羅したという意味ではない。
5. 作業の節目に追記する。ユーザーが意図的に変更したUIは勝手に修正対象としない。
6. 記録の修正は上書きで事実を消さず、訂正経緯も残す。

## リポジトリ

- [Scarlet X](https://github.com/Botanium-git/SCARLET-X) — アプリ本体と公式X静的解析の台帳
- [SCARLET-X-Probe](https://github.com/Botanium-git/SCARLET-X-Probe) — 公式X iOS側の読み取り専用動的診断
- [設計方針・引継ぎ](https://github.com/Botanium-git/SCARLET-X/blob/develop/Docs/ScarletXArchitectureAndHandoff.md) — 製品方針の正本

## 移管された解析資料（2026-10-23）

Scarlet X の `main` / `develop` から、解析専用のファイルを移動。**アプリ用の診断ソースコードは移動していない**。

- [公式X解析の台帳・証拠（6ファイル）](SCARLET-X/OfficialXResearch/OfficialXFindingsLedger.md)
- [解析用Pythonスクリプト（9本）と説明](SCARLET-X/ResearchTools/README.md)
- [旧main版の説明（差分保存）](SCARLET-X/ResearchTools/README-main-baseline.md)

移管元と移管先のblob SHA照合を実施してから削除。ファイルは移動先に残し、Scarlet Xの本体ビルドに含まれる診断用 `.m/.h` ファイルは維持した。
