# T1UserView bio計測の共通ディスパッチ（2026-10-23）

## 既存解析との照合
OfficialXTextSizingDisassembly.md の OX-020/021 は主に投稿本文の TTAStatusBodyTextView 系であり、プロフィールbioに同じ書体・行高規則が適用されると決めつけない。

## 今回の検証
一次資料: ユーザー提供X v12.25のT1Twitter Mach-O、SQLite objc_methods、llvm-objdump。

- `T1UserView sizeForDynamicComponent:withOptions:size:layoutMetrics:cachedTextSizing:` は IMP `0x14162C`。
- 入口 `0x141644..0x14165C` は component x2→x23、options x3→x19、layoutMetrics x4→x20、cachedTextSizing x5→x21 を保持する。
- `0x141678` はcomponent値と12を比較。範囲内なら `0x141680..0x141694`でbyte jump table (`0x144DEE1`)を使って分岐する。
- 各分岐はx0=self、x2=options、x3=layoutMetrics、x4=cachedTextSizing、v0/v1=sizeを準備して別個のObjC selector stubへBLする。
- 同じクラス・method list `0x1393E10` に `_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:` (IMP `0x141CD8`) が存在する。ただしjump tableのcomponent番号→bio selector対応は今回まだ未解決。
- `0x141C34..0x141CD4` はbioではなく `_t1_sizeForAvatarWithOptions:size:layoutMetrics:cachedTextSizing:`。この中にある `T1UserViewLayoutDelegateClassForStyle` 呼び出しをbioの経路と混同しない。

## 未確定・次
bio case selector stubの正確な割当て、上位caller、layoutMetrics生成元、bioCGSizeとbioLabelFrameの接続、UIFont実数値、X v12.35実機での採用は未確定。
次にjump tableを復元し、その後上位callerの引数を追う。
Scarlet X本体・Probeは変更しない。