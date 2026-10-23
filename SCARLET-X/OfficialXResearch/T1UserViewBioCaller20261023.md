# X v12.25 T1UserView bio計測の上位caller追跡（2026-10-23）

## 既存記録との照合
- `T1UserViewBioDispatcher20261023.md`: dynamic component 4 はbioにdispatch。
- `T1UserViewMetricsProvenance20261023.md`: bio計測内でmetricsFont/lineHeightへ引数が伝搬。
- OX-020/021は投稿本文側のfont/lineHeight経路であり、プロフィールbioへの転用は未実証。

## 新規一次確認
ユーザー提供のX v12.25 `T1Twitter` Mach-O + `scarlet_macho(1).sqlite`。
- ObjC selector `sizeForDynamicComponent:withOptions:size:layoutMetrics:cachedTextSizing:` のC文字列 VM `0x168C7E5` → selref `0x1ABE738` → stub `0x13595C0`（ADRP/LDRを復号）。
- `__TEXT,__text` のARM64 direct BL走査で、stub `0x13595C0`へのBLは**1件**：`0x2526D8`。
- 呼出元はSQLiteのindexと実命令を照合した `sizeForComponent:withSize:layoutMetrics:` IMP `0x252654`（method list decimal `20870144`）。このmethod listには`initWithOptions:layoutMetrics:estimated:` IMP `0x252510`、`cachedTextSizing` IMP `0x25271C`、`options` IMP `0x25274C`もある。
- 呼出元の命令：`0x252668 mov x19,x3`で渡されたlayoutMetricsを退避、`0x2526D4 mov x4,x19`、`0x2526D8 bl 0x13595C0`。同時にcomponentはx2（x21保持）を転送、optionsはself+0x20、cachedTextSizingはself+0x10から取得する。
- `0x1AD3790` classrefの値は下位36bit `0x1AE5C08`で、SQLite symbolsは`_OBJC_CLASS_$_T1UserView`と照合。つまりこのcallsiteは**T1UserViewクラス宛**。
- `0x252510`のinitializerは引数options（x2）とlayoutMetrics（x3）を受け取るが、実際にinit後のオブジェクトに保存される構造や呼出元のさらなる追跡は未完了。

## 解釈の境界
- **確定**：component4→bio; `sizeForComponent`→`T1UserView.sizeForDynamicComponent`→bio計測→text modelの静的連鎖。
- **未確定**：上位呼出元がどこでlayoutMetricsを生成するか、プロフィール画面の実際の採用経路、CGSize→bioLabelFrameの接続。ObjC動的呼出・他バイナリへの完全な呼出追跡ではない。
- SE v12.35実測、アプリ・Probeコード変更、ビルドは今回なし。

## 次
1. method list `20870144` のclass_ro所有クラスを確認。
2. `sizeForComponent:withSize:layoutMetrics:` selector stubのcallersを追う。
3. `initWithOptions:layoutMetrics:estimated:`呼出元からlayoutMetrics作成元を探し、最終CGRectへのデータフローを検証。

## 所有クラス追加確認
- 元Mach-O実バイトを走査し、method list VM `0x13E7400` (=20870144)を指すclass_ro `baseMethods`ポインタをVM `0x19882B8`で検出。class_ro先頭VM `0x1988298` のname pointerが指すC文字列は **`T1UserViewLayoutParameters`**。
- よって`initWithOptions:layoutMetrics:estimated:`および`sizeForComponent:withSize:layoutMetrics:`は **`T1UserViewLayoutParameters`のメソッド**と確認。前節の「所有名未確認」は本追加確認で解消。

