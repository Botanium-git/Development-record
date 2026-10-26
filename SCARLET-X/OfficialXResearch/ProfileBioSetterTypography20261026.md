# 2026-10-26 — Profile detail Bio setter typography/model pipeline

## Scope and duplicate avoidance
Extends the already verified v12.25 `T1ProfileUserInfoView.updateFontSize` → `_t1_refreshBioLabel` → `_t1_setBioLabel:withText:entities:isUTF16Entities:` path, verified in ProfileUpdateFontSizeIMP20261026.md. Prior `T1UserViewBuildTextModel` metricsFont fallback is a **different** UI path; do not substitute its layout metrics assumptions here.

Original artifact: user-provided official X v12.25 IPA `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter`, Mach-O ARM64. Target selector owner `T1ProfileUserInfoView`, IMP `0x257E98`.

## New instruction/stub evidence

- At `0x257F84`, classref is passed to selector `xds_body` via stub `0x1373CA0`; at `0x257F98`, returned UIFont is messaged `lineHeight` (stub `0x1315580`), result increased by immediate double **2.5** at `0x257F9C..0x257FA0`. The font is fetched a second time at `0x257FB0`. This is the font argument (x3) passed at `0x257FE4` to initializer stub `0x13039C0` = `initWithString:font:alignment:lineBreakMode:paragraphSpacing:lineHeight:`, with text argument x2, x5=0, d0=0, d1 holding the 2.5-augmented line height. **Exact downstream typography and font-size provider inside xds_body are not yet known.**
- `0x257FF0` invokes that initializer; at `0x25809C..0x2580A0`, alloc+ `initWithAttributedString:` (`0x12FD160`) produces a text-model object. At `0x258218..0x258220`, recipient Bio label x19 receives `setTextModel:` (stub `0x134EEC0`), with its view model x27. Relevant color/active text styling and `setAttributedTrailingString:` (stub `0x13368C0` at 0x2581D8) occur along the way.
- `0x2580C4` calls `isBioExpanded` (stub `0x1308000`). If false, same model x27 receives `setMaxLines:` (stub `0x1344240`) with w2=4 at `0x2580D0..0x2580D4`, and `setLineBreakMode:` (stub `0x13433E0`) with w2=4 at `0x2580DC..0x2580E0`. Do **not** infer that UIKit's numerical lineBreak enum and the proprietary TFN interpretation are exactly interchangeable without confirming API contract.
- `0x258068` branch checks active range metadata; it can change subsequent formatting, and `0x258200` hides bio label when content absent. All callsite observations concern official v12.25 **static** implementation.

Selector identities resolved from original objc-stub `adrp x1 / ldr x1` selrefs then original Mach-O chained pointers (rather than guesswork from symbol labels):
`0x1373CA0 = xds_body`, `0x1315580 = lineHeight`, `0x13039C0 = initWithString:font:alignment:lineBreakMode:paragraphSpacing:lineHeight:`, `0x12FD160 = initWithAttributedString:`, `0x134EEC0 = setTextModel:`, `0x1308000 = isBioExpanded`, `0x1344240 = setMaxLines:`, `0x13433E0 = setLineBreakMode:`.

## Consequences / remaining work

This verifies a direct **profile-detail bio model uses UIFont xds_body**, which is more specific than the previous generalized T1UserView metricsFont fallback. It does NOT establish xds_body's dynamic font size source, actual 12.35 baseline or cross-device category effects, and does NOT establish the archived fontSizeWithBaseline... formula is called for this bio. Next inspect `+[UIFont xds_body]` implementation and its token/provider; then identify how it resolves size via category/windowSize. Also inspect `TFNAttributedTextModel` trailing string and maxLines paths when tackling 'さらに表示' separately.

Reproduce:
```
llvm-objdump -d --start-address=0x257e98 --stop-address=0x2582a8 T1Twitter
llvm-objdump -d --start-address=0x13039c0 --stop-address=0x13039d4 T1Twitter
llvm-objdump -d --start-address=0x134eec0 --stop-address=0x134eed4 T1Twitter
```
No SCARLET-X or SXP modification, no device test, no build.
