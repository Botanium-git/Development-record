# 2026-10-26 — T1ProfileUserInfoView.updateFontSize original IMP and dispatch

## Prior work / duplicate check
Earlier Development-record reports v12.35 SE runtime method `-[T1ProfileUserInfoView updateFontSize]` and v12.25 index missing it, and 2026-10-26 ProfileUpdateFontSizeBinaryCheck20261026.md confirmed raw selector presence. This note **adds method ownership, concrete IMP and dispatched selectors**; these were not previously established there.

## Source and direct reproduction
Original user-provided official X v12.25 `T1Twitter.macho` (extracted from the IPA). Objective-C class `T1ProfileUserInfoView` class address 0x1AECF58 → class_ro 0x1989140 → relative instance method list 0x13C6F88 (145 entries). Method record `updateFontSize`, encoding `v16@0:8`, IMP **0x25521C**. Verified relative-name selref pointer, types and signed-relative IMP for that class; method absence from earlier sqlite index was incomplete coverage, not absent implementation. The same method list contains `_t1_refreshBioLabel` IMP 0x257C70, `_t1_setBioLabel:withText:entities:isUTF16Entities:` IMP 0x257E98 and `_t1_buildBioLabelWithAccessibilityIdentifier:` IMP 0x256FB4.

Disassembly `llvm-objdump -d --start-address=0x25521c --stop-address=0x255298 T1Twitter.macho`:
- 0x25522C BL 0x12C52C0 → selector `_t1_refreshBioLabel` (selref 0x1A99678 → 0x15C37FB).
- 0x25523C BL 0x136DCC0 → `useGrokInlineTranslation` (selref 0x1AC38F8). Conditional branch tests bit0 before next call, so translation eligibility affects subsequent refresh.
- 0x255248 BL 0x12C5440 → `_t1_refreshTranslatedBioLabel` when condition permits.
- 0x255250 / 0x255258 / 0x255260 / 0x255268 / 0x255270 / 0x25527C / 0x255284 dispatch `_t1_refreshBulletpointButton:withTitle:image:linkable:invisibleLink:accessibilityFormat:`, `_t1_refreshCreatedDateButton`, `_t1_refreshIfNecessary`, `_t1_refreshLocationButton`, `_t1_refreshProfessionalCategoryButton`, `_t1_refreshTranslateBioButtonAnimated:`, `_t1_refreshExpandedBioButton` (full selector list cross-check with 0x1A99678..0x1A996E8).
- 0x255294 tail B 0x1345580 → `setNeedsLayout` (selref 0x1AB9728).

## Interpretation and limits
This IMP is a **refresh orchestrator**, not a direct call to the archived `fontSizeWithBaselineFontSize:contentSizeCategory:contentSizeModifier:windowSize:` helper in its observed instructions. Its bio refresh first forwards to `_t1_refreshBioLabel`, and indirect model/font setup may occur downstream. This does **not** disprove that the helper is ultimately used. Next trace `0x257C70` and `0x257E98`, selectors and receiver classes, then CT font provider. Official v12.35 runtime method existence is separately established but its implementation may differ. No application changes, no device experiment.
