# 公式X v12.25 — bio計測経路の追加候補（2026-10-23）

## 検証した一次資料
- ユーザー提供IPA `com.atebits.Tweetie2_12.25_und3fined(1).ipa` から抽出した `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter`。
- ユーザー提供 `scarlet_macho(1).sqlite` の `objc_methods` 索引。
- `llvm-objdump -d --start-address=0x141cd8 --stop-address=0x141f00` による実命令確認。

## 確認済み
- `_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:` が`T1Twitter`内に存在。IMP `0x141CD8`（decimal 1318104）、method list `0x1393E10`（decimal 20528656）、list index 9。
- 同一method listには`sizeForStaticComponent:withOptions:layoutMetrics:`、`sizeForDynamicComponent:withOptions:size:layoutMetrics:cachedTextSizing:`、`_t1_sizeForUsernameWithOptions:size:layoutMetrics:cachedTextSizing:`、`_t1_sizeForFullNameWithOptions:size:layoutMetrics:cachedTextSizing:`、`_t1_sizeForAvatarWithOptions:size:layoutMetrics:`など、計22メソッドが並ぶ。
- `0x141CD8`以降のARM64実命令に複数の条件分岐と外部スタブ呼び出しがある。単なる `bioLabelFrame` getter/setterとは別の処理であることは確認できる。

## 評価と限界
- メソッド名と周辺APIから、ユーザー表示要素としてのbioの**サイズ計測に関係する候補**として優先調査する価値がある。
- **現時点でこのmethod listの所有クラスをObjective-C metadataから照合していない。通常プロフィール画面のレイアウト計算元とも、`T1UserViewLayoutState.setBioLabelFrame:`の呼出元とも確定していない。**
- メソッド名中の`cachedTextSizing`はキャッシュの受け渡しを示唆するが、実際のキャッシュの使い方、文字列・フォント・幅・高さの具体的な導出はスタブ解決前で未確定。
- すでに判明している`T1UserRecommendationViewLayoutDelegate._t1_layoutBio:user:width:yCoordinate:`→`setBioFrame:`とは別のmethod list。
- 今回、Scarlet X本体/Probeへのコード変更、ビルド、実機検証は行っていない。

## 次の手順（SEなしで可能）
1. `0x1393E10`の所有クラスをclass_roとbaseMethodsで特定。
2. `0x141CD8`のObjC stubをselector名に解決し、文字列、サイズ、フォント、キャッシュの利用箇所を切り分ける。
3. この計測処理の戻り値が`T1UserViewLayoutState.bioLabelFrame`に接続するか、呼出元・layoutState生成経路を確認する。

## 追加検証：所有クラスを実バイナリで確認
- Mach-O LC_SEGMENT_64でVMアドレスとファイルオフセットを対応させ、8バイト境界のbaseMethodsポインタ（下位36bit `0x1393E10`）を検索。
- 当該baseMethodsを持つclass_roはVM `0x195EDC8`、`baseMethods`フィールドは `0x195EDE8`。
- class_roの`name`ポインタが指すC文字列は**`T1UserView`**。
- したがって、`_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:`を含む同一method listの所有クラスは**`T1UserView`**と確認できた。
- ただし`T1UserView`のメソッドであることと、プロフィール自己紹介欄の実際のCGRect値への接続は別問題。呼出元と戻り値の伝搬は未確認。
