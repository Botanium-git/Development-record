# Font fallback trace

Verified using supplied X v12.25 binary: `T1UserViewBuildTextModel` loads `metricsFont` from its layout-metrics argument. If nil, it calls `xds_body` as fallback. It separately obtains `lineHeight` from the same metrics argument. Concrete UIFont properties and receiver class for `xds_body` remain unresolved. No app code changed.
