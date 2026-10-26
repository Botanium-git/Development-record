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
