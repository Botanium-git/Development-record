# 2026-10-26 — xds_body historical work check and fontForToken original instructions

## Duplicate review
Earlier records: OfficialXFindingsLedger OX-H01 documents prior `fontForToken:` IMP 0x369F0C and token 6=`xds_subtext2` and token 8=`xds_subtext2Bold`, but these belong to follow-count context, not Bio. OX-H03 documents the baseline/category/modifier/window helper at 0x244F2F4. NEXT.md and T1UserViewFontFallback20261023.md show prior `T1UserViewBuildTextModel` uses layoutMetrics.metricsFont and falls back to `UIFont xds_body` only when nil. ProfileBioSetterTypography20261026.md newly establishes a **different profile-detail** Bio setter path fetching `UIFont xds_body` directly. Never treat the earlier fallback as direct evidence for profile-detail Bio.

## This session's original binary checks
Artifacts: original user-provided official X v12.25 IPA extracted `XServiceLibraries.macho` and `T1Twitter.macho` already mounted. Original bytes contain NUL-terminated selector strings in XServiceLibraries at file offsets `xds_body=58147141`, `fontForToken:=57513632`, `fontSizeWithBaselineFontSize:contentSizeCategory:contentSizeModifier:windowSize:=57513798` (one exact match each); T1Twitter has `xds_body` at 23803098 (one exact match). This is presence, not graph ownership.

Scoped `llvm-objdump -d --start-address=0x369f0c --stop-address=0x369f94 XServiceLibraries.macho` verified executable code at archived `fontForToken:` reported IMP 0x369F0C: 0x369F28 moves x2 to x20; 0x369F30 compares x2 to 0x1B (27); 0x369F38..4C loads a per-token byte offset table at VM 0x302B018 and branches to computed instruction target (`br x9`). This proves selector-reported IMP is a token-indexed switch *if method owner and linkage are confirmed*, but NOT which token `xds_body` requests nor what font-size provider its branch uses. Numeric token-to-style map in prior OX-H01 is not rederived here.

## Next proof target
Locate Objective-C category method registration for `+[UIFont xds_body]` in XServiceLibraries and its actual IMP; disassemble it and record whether it calls `fontForToken:`, its token integer, and downstream baseline/category/window function path. If absent from this binary, identify the framework defining the category; do not infer a call solely from matching strings.

No application or SXP edits; no build or device measurement.
