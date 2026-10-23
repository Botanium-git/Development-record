# 公式X v12.25 — bioLabelFrame の所有クラス（2026-10-23）

## 対象と再現可能な根拠

- 一次資料: ユーザー提供 `com.atebits.Tweetie2_12.25_und3fined(1).ipa` の `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter`
- 当該Mach-O SHA-256: `1af51e57d746a81546da299e103241da9682ac397f33475defec7f6a3e6e5ccd`
- 併用: `scarlet_macho(1).sqlite` の `objc_methods` 索引
- 確認方法: SQLiteの同一 `list_address` 群を列挙。Mach-O内のObjective-C class `data`→class_ro `baseMethods`→method list、およびclass_ro `name` を実バイトから追跡。ポインタの下位36bitが当該VMアドレスを示す観測形式を用いた。

## 確認済み

- `bioLabelFrame` の索引レコード: binary `T1Twitter`, IMP `0x3AFDDC`, method list `0x13E7490`, entry_index 15。
- `0x13E7490` のメソッドリストは51件で、`size`, `applyToUserView:`, `userNameLabelFrame`, `fullNameLabelFrame`, `setBioLabelFrame:`, `avatarFrame` などを含む。
- 実バイナリのclass_roはVM `0x19ABE10`; `baseMethods` pointer（class_ro+32）が `0x13E7490` を指す。
- class_ro+24の名前ポインタは `T1UserViewLayoutState` を指す。
- VM `0x1AF2688` にあるObjective-Cクラスのdata（+32）が上記class_ro `0x19ABE10` を指す。
- したがって、**`bioLabelFrame` を含む51個のインスタンスメソッドを保持するクラスは `T1UserViewLayoutState`**。

## 解釈と未確定部分

- クラス名・フィールドの構成から、プロフィール各要素のレイアウト状態オブジェクトである可能性が高い。ただし、プロフィール画面のどの状態で生成・適用されるか、各CGRectを計算するメソッドとその呼出元はまだ未確定。
- `applyToUserView:` は描画への適用経路を調べる次の有力候補。存在だけでは特定のviewへの適用・最終CGRectは証明できない。
- SQLite/IPAはv12.25、SEで動かすProbe側はv12.35なので、クラス名・オフセット・挙動が同じとは仮定しない。
- 実測のフォント、本文サイズ、bioのCGRect値は未取得。Scarlet Xの本体UIに変更を加える根拠にはまだ不足。

## 次の限定的調査

1. `T1UserViewLayoutState applyToUserView:` の呼出し・プロパティ読み出しを解析し、`bioLabelFrame` の利用先を追う。
2. `setBioLabelFrame:` に書き込む計算・呼出元を追い、テキスト幅・高さの決定過程へ接続する。
3. v12.35のSE実機で必要なクラス・メソッドの存在と測定値を読み取り専用Probeで検証（端末が使えるとき）。
