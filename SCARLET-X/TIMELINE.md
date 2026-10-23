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

### 2026-10-23 — 公式Xプロフィールbioレイアウト所有クラスの確定
- 対象: 公式X v12.25 IPA内`T1Twitter`、SQLite索引。Scarlet X本体・Probeコード変更なし。
- 方法: `bioLabelFrame`のmethod list `0x13E7490`からMach-O Objective-C class_roとクラス名ポインタを直接照合。
- **確認済み:** 所有クラスは`T1UserViewLayoutState`（class `0x1AF2688`, class_ro `0x19ABE10`, 51 methods）。[調査の根拠](OfficialXResearch/ProfileBioLayoutOwner20261023.md)。
- **未確認:** レイアウト値の生成・最終適用・実機測定。ビルド／実機検証は今回対象外。
- 次: `applyToUserView:` と `setBioLabelFrame:` の利用経路を追う。

### 2026-10-23 — bio CGRect生成元の候補を区別
- 対象: 公式X v12.25 `T1Twitter` Mach-OとSQLite索引。Scarlet X本体・Probeの変更なし。
- 確認: `_t1_layoutBio:user:width:yCoordinate:` (`0x1C0F70`) がbio文字列取得、空判定、`fitToWidth:`、`actualSize`の後に`setBioFrame:`へCGRectを渡す経路。
- クラス所有者: `T1UserRecommendationViewLayoutDelegate` → `T1UserRecommendationViewLayoutState`。
- **重要:** これはユーザーおすすめビュー用。通常プロフィールの `T1UserViewLayoutState.bioLabelFrame` との関連は未確認。類似名による取り違えを避ける。
- 根拠: [ProfileBioFrameProducer20261023.md](OfficialXResearch/ProfileBioFrameProducer20261023.md)。次は通常プロフィール側 `setBioLabelFrame:` の呼出元を追う。

### 2026-10-23 — 通常プロフィールbio setterの直接BL探索
- 公式X v12.25 `T1Twitter`のARM64実命令を走査。 `setBioLabelFrame:` stub `0x1337B60` とIMP `0x3AFDE8`への直接BLはいずれも0件。getterへは1件で、既出のapply経路と一致。
- **未確定:** 動的ObjC呼出、他framework、直接ivar書込、値の計算元。
- [詳細](OfficialXResearch/ProfileBioSetterCallers20261023.md)。SCARLET-X・Probeコード変更なし。

### 2026-10-23 — SE不使用の追加静的解析
- 公式X v12.25 IPA実物、SQLite `objc_methods`、ARM64命令を確認。
- `_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:`（IMP `0x141CD8`）が、username/fullname等の計測APIと同一method listにあることを確認。
- 画面ごとの利用先、所有クラス、bio CGRectの計算規則は未確定。[詳細](OfficialXResearch/BioSizingAlternative20261023.md)。SCARLET-X/Probeのコード変更なし。

### 2026-10-23 — T1UserViewのbioフォントfallback分岐確認
- X v12.25 ARM64命令より`metricsFont`がnilの場合のみ`xds_body`を呼び、`lineHeight`はmetricsから独立取得。具体値・受信クラス未確定。
- [記録](OfficialXResearch/T1UserViewFontFallback20261023.md)。Scarlet X/Probeのコード変更・実機確認なし。

### 2026-10-23 — bio metrics伝搬とUIFont fallback解決
- 公式X v12.25 T1Twitter実命令／chained fixupsを解析。`layoutMetrics:`引数から`metricsFont`・`lineHeight`へ到達するデータフロー、およびnil fallback先の`_OBJC_CLASS_$_UIFont`を確認。
- 実際のフォント・行高の数値とmetrics初期化元は未確認。 [証拠](OfficialXResearch/T1UserViewMetricsProvenance20261023.md)。アプリ・Probe変更なし。

### 2026-10-23 — T1UserView bio component=4を実命令から確定
- X v12.25 T1Twitterのジャンプテーブル `0x144DEE1` を実バイト読取り。component4→`0x1416D8`→ObjC stub `0x12C9780`→`_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:`をselrefで確認。
- 既存OX-020/021は投稿本文側の解析として参照し、bioと同じレイアウト規則と断定しない。[証拠](OfficialXResearch/T1UserViewBioDispatcher20261023.md)。本体・Probe変更なし。

