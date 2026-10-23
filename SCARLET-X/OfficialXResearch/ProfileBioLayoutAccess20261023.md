# 公式X v12.25 — bioLabelFrame の格納と applyToUserView: （2026-10-23）

## 一次資料と調査方法
- ユーザー提供IPA `com.atebits.Tweetie2_12.25_und3fined(1).ipa` の `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter` を抽出。
- ユーザー提供 `scarlet_macho(1).sqlite` の `objc_methods` で `T1UserViewLayoutState` 所属の IMP を照合。
- `llvm-objdump -d --start-address=0x3afdc0 --stop-address=0x3afe18 T1Twitter` と `--start-address=0x3af410 --stop-address=0x3afd38` でARM64実命令を確認。

## 実命令で確認済み
- `bioLabelFrame` IMP `0x3AFDDC` は `ldp d0,d1,[x0,#0xD8]` / `ldp d2,d3,[x0,#0xE8]` / `ret`。CGRectの4 doubleをオブジェクトのオフセット `0xD8..0xF7` から読み出す。
- `setBioLabelFrame:` IMP `0x3AFDE8` は `stp d0,d1,[x0,#0xD8]` / `stp d2,d3,[x0,#0xE8]` / `ret`。渡されたCGRectをそのまま保存。**このsetterにフレーム計算処理は存在しない**。
- `applyToUserView:` IMP `0x3AF410` は数多くの外部スタブ経由のCGRect getter/setter呼出しと、返された4 double（v0..v3）の受渡しを含む。呼び出し先スタブ `0x133F080` が繰り返し登場し、ビュー側に複数のレイアウト状態を渡す処理であることが命令上裏付けられる。
- ただし現時点で `applyToUserView:` の各スタブをselectorまで解決しておらず、**bioLabelFrameに対応する個別callsite・適用先UIView/UILabelは未確定**。またsetterの呼出元は未特定。

## 解析で判明した境界
- `bioLabelFrame` と `setBioLabelFrame:` は保存場所を示すaccessorであって、幅・行高・位置の決定規則そのものではない。
- `applyToUserView:` は有力な最終適用経路だが、実際にどのviewへbio CGRectを代入するかの証明にはスタブ／selector解決が必要。
- v12.25静的結果を、SE公式X v12.35でも同じ実装と仮定しない。公式X実測値は未取得。

## 次の限定調査
1. `applyToUserView:` 内の呼出先スタブをObjC selectorへ解決し、`bioLabelFrame`→`setFrame:` の対応箇所を特定。
2. `setBioLabelFrame:` 呼出元、または対応IVARへの直接書込みがないかを調べてフレーム計算元を追う。
3. アプリ本体UIは規則が判明するまで変更しない。
