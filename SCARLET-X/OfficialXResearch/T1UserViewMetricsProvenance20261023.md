# T1UserView bio layoutMetrics trace (2026-10-23)

Analyzed the user-supplied X v12.25 T1Twitter Mach-O.

- At `0x141CFC`, `_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:` preserves its `layoutMetrics:` argument x3 in x21.
- At `0x141F44`/`0x141F48`, it passes this to `T1UserViewBioAttributedTextModelWithString` as x1.
- At `0x13CB78`/`0x13CB80`, a cache miss passes the same metrics object to `T1UserViewBuildTextModel`.
- At `0x13C938` and `0x13C9A4`, the model builder requests `metricsFont` and `lineHeight` on that object.
- The nil fallback loads the class ref from `0x1AD18F8` with chained-fixup bind ordinal `0x3ADC` (15068). The import symbol resolves to `_OBJC_CLASS_$_UIFont`. It therefore invokes `[UIFont xds_body]`.

Unverified: creation of layoutMetrics, actual font/line-height values, and path from bio CGSize to bioLabelFrame. No SCARLET-X or Probe source changes, build, or device validation.

Next: trace the caller of `_t1_sizeForBioWithOptions:size:layoutMetrics:cachedTextSizing:` and the source of its layoutMetrics argument.
