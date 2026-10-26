# OX-FONT-20261026 — Dynamic font size helper: binary recheck and remaining gaps

Date: 2026-10-26. Source: user-provided official X v12.25 IPA, `Payload/Twitter.app/Frameworks/XServiceLibraries.framework/XServiceLibraries`. Earlier OX-H03 already documented the entry IMP 0x244F2F4, helper 0x244ED5C, 12 category offsets, and frinta. **Do not treat this as a new discovery of OX-H03.** This note records additional instruction-level confirmation and precise outstanding unknowns. The IPA is not the official X v12.35 currently observed on first-gen SE.

## Newly rechecked original instructions

- `0x244F2F4` entry: saves float registers; `0x244F328..330` calls `_TFNWindowSizeClassForSize` at `0xD57124`, places resulting size-class enum in x2; `0x244F334..340` forwards x0=content category, x1=modifier, x2=size class, d0=baseline to helper `0x244ED5C` (register role based on selector name and ABI; concrete ObjC binding still needs tracing).
- Helper `0x244ED5C`: `0x244EDC8..D4` optionally substitutes 0 for window class when a boolean selector returns nonzero; `0x244EDE0..F8` calls selector stub `0x2F161C0` with selected category object, adds returned integer to x20(modifier) and x21(effective window class), converts signed sum to double and adds baseline. `0x244EDF8 frinta d8,d0` rounds to integral double (ARM64 ties-to-even FP instruction mnemonic frinta: nearest with ties away from zero, verify ISA precise tie behavior before reproducing). **Correction: frinta means nearest with ties away from zero.**
- `_TFNWindowSizeClassForSize` `0xD57124..0xD5716C` compares `min(width,height)` with three double thresholds loaded from VM `0x3082030/0x3082038/0x3082040` (load addresses verified from adrp+ldr). Returns enum 0,1,2,3 in increasing size bands. Threshold doubles directly decoded from original Mach-O: 513.0 at 0x3082030, 401.0 at 0x3082038, 376.0 at 0x3082040 (mapped __TEXT VM address to file offsets). Thus min dimension <376=>0, [376,401)=>1, [401,513)=>2, >=513=>3 for ordinary finite dimensions. Class 0/1 division is from comparison at 0xD57154, class 2 at 0xD57168, class 3 at 0xD57160.
- The helper calls an as-yet unresolved selector stub at `0x2F188A0` for the flag gating the class adjustment. **Do not identify it as `ios_ui_font_size_ignore_device_width_enabled` until its selref/target is resolved.** A previous ledger reports that feature key but this precise caller association remains unverified.
- The prior OX-H03 offsets `[-2,-1,0,0,+1,+3,+6,+10,+15,+20,+25,+30]` were NOT re-extracted in this check. Category selector `0x2F161C0` is unresolved here.

## What remains unconfirmed

1. Resolve Objective-C selrefs for `0x2F161C0` and `0x2F188A0`, and confirm the config gate mapping, modifier semantics and selector ownership.
2. Prove that profile-detail bio on official X v12.35 uses this helper, its baseline, window size, current X text setting, and bounds. v12.25/v12.35 versions differ; static function presence alone is insufficient.
3. Compare SE v12.35 live category/width/weight against same-device change and iPhone 15 comparison; no iPhone 15 runtime hook data currently.
4. Do not port a formula to Scarlet X Bio until provider linkage and inputs are proven.

## Reproduction

```sh
unzip -p 'com.atebits.Tweetie2_12.25_und3fined(1).ipa' 'Payload/Twitter.app/Frameworks/XServiceLibraries.framework/XServiceLibraries' > XServiceLibraries
llvm-objdump -d --start-address=0x244F2F4 --stop-address=0x244F368 XServiceLibraries
llvm-objdump -d --start-address=0x244ED5C --stop-address=0x244EE1C XServiceLibraries
llvm-objdump -d --start-address=0xD57124 --stop-address=0xD57170 XServiceLibraries
```

No changes to SCARLET-X / SXP, no new build, no device test.


## 2026-10-26 follow-up — resolved flag and category offset selector from original IPA

Extracted bytes directly from the user-supplied official X v12.25 IPA XServiceLibraries Mach-O and disassembled only scoped original instruction windows. This corrects the earlier unresolved selector status; no prior offsets or category formulas are claimed as newly discovered.

- Helper at `0x244EDBC` obtains receiver from `0x43E9C50`; ObjC stub `0x2F188A0` loads selector pointer from `0x43C2C20`. Mach-O chained pointer low address `0x3705875` resolves to original `isFontSizeIgnoreDeviceWidthEnabled` C string. This gives direct method-name evidence matching prior archived key `ios_ui_font_size_ignore_device_width_enabled`; actual config source/value on each device remains unobserved.
- The `0x2F161C0` stub reads selector pointer `0x43C2268`, original string `integerValue` at `0x37025EC`. Receiver is selected category object (`x19`) prepared by the helper. Thus this path converts a category-derived object to integer, but the exact category-to-offset selector/provider remains to trace.
- `0x244EDC8 cmp x21,#1`; `0x244EDCC csinc x8,x21,xzr,lt`; `0x244EDD0 cmp w0,#0`; `0x244EDD4 csel x21,x8,x21,ne`: **if flag true effective window class = min(original class,1),** for nonnegative class values, not zero. If flag false original class retained.
- `0x244EDE4 bl integerValue`; `0x244EDE8 add x8,x0,x20`; `0x244EDEC add x8,x8,x21`; `0x244EDF0 scvtf d0,x8`; `0x244EDF4 fadd d0,d8,d0`; `0x244EDF8 frinta d8,d0`. For ordinary finite input and signed integer arithmetic without overflow the resulting calculation is `roundTiesAway(baseline + categoryInteger + modifier + effectiveWindowClass)`; no min/max clamps in THIS helper. Callers may clamp elsewhere. Flag does not eliminate the difference between screen classes 0 (320pt SE) and 1 (393pt iPhone 15).
- `_TFNWindowSizeClassForSize` takes the lesser dimension and compares against original in-Mach-O constants 376.0,401.0,513.0 (VM 0x3082040, 0x3082038, 0x3082030 respectively) to return 0/1/2/3. This reproduces the exact prior decoded thresholds for ordinary finite dimensions; do not assume the bio call passes physical screen bounds rather than another windowSize.

### Remaining blockers

- Identify how category offset object is produced before its `integerValue` message and confirm exact 12-category mapping at this call site.
- Confirm actual bio font provider calls `fontSizeWithBaselineFontSize:contentSizeCategory:contentSizeModifier:windowSize:` (or distinct path); find baseline, modifier, content category, runtime flag, and passed windowSize for profile-detail bio. Static v12.25 evidence does not prove live v12.35 selection.
- Implement no guessed font value or global UI changes. SCARLET-X and SXP untouched during this follow-up.
