# 2026-10-26 — Profile Bio xds_body to fontForToken:0 verified

## Prior work
OX-H01 had fontForToken: IMP 0x369F0C and token 6/8 for follow-count/subtext. 2026-10-26 ProfileBioSetterTypography20261026.md confirmed detailed profile Bio setter calls UIFont xds_body. XdsBodyPriorWorkAndTokenDispatch20261026.md had not determined xds_body's token. This note supplies that missing link, without re-claiming earlier findings.

## Primary evidence
Official X v12.25 user-provided IPA, XServiceLibraries Mach-O (86,030,256 bytes), directly read.

- Literal `xds_body\0` at Mach-O file/VM offset 0x3774145. Chained selector reference pointer at 0x43D5830 points to that string. Relative ObjC method-name field at 0x2F8A390 resolves to 0x43D5830; method encoding field points to 0x3776F00; relative IMP entry points to **0x36AFF4**.
- `fontForToken:\0` at 0x36D96A0, selector ref 0x43BD8B8, relative method-name field 0x2F8A9E8; relative IMP resolves to **0x369F0C**. Need enclosing method/category metadata to confirm exact declaring category/class.
- `0x36AFF4 adrp x8`, `0x36AFF8 ldr x0,[x8,#0x288]` (receiver from VM 0x43E9288), `0x36AFFC mov x2,#0`, `0x36B000 b 0x2F03B00`. The ObjC stub at 0x2F03B00 reads selref **0x43BD8B8 = fontForToken:**. Therefore **xds_body sends fontForToken: with token integer 0** on the observed v12.25 code path.
- Jump table for fontForToken: at VM/file offset 0x302B018 has first byte 9. `0x369F40 adr x9,0x369F50`, `0x369F44 ldrb w10,[table,x20]`, `0x369F48 add x9,x9,x10,lsl #2`, `0x369F4C br x9`. For token 0, 0x369F50 + 9*4 = **0x369F74**. Thus token0 selects the branch starting 0x369F74. The branch contains a flag-dependent font style choice and then common font-size processing after 0x369FCC; downstream selectors and actual baseline have NOT been resolved yet.

## Remaining gaps
1. Resolve ObjC stub calls within token0 branch and common font-size handling (0x369F74 onward): determine the base token size, category modifier, screen-size input, and whether it invokes helper at 0x244F2F4. Do not assume direct call or category formula yet.
2. Decode method-list owner and category identity to fully establish `+[UIFont xds_body]` ownership, though its use from Bio setter as class-method send and token0 is now established.
3. v12.25 static only; profile v12.35 runtime and iPhone15 effective values not proven.

Reproduce: `llvm-objdump -d --start-address=0x36AFF4 --stop-address=0x36B014 XServiceLibraries.macho`; `llvm-objdump -d --start-address=0x369F0C --stop-address=0x369FF8 XServiceLibraries.macho`; inspect file byte at 0x302B018.

No Scarlet X / SXP edit, build or device validation.

## Same-session selector follow-up (original bytes)

Decoding the adrp/ldr pair in each direct Objective-C stub and the chained selector pointer gives:
- `0x2F54860` → `tfn_isChirpFontEnabled` (used in token0 branch `0x369F78`);
- `0x2F54160` → `tfn_dynamicFontSizeOffsetBy:` (common processing at `0x369FEC`; d0 supplied from token-indexed double table, for token 0 the table lookup defaults to 0 because token−3 underflows unsigned range);
- `0x2F24400` → `numberWithInteger:` (token identifier boxed at `0x36A000`);
- `0x2F24760` → `objectForKeyedSubscript:` (cache lookups at `0x36A030` etc);
- `0x2F54460` → `tfn_fontWithName:size:` (called after computed font-size in `0x36A088`).

These direct stub identities strengthen the font-size path: token0 enters `fontForToken:`, which uses `tfn_dynamicFontSizeOffsetBy:` with **0** as its offset parameter and passes the returned size to `tfn_fontWithName:size:`. It is still unproven whether `tfn_dynamicFontSizeOffsetBy:` calls the archived baseline/category/window helper `0x244F2F4` or how its input default/base size is chosen. Do not claim full formula-to-Bio linkage yet.

## 2026-10-26 — dynamic size global-state and clamp instruction follow-up

