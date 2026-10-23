# TFN Flexible Layout: bio callback ABI and CGRect boundary (2026-10-23)

## Scope and original evidence
Official X v12.25 supplied IPA, extracted `T1Twitter.framework/T1Twitter`. This analysis cross-checks previous `T1UserViewBioCaller20261023.md` with actual ARM64 instructions (`llvm-objdump -d`) and the supplied SQLite `strings` records. It does not examine runtime SE v12.35.

## Native instructions independently checked
- In layout assembly routine `0x483C58`, `0x483C7C..0x483C88` supplies constant block `0x1882490` to objc stub `0x1365180`, previously resolved to `tfn_withFlexibleLayoutSizeCalculator:`.
- `0x483CAC..0x483CD8` writes invoke pointer `0x483DC0` into a stack block and passes it to stub `0x1365140`, previously resolved to `tfn_withFlexibleLayoutFrameListener:`. The captured object at stack `+0x28` comes from the routine's input x3.
- Size-calculator invoke `0x483D40` preserves incoming x2 as x19 and v0/v1 as v9/v8; asks the item for `tfn_flexibleLayoutViewModel` at `0x483D6C`; then passes component=4 (`mov w2,#4`) and x3=x19 to `sizeForComponent:withSize:layoutMetrics:` at `0x483D8C`. It returns its v0/v1 pair as the size result. No CGRect origin is created here.
- Frame-listener invoke `0x483DC0` loads x0 from the block's captured field at `+0x20`, then **tail branches** `b 0x1337B60` (selector stub `setBioLabelFrame:`). There is no local frame formula or conversion in this 2-instruction callback; it forwards the incoming SIMD arguments to the setter.
- Therefore the element/engine producing the frame argument is **outside** these two callbacks, not a computation inside `setBioLabelFrame:`.

## Type information still present in original T1Twitter
- SQLite `strings` includes block prototype `{CGSize=dd}40@?0@"<TFNFlexibleLayoutItem>"8{CGSize=dd}16@"TFNLayoutMetrics"32`. This is consistent with calculator output CGSize and input TFNLayoutMetrics.
- Also present: `v56@?0@"<TFNFlexibleLayoutItem>"8@"TFNLayoutMetrics"16{CGRect={CGPoint=dd}{CGSize=dd}}24`. This declares a void-style listener receiving a CGRect along with item and metrics. **Not independently proven as the exact block signature associated with invoke `0x483DC0`**; matching to that particular block requires checking its descriptor signature pointer.

## New conclusion and remaining work
- The **bio sizing callback** and **bio frame listener** are registered together, and the listener is only forwarding the rectangle to `setBioLabelFrame:`. There is no offset/position arithmetic in the listener.
- Thus the **CGSize→CGRect origin & spacing** calculation must be followed in the TFN flexible-layout implementation/caller. Next identify the actual implementation module and layout algorithm for `TFNFlexibleLayoutItem`/container; trace the registration receiver at `0x483C58`, the parent layout constraints, spacing, and the source of the `TFNLayoutMetrics` instance.
- Existing status-body OX-020/021 height constants are not automatically applicable to profile bio.
- No code edits to Scarlet X/Probe, no build or device validation.
