# Scarlet X — 開発年表

**注意:** これまでのチャットと確認済みGitHub記録をもとに、重要な節目を再構成。全変更履歴を漏れなく復元したものではない。各項目の根拠レベルを明示する。

| 時期 | 出来事 | 根拠・確度 |
|---|---|---|
| 2026年9月 | Web版Xを土台にしたScarlet X開発。独自メニュー、プロフィール・投稿、診断、Google Driveの機能を段階的に追加 | 過去のプロジェクト記録。詳細コミットは要照合 |
| 2026年10月初旬 | プロフィール一覧のDirectTable構造、手動フレーム・高さ、読み込み・タブ動作などを改善。v1.4.118で文字サイズが良好との実機評価あり | 過去のユーザーフィードバック。現在の最新版の正否を保証しない |
| 2026-10-19〜22 | 公式X v12.25 IPAの静的解析を蓄積し、公式X解析台帳 OX-001〜152へ（OX-150独立エントリは未保存） | [公式解析台帳](https://github.com/Botanium-git/Development-record/blob/main/SCARLET-X/OfficialXResearch/OfficialXFindingsLedger.md) |
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

### 2026-10-23 — SCARLET-X-Probe 0.1.11 (SE実機未検証)
- 対象: `Botanium-git/SCARLET-X-Probe` の `develop`。
- 理由: 「クラスメソッド情報を記録」ボタンを毎回押す必要をなくす。
- 変更: メタデータ記録ON時、公式Xの起動後に既存4クラスのメソッド情報をプロセスにつき1回自動記録。ON切替時にも収集。手動操作・Drive送信は維持。
- 対象ファイル: `ProbePanel.m`, `Tweak.xm`, `control`。バージョン 0.1.10 → 0.1.11。
- コミット: `e337c04f`, `27500bcc`, `f30289e0`。
- ビルド: GitHub Actions run `39877632072`（確認時 in_progress）。実機動作: 未検証。
- 次: Actionsの結果確認。SEで新バージョンを適用した後、メタデータONで起動し、手動記録ボタンなしに診断JSONの `method-probe-completed` を確認。

### 2026-10-23 — 解析資料移管の再確認とドキュメント整理
- 対象: `Botanium-git/Development-record` の `main`。Scarlet Xのアプリ実装コードは変更しない。
- 確認済み: `OfficialXResearch` の解析報告6件、`ResearchTools` のPythonスクリプト9本、develop由来README、旧main由来 `README-main-baseline.md` が存在する。
- 確認済み: Scarlet X `main` 上の旧 `ResearchTools/README.md` は既に存在せず、移行先の旧main版は保存済み。再削除は不要。
- 修正: 年表の旧台帳リンクを移管先に変更。Development-recordの入口・運用方針、解析ツールREADME内の古いリンクを整理。
- 留保: リポジトリ全ファイルの再帰的な一覧比較を完了したわけではない。確認済み対象以外に資料が残っていないとまでは断定しない。
- 次: 両ブランチの残存ファイル・参照整合を引き続き点検。開発の節目には本年表と必要な状態・解析資料を更新。