Verified with original supplied official X v12.25 `XServiceLibraries.macho` (independent read of Mach-O mapped bytes and llvm-objdump; no runtime execution). Already known selector IMPs: `tfn_dynamicFontSizeOffsetBy:` = `0x3676F0`, `tfn_dynamicFontSize` = `0x367644`. The latter reads doubles from VM `0x448CB88`=**17.0** and `0x448CB90`=**14.0**, then uses `fcmp/fcsel` to select the lesser under ordinary finite values. The former calls the getter at `0x367700`, adds offset in `d8` to returned value at `0x367704`, then compares against upper candidate `*(double*)0x448CB80 + *(double*)0x3015B08` at `0x367718..0x367720`. The original mapped data at `0x448CB80` is **3.4028234663852886e38** and `0x3015B08` is **0.001**. Remaining branch `0x367724..0x36773C` conditionally selects from candidate and `*(double*)0x46BD010`, but **the latter is in runtime writable/zero-fill area** and its effective value and setter are not yet established. It would be incorrect to infer a globally fixed 14–17pt output or complete clamp law from this data alone.

Other nearby static code `0x367668` reads `0x46BD018` and checks against 14.0 and 0.001, and `0x367694` calls another selector and uses same boundaries; static adjacency is not evidence of the setter source. The previously reported direct call to the other helper `fontSizeWithBaselineFontSize:... (0x244F2F4)` has not been proven for the Bio path. Next identify writes to `0x46BD010/018`, relevant setter methods, and distinguish v12.25 static inputs from v12.35 runtime. No Scarlet X or SXP code changes.

## 2026-10-26 — original Mach-O state writers (verified this session)

Read-only static analysis of supplied official X v12.25 `XServiceLibraries.macho`, using scoped `llvm-objdump`, not previously inferred from the ledger. An instruction scan for stores close to ADRP 0x46BD000 identified:

- Function entry `0x367360`: `0x367390 str d1,[x8,#0x10]` writes a passed double to global VM **0x46BD010**. At `0x367398 str d2,[x8,#0xB80]` and `0x3673A0 str d3,[x8,#0xB88]` it writes configuration doubles to VM 0x448CB80/88. At `0x3673A8 str xzr,[x8,#0x18]` it zeros VM **0x46BD018**. At `0x3673D0 str d8,[x8,#0xB90]` it conditionally updates VM 0x448CB90. The function may continue into notification/observer logic, and must not yet be modeled as an unconditional setter of all values.
- Function entry `0x367604`: `bl 0x2ED5BBC` then `0x36760C str d0,[x8,#0x18]` writes a computed double to **0x46BD018**. Identity of the call at 0x2ED5BBC must be resolved before assigning a numerical meaning.
- Read access `0x36766C ldr d0,[x8,#0x18]`, `0x36772C ldr d3,[x8,#0x10]` and additional nearby access establish the two mutable globals contribute to downstream typography behavior.
- `0x367644` currently selects the lesser of double globals 0x448CB88 / 0x448CB90 for finite values (their original file initial values are 17 and 14), which can be modified by the setter. The fixed original file bytes are not sufficient to deduce a live Bio font size across devices/settings.

**Next:** resolve the Objective-C method registration/selector for `0x367360` and the 0x2ED5BBC computation, find their callers and the incoming double argument values. Establish whether `fontSizeWithBaselineFontSize:... 0x244F2F4` is upstream of any setter; its connection is not proven. Also compare v12.35 live runtime independently. No SCARLET-X/SXP changes or runtime verification.

## 2026-10-26 — setter identity confirmed and exceptional-path correction

Read original v12.25 XServiceLibraries Mach-O directly. Relative Objective-C method list: relative IMP field 0x2F8A50C resolves to 0x367360; its selector field at 0x2F8A504 resolves via selref 0x43D1FF0 to the original string **tfn_setContentFontSize:minimum:maximum:maximumDynamic:** (at 0x375CEF4). This **confirms the setter IMP and selector**, rather than the earlier unnamed writer.

The setter's entry takes d0/d1/d2/d3 as incoming numeric parameters; at 0x367390 writes d1 to 0x46BD010; 0x367398 writes d2 to 0x448CB80; 0x3673A0 writes d3 to 0x448CB88; 0x3673A8 resets 0x46BD018; d0 is preserved as d8 and conditionally stored at 0x3673D0 into 0x448CB90 after comparing existing and incoming values with a small epsilon. The ordering of the four parameters is supported by the selector's names and ABI, but exact meaning under all branches/runtime flags still needs caller examination.

