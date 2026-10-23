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
