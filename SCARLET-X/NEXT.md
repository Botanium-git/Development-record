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

## 2026-10-23 — componentごとのcallsite実測
- T1Twitter実命令でstub `0x1359540`への13箇所にcomponent 0〜12が1件ずつ渡されることを確認。**bio=4は`0x483D8C`**で`mov w2,#4`直後にBL、`x3=x19`。
- 13件は複数関数にまたがる。前回の「全部同一`sizeThatFitsHelper`メソッド内」という推定は誤りとして訂正した。[詳細](OfficialXResearch/T1UserViewBioCaller20261023.md)。
- 次は`0x483D8C`の関数境界・x19の生成元を追う。

## 2026-10-23 — bio helper関数入口とsetter blockを実バイナリ確認
- bioサイズ計測の関数入口は`0x483D40`。第3引数x2をx19に保持し、`tfn_flexibleLayoutViewModel` (`0x1363600`)の戻り値へ`sizeForComponent:withSize:layoutMetrics:`をcomponent=4で送る (`0x483D8C`)。
- 近傍`0x483DC0`はcaptured object `+0x20`に対して`setBioLabelFrame:` stubへ**B（tail branch）**。`0x483C58`の別関数がこのinvoke pointerをブロック様構造へ格納する。以前のBLだけのcaller検索ではこの経路を検出できない。
- **未確定:** block実行順、CGRect算出、x2 metricsの上位生成元、runtimeでの採用。
- [証拠](OfficialXResearch/T1UserViewBioCaller20261023.md)。次はblock登録メソッド`0x1365140`のselector解決と上位caller追跡。

## 2026-10-23 — bio flexible layout両コールバック接続確定
- `0x483C58`が` tfn_withFlexibleLayoutSizeCalculator:`へ定数block `0x1882490`を渡す。このblockのinvokeポインタは実バイトで`0x483D40`（bio component4計測）と確認。
- 同じ関数が` tfn_withFlexibleLayoutFrameListener:`へinvoke `0x483DC0`のstack blockを登録。後者は`setBioLabelFrame:`へtail branch。
- [実命令とblockポインタ](OfficialXResearch/T1UserViewBioCaller20261023.md)。次はflexible layout engineによるサイズ→CGRect算出規則と上位layoutMetrics生成元。実機値/ランタイム採用は未確認。

## 2026-10-23 — TFN bio callback ABI実命令確認
- v12.25 `T1Twitter` のsize callback `0x483D40`はcomponent4の計測結果（v0/v1 CGSize相当）を返す。frame listener `0x483DC0`は受け取ったframeを加工せず`setBioLabelFrame:`へtail branchで転送。
- flexible layout関連の型文字列に`TFNLayoutMetrics`、`CGSize`、`CGRect`を確認。ただし文字列と当該callbackを1対1対応させるblock descriptorは未確認。
- [根拠](OfficialXResearch/TFNFlexibleBioCallbackABI20261023.md)。次はTFNエンジン実装モジュール、配置のorigin/spacing計算、metrics作成元を特定する。

## 2026-10-23 — SE再開時のProbe v0.1.12検証
- SCARLET-X-Probe/develop: ProbePanel.mに既存4クラスを保ったまま T1UserView, T1UserViewLayoutParameters, T1UserViewLayoutState, T1ProfileUserInfoView, T1ProfileHeaderView, T1ProfileUserInfoViewController の6クラスを追加し、10クラスを読み取り専用で記録するよう変更。
- control/Tweak.xmを0.1.12へ更新。最終コミット62acee60c2e04a2df5a4f47e554c09f06349b016。main変更なし。
- 未確認: Actionsビルド、SE実機導入、X v12.35でのクラス/メソッド、CGRect/フォントの実測。
- 次: Actionsビルド確認後、SEでSX→クラスメソッド情報を記録→診断JSONをDriveへ送信。メソッド存在確認後に限定CGRect計測を検討。

