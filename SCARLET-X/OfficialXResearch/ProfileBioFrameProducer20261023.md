# 公式X v12.25 — プロフィールbio CGRect生成候補（2026-10-23）

## 一次資料
- ユーザー提供の公式X v12.25 IPAから取り出した `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter`。
- `scarlet_macho(1).sqlite` の `objc_methods` に対する selector/IMP照合。
- `llvm-objdump` のARM64実命令と、ObjC stubのADRP/LDR→selref→selector文字列を直接確認。

## 確認済み
- `_t1_layoutBio:user:width:yCoordinate:` のIMPは `0x1C0F70`。
- 関数内部 `0x1C0FAC` のstub `0x12DACA0` = `bioDisplayText`。
- `0x1C0FBC` のstub `0x130D6A0` = `isNotEmptyAfterTrimmingWhitespace`。
- 文字列が空でない分岐で、`edgePadding` (`0x12EE840`)、`_t1_bioTextModelForUser:` (`0x12B94E0`)、`fitToWidth:` (`0x12F2580`)、`actualSize` (`0x12D2A60`) のselectorを確認。
- `0x1C102C`のstub `0x1337B20` は `setBioFrame:`。この直前 `0x1C101C`〜`0x1C1028` ではv0〜v3にCGRect相当の4 doubleをセットするため、計算した矩形をlayout-state相当の受信オブジェクトへ渡す経路と判断できる。
- 空の分岐にも `setBioFrame:` (`0x1C1054`) があり、0のCGRectを読み込む命令が存在する。
- **重要: `setBioFrame:` と `setBioLabelFrame:` は別selector。** SQLiteでは`setBioFrame:`のIMPが `0x1BE380`（別method list）、`setBioLabelFrame:`のIMPが `0x3AFDE8`。今回の関数を前回の`T1UserViewLayoutState.bioLabelFrame`に直接結びつける証拠はまだない。

## 未確定
- `setBioFrame:`の所有クラス、`_t1_layoutBio...`の所有クラスと各メソッドの呼出元。
- `setBioFrame:`に渡された矩形と`T1UserViewLayoutState.bioLabelFrame`の関係。画面やUIコンポーネントが別系統の可能性を排除しない。
- `actualSize`の高さが最終的なUILabel/UITextViewのframeにどのように反映されるか、fontや行高の実値。
- iOS実機の公式X v12.35で同一経路が利用されること。

## 次の優先調査
1. `setBioFrame:` / `bioFrame` のmethod list所有クラスを実バイナリのObjC metadataで解決。
2. `_t1_layoutBio:user:width:yCoordinate:`を呼び出す処理と、`setBioFrame:`を受けるlayout-stateの画面適用処理を追う。
3. `T1UserViewLayoutState.bioLabelFrame`との関連有無を確かめる。関係がなければ異なる画面レイアウト経路として記録し、元の探索へ戻る。

## 追加検証：両クラスの所有者を確定（同日）

SQLiteのmethod listアドレスを実Mach-Oのclass_ro `baseMethods` ポインタに照合し、class_roの名前参照を確認した。

- `_t1_layoutBio:user:width:yCoordinate:`: IMP `0x1C0F70`、method list `0x13E65F8`、class_ro `0x1972528`、**所有クラス `T1UserRecommendationViewLayoutDelegate`**。
- `bioFrame`: IMP `0x1BE374`、`setBioFrame:`: IMP `0x1BE380`、method list `0x13E6688`、class_ro `0x1971E58`、**所有クラス `T1UserRecommendationViewLayoutState`**。
- したがって今回確認したCGRect計算・保存は **ユーザーおすすめビュー系のレイアウト**。前回の `T1UserViewLayoutState.bioLabelFrame` と同一経路だとは確認できない。名称から一般的なプロフィールヘッダーの実装だと扱うことはしない。
- 次にプロフィール通常表示を追うなら、`T1UserViewLayoutState`への`setBioLabelFrame:`呼出元（またはオフセット`+0xD8`への直接書込み）を改めて探索する。
