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

## 2026-10-23 — bioLabelFrame所有クラスを実バイナリで確定
- **確認済み:** 公式X v12.25の`T1Twitter`内で、`bioLabelFrame` と同一51メソッド群を持つ所有クラスは `T1UserViewLayoutState`。class `0x1AF2688`、class_ro `0x19ABE10`、methods `0x13E7490`。
- [実バイト根拠・手順](OfficialXResearch/ProfileBioLayoutOwner20261023.md)。
- **未確定:** `applyToUserView:` での適用先、`setBioLabelFrame:` の値の生成経路、公式Xの自己紹介欄の実測CGRect・フォント・幅・高さ。SE実行版はv12.35で静的資料とバージョン差がある。
- **次:** 上記2メソッドの呼出関係を限定して追う。Scarlet Xの表示コードは根拠が確定するまで変更しない。

## 2026-10-23 — bio生成候補の切り分け（追記）
- `_t1_layoutBio:user:width:yCoordinate:` は `T1UserRecommendationViewLayoutDelegate` のメソッド。
- 計算結果を書き込む `setBioFrame:` は `T1UserRecommendationViewLayoutState` に属する。**通常プロフィール側の `T1UserViewLayoutState.setBioLabelFrame:` とは別経路**。
- 解析証拠: [ProfileBioFrameProducer20261023.md](OfficialXResearch/ProfileBioFrameProducer20261023.md)。
- 次は `T1UserViewLayoutState.setBioLabelFrame:` (`0x3AFDE8`) へのcallsite、またはCGRect格納域 `+0xD8`への直接書込みを調べる。プロフィールUIへの仮実装はまだ行わない。

## 2026-10-23 — 通常プロフィールsetBioLabelFrame:直接呼出探索
- v12.25 `T1Twitter`内の直接BL走査では `setBioLabelFrame:` stub `0x1337B60`およびIMP `0x3AFDE8`への直接呼出は0件。getter `bioLabelFrame`は `applyToUserView:`で1件検出。
- [調査・限界](OfficialXResearch/ProfileBioSetterCallers20261023.md)。**objc_msgSend/他framework/直接ivar書込みは未除外**。
- 次は他framework・レイアウト状態生成経路の探索。SE実機なら計測値取得が近道だが、Probe機能追加と複数条件検証が必要。

## 2026-10-23 — SEなしでのbio計測候補の発見
- 公式X v12.25 T1Twitterに `_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:`（IMP `0x141CD8`、method list `0x1393E10`）がある。username/fullname等のサイズ計測メソッドと同一リスト。
- [一次証拠と未確定部分](OfficialXResearch/BioSizingAlternative20261023.md)。所有クラス、bio CGRectとの接続、計測アルゴリズムは未特定。
- SE不要の次作業: 所有クラス特定→呼出スタブ解決→profile側layoutStateへの接続検証。

## 2026-10-23 — bioフォントの分岐を命令確認
- `T1UserViewBuildTextModel`はlayout metricsの`metricsFont`を優先し、nil時に限り`xds_body` fallback。`lineHeight`は同じmetricsから別途取得。ソース: [T1UserViewFontFallback20261023.md](OfficialXResearch/T1UserViewFontFallback20261023.md)。
- 次: fallback受信クラスのchained fixup解析、layout metrics生成元、bio CGSize→CGRectの伝搬経路。

## 2026-10-23 — bioレイアウト指標の引数伝搬・UIFont fallback確認
- v12.25実命令により、`T1UserView._t1_sizeForBio...`の`layoutMetrics:`引数は、キャッシュ付きテキストモデル生成を経て`metricsFont`/`lineHeight`の受信者へ渡される。
- fallbackのchained-fixup import ordinal `0x3ADC`は `_OBJC_CLASS_$_UIFont` と解決。`[UIFont xds_body]`呼び出しを確認。
- 詳細: [T1UserViewMetricsProvenance20261023.md](OfficialXResearch/T1UserViewMetricsProvenance20261023.md)。次は上位呼出元とlayoutMetrics生成元。具体フォント値は未確認。

## 2026-10-23 — T1UserViewのbio component番号確定
- 公式X v12.25実バイナリのbyte jump table `0x144DEE1`とObjC selrefを照合し、`sizeForDynamicComponent:withOptions:size:layoutMetrics:cachedTextSizing:`の**component=4が`_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:`へdispatch**されると確定。
- [根拠](OfficialXResearch/T1UserViewBioDispatcher20261023.md)。次は共通dispatcherの上位callerとlayoutMetrics生成元を追う。投稿本文のlayoutMetrics規則をbioに転用しない。

## 2026-10-23 — bio計測の上位呼出元と所有クラス（実バイナリ検証）
- `T1UserViewLayoutParameters.sizeForComponent:withSize:layoutMetrics:` IMP `0x252654` から `T1UserView.sizeForDynamicComponent:withOptions:size:layoutMetrics:cachedTextSizing:` ObjC stub `0x13595C0` を `0x2526D8` で呼ぶ。直接BLはT1Twitter内1件。
- x3で受け取ったlayoutMetricsをx4へ渡す。component 4がbio計測へdispatchするのは前回確定済み。
- method list `0x13E7400` の所有クラスはclass_roの実バイトで `T1UserViewLayoutParameters` と確認。
- [詳細](OfficialXResearch/T1UserViewBioCaller20261023.md)。次は `sizeForComponent:withSize:layoutMetrics:` と `initWithOptions:layoutMetrics:estimated:` の上位呼出元からmetrics生成元を追う。
