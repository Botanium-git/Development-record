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
