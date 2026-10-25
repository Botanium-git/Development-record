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
