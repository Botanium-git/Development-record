# Development Record

Scarlet X と関連開発（SCARLET-X-Probe、公式X解析）の**開発記録と解析資料の保管場所**。

**次のチャットではまず本READMEを読み、STATUS → DECISIONS → NEXT → 必要に応じTIMELINE・OfficialXResearchを参照する。** 作業開始時には各開発リポジトリの実際のブランチ・最新コードを再確認する。ここにある状態記録を最新のコードとみなさない。

## ここから読む

- [開発の全体像・進捗](SCARLET-X/STATUS.md)
- [開発年表](SCARLET-X/TIMELINE.md)
- [決定事項と設計上の制約](SCARLET-X/DECISIONS.md)
- [これからの作業と未解決事項](SCARLET-X/NEXT.md)

## 記録のルール

1. **事実・推測・未検証**を分離する。GitHub Actions成功と実機での動作確認を混同しない。
2. 日付・対象リポジトリ・ブランチ・コミット・変更意図・結果・未解決事項を、確認できる限り残す。
3. アプリ実装コードの正本はSCARLET-X、Probe実装コードの正本はSCARLET-X-Probe。**移管済み公式X解析資料・解析スクリプトの正本はこのDevelopment-record**。実行バイナリやユーザー保持のIPA/SQLiteは別管理。
4. 初期記録は**これまでのチャット／公開済みGitHub記録から確認できる範囲をまとめたもの**。過去の全コミットを完全網羅したという意味ではない。
5. 作業の節目に追記する。ユーザーが意図的に変更したUIは勝手に修正対象としない。
6. 記録の修正は上書きで事実を消さず、訂正経緯も残す。

## リポジトリ

- [Scarlet X](https://github.com/Botanium-git/SCARLET-X) — アプリ本体（解析台帳・ツールは本リポジトリに移管）
- [SCARLET-X-Probe](https://github.com/Botanium-git/SCARLET-X-Probe) — 公式X iOS側の読み取り専用動的診断
- [設計方針・引継ぎ](https://github.com/Botanium-git/SCARLET-X/blob/develop/Docs/ScarletXArchitectureAndHandoff.md) — 製品方針の正本

## 移管された解析資料（2026-10-23）

Scarlet X の `main` / `develop` から、解析専用のファイルを移動。**アプリ用の診断ソースコードは移動していない**。

- [公式X解析の台帳・証拠（6ファイル）](SCARLET-X/OfficialXResearch/OfficialXFindingsLedger.md)
- [解析用Pythonスクリプト（9本）と説明](SCARLET-X/ResearchTools/README.md)
- [旧main版の説明（差分保存）](SCARLET-X/ResearchTools/README-main-baseline.md)

移管元と移管先のblob SHA照合を実施してから削除。ファイルは移動先に残し、Scarlet Xの本体ビルドに含まれる診断用 `.m/.h` ファイルは維持した。

## 今後の開発での記録先

- 現在地・確定事項・未解決問題は [`STATUS.md`](SCARLET-X/STATUS.md)、決定事項は [`DECISIONS.md`](SCARLET-X/DECISIONS.md)、次の作業は [`NEXT.md`](SCARLET-X/NEXT.md) を更新する。
- 実施した調査・コード変更・コミット・Actions結果・実機結果は [`TIMELINE.md`](SCARLET-X/TIMELINE.md) に時系列で追記する。
- 公式Xの新しい静的解析結果は [`OfficialXFindingsLedger.md`](SCARLET-X/OfficialXResearch/OfficialXFindingsLedger.md) 等に、根拠・対象版・確度・未確定事項を明記して追記する。
- すべての小操作を機械的に記録する必要はない。調査結果、設計判断、実装、重要な検証結果など**引き継ぎに必要な節目**で更新する。既存の記録を黙って書き換えない。
- 実装・解析結果の更新時もアプリ本体の正常動作を壊さない。`main` は安定版として扱い、アプリ変更は原則 `develop` で実施する。