**Important correction to previous entry:** The bytes at 0x367604 are a stack-check failure continuation (after 0x3675E8 comparison and 0x3675EC B.NE 0x367604); the call to stub 0x2ED5BBC there is an exceptional guard/failure path. The subsequent 0x36760C STR d0 to 0x46BD018 is not evidence that normal application execution routinely computes and updates that value. Accordingly withdraw the prior claim that this is a normal 'calculated double writer' until function boundaries and intended reachability are proved. Normal return is 0x367600.

The original X v12.25 binary establishes setter identity, but who invokes it and which runtime values (14/17 among others) are passed remain unverified. In particular the connection from prior 0x244F2F4 category/window helper to this setter or profile Bio remains unproven. No SCARLET-X/SXP code changes.


## 2026-10-26 — Setter caller found in XServiceLibraries (original binary)

Prior work established setter `tfn_setContentFontSize:minimum:maximum:maximumDynamic:` IMP 0x367360 and profile Bio -> xds_body -> token0 -> dynamic offset. New scoped read-only binary scan:

- Original `XServiceLibraries` Mach-O selector ref **0x43D1FF0**, stub at **0x2F557E0**, selector `tfn_setContentFontSize:minimum:maximum:maximumDynamic:`. A whole `__TEXT` direct BL scan finds **one** direct BL to this specific stub at **0x2439330**. This excludes other ObjC message-send stubs, indirect calls and other images; no claim of unique runtime caller.
- Immediately earlier in the same code region, two selector sends at **0x24392CC** and **0x2439300** resolve to `fontSizeWithBaselineFontSize:contentSizeCategory:contentSizeModifier:windowSize:` (stub 0x2F03BC0).
- At **0x243930C** the receiver gets `tfn_minimumFontSize` (stub 0x2F54D80), and at **0x2439318** it gets `tfn_maximumFontSize` (stub 0x2F54D60).
- Just before setter call: `0x2439324 mov v0,v10`; `0x2439328 mov v1,v11`; `0x243931C mov v2,v0` (returned maximum value); `0x243932C mov v3,v9`; `0x2439330 bl 0x2F557E0`. This validates transfer of computed double arguments into the four setter slots. Note first and fourth setter sizes are sourced from different preceding formula calls (the second formula return saved in v9); nominal semantic labels are suggested by method selector but verify callers/branch conditions before claiming exact runtime numbers.
- `0x2439264` also sends `tfn_defaultFontSize` (stub 0x2F53F20), and `0x24392CC` passes a computed category/provider and window-size vector. Exact top-level method owner and dynamic settings origin are **not yet resolved**.

**New conclusion:** font-size calculator is upstream of a setter of shared dynamic typography state within original v12.25 code. It is **not yet proven that the same path runs for every Bio render or that 12.35 uses identical code**, and runtime size on iPhone15 remains unknown. No implementation change to SCARLET-X/SXP; no build/device test.

## 2026-10-26 — formula caller argument-register provenance (follow-up)

Original supplied official X v12.25 `XServiceLibraries.macho`, scoped disassembly `0x2439244..0x2439334`. This extends the existing caller finding, not a second discovery of the callsites.

- `0x2439254..0x2439260`: loads class references at **0x43EB548** and **0x43E8DF0**, invokes stub 0x2ED6594 on both, retains receivers in x19/x21. Their concrete class/object identities remain to resolve through chained binds/selector, not guessed.
- `0x2439264`: sends `tfn_defaultFontSize` to x21; result saved in v10. **The initial baseline for the first formula is this returned double** (v0 restored from v10 at 0x24392B8), not a hardcoded literal at the callsite.
- `0x2439290..0x2439294`: derives object x20 from helper 0x243BB78 (receiver in x22). This x20 is passed to the first formula as x2 (content-size category slot) at 0x24392BC. The exact category and x22 provenance need more examination.
- `0x24392B4..0x24392CC`: x0=x19, d0=v10, x2=x20, **x3=0**, v1=v8, v2=v9, then calls selector stub 0x2F03BC0 (fontSizeWithBaselineFontSize:contentSizeCategory:contentSizeModifier:windowSize:). d0 returned saved into v10.
- `0x24392DC..0x2439300`: obtains another `tfn_defaultFontSize` from x21; x2 is read via pointer VM **0x3DF7C68**, x0=x19, **x3=0**, v1=v8, v2=v9 before calling same formula again; result saved in v9. The pointer's target/type is not established; do not label it as a concrete size category.
- `0x2439308..0x2439330`: x21 receives `tfn_minimumFontSize` into v11 and `tfn_maximumFontSize` into v2. Then invokes `tfn_setContentFontSize:minimum:maximum:maximumDynamic:` with **v0=first formula result, v1=minimum, v2=maximum, v3=second formula result**. This establishes position-level provenance for all four doubles at this callsite. The numeric values in use depend on receivers, input category, window and settings.