## さらに上位の呼出箇所（2026-10-23追記）
- `sizeForComponent:withSize:layoutMetrics:`のC文字列VM `0x168C762`→selref `0x1ABE718`→ObjC stub `0x1359540`。
- 元T1Twitter実命令の直接BL走査でこのstubへのcallsiteは**15件**: `0xD1CD4`, `0x14C998`, `0x4835C8`, `0x483738`, `0x4839A8`, `0x483C1C`, `0x483D8C`, `0x483FFC`, `0x48416C`, `0x4842DC`, `0x48444C`, `0x4845BC`, `0x48482C`, `0x484AC0`, `0x484D38`。
- SQLiteで直前のメソッド開始位置を照合すると、`0x14C998`は`layoutItemsForLayoutContainer:withParameters:layoutState:contentRectProvider:`（開始`0x14C05C`）の範囲内、`0x4835C8..0x484D38`の13件は`sizeThatFitsHelper:layoutSubviewsForView:atWidth:`（開始`0x483408`）の範囲内。`0xD1CD4`も同名のlayoutItems系開始`0xD1594`以降。ただし境界・呼出receiverの確認を要するため、これをプロフィール画面採用と断定しない。
- `0x14C998`では`component=10`、x3はx19から渡す。**component4（bio）を呼んでいるとの証拠ではない**。各callsiteのcomponent番号とmetricsの出自を別々に確認する必要がある。

## Correction 2026-10-23: function boundaries and component calls

Direct llvm-objdump of original T1Twitter shows multiple independent function entries between the thirteen callsites, so they must NOT all be attributed to a single `sizeThatFitsHelper` method based on the preceding indexed IMP. This corrects the preliminary statement in the section above.

At calls to stub `0x1359540` the immediate component argument `w2` is:

| BL address | component |
|---|---:|
| `0x4835c8` | 0 |
| `0x483738` | 1 |
| `0x4839a8` | 2 |
| `0x483c1c` | 3 |
| `0x483d8c` | 4 |
| `0x483ffc` | 5 |
| `0x48416c` | 6 |
| `0x4842dc` | 7 |
| `0x48444c` | 8 |
| `0x4845bc` | 9 |
| `0x48482c` | 10 |
| `0x484ac0` | 11 |
| `0x484d38` | 12 |

**Bio confirmed:** `0x483D7C mov w2,#4`, `0x483D88 mov x3,x19`, `0x483D8C bl 0x1359540`. Caller-function ownership, x19 provenance, runtime screen selection and final CGRect remain unverified.


## 2026-10-23 — bio呼出関数の入口・実際の入力・CGRect setterブロック（追加）
- ユーザー提供v12.25 T1Twitter実バイナリを `llvm-objdump` で `0x483C58..0x483DC4`まで確認。
- `0x483D40` に独立したARM64関数プロローグ、`0x483DBC ret` を確認。`0x483D50 mov x19,x2`で第3引数を保持、`0x483D5C mov x20,x1`で第2引数を保持。
- `0x483D68 mov x0,x20; 0x483D6C bl 0x1363600`: stubのADRP/LDR→selref VM `0x1AC0F48`→selector **`tfn_flexibleLayoutViewModel`** と解決。返ったモデル相当のobjectを `0x483D78`でx20に保持。
- `0x483D7C mov w2,#4`, `0x483D88 mov x3,x19`, `0x483D8C bl 0x1359540`: `sizeForComponent:withSize:layoutMetrics:` を component=4、metrics=x19 で送る。呼出 receiver は直前の `tfn_flexibleLayoutViewModel`の返り値（x20）。以前の「x19の由来未確定」は、この関数内では**第3引数x2**まで解決。
- 近接する `0x483DC0 ldr x0,[x0,#0x20]; 0x483DC4 b 0x1337B60` はstub selector `setBioLabelFrame:`。受信対象は先頭引数のblock/objectのoffset+0x20の参照先。ただし実際のclassは未確定。
- その直前の別関数 `0x483C58` は `0x483CB0 add x8,x8,#0xDC0` で `0x483DC0`へのfunction pointerをブロック状の構造に格納し、`0x483CC4 str x19,[sp,#0x28]` でcaptureを組み立てる。`0x483CD8 bl 0x1365140`でこのblockを渡す。
- **重要:** setter側のスタブに対する通常BLがゼロでも、block invokeからtail branch（B）する呼出経路が存在する。これが以前のBLのみの検索で拾えなかった理由の一つ。
- 以上によりbio計測経路と`setBioLabelFrame:`のblock化処理が近接し、共通のレイアウト実装群にあることを実証。ただし **個別のblockがどの条件・順番で実行され、得たCGSizeがどのCGRect座標へ変換されるかは未確定**。画面ごとのruntime採用・v12.35の同一性も未確認。
- 次: `0x1365140` のObjC selectorとblock保持先、`0x483C58`のowner/呼出元、x2で渡るmetricsの生成元を追う。アプリ・Probeは未変更。