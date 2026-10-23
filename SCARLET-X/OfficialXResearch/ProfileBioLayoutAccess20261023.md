# 公式X v12.25 — bioLabelFrame の格納と applyToUserView: （2026-10-23）

## 一次資料と調査方法
- ユーザー提供IPA `com.atebits.Tweetie2_12.25_und3fined(1).ipa` の `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter` を抽出。
- ユーザー提供 `scarlet_macho(1).sqlite` の `objc_methods` で `T1UserViewLayoutState` 所属の IMP を照合。
- `llvm-objdump -d --start-address=0x3afdc0 --stop-address=0x3afe18 T1Twitter` と `--start-address=0x3af410 --stop-address=0x3afd38` でARM64実命令を確認。

## 実命令で確認済み
- `bioLabelFrame` IMP `0x3AFDDC` は `ldp d0,d1,[x0,#0xD8]` / `ldp d2,d3,[x0,#0xE8]` / `ret`。CGRectの4 doubleをオブジェクトのオフセット `0xD8..0xF7` から読み出す。
- `setBioLabelFrame:` IMP `0x3AFDE8` は `stp d0,d1,[x0,#0xD8]` / `stp d2,d3,[x0,#0xE8]` / `ret`。渡されたCGRectをそのまま保存。**このsetterにフレーム計算処理は存在しない**。
- `applyToUserView:` IMP `0x3AF410` は数多くの外部スタブ経由のCGRect getter/setter呼出しと、返された4 double（v0..v3）の受渡しを含む。呼び出し先スタブ `0x133F080` が繰り返し登場し、ビュー側に複数のレイアウト状態を渡す処理であることが命令上裏付けられる。
- さらにObjC stubの`ADRP/LDR`から`__objc_selrefs`をたどり、**`applyToUserView:`の`0x3AF874`で`bioLabelFrame`を呼び、直後に保持したCGRectを`0x3AF8AC`の`setFrame:`呼出しへ渡す経路を確認**。`bioLabelFrame`側のstub `0x12DADE0`、`setFrame:`側のstub `0x133F080`も実バイトからselector解決した。ただし`setFrame:`の**受信ビューのクラス・プロパティ名は未確定**。またsetterの呼出元は未特定。

## 解析で判明した境界
- `bioLabelFrame` と `setBioLabelFrame:` は保存場所を示すaccessorであって、幅・行高・位置の決定規則そのものではない。
- `applyToUserView:` は有力な最終適用経路だが、実際にどのviewへbio CGRectを代入するかの証明にはスタブ／selector解決が必要。
- v12.25静的結果を、SE公式X v12.35でも同じ実装と仮定しない。公式X実測値は未取得。

## 次の限定調査
1. `applyToUserView:` の `0x3AF874`→`0x3AF8AC` でCGRect受渡しは確認済み。`setFrame:`の受信ビューを返す直前の呼び出し（`0x3AF888`付近）をselector解決し、その所有ビューを特定。
2. `setBioLabelFrame:` 呼出元、または対応IVARへの直接書込みがないかを調べてフレーム計算元を追う。
3. アプリ本体UIは規則が判明するまで変更しない。

## 追加確認 — bioの適用先プロパティを解決
- `applyToUserView:` の `0x3AF88C` が呼ぶObjC stub `0x12DAC80` を`__objc_selrefs`から解決し、selector **`bioAttributedTextView`** と確認。
- この戻り値が `objc_retainAutoreleasedReturnValue` を経由して `x21` に置かれ、`0x3AF8AC` の `setFrame:` のreceiverに渡される。
- **確認できた連鎖:** `T1UserViewLayoutState.bioLabelFrame` → （引数`userView`の）`bioAttributedTextView` → `setFrame:`。これにより表示適用対象のプロパティ名まで特定できた。
- `bioAttributedTextView` が返す実際のビュークラス、`setBioLabelFrame:`への入力CGRectを計算する関数、フォント／行高は引き続き未確定。