Limit: enclosing code is in a large compiled region with Swift/ObjC bridging and indirect dispatch. **This pass has not identified the containing method owner or confirmed top-level on-screen event, caller chain or official X v12.35 equivalent.** Neither instance-category details nor runtime 14/17 values are asserted for Bio/iPhone 15. Prior Core Text findings are unchanged. No application/SXP modification.

## 2026-10-26 follow-up — caller scope and indirect callback caveat

Rechecked original supplied X v12.25 `XServiceLibraries.macho` scoped disassembly at `0x2438F80..0x2439360`. This confirms prior input register provenance, not a new numerical value. The `0x24392CC` and `0x2439300` calculation calls feed the setter at `0x2439330`, and both pass `x3=0`. The baseline at `0x24392B8` comes from `tfn_defaultFontSize` at `0x2439264`; second gets the same selector at `0x24392E0`. Window dimensions are held in `v8/v9` before the formula calls, but a preceding data-dependent path at `0x2438FC0..0x2439240` sets these values and includes indirect `blr x22` and `blr x8` calls. Consequently, the upstream window/category state and runtime dispatch are not established from this localized window. The symbol printed by llvm-objdump for these addresses is a nearby broad symbol (`_TFSLocalizable_US_STATE_WY+...`); it must **not** be taken as verified ownership of this function without a real boundary/symbol mapping. Original XServiceLibraries is not contained in the supplied sqlite indexes, so an sqlite miss is not negative method evidence. To determine owner and true inputs, next inspect Swift descriptors/function starts, the body of `0x243BB78`, classrefs at `0x43EB548/0x43E8DF0`, and the data reference at `0x3DF7C68`. No definitive method-owner or runtime sizes were identified in this pass. No app/probe changes.

## 2026-10-26 — category selection original binary details and bind caveat

Read user supplied X v12.25 `XServiceLibraries.macho` using scoped llvm-objdump disassembly of `0x243BB78..0x243BD78`. Earlier notes identified `0x243BB78` as a call returning the category argument at `0x2439290`. New instruction observations: function starts at `0x243BB78`; through `0x243BB90..0x243BBD4` it allocates a temporary value buffer and invokes operations through the type metadata/witness-style indirect function pointers `blr x16` and `blr x8`. Then it compares `w0` against a series of values loaded from globals in `0x3DFBxxx`, selecting global object references from `0x3DF7xxx`, and returns `ldr x0,[x8]` at `0x243BD64`. This proves a switch-like category object selection, but **does not yet map enum cases to the human-readable XS/S/M category names**.

Chained-pointer bytes checked at VM/file offsets `0x43EB548`, `0x43E8DF0`, `0x3DF7C68`: the first is a rebase-like pointer with low36 target `0x4445B10`; the latter two contain bind-ordinal encodings (`0x1913` and `0x228` respectively). It is **incorrect** to interpret low36 of a bind pointer as a string/address. Bind import ordinal must be resolved using `LC_DYLD_CHAINED_FIXUPS` import table before claiming the receiver class or category name.

Next: use the chained-fixups import table to resolve these ordinals and the enum case values, and inspect the type metadata of `0x243BB78`; then prove category mapping and any top-level update hook. Distinguish official X 12.25 static findings from 12.35 live runtime. No Scarlet X or SXP changes or SE runtime measurement during this pass.

## 2026-10-26 — static baseline confirmed
In the original v12.25 XServiceLibraries ObjC relative method list, selector `tfn_defaultFontSize` at selref 0x43D19C0 maps to IMP 0x367660 (relative IMP field 0x2F8A56C). At 0x367660: `fmov d0, #14.0`; 0x367664: `ret`. Thus the default font baseline getter returns 14pt in this v12.25 binary. The caller at 0x2439264 obtains its baseline through this selector on UIFont (class bind ordinal 0x1913 at slot 0x43E8DF0). The other input at bind slot 0x3DF7C68 (ordinal 0x228) is `UIContentSizeCategoryExtraExtraExtraLarge`. This explains the SE v12.35 XS/S/M measured 12/13/14 if that version's category mapping and state setter inputs are unchanged, but runtime equivalence remains unproven. No app code changes.
