# Scarlet X — 開発年表

**注意:** これまでのチャットと確認済みGitHub記録をもとに、重要な節目を再構成。全変更履歴を漏れなく復元したものではない。各項目の根拠レベルを明示する。

| 時期 | 出来事 | 根拠・確度 |
|---|---|---|
| 2026年9月 | Web版Xを土台にしたScarlet X開発。独自メニュー、プロフィール・投稿、診断、Google Driveの機能を段階的に追加 | 過去のプロジェクト記録。詳細コミットは要照合 |
| 2026年10月初旬 | プロフィール一覧のDirectTable構造、手動フレーム・高さ、読み込み・タブ動作などを改善。v1.4.118で文字サイズが良好との実機評価あり | 過去のユーザーフィードバック。現在の最新版の正否を保証しない |
| 2026-10-19〜22 | 公式X v12.25 IPAの静的解析を蓄積し、公式X解析台帳 OX-001〜152へ（OX-150独立エントリは未保存） | [公式解析台帳](https://github.com/Botanium-git/SCARLET-X/blob/develop/Diagnostics/OfficialXFindingsLedger.md) |
| 2026-10-21〜22 | SEとRootHideでSCARLET-X-Probeを動かす準備・検証。Probe v0.1.10のボタン・パネル表示をユーザーが確認 | [Probe](https://github.com/Botanium-git/SCARLET-X-Probe)・会話での実機報告 |
| 2026-10-22 | `main` を `develop` にfast-forward同期、SHA `19029288` | GitHubで更新・両ブランチ確認 |
| 2026-10-22 | プロフィール投稿のヘッダー→本文間隔を2→3ptに変更し、計測行高も調整。v1.4.168を `develop` に追加 | [コミット c6251002](https://github.com/Botanium-git/SCARLET-X/commit/c62510022f1273560ea5cfab2867e3324a646cc3)。表示の実機一致は未確認 |
| 2026-10-22 | 方向性を明文化：WebKit取得＋ネイティブ表示、公式Xと同じ表示決定プロセス、独自再実装可、最終的にネイティブ全体、まずプロフィール | [設計引き継ぎ](https://github.com/Botanium-git/SCARLET-X/blob/develop/Docs/ScarletXArchitectureAndHandoff.md) |
| 2026-10-22 | Windows 11メインPC（i5-12400F / RTX3060Ti / 32GB）で土日にGhidra導入を検討。それまでは既存のIPAとSEのProbeを利用して分析を継続 | 会話での計画。**導入済みではない** |
| 2026-10-22 | 専用リポジトリDevelopment-recordを開発記録拠点として初期化 | [Development-record](https://github.com/Botanium-git/Development-record) |

## 次の追記フォーマット

```md
### YYYY-MM-DD — 変更テーマ
- 対象リポジトリ／ブランチ・コミット：
- 背景／目的：
- 実際の変更（コード・ファイル）：
- 確認済み：ビルド / 実機 / ログ
- 未確認と推測：
- 次にすること：
```

同日に複数変更があればそれぞれ独立して記録する。
