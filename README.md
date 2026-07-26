# mesh

HTTP mesh-route runtime for compiled Kotoba guests.

This repository owns route compilation/admission at startup, HTTP serving, and
runtime dispatch. It consumes the language compiler/runtime and self-host
contract; it does not own language semantics, fleet placement, or security
policy.

## Test

```sh
clojure -M:test
clojure -M:lint
```
