# 2026-10-26 — profile updateFontSize: original Mach-O selector-presence check

## Context and prior work

Reviewed Development-record NEXT.md and OfficialXFindingsLedger.md. Previous 2026-10-25 NEXT.md notes say the v12.25 indexed objc_methods lacks exact `updateFontSize` and `_t1_setBioLabel:withText:entities:isUTF16Entities:`, while **live official X v12.35 on first-gen SE** reports `-[T1ProfileUserInfoView updateFontSize]` encoding `v16@0:8`, implementation in T1Twitter, and active bio setter. The earlier index-negative result is not evidence of absence from a binary.

## New direct v12.25 checks

Source is the provided original `com.atebits.Tweetie2_12.25_und3fined(1).ipa`, extracted `Payload/Twitter.app/Frameworks/T1Twitter.framework/T1Twitter` (33,287,408 bytes). Searching raw original binary bytes, without relying on objc_methods indexing:

| NUL-delimited selector/string | File offset | Occurrences of queried byte sequence |
|---|---:|---:|
| `updateFontSize\0` | 23,758,704 | 1 |
| `_t1_setBioLabel:withText:entities:isUTF16Entities:` | 22,831,525 | 1 |
| `T1ProfileUserInfoView\0` | 21,122,747 (first) | 3 |
| `_t1_sizeForBioWithOptions:` (prefix) | 22,843,025 | 1 |
| `metricsFont\0` | 23,331,969 | 1 |

Separately sqlite table `strings` contains `updateFontSize` at address/file offset 23,758,704 and the setter at 22,831,525, but the prior `objc_methods` index lacks them. **Correction/clarification:** These selectors are present as strings in v12.25 T1Twitter; no class-owner/IMP or active call path has been established from strings alone. This may be index incompleteness or a genuinely different method registration situation. Do not claim the method absent.

## Next smallest evidence step

Find method-list entry/owner for the v12.25 `updateFontSize` selref or relative method metadata, confirm whether it is owned by `T1ProfileUserInfoView`, inspect real IMP and its delegates, then identify whether the font provider flows through `fontSizeWithBaselineFontSize:contentSizeCategory:contentSizeModifier:windowSize:` or another pipeline. Version v12.25 static and v12.35 live must remain distinct. The prior T1UserView metricsFont fallback remains known and is not itself proof that this detailed profile view uses the same provider.

No Scarlet X/SXP code or version change; no Actions or device verification.