## 2026-10-23 — private Probe updater MVP 実装（SE未検証）
- SCARLET-X-Probe/developに `tools/private_probe_update.py` を追加。GitHub fine-grained PATを実行時に非表示入力、Actions-readで成功したdevelop push buildを取得、ZIP内deb一件・パッケージID・arm64e architecture・既存バージョン超過・dylib/plist存在を確認後、`~/Documents/SXProbeUpdates/`に保存。rootでの自動インストールなし。
- 認証付きAPIへのHTTP redirectは拒否。artifactの署名付き配信URLにはPATを送らない。コード・debの公開APTなし。
- 実装 commits `e91f00001ccff6959d9be0901e3e0b4cda96d2ab`, `ec28df51c546b0f51b2a02b75d9f16048c0150e5`; [設置ガイド](https://github.com/Botanium-git/SCARLET-X-Probe/blob/develop/tools/PRIVATE_UPDATER.md)。
- **未検証:** SEのPython3/dpkgコマンド有無、実ダウンロード、Sileoインストール、ワンタップShortcut、v0.1.12のActionsビルド・実機。次はSEで`command -v python3 dpkg-deb dpkg-query dpkg`等を確認してから実行試験。初期実装は実行毎にtoken入力が必要。

## 2026-10-24 — Private updater: validated deb → Filza staging automation
- `SCARLET-X-Probe/develop/tools/private_probe_update.py` now stages validated new .deb into `/rootfs/var/mobile/Media/Downloads` after saving locally, for Filza handoff. This exact rootfs directory was confirmed visible in Filza on SE using a test file; previous standard RootHide /var/mobile and /var/mobile/Media paths were not shared.
- Commit `4f1c108023345709e22c5961cda5a47fc8f01891`; guide `58e6c6fcc6b209bca022056e21ca14083def2f2b`.
- User reports Filza installing the .deb and Sileo displaying Probe version. Remaining: secure Keychain-backed token persistence and one-action launcher; neither is yet implemented, do not call this a completed one-tap update. Before designing home screen launcher, verify device-side OpenSSH/Shortcuts or implement an SX UI Keychain helper. Do not store PAT in cleartext scripts or Shortcut action.

## 2026-10-24 — Probe v0.1.13 更新画面第1段階（未検証）
- `SCARLET-X-Probe/develop` に新規 `ProbePrivateUpdate.m`。既存の `ProbePanel.m` 第3セクション末尾へ「Probe更新（準備中）」を追加、Keychainのトークン登録・削除、およびAPIを使った成功済みdevelop pushビルド確認機能だけを追加。元の診断/Driveフローは維持。
- `Makefile` に追加ソース登録、`control` と `Tweak.xm` を **0.1.13** に更新。Github API認証つき要求はredirect拒否。
- **未実装:** native deb download/ZIP extraction/package validation、Filzaへの受け渡し、ホーム画面1タップ起動、実際のインストール。Python updater は別に残す。**未検証:** Actionsビルド、SE Keychain、API認証実行、UI表示。最初にCIを確認してから実機試験すること。
- **注意:** Probeは公式Xのプロセス内で動くので、GitHubトークンのKeychainはそのアプリ実効IDに属する。Keychain保存は平文ファイルより良いがXや同じプロセス内のコードから絶対に隔離されることを意味しない。必要最小権限・短期失効を守ること。

## 2026-10-24 — SE validates Probe 0.1.13 native GitHub authentication
- User installed v0.1.13, entered token in new SX update panel and saw `ビルド確認済み` after pressing latest-build check. This is an actual device validation of Keychain token reuse and successful GitHub API workflow discovery; not validation of deb download or installation.
- The following stage must download the private artifact and validate archive, package identity, architecture and versions before exposing to Filza's share/install UI. Keep a separate explicit installation confirmation; do not auto-install from the X process or log tokens.
- No fully operational native deb downloader or install button has been verified yet. Python updater remains the known functional fallback.

## 2026-10-24 — Probe 0.1.14: native private Actions ZIP retrieval (CI/device unverified)
- User confirmed SE v0.1.13 update screen displays `ビルド確認済み` after Keychain token enrollment. GitHub auth and listing confirmed on-device.
- On SCARLET-X-Probe/develop `ProbePrivateUpdate.m`, added separate `成果物ZIPを取得・共有` row: authenticated GitHub Actions artifact lookup, signed URL download without token, size cap and ZIP magic check, UIActivityViewController handoff to Filza. No root install. GitHub update commit `123c1ae1c042498c452d609b22eb4ea25a9b6b95`; version bumped to 0.1.14 via control `150b8ecd13176a1385bf8515a081a267e166b2a3` and Tweak marker `fc25d7362c2f5f0d78f1ee982c37c73df5273c54`.
- **Important limitation:** It shares ZIP, not a validated extracted .deb. Native ZIP extraction and package ID/version verification NOT implemented. User must extract in Filza, inspect deb, explicitly install. Workflow build status and SE runtime not confirmed. Do not claim one-tap done; Python downloader remains known validated path.

## 2026-10-24 — Probe 0.1.15: ZIP→deb direct Filza share (build/device pending)
- User validated v0.1.14: native private ZIP download, iOS share to Filza, manual installation successful, and Sileo records version. Native Sileo local-package handoff is not confirmed, so retain Filza.
- Updated `SCARLET-X-Probe/develop/ProbePrivateUpdate.m`: single-file ZIP central directory checks, rejects encrypted/multipart/paths/non-regular entries, deflate or stored ZIP extraction via zlib, CRC32 verification, Debian `ar` magic validation; share extracted `.deb` directly. UI button changed to `debを取得・共有` and guidance requires inspecting package ID/version in Filza before manual install.
- `Makefile` adds zlib linkage, `control` and `Tweak.xm` bumped 0.1.14→0.1.15. Key source commits `66139c1671e875246461a05e01b01e1162dead74`, `824efbe31c31cc2714a87bedabf8fe16db94a718` and build commits `3aecd51c043d153778931f7425749de01d9df2cc`, `c640eb74b49b5a681ddb6bf394809243f0cf31e7`, `a6e9a62db430786b4f2e8c02e4d367beda0ab169`.
- **Pending:** GitHub Actions build, SE runtime, ZIP extraction and direct deb share. `ar` magic is not internal package metadata verification! This does NOT yet compare installed/new versions, parse `control.tar.*`, or auto-install. Must fail closed if ZIP layout unsupported; user verifies Package `com.botanium.scarletxprobe`, Architecture `iphoneos-arm64e`, Version before Filza installation.

## 2026-10-24 — Probe 0.1.15 SE device confirmed Sileo update
- User reports `debを取得・共有` returned a .deb, iOS share could hand that deb to Sileo, and the update was successfully completed within Sileo. Thus native GitHub Actions access → ZIP extraction → deb handoff → Sileo package update is confirmed by user on SE.
- Preserve this native working path; Filza is no longer required for the normal case. Do not replace it with a speculative installer. Next planned improvements are version comparison (dpkg installed vs remote), Debian control-field validation (package ID/arch/version), and no-update UX; do not claim these exist in v0.1.15.

## 2026-10-24 — SXP 0.1.16 menu redesign (not yet device/build validated)
- User approved SXP menu redesign and **update check only when opening SXP menu** (not when X launches).
- `SCARLET-X-Probe/develop/ProbePanel.m` organized into Information (X version, SXP version+update notice), Diagnostic (metadata toggle, record class methods, clear logs with confirmation, event count), Connected/Management (Drive connection, send diagnostics, SXP update). Hid redundant class-existence row, diagnostic JSON share row and latest event row; backend diagnostic features preserved; top-right JSON share preserved.
- `ProbePrivateUpdate.m`: consolidated GitHub token register/change/delete in one row; separate latest-build-check row hidden; native proven ZIP→deb→Sileo share retained. On panel open, checks successful develop push workflow and reads `control` from exact workflow `head_sha` through authenticated GitHub Contents API; compares numeric dotted `Version` to compiled `SXProbeOwnVersion()` = 0.1.16, displays notice only if greater, otherwise blank; failures say unable to check. This checks source metadata for successful run, NOT package file's signed metadata or availability; must be verified against packaged build in future.
- Version in `control`/`Tweak.xm` bumped to 0.1.16. Commits: panel `d7ec8e372218681fcaa4cc4642840c51bce28638`, update checker `8acfdea43ca121e986ab0a84a73f26b0f77011e8`, token UI `ccf3c9e9278949f672cd083205676c3ce390119f`, version `3fc85856970f207d9c1ba50a490b563a7a8136c9`, `d221b630b9f3ea42d983b06bf1ba25cd1d468755`.
- Do not claim CI or SE validation; known-good v0.1.15 remains available. Verify build before installing 0.1.16 and test UI plus Sileo update path.

## 2026-10-24 — SXP 0.1.17 UI refinement (build/device unverified)
- User confirmed installing v0.1.16 and likes redesigned UI. Requests: tapping SXP version opens integrations only (`Google連携`, `GitHub認証`); main `SXP更新` invokes the same direct deb download/share operation; `現在の状態` directly below; SXP app messages/alerts formal polite Japanese (character voice only in chat).
- Develop changes: `ProbePrivateUpdate.m` now integration settings controller with Google action and GitHub credential management, exported `SXStartProbeDownload(host,status)` to reuse native verified download path, retains worker over dispatch until completion, shows native share sheet from SXP panel. Updated token prompts and result messages to polite neutral Japanese, `SXProbeOwnVersion` 0.1.17. `ProbePanel.m` now main rows Drive JSON send / SXP update / current status; SXP version row opens integration settings; redundant Google link removed. Control and Tweak marker bumped 0.1.17.
- Commits: `1a2009c8c6c00752523cd4e77b7c57611be1798f`, `53f0d1f6a69b25622fa61b026513ba24eaaba236`, `6bd81e1c04d98c7824476c556f609f2f2901fb31`, `4bd43547da0498ca3bc8e8371f9117195d5fecfa`, `749581515dcf8a615871d00704c28d8e8dc0a917`.
- **Not verified:** Actions compilation, SXP 0.1.17 SE runtime, direct download share presentation, Drive button in integration settings. Preserve last working v0.1.16 and verify CI before device update. No auto-install; Sileo must confirm.

## 2026-10-24 — SXP 0.1.18 operation status unification (CI/device pending)
- User used SXP class method recorder and attempted Drive upload; asked to show Drive send state OR updater outcome in the same `現在の状態` row, not only update progress.
- `ProbeDriveOAuthTest.m` now broadcasts `SXPOperationStatusChanged` with `message: Drive: ...` from Drive upload stages and success/failure after Drive API callback; doesn't incorrectly announce start as success. `ProbePanel.m` subscribes, shows most recent Drive/update operation under main `現在の状態`, saves last message in NSUserDefaults and uses multiline subtitle layout. Update download callback also writes status with `更新: ...`. Existing Drive and Sileo update paths retained.
- Version bumped to 0.1.18 in `ProbePrivateUpdate.m`, `control`, `Tweak.xm` (commits `489a13ce46a225b770a09444a37ac6d0c0da35d7`, `bf77afee135b38a1efaf13a0043e829761e23a4c`, `2d7e15bcd215a327e46ae140bf88af45db6a1f74`, `ef0f4f00d77ee6d1dfc72471cdc4eabfe43692d5`, `55fda8e1c788a2736ce2362a6c28e872e2a35bb0`, `7d6c32a59ce631b94a41adf0e9e4ec7d927b160c`). No Actions or SE validation yet.
- Google Drive connected search for SXProbe_ returned no files; cannot confirm user upload through connector search; do not claim Drive-side verification. Ask for SXP's final status after test, and continue analysis once content available.

## 2026-10-24 — SXP 0.1.19 read-only on-screen profile bio geometry probe (CI/SE unverified)
- User requested continuing official X v12.35 vs static v12.25 profile-bio layout analysis. Latest Drive JSON confirmed runtime classes and selectors, but not live CGRect/font values.
- SCARLET-X-Probe/develop `ProbePanel.m` adds opt-in `プロフィール表示を計測` in Diagnostics, manually scans foreground UIKit window view trees for `T1ProfileUserInfoView`, then captures class names, frame/bounds for matching subtree elements and UIFont name/point size for UILabel/UITextView only. Logs event `profile-bio-view-snapshot`, no attributed/plain text, handles or account contents; capped traversal. Does not hook private methods, does not modify frames. May not find profile view when SXP presented modally; this is a probe to test, not a runtime measurement validation.
- Package bumped to 0.1.19; commits source `c42bd23d0b7f74355975cb8738133e88d75258b3`, versions `a065f13ca95bbbb83b273953a516898794ba5bc1`, `bb7d82fce83b7e7b82f02642708fcc49c29506ea`, `9467e81088c8806a1e348d49e327d36642e17579`.
- Next: check GitHub Actions build; install via established SXP/Sileo; open official X user's profile (prefer non-sensitive test profile), tap `プロフィール表示を計測`, check profile-view count, send JSON to Drive; inspect event before building any hooks or modifying Scarlet X UI. If no profile view found, revisit modal view hierarchy scanning before more invasive instrumentation.

## 2026-10-24 — SXP 0.1.20 compile fix
- User provided GitHub Actions screenshots for v0.1.19 failing `Build RootHide package` in `ProbePanel.m:131:5`, errors undeclared `_cmd` and `self`, both stemming from `NSAssert` in free C function `SXBioSnapshot`.
- Fix in develop `ProbePanel.m`: replace NSAssert with safe main-thread guard returning explicit `error=main-thread-required` and zero-result snapshot; commit `f413d7c68a93ed8915ea6ae9e1c2c0e736f6db09`.
- Bump package version, compiled own-version, launch marker to `0.1.20`; commits `84e4b3619c4453900c34d1bb5eb0b7bdfe8aa04e`, `13d360d5689198b5b9d0fc3a7d820b64b6ea5fd4`, `1034a1f413ba077ca7bd4dba4f9165925a79b21b`.
- CI after fix not yet confirmed; runtime unverified. User asked additionally for real upload/download percent and build queued/in-progress notifications. Keep these as separate follow-up development stages after CI, avoiding invented percentages for API queries/build jobs without known total.

## 2026-10-24 — Profile bio runtime sample + SXP 0.1.21 progress/build UI
- Verified Drive `ScarletX/Probe/SXProbe_2026-10-24_185604.json` (X 12.35, 151 entries), 3 `profile-bio-view-snapshot` events: first none, then 2 successful samples each with `T1ProfileUserInfoView` frame `[0,257.5,320,98]`, `TFNAttributedTextView` frame `[9,12.5,23,54]`, another `[0,0,0,0]`. UI button labels font `SFUI-Semibold` 10/18 are NOT proof of bio text font. Text not captured. View count 0→1 suggests screen visibility matters; no comparison with Scarlet X UI yet. Need targeted rich-text rendering metrics capture, without content collection, before drawing conclusions.
- SXP develop 0.1.21: `ProbePrivateUpdate.m` menu-open checker now selects newest develop push workflow of any status (not only success), reports queued / in-progress / failure; on successful completed build reads exact head SHA `control` and compares version. Existing deb downloader still deliberately selects successful completed build for safe Sileo handoff. `ProbeDriveOAuthTest.m` switches actual upload to NSURLSessionUploadTask with didSendBodyData delegate, reports real bytes sent / expected percentage, retains HTTP API response validation for upload success. Version bumped in `control`, `ProbePrivateUpdate.m`, `Tweak.xm`. GitHub commits: build state `f3851131af1cfd1a991d42909365c9bc528db410`, Drive progress `40fa0955bfeca3346147bdfa54f7a7b5ec8f4e3c`, version `19109f1dbcf0b86df2587f40c7e2e5cd657d031d`, `5bf2fa4a2648531dab59b8f7e3f7ffcadbd86974`, `d508083246bfe730321c184a2dd88c1fcdfb434c`.
- CI build and SE device unverified. Percentage of ZIP **download** has not been implemented in this version; HTTP API / ZIP inflate stages have indeterminate progress. Do not invent percentages. Next: inspect Actions and validate 0.1.21; then incremental downloader delegate implementation if desired. Latest known working on-device version 0.1.20 per user.

## 2026-10-24 — SXP v0.1.22 profile typography probe
- User explicitly requested implement enhanced runtime profile measurements, not merely suggest them. In `SCARLET-X-Probe/develop/ProbePanel.m`, added `SXTypographyMetrics`: read-only sampling of `NSFontAttributeName`/`NSParagraphStyleAttributeName` for UIKit UILabel/UITextView attributed strings, returning bounded distinct font name/size/line height and paragraph metrics without text, ranges or account identifiers. Captures each matching view's hidden/alpha/onScreen status and fontLineHeight. Records available font/attributed/layout selector names of `TFNAttributedTextView` to plan later precise custom rendering analysis; **does not call unknown private selector**. Commit `78c9dcd22d47d0a926532aaee86d7c86da63edbb`.
- Updated `ProbePrivateUpdate.m` own-version to 0.1.22 (commit `c2eae4f1b632270814dec947fd04922889016324`) and `control` Version 0.1.22 (commit `29cd7cd4f82d340b78cf655af527d8924bb733e6`). **Tweak.xm version marker still 0.1.21** at last check; repeated connector update attempts were blocked. This must be synchronized before declaring the release complete, along with verifying Actions build and device test. Note this marker is informational but should match.
- Testing: official X v12.35, open actual profile, SXP > Diagnostics > Profile display measurement, Drive send, inspect new event for `typography`, `availableMetricSelectors`, `onScreen`. No runtime/build success yet.

## 2026-10-25 — SXP 0.1.23 font-update method ABI preflight
- User requested next actual implementation for official X 12.35 profile bio font diagnosis. To avoid unsafe private selector interception with unverified Objective-C ABI, added `SXFontUpdateABIMetadata` to `SCARLET-X-Probe/develop/ProbePanel.m`. During manual profile snapshot, each `TFNAttributedTextView` includes `fontUpdateABI`: exact `method_getTypeEncoding` and `NSMethodSignature` argument encodings for `ocf_updateForText:activeRanges:font:color:activeRangesFont:textAlignment:lineBreakMode:`, `ocf_updateForExpandedRichText:font:color:activeRangesFont:textAlignment:lineBreakMode:`, `_ocf_updateForText:activeRanges:attributedStringFactory:`. **Metadata-only, no hook, no captured text/font values yet.** Commit d52a61f85227ea3bbe18c6b74e1e3c7c50bc328f.
- Version bumped to 0.1.23 in ProbePrivateUpdate.m (1ba579c61471c18ea7fe92016b0c23e2e17581f0), control (3291a879e0cbfcfd59c282a973a6064b10382f5e), and Tweak.xm launch marker (863f6d8dc3cfa0f90b3c65dcd0e14d6ead369cfd), resolving prior marker mismatch. Build and SE device unverified.
- Next: verify Actions, update Sileo, display multi-line official X profile, run SXP > Diagnostics > Profile display measurement, Drive upload. Analyze `fontUpdateABI` signatures, then design properly typed read-only interception (if safe) without collecting text or changing layout. Keep ScarletX main repo unchanged.

## 2026-10-25 — SXP 0.1.24 runtime profile font observer
- Added two read-only `TFNAttributedTextView` update interceptors in ProbePanel.m, commits `964dad8bdb95c142da65e53a432829d6f4513068` and include fix `8da57501f5e9a860cf3047dec8f28f72bbe5690f`. Installs on app active only if BOTH exact encodings match ABI from user diagnostic 0.1.23. `class_replaceMethod` subclass override, preserves original IMP and forwards all arguments unchanged. Only records `font` class/name/pointSize/lineHeight, view width/height, source and privacy marker when text view has `T1ProfileUserInfoView` ancestor and metadata recording enabled. Caps to 40 events/process; no text/content or other objects serialized. `profile-font-observer-installed` event records guard success. No guarantee callback runs (profile may already be rendered or no parent at invocation).
- v0.1.24 versions: ProbePrivateUpdate.m commit 676c781a9d6dc9b0c69f51ce0d5c8501b84ea440, control 7e0508a3793eb71bb22db9919e85e5045be22cb6, Tweak.xm 8687b199ee4595a6fa28fcbeb2495f5a50a17adc. Main ScarletX untouched.
- Build/actual iPhone testing not verified yet. On install open X, ensure metadata recording ON, *navigate to a different multi-line profile after app launch* so update events fire, then capture profile layout and upload Drive JSON; search `profile-font-observer-installed` + `profile-bio-font-argument`. If missing, investigate installation timing or parentage instead of claiming font. Device stability first; rollback 0.1.23 if crashes.

## 2026-10-25 — SXP 0.1.25 call/rejection counters
- After v0.1.24 produced one `profile-font-observer-installed` event but zero `profile-bio-font-argument` events despite user-reported app stability, committed `ProbePanel.m` diagnostic counters: `textCalls`, `richCalls`, `accepted`, `rejected`, `noSuperview`, `notProfileAncestor`. Emits `profile-font-observer-counters` once per ~25 intercepts and at manual profile snapshot. No user text is captured, hook ABI and forwarding unchanged. Commit `e1b8201e55aaec6105adb39dc1ea524a31f9b0eb`.
- Version 0.1.25: ProbePrivateUpdate.m `f38d44691549af69c521e520a745c34c22a7ac55`, control `6866de5a8a9fddc2558a1299f7d38c3a7ba80437`, Tweak.xm `12c61dc8a752d74785d825e28c7cefeef036bd12`. Build/device unverified. Test: update, open official X, navigate between profiles, manual `プロフィール表示を計測`, upload JSON, examine last counters event. If textCalls+richCalls=0 then hooked methods not called/incorrect dispatch/timing; if rejected>0 with noSuperview>0 likely parent not attached at update time. Do not assume until results. ScarletX main repo untouched.

## 2026-10-25 — SXP v0.1.26 font dispatch ownership diagnostics
- v0.1.25 showed `profile-font-observer-installed` but `textCalls=0,richCalls=0,accepted=0,rejected=0`, so whether dispatch reached the new IMP is unverified; no evidence to blame the profile ancestor filter.
- `ProbePanel.m` commit `ebaec804aeea7fa413c5fb6cce070e42c85d1176`: new `profile-font-dispatch-status` metadata at installation and manual profile snapshot, including original/active IMP image basename (`dladdr`), class owners derived with `class_copyMethodList`, boolean `currentIsObserver/currentIsOriginal`, no raw pointer or text content. No new interceptions.
- v0.1.26: `ProbePrivateUpdate.m` bae0fc33fe9a9b709273ad229c5269d069cfc3f8, `control` 12c7616f2cc8c43c356b1a6d55729b674bda5b7c, `Tweak.xm` 29f34264675f1579f7f869355c9d2218c9f5111c. Actions build and device tests not verified. Main ScarletX repo unchanged.
- Test v0.1.26: open official X after install, navigate multi-line profile, SXP > manual profile measurement, Drive JSON; inspect `profile-font-dispatch-status` vs `profile-font-observer-counters`. Do not infer implementation dispatch from class selector existence alone.

## 2026-10-25 — SXP v0.1.27 updater UI polling/spinner
- User requested avoiding artificial percent/progress bar; show iOS native spinning indicator immediately to right of `ビルド中`, poll status every **5 seconds** because builds often complete in ~40 seconds. Implemented in `ProbePanel.m` commit `2eece357eaeaaaa4f9c069960c5c9c446d77034f`: `updateRequestInFlight` prevents concurrent status queries, one-shot `NSTimer` while `updateNotice` is `ビルド中` or `ビルド待機中`, cancels on view disappear/dealloc, resumes check on viewDidAppear, UITableViewCell accessory uses horizontal UIStackView UILabel + UIActivityIndicatorView. On success/failure stop polling and display existing notice. No percentage, no changes to download/extract or profile font diagnostics.
- Version 0.1.27 in `ProbePrivateUpdate.m` commit `16658f84fcdd5185b74b4cf85048b2ae3c808ba2`, `control` commit `3cde540d79068ac1d82b306d9225dfb9e1f456ef`, and `Tweak.xm` marker commit `bd2f0f27fbb6988ab0d4fa6f08977db5afb3e199`. Builds and iPhone verification outstanding; latest GitHub Actions should be checked before installing. Main ScarletX untouched.

## 2026-10-25 — SXP v0.1.28 profile bio setter ABI preflight
- User deferred testing v0.1.27 build-spinner until an actual in-progress build exists and asked to continue profile typography analysis.
- `ProbePanel.m` commit `b1fed511e88a14fb3dab5db3d8d52f247b636d06`: during manual `プロフィール表示を計測`, emit `profile-bio-setter-abi` for `T1ProfileUserInfoView` instance selector `_t1_setBioLabel:withText:entities:isUTF16Entities:`: existence, exact method type encoding, argument/return ABI, IMP image basename. Strict read-only; no calls to selector, no hook, no user text extraction. Original TFN observers and updater polling remain unchanged.
- v0.1.28 versions: `ProbePrivateUpdate.m` e46d0e9e4000b2d296eff436bf4d49585545ae51, `control` 6cdf0fd96fdac9da732844b0ca1930bc9d30a742, `Tweak.xm` 151acb3a0bbbcea15b9048c295478d5b91929e68. Build/device verification pending. ScarletX main untouched.
- Next: after build and install, open profile, SXP > プロフィール表示を計測 > export Drive JSON; inspect `profile-bio-setter-abi` before deciding whether to introduce typed observation hook.

## 2026-10-25 — SXP v0.1.29 manually sampled bio text
- User explicitly authorized capturing displayed bio text and keeping Google Drive JSON export; build-spinner seen working in v0.1.28/0.1.27 flow, but five-second auto-transition still needs validation on an active build.
- ProbePanel.m `a239564bd2435ca527b053a73c41ccfd12848a7e`: on manual `プロフィール表示を計測` only, if a visible `TFNAttributedTextView` under `T1ProfileUserInfoView` has an accessible label, include `visibleText` (max 1500 UTF-16 code units), `textLength`, `textTruncated`, `explicitNewlines`, `textSource=accessibilityLabel` along with existing bounds/frame data in same entry; no automatic scraping, no new hooks. If accessibilityLabel is unset, text will not be captured; do not claim completeness. Existing arbitrary labels remain metadata-only. `manualVisibleTextCapture=true` snapshot flag (capability not evidence text was found).
- Versions: ProbePrivateUpdate.m `13631ce3ec699be11adbcea2b0d4192798ab5322`, control `3558ee82ce2da263732c04a5707a453224cdd66a`, Tweak.xm `b0a2de8e8c95df4126e90468b4e81feaa1092d3a`. Build and device test unverified. Next: install, show multi-line profile, manual measure, upload Drive JSON, look for `visibleText` and `profile-bio-setter-abi`. Main ScarletX unchanged.

## 2026-10-25 — SXP v0.1.29 device result and v0.1.30 bio setter call probe
- v0.1.29 device JSON `SXProbe_2026-10-25_063612.json`: reported 160 events, 7 manual visible bio text samples from `TFNAttributedTextView.accessibilityLabel` with text-to-frame pairing. `profile-bio-setter-abi` confirmed on X 12.35 Build 27: `_t1_setBioLabel:withText:entities:isUTF16Entities:` encoding `v44@0:8@16@24@32B40`, implementation binary `T1Twitter`. Do not confuse height samples with confirmed font sizes.
- v0.1.30: `ProbePanel.m` commit `02c23905d1029727f7da0fb576624ee71f2af782`: typed observer for `T1ProfileUserInfoView` bio setter, exact-ABI guard; original function called with unchanged arguments. Up to 40 metadata-only `profile-bio-setter-call` events plus manual `profile-bio-setter-counters` with observed call count; do not record passed text or account IDs. Main ScarletX unchanged.
- A transient tool safety check blocked first attempt to bump versions; successfully retried. `control` 0.1.30 commit `0c591789ced1293f2d370cab9d8e549cb37c6f62`; `ProbePrivateUpdate.m` 0.1.30 commit `fd4023e87dbbeaa5658083f583f6d18d0f0ce6b7`; `Tweak.xm` 0.1.30 commit `543c151186137d2afe98a4f668a0cced4b815748`. GitHub Actions build and iPhone test not yet verified.
- Next: verify Actions and install v0.1.30; visit several profiles with bio text, run manual profile measurement, upload diagnostic JSON. Compare setter installed/call count and the manual snapshot. 5-second polling and spinner already confirmed on device for v0.1.29 update path; completion transition not independently confirmed.

## 2026-10-25 — SXP v0.1.30 device validation and v0.1.31 typography diagnostics
- Device v0.1.30 JSON `SXProbe_2026-10-25_065524.json`: 103 events, 26 stored `profile-bio-setter-call` events (final counter 28). `T1ProfileUserInfoView` bio setter observer installed and called; `TFNAttributedTextView` label, `__NSCFString` text, `TFSTwitterEntitySet` entities. Existing text/rich font observers still show 0 callbacks. The bio setter is a proven active observation point, but specific font size and paragraph spacing are not yet identified.
- v0.1.31 code `ProbePanel.m` commit `627fefb03784103b9e35c44f722c4085819e96f6`: after forwarding unchanged bio setter arguments to original method, observe supported zero-argument, object-returning getters on the bio label: `font`, `attributedText`, `attributedString`, `attributedActiveTextModel`. Gate on runtime method signature before invocation, log getter availability, returned object class, and font/attributed typography metadata only (no text content), capped at 20 calls. New event `profile-bio-postset-typography`. No ScarletX app changes.
- Versions all v0.1.31: `control` commit `2cb6080003b0cbfe8c1bc656705bf8ecb5f5959a`; `ProbePrivateUpdate.m` commit `9671a9c5ca44df5d39905660834a85c162dc6fdf`; `Tweak.xm` commit `a40f406a9d46ef3240f335dea445cd3c79f4dc8d`.
- v0.1.31 build and device behavior not yet verified. Check Actions before installing. Then visit profiles, manually measure, send Drive JSON, inspect `profile-bio-postset-typography` for actual UIFont / attributed-text metrics; if no supported getter, choose a different trace based on the observed results, not guessed size.

## 2026-10-25 — SXP v0.1.31 result and v0.1.32 bioFont ABI preflight
- Received `SXProbe_2026-10-25_070307.json`, 105 events. 16 `profile-bio-postset-typography` probes all reported `font`, `attributedText`, `attributedString`, `attributedActiveTextModel` unavailable under the object-return/zero-argument runtime signature check. Setter still observed 16 times. No reliable font size/line spacing found.
- Older official X 12.25 static index identified a `bioFont` method name; current target X is 12.35, so existence/usage there remains UNVERIFIED. Do not infer a font value from older-version symbol names.
- v0.1.32 ProbePanel.m commit `65ebe4bdc11f63c8ce901eb8feda2e35460f4137`: at manual profile measurement emit `profile-bio-font-candidates`, read-only runtime existence/type encoding/IMP image for instance selector `bioFont` on `T1ProfileUserInfoView`, `T1UserView`, and `TFNAttributedTextView`. Does NOT invoke the getter or read user content. Existing bio setter and font observers unchanged.
- Version 0.1.32: `control` 5bde7d6b281a8234f89c172fd25d9e2d27177697; `ProbePrivateUpdate.m` 15c131200b750743e0b2e89b709a28ff75f3f94c; `Tweak.xm` 3bb3441606fb669968ee8ca2668c68abde1990e1. GitHub Actions and on-device verification pending. Main ScarletX untouched.
- Next: confirm successful build; install SXP v0.1.32, open a profile, run manual measurement and Drive-export diagnostic JSON; inspect `profile-bio-font-candidates`. Only plan a typed value read after runtime encoding supports it.

## 2026-10-25 — SXP v0.1.32 device result
- Received `SXProbe_2026-10-25_121420.json` (official X 12.35; 157 retained events). New `profile-bio-font-candidates` occurred twice. Both reported `bioFont` instance selector absent for `T1ProfileUserInfoView`, `T1UserView` and `TFNAttributedTextView` (`exists=0`, empty type encoding, missing IMP). This excludes these specific instance-class paths on X 12.35, not other classes or class methods.
- Existing bio setter observer remains installed and active; after-set typography getter checks remain unavailable. No confirmed profile bio font size, style, or paragraph line spacing. Do not treat other visible UILabel typography as bio typography.
- Next: use older X 12.25 static index to identify the declaring class of `bioFont` before further runtime inspection; do not add blind getter invocations. No SXP source change and no version bump from v0.1.32 for this result.

## 2026-10-25 — X 12.25 static bioFont method-list follow-up
- Checked supplied `scarlet_macho(1).sqlite` and original X 12.25 IPA `T1Twitter` Mach-O. SQLite `objc_methods`: sole `bioFont` selector record binary_id=1, types=`@16@0:8`, imp=1841156, list_address=20528456, entry_index=3.
- Same selector list includes `fullNameFont`, `usernameFont`, `bannerImageRatio`, `badgeImageSize`, `edgePadding` and related layout selectors. It is a candidate appearance/layout configuration class, but declaring class is NOT established.
- The SQLite schema has no class ownership mapping. Raw 64-bit pointer search for `list_address` in Mach-O found zero literal pointers, so Objective-C metadata / chained fixups must be interpreted; do not infer class identity.
- On X 12.35, SXP v0.1.32 failed to locate instance `bioFont` on the three previously selected view classes. No SXP release change from this static investigation.

## 2026-10-25 — X 12.25 bioFont declaring class RESOLVED
- Reopened user-provided X 12.25 IPA `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter` and cross-checked `scarlet_macho(1).sqlite` method index.
- `bioFont` method list address `0x1393D48` resolves through ObjC class_ro_t `baseMethods` pointer at `0x1972590` (class_ro_t begins `0x1972570`). Its name pointer at `0x1972588` resolves to **`T1UserRecommendationViewLayoutSpec`**. The direct Mach-O bytes and 12.25 static index agree.
- This is a user-recommendation-view layout spec class; it is NOT proof of profile-detail (`T1ProfileUserInfoView`) bio typography. Prior SXP v0.1.32 checks against three view classes were not tests of the actual owner class. No confirmed font or line spacing value for the profile-detail screen.
- Next: avoid blindly using `bioFont` from recommendation UI. Prioritize `T1ProfileUserInfoView`'s `updateFontSize` and actual rendering path or obtain a corresponding 12.35 binary to inspect. No SXP code changes in this static analysis.

## 2026-10-25 — SXP v0.1.33 profile typography class-structure probe
- Previously confirmed X 12.25 `bioFont` belongs to `T1UserRecommendationViewLayoutSpec`; do not confuse recommendation UI with profile detail. X 12.35 `T1ProfileUserInfoView` has `updateFontSize` and active `_t1_setBioLabel:withText:entities:isUTF16Entities:` setter.
- v0.1.33 `ProbePanel.m` commit `5cf1fb8bd19e48f17c061a6c32fb24f067b212d9`: manual profile capture additionally emits `profile-typography-structure`, recording ABI of the two methods and class-declared ivar names/type encodings containing bio/font/text across the `T1ProfileUserInfoView` hierarchy. No private method invocation or ivar value access; keep existing hooks unchanged and do not capture text content in new event.
- Version 0.1.33 commits: `control` de4c9f814c88f9990352f6942c2d8b2613795922; `ProbePrivateUpdate.m` c39a2a3d89bd2d6d21f960cac502aa08742b8fe2; `Tweak.xm` fe72c9d08f96d59c8b0b170b1ddcaed2ac03b645. Main Scarlet X unchanged.
- Build and device behavior unverified. Next: check Actions, install 0.1.33, open one bio-bearing profile, tap manual profile measurement and send Drive JSON. Inspect `profile-typography-structure` before planning any value read.

## 2026-10-25 — SXP v0.1.33 device result
- Received Drive log `SXProbe_2026-10-25_132414.json`, xVersion 12.35, 200 entries (includes accumulated prior-session events; do not treat every entry as v0.1.33-only).
- New `profile-typography-structure` event captured once: `T1ProfileUserInfoView.updateFontSize` exists, encoding `v16@0:8`; `_t1_setBioLabel:withText:entities:isUTF16Entities:` exists, encoding `v44@0:8@16@24@32B40`.
- `T1ProfileUserInfoView` directly declares `_bioLabel` (`@\"TFNAttributedTextView\"`) and `_translatedBioLabel` (same type), plus `_bio` (`NSString`), `_bioEntities` (`TFSTwitterEntitySet`), and state fields. No `font`-named ivar was present in emitted matching set. `valuesRead=false`: neither UIFont nor paragraph styling confirmed.
- Next diagnose `TFNAttributedTextView`'s own ivar/property metadata or trace concrete font setup in `T1ProfileUserInfoView.updateFontSize`; do not assume `_bioLabel` holds a directly readable UIFont. Keep scope read-only until ABI/type is confirmed.

## 2026-10-25 — Archived profile investigation reconciliation / SXP v0.1.34
- Revisited prior Scarlet X archived discussion findings: X 12.25 `T1UserViewLayoutState.bioLabelFrame` controls user-list/user-view bio layout, alongside `userNameLabelFrame`, `fullNameLabelFrame`, `avatarFrame` and `size`. Do NOT assume these are the profile-detail screen's geometry. Previous design goal: match exact X font, line height, linebreaks, measurement, width, dynamic type and spacing; use current code as primary evidence before altering Scarlet X rendering.
- Current X 12.35 profile-detail route is `T1ProfileUserInfoView` with ivars `_bioLabel` and `_translatedBioLabel` typed `TFNAttributedTextView`; active `_t1_setBioLabel:withText:entities:isUTF16Entities:` and `updateFontSize` method exists (`v16@0:8`). Font value still unknown.
- SXP v0.1.34 changes ONLY probe diagnostics: `ProbePanel.m` commit `5e68d84f24e2459f9d7b7b8466d7d45537079e0d` adds `profile-bio-textview-structure`, enumerating TFNAttributedTextView superclass-chain ivar/property names and type declarations related to text, fonts, attributes, styles, renderers or layouts. It invokes no private getter, reads no ivar values, and leaves observer behavior unchanged.
- Bumped version to 0.1.34: control commit bfa52dfb47b8358a3e4d55ce0d0899a5882b5b94; ProbePrivateUpdate.m commit b0f04d4ef3d5ca878d5f8523160d5ae8b085a2b0; Tweak.xm commit d78e4d73f953867d29e9895ac44eda7612ec8f3b. Main Scarlet X repo unchanged. Build and on-device behavior pending verification.
- Next: verify Actions build, install SXP 0.1.34, open one profile with bio, manual profile measurement, send Drive JSON, then inspect `profile-bio-textview-structure` for typed safe follow-up targets. Avoid confusing list/profile-detail views or attributing unrelated UILabel fonts to the bio.

## 2026-10-25 — SXP v0.1.34 device result
- Received Drive `SXProbe_2026-10-25_135203.json`, X version 12.35, 44 events, one `profile-bio-textview-structure`; diagnostic executes successfully.
- `TFNAttributedTextView` directly declares `_textModel` (`TFNAttributedTextModel`), `_textRenderer` (`TFNAttributedTextRenderer`), `_preferredMaxLayoutWidth` (`double`), `_updateForFontSizeChanges` (`BOOL`), and `_updateTextAndRangesBlock` (block). Properties include `textModel` and readonly `textRenderer` with matching ivars.
- This is metadata only (`valuesRead=false`); no actual font name, point size, line spacing, attributed-string attributes, or paragraph spacing confirmed. Do not use unrelated UILabel measurements for profile bio.
- Next narrow read-only metadata survey to `TFNAttributedTextModel` and `TFNAttributedTextRenderer` classes (properties, ivars, signature only). Only after types and concrete object path are verified consider safe nonmutating value inspection; avoid arbitrary private calls and user text capture. Main Scarlet X remains untouched; SXP v0.1.34 stays installed until a next build is ready.

## 2026-10-25 — SXP v0.1.35 model/renderer investigation and roadmap assessment
- Original Scarlet X goal (see STATUS/DECISIONS and architecture handoff): WebKit remains for data/auth/session, while UI becomes native progressively; recreate official X *decision rules* (font, measurement, line breaks, spacing, Dynamic Type, device widths), not iPhone-only screenshot tuning. Preserve intentional UI differences and existing DirectTable.
- X 12.35 SXP v0.1.34 confirms profile detail: T1ProfileUserInfoView._bioLabel -> TFNAttributedTextView, whose own ivars include _textModel (TFNAttributedTextModel) and _textRenderer (TFNAttributedTextRenderer). These are storage/renderer candidates; neither actual instance contents nor font metrics known.
- X 12.25 static research already showed separate T1UserView path: layoutMetrics.metricsFont (fallback UIFont xds_body), layoutMetrics.lineHeight, bio component=4; **do not conflate list/user-view rendering path with profile-detail T1ProfileUserInfoView**.
- Local 12.25 SQLite includes class symbols for both TFNAttributedTextModel and TFNAttributedTextRenderer, but cannot by itself prove 12.35 runtime instance typography. Prefer constrained live class metadata observation to guessing.
- SXP v0.1.35 ProbePanel.m commit 993dbcf44804a79737e7f70ec198cb3b39731a8e: emit profile-bio-model-renderer-structure during manual profile capture, enumerating ONLY the two classes' directly declared ivars/properties and typography-related method names/type encodings. No private method invocation or instance values, no text capture by this new event.
- v0.1.35 version bumps: control 2d1d955555924138ae94a5664bd9be9bd315a6b9; ProbePrivateUpdate.m dbde4efe2f72598edc60cf89ff02dac96ffdc4a7; Tweak.xm 352bf2583ed050d0002fb45e7c494fbeb1dc7e8d. Main Scarlet X unchanged. Build/device not yet verified.
- Prioritization: (1) check Actions, install/test v0.1.35, single bio profile measurement and Drive JSON; (2) identify actual bio typography via verified types/instance path; (3) compare same conditions to Scarlet X existing NativeProfileViewController/NativeProfileDirectTable278/NativeTextMeasureProbe; (4) implement minimal isolated measurement/layout change and test at multiple widths, font settings, line wrapping and translations; (5) expand to other native screens. Existing project-wide completion is NOT quantified from these probe logs: avoid falsely treating investigation as implementation completion.

## 2026-10-25 — SXP v0.1.35 result; SE availability and archived research reminder
- Received latest Drive JSON `SXProbe_2026-10-25_135842.json` (official X 12.35 build 27, 45 events). Exactly one `profile-bio-model-renderer-structure` event; both classes exist. The new inspection reads runtime *class metadata only*, not model instance values.
- `TFNAttributedTextModel`: declared `_attributedString` and `_renderedAttributedString` (NSAttributedString), `_lineRanges` (NSArray), `_actualSize` (CGSize), `_coreTextFrame` (CTFrameRef), `_lineBreakMode`, `_maxLines`, `_contentTopPadding`, `_contentBottomPadding`. Runtime methods include `renderedAttributedString` (`@16@0:8`), `attributedString` (`@16@0:8`), `lineRanges` (`@16@0:8`).
- `TFNAttributedTextRenderer`: `_attributedTextModel`, `_frame`, `_verticalAlignment`, `_selectionInsets`, and CoreText draw-related methods. This makes the model's NSAttributedString attributes the leading candidate for font and paragraph metrics. No actual font size or line spacing yet; v0.1.35 was not a measurement of values.
- User reminder to preserve and reuse the extensive **archived Scarlet X profile investigations**, primary design/handoff, Mach-O research, and real source code before implementing fixes; do not rediscover already-proven class/metric paths. Explicitly preserve distinction between T1UserView (listing/user-view) and T1ProfileUserInfoView (profile detail).
- **Impact of restored iPhone SE access:** Prior X 12.25 static IPA analysis established layout paths but not runtime 12.35 values; after SE access, SXP injection/overlay and Drive logs confirm real X 12.35 classes, active bio setter, concrete bio label view, text-model/renderer ivars and their exact runtime types. This materially improves proof quality and allows empirical falsification of hypotheses. SE alone does not prove cross-device layout parity or complete Scarlet X implementation.
- Next prioritize read-only, ABI-guarded *instance-targeted* inspection of the actual on-screen profile-detail `TFNAttributedTextView.textModel` / `renderedAttributedString` attributes (font and NSParagraphStyle metrics only; do not record actual user text). Confirm sender/model types and object lifetimes; avoid unrelated UILabel data. Compare to Scarlet X production measurement with identical text, viewport, and Dynamic Type. SE-to-iPhone/iPad comparisons remain needed.

## 2026-10-25 — Pending SXP bio rendered typography capture
- ProbePanel.m change e8c8af46dec8083e82b13923fb369dbb807fcbda adds manual profile-bio-rendered-metrics using the verified T1ProfileUserInfoView._bioLabel -> TFNAttributedTextView._textModel -> TFNAttributedTextModel.renderedAttributedString path. It records font/paragraph attributes and geometry, not text content. ABI guards and no-view status reporting are included.
- Version bump from v0.1.35 to v0.1.36 was blocked by the tool safety gate. Release NOT READY; Actions and device behavior unverified. Do not request installation until version consistency and build have been verified.

## 2026-10-25 — SXP v0.1.36 version sync retry succeeded
- Previous `ProbePanel.m` commit `e8c8af46dec8083e82b13923fb369dbb807fcbda` added manual `profile-bio-rendered-metrics`, intended to capture rendered bio typography attributes (without body text) via verified profile bio view/model path. This code is not yet proven by a successful build or device run.
- Version synchronization was retried successfully. `control`: `365a80fe1d539323c0b300931c6d9c0e915d8f58`; `ProbePrivateUpdate.m`: `0b293464bea0d24ba95aa4e638378cc12964d518`; `Tweak.xm`: `c6c0350c15ab8a0ef3c0c578c46a330d145ee1d0`. Re-fetched each on `develop`: contains 0.1.36, no 0.1.35 occurrences.
- Workflow status NOT confirmed: commit-run connector only lists PR-triggered runs, returning zero for the tip; this is not a proof of build success/failure. Check GitHub Actions explicitly before installing. No device measurements from v0.1.36 yet. Main Scarlet X unchanged.

## 2026-10-25 — SXP v0.1.36 compile failure fixed in v0.1.37
- User-provided GitHub Actions screenshot: `Build RootHide package` failed in `ProbePanel.m:621` with four instances of `property 'frame' not found on object of type '__strong id'`, all caused by accessing `label.frame` on an `id` variable in `profile-bio-rendered-metrics`. This is a confirmed compiler error, not a runtime diagnosis.
- Minimal source fix: after verifying `label` is `TFNAttributedTextView` (UIView subclass), take `CGRect bioFrame=((UIView *)label).frame;` and use `bioFrame` coordinates instead. ProbePanel.m commit `36028ad777fae2b1ff61c9ab56dc8f80ec76538e`.
- Bump per-version change to v0.1.37: control `a7ab7ea058e2e34bc06a9db3eafde9db0456ccb7`, ProbePrivateUpdate.m `377a106925e82a82bc0a5cf9706297e15b0f6a97`, Tweak.xm `3ddcb79258672d02b89a925b7d8073e39c1ce013`.
- Build of v0.1.37 remains UNVERIFIED; do not equate commit with successful compilation. Need inspect Actions and on-device behavior. Main Scarlet X untouched.

## 2026-10-25 — SXP v0.1.37 first confirmed on-screen bio typography values
- User sent Drive `SXProbe_2026-10-25_141906.json` (official X 12.35, 46 events). `profile-bio-rendered-metrics` status=`captured`, `textCaptured=false`, from a live T1ProfileUserInfoView `_bioLabel` -> TFNAttributedTextView `_textModel` -> `renderedAttributedString`.
- Sample: textLength=7, bioFrame=[9,12.5,23,54] pt, one style run length 7; UIFont `fontName=.SFUI-Semibold`, `pointSize=12`, `lineHeight=14.3203125`. These are the first confirmed actual font metrics **for this specific bio instance**. Do NOT generalize to all bios/font settings or use it as a fixed Scarlet X default.
- No NSParagraphStyle values were logged in that style run; absence is not proof `lineSpacing=0`. Frame height 54 does not equal font line height; distinguish visible layout height and glyph metrics. Need a longer ordinary multiline bio with wider frame and at least one additional sample before deciding default font/line spacing. Prefer limiting future exported diagnostic to metadata rather than accessible bio string; older snapshot captures visibleText explicitly on manual command.
- No Scarlet X app UI changes made, and no newer SXP code required to acquire another sample. Next: collect 1-2 representative bios and compare typography/style runs, then decide if paragraph/line metrics need extra probing. Build status of v0.1.37 can be inferred from running SXP code, but Actions success specifically was not queried.

## 2026-10-25 — SE controlled multi-line bio / iPhone 15 constraint
- User confirms SXP official-X logs are recorded on first-generation iPhone SE (320×568 logical points, 2x scale), not iPhone 15. User's iPhone 15 has no TrollStore/SXP; do not suggest Probe runtime measurements there. For comparison, use official X runtime readings from SE and Scarlet X screenshots on iPhone 15, while acknowledging cross-device conditions differ.
- Drive SXProbe_2026-10-25_142609.json: X 12.35, 26 events. Live bio rendered metrics: status captured, frame [9,12.5,292,71] pt, model textLength 89, style run lengths 82/1/6. First and last runs: .SFUI-Semibold, 12pt, UIFont lineHeight 14.3203125pt. Middle one-character run has no explicit font property. No paragraph-style metrics recorded. Cause of run split remains unknown.
- Visible snapshot (TFNAttributedTextView): frame [9,12.5,292,71], accessibility textLength 112, four explicit newlines, not truncated. Rendered string and accessible text lengths differ; explanation not yet verified.
- Earlier short sample also yielded .SFUI-Semibold 12pt at width 23pt. Samples are same-device observations, not universal values. Next: consider targeted lineRanges/actualSize capture before editing Scarlet X native layout.

## 2026-10-25 — SXP v0.1.38: live profile bio line metrics
- Carry forward the confirmed test conditions: first-generation iPhone SE, 320×568pt; official X 12.35; do not ask for TrollStore/SXP on user's iPhone 15. The prior controlled bio sample gave 12pt `.SFUI-Semibold`, UIFont lineHeight 14.3203125, live bio view 292×71pt; rendered model length 89 differs from accessible text length 112.
- ProbePanel.m change `ceda0e393e5b7b797f9ca3cbae3125944ab79b5a`: extend the existing manual `profile-bio-rendered-metrics` event with `layoutMetrics`, reading the proven `TFNAttributedTextModel` ivars `_lineRanges` (bounded NSValue/NSRange list, maximum 40), `_actualSize` (CGSize), `_contentTopPadding` and `_contentBottomPadding` (double). Check ivar type and finite size/padding; do not capture string contents. Include `<math.h>` (`502a375c0e8a39b5a897d4a79d0326b9a037777c`). No changes to actual X rendering, hooks, or Scarlet X app.
- SXP version 0.1.38 bumped in control (`70525b8af31603da34f6744eed5bd5c5e1ba7631`), ProbePrivateUpdate.m (`38490768b35c14c90a93674e792f36bccb06bb53`), Tweak.xm (`7769530e2d99007b55e35738226e6b782afb12da`). Build and on-device success NOT verified. Check Actions before installing; then repeat manual capture on the same multi-line bio and upload Drive JSON. Compare line range count and actualSize to 292×71 frame and font lineHeight. If ranges have unexpected types, treat as missing, not valid data.

## 2026-10-25 — SXP v0.1.38 verified bio line-range/size results
- Received Drive `SXProbe_2026-10-25_143344.json`, official X 12.35 on first-generation iPhone SE, 26 events. Manual `profile-bio-rendered-metrics` reports status `captured`, textLength 89; `bioFrame=[9,12.5,292,71]` pt.
- Newly measured model `layoutMetrics`: `actualSize=[292,71]` pt EXACTLY matches text view frame size; `contentTopPadding=0`, `contentBottomPadding=0`, `lineCount=4`, NSRange line segments `(location,length)=(0,21),(21,25),(46,26),(72,17)`, covering all 89 model UTF-16 units without gaps or overlaps. Model values are from actual on-screen `_bioLabel`.
- Attributed run lengths 82/1/6; first/last `.SFUI-Semibold`, pointSize 12, UIFont lineHeight 14.3203125. Middle one-unit run lacks explicit UIFont. No NSParagraphStyle fields measured. Snapshot has accessibility textLength 112, explicitNewlines 4; this is distinct from rendered model textLength 89 and lineCount 4. Why they differ is not determined; do NOT equate logical newlines with actual line ranges or infer effective baseline spacing from `71/4`.
- v0.1.38 runtime diagnostic is functioning in this sample; no Scarlet X production changes. Next: compare exact source/processed text relationship and Core Text typographic line origins/line heights if required, plus compare Scarlet X existing header code and an equivalent SE rendering. Avoid extra probe cycles before using already-confirmed measurements.
