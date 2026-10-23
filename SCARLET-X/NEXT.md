# Scarlet X — 次の作業・未解決事項

## P0 — SEで既存Probeを検証（現在）
- **ユーザーの確認済み事項:** SX Probeボタンと診断パネルが表示できる。
- **既存の手動操作:** パネルの「クラスメソッド情報を記録」を押す。これは `ProbePanel.m` で実装済み。
- **期待する既存コード動作:** 4クラス `TTAStatusBodyTextView` / `T1StatusCell` / `T1StandardStatusView` / `T1StatusTableRowAdapter` の情報をJSONに記録。完了ダイアログ、記録件数・最新イベントを更新。
- **検証待ち:** SEで記録完了になるか、JSONエクスポート内容に実際に何が含まれるか。これまでは未受領。
- 診断JSONの共有は「診断JSONを共有」から実施可能。アカウント情報・投稿内容の共有は不要。

## P1 — 公式Xのプロフィール表示状態を特定
- **まだ未実装:** プロフィールの自己紹介UILabel/描画ビューのフォント・描画幅・行高・frame・端末条件の収集。
- 先に既存クラス／ビューを観察して**対象ビューを識別できる根拠**を作る。推測による全UILabelフックや投稿文の収集はしない。
- ユーザーが許可し、観測対象・収集項目・オーバーヘッドを明確にした**読み取り専用Probe**を段階実装・SEで検証する。
- 公式Xの元の表示値とScarlet Xのヘッダー／DirectTableの入力・計測・最終CGRectを比較する。

## P2 — 表示規則を必要最小限実装
- プロフィール用の画面幅／フォント条件・文字計測を先行検証。複数幅で値が変わる仕組みを確認。
- 現行の正常動作を破壊しない比較実装から開始。実機一致が確認できた変更を本番採用する。
- 他のネイティブ画面へ適用を展開する。

## PC側の将来作業
- Windows 11にGhidraを導入、公式X IPAから必要関数の擬似コード・参照関係・定数を自動抽出するパイプラインを構築。
- 元のソースコード一式の完全復元ができるわけではない。自動抽出はプロフィール表示等の課題に必要な範囲へ限定する。

## 未解決
- 公式X v12.25の静的解析とSE実行中の公式Xの**バージョン差**、端末幅／文字サイズ条件の違い。
- 公式Xプロフィール自己紹介欄の具体的な対象ビューと実値。
- `additionalEdgeInsets` から最終的なCGRectへの適用経路。
- 公式Xの表示決定規則がScarlet Xで複数端末に再現できることの検証。

## 2026-10-23 — Drive受信結果（実測）
- Drive `Probe/SXProbe_2026-10-23_001123.json` を取得して確認（公式X **12.35 build 27**）。JSONは有効、`schemaVersion=1`、`entries` は **1件**。
- 唯一の記録は `recording-restored` / `enabled=true`。**今回のJSONに `method-probe-invoked` / `method-names` / `method-probe-completed` は存在しない**。したがって「クラスメソッド情報を記録」押下の成功は確認できない。
- 過去の `SXProbe_2026-10-22_031755.json` には `probe-enabled` と4件の `class-check`（各 `present=1`）がある。これはクラス存在チェックの証拠であり、メソッド列挙成功の証拠ではない。
- 次は端末で「クラスメソッド情報を記録」を押して**記録完了ダイアログ**を確認し、その直後に「診断JSONをDriveへ送信」した新しいJSONを調べる。既存Probeのコード変更は現時点では不要。

## 2026-10-23 — 公式X v12.25プロフィール自己紹介欄の静的索引調査（SE不要）

**証拠:** 手元の `scarlet_macho(1).sqlite` 内の `symbols`, `strings`, `objc_methods` の直接照会。逆アセンブリで呼び出し経路を証明したわけではない。SE実行版はv12.35でバージョン差がある。

### 確認できた候補
- `T1ProfileUserInfoView`, `T1ProfileUserInfoViewController`, `T1ProfileUserInfoViewConfiguration`, `T1ProfileHeaderView`, `T1ProfileHeaderViewController` はシンボル／文字列索引に存在する。
- プロフィール自己紹介と関連しそうなセレクタ文字列として `_t1_buildBioLabelWithAccessibilityIdentifier:`, `_t1_refreshBioLabel`, `_t1_setBioLabel:withText:entities:isUTF16Entities:`, `_t1_refreshTranslatedBioLabel`, `bioLabel`, `translatedBioLabel` を確認。ただし現SQLiteの `objc_methods` はこれらを全て網羅しておらず、名前だけで所有クラス／IMPを確定できない。
- `objc_methods` 上 `bioLabelFrame` IMP=0x3AFDDC (T1Twitter binary 1) は `list_address=0x13E7490` の51メソッド群に所属。同群には `userNameLabelFrame`, `fullNameLabelFrame`, `socialTextFrame`, `infoTextLabelFrame`, `avatarFrame` 等のフレームgetter/setterが存在する。**プロフィール表示のレイアウトデータ候補**。
- `_t1_bioTextModelForUser:` は `objc_methods` でIMP=0x1BF0EC、単独記録のメソッドリストがある。実際の描画対象との対応は未確定。
- `ProfileHeaderBio`, `ProfileHeaderInfoView`, `ProfileHeaderTranslatedBio` も文字列に存在するが、クラス名や最終描画ビューと断定しない。

### 結論・次の作業
- **プロフィール自己紹介欄の最終描画クラスはまだ確定していない。** 上記のセレクタ群が最優先の静的解析対象。
- まず、`bioLabelFrame` を持つレイアウトオブジェクトの所有クラスをObjective-Cメタデータ／IPAから特定し、`T1ProfileUserInfoView` などとの関係を追う。
- SEが使えるときに、v12.35の `T1ProfileUserInfoView` 等の存在・メソッドを限定的に照合する。現在のProbeの固定4クラスは投稿セル寄りなので自己紹介欄の証明には使えない。
- 実測する候補: bioビューの実クラス、font/pointSize、bounds/frame、表示条件とsafe area。投稿本文やユーザーのプロフィール文字列は採取しない。
- Scarlet Xの本番UIには、この調査だけを根拠に変更を加えない。