### 2026-10-23 — bioサイズ計測callerを特定
- X v12.25 T1Twitterのselector参照（`0x1ABE738`）からstub `0x13595C0`を解決。direct BL `0x2526D8`を確認。
- 呼出元は `T1UserViewLayoutParameters.sizeForComponent:withSize:layoutMetrics:`（IMP `0x252654`）。layoutMetricsはこの呼出元のx3からx4へ伝搬。method list `0x13E7400` とclass_ro `0x1988298`によりクラス所有を確認。
- 生成元・通常プロフィールへの実際の利用、bio CGSize→CGRect経路は未確定。[記録](OfficialXResearch/T1UserViewBioCaller20261023.md)。Scarlet X/Probe変更・ビルド・実機検証なし。

### 2026-10-23 — bio component 4 callsite確定・関数境界誤認訂正
- `0x483D8C`で`mov w2,#4`→`sizeForComponent:withSize:layoutMetrics:`を実命令確認。`0x4835C8..0x484D38`の13件がcomponent 0〜12をそれぞれ受け持つ。
- **訂正:** 13件は個別の関数にまたがるため、1つの`sizeThatFitsHelper:`所有メソッドへ全件帰属する前の説明は不正確だった。[証拠](OfficialXResearch/T1UserViewBioCaller20261023.md)。
- 端末・アプリ本体・Probeは変更なし。

### 2026-10-23 — bio helperからsetter blockまでの静的接続候補
- X v12.25 T1Twitter `0x483D40..0x483DC4`実命令確認。`0x483D6C`は`tfn_flexibleLayoutViewModel`、`0x483D8C`でcomponent=4を`sizeForComponent:withSize:layoutMetrics:`へ送る。x19は関数第3引数x2。
- `0x483C58`で`0x483DC0`をblock invoke pointerへ設定し、`0x483DC0`は`setBioLabelFrame:`へのtail B (`0x1337B60`)。以前のBL検索では拾えなかった経路。保持・実行条件、CGRect値は未確定。
- [詳細](OfficialXResearch/T1UserViewBioCaller20261023.md)。Scarlet X本体/Probe変更なし。

### 2026-10-23 — flexible layout bio計測とframe listenerの両登録を解決
- T1Twitter実バイナリで`0x1365180`=`tfn_withFlexibleLayoutSizeCalculator:`、`0x1365140`=`tfn_withFlexibleLayoutFrameListener:`を解決。
- size block`0x1882490`のinvokeは`0x483D40` (component4→bio計測)、frame listenerのinvokeは`0x483DC0` (`setBioLabelFrame:`)。`0x483C58`で両者登録を確認。
- 計算サイズからCGRectへの変換・layoutMetrics生成元は未確定。[詳細](OfficialXResearch/T1UserViewBioCaller20261023.md)。本体/Probe変更なし。

### 2026-10-23 — TFN bio frame listenerがCGRectを転送するのみと確認
- 提供X v12.25 IPAのARM64命令を再確認。bio size callback `0x483D40` はCGSizeを返し、frame listener `0x483DC0` はsetterへtail Bする2命令で計算式なし。TFN block型情報にもCGSize/TFNLayoutMetrics、CGRect引数が存在。
- **未確定**: 指定blockへの型情報の最終紐付け、TFNエンジン内部のCGSize→CGRect変換、親レイアウト制約・spacingの数値、実機条件。
- [詳細](OfficialXResearch/TFNFlexibleBioCallbackABI20261023.md)。Scarlet X/Probe変更なし。

### 2026-10-23 — Probe v0.1.12プロフィールクラス拡張
- GitHub developの現行ファイルをSHA確認して編集。既存投稿4クラスを残しプロフィール6クラスを追加、存在チェックと記録件数の母数を10に修正。
- ProbePanel.m commit 61a99358cc9249eb0ea4fbafc7e1dea0c26bfc8e。control commit cd01b5fdb19c2609ea59b3d033ae5e45167c0d1c。Tweak.xm commit 62acee60c2e04a2df5a4f47e554c09f06349b016。
- ソース再取得で変更内容と既存Drive/手動操作を確認。Build/SE実機の成功は未確認。Scarlet Xアプリ本体とProbe/mainは未変更。

### 2026-10-23 — 非公開GitHub Actions→Sileo用deb取得MVP
- SCARLET-X-Probe/developに認証付きダウンローダ `tools/private_probe_update.py`とガイド `tools/PRIVATE_UPDATER.md`を追加。PATを保存/コマンドラインに載せず、artifact配信先への認証転送を防止。パッケージ検証後のみプライベート領域に保存。
- commits `e91f00001ccff6959d9be0901e3e0b4cda96d2ab`, `ec28df51c546b0f51b2a02b75d9f16048c0150e5`, `e875e8b8b48639413078e55dc4ea3e46c097588f`。
- 自動インストールもショートカットも未実装。SE実行環境・DL・Sileo導入・Actionsの成否未検証。Probe本体のこの作業によるversion変更なし、main未変更。
