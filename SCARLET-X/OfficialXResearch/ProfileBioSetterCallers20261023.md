# 公式X v12.25：通常プロフィール bio CGRect setter 呼出元探索（2026-10-23）

## 一次資料
ユーザー提供 `com.atebits.Tweetie2_12.25_und3fined(1).ipa` 内 `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter`、`scarlet_macho(1).sqlite`。ローカル環境でARM64命令・ObjC stub・`__objc_selrefs`を照合。バイナリの固定VM addressを用いている。

## 確認
- `T1UserViewLayoutState.setBioLabelFrame:`: IMP `0x3AFDE8`; method list `0x13E7490`。
- `setBioLabelFrame:`に対応するObjC stubは **`0x1337B60`**（ADRP/LDRで参照するselrefからselector文字列を解決）。
- `bioLabelFrame`のObjC stubは **`0x12DADE0`**。
- T1Twitterの `__text` 範囲 `0x4000..0x129A124` を4バイト境界で走査し、ARM64 `BL`の分岐先を算出したところ、**`0x1337B60`への直接BLは0件**、**IMP `0x3AFDE8`への直接BLも0件**。
- 対照として、getter `0x12DADE0`へのBLは1件（`T1UserViewLayoutState.applyToUserView:`中の`0x3AF874`）を検出。既存調査と一致。
- 同クラスには、他にも多数のCGRectのgetter/setterが存在（`userNameLabelFrame`, `fullNameLabelFrame`, `avatarFrame`, `bioLabelFrame`など）。ただしこれは計算元の証明ではない。

## 結論の限界
- **「直接BL呼出しが見つからなかった」ことのみ確認。** Objective-Cの`objc_msgSend`経由、間接呼出し、他frameworkのコード、直接ivar書込み等は今回のBL走査の対象外なので「未使用」「呼出元なし」と断定しない。
- 既出の `T1UserRecommendationViewLayoutDelegate._t1_layoutBio:user:width:yCoordinate:` → `T1UserRecommendationViewLayoutState.setBioFrame:` は別系統。この経路と通常プロフィール用 `setBioLabelFrame:` を混同しない。
- 通常プロフィールのCGRect計算規則／本番画面の実測値はいまだ未確認。Scarlet XへのUI修正は行わない。

## 次の調査
1. T1Twitter以外のframeworkからのselector参照と対応する呼出を調べる。または`T1UserViewLayoutState`生成・組立経路を追う。
2. `+0xD8`への直接格納候補をクラスの生成経路と照合して、計算処理を切り分ける。
3. SEの公式X v12.35上で、対象ビューとサイズの実測を行うにはProbeの追加計測機能が必要（v12.25静的オフセットをそのまま流用しない）。

## SEに関する判断
- 実機でプロフィールbioのCGRect・フォント・幅・実画面の状態が読み取れれば、望む表示の**値**は短縮して確認できる。
- 一方、異なる画面幅・Dynamic Type・折返しなどを包含する**一般的な配置規則**の解明は1回の実機計測だけでは足りない。複数画面・設定条件での比較、または静的解析との照合が必要。
