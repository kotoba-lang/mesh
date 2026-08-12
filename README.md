# mesh

HTTP mesh-route runtime for compiled Kotoba guests.

This repository owns route compilation/admission at startup, HTTP serving, and
runtime dispatch. It consumes the language compiler/runtime and self-host
contract; it does not own language semantics, fleet placement, or security
policy.

## Is the Kotoba running? (ADR-2608120200)

**Yes, since 2026-08-12: the HTTP contract is decided by a shipped Kotoba
core.** `src/kotoba/mesh/route.kotoba` is compiled to
`resources/mesh/oracle/route.kir.edn`, and `kotoba.mesh.node` executes it
through `kotoba.mesh.kotoba-oracle` on every request — which url shapes this
node recognises, which route a path names, whether a guest that ran had
anything to answer with, what outcome that is, and what status it gets. There
is no `.cljc` twin of those rules to drift from, because there is no second
copy.

What stayed in the host, and why:

| stays in `node.clj` | because |
|---|---|
| the route table | a collection that grows with the deployment. ADR-2608112100 measured the boundary — the core is handed a *boolean* saying the route was bound, so it sees the same four scalars whether the node serves one route or ten thousand |
| Chicory dispatch, reading guest memory, writing the socket | effects, not decisions |
| `compile-route`'s check-before-emit | the rule IS an order between two effects a guest cannot call. Delegating it would move the shadow and leave the rule |
| the `/health` body | it names which runtime answered — a fact about this host, not something the contract derives |

Two gates exist that a behavioural test cannot replace
(`test/kotoba/mesh/kotoba_oracle_test.clj`):

- **drift** — the shipped artifact is the current source, compiled. The
  existing suite never reads the artifact, so without this a stale one is
  invisible.
- **delegation** — a substituted core with the same exports and deliberately
  different answers is registered, and the live server is required to *follow*
  it over real HTTP. A host that had kept its own `cond` passes every other
  test in this repository; this is the only one that separates them. Verified
  by mutation: reverting one delegated call to a host copy fails this and
  **nothing else**.

### `examples/*.kotoba` are not decision cores

The three `.kotoba` files under `examples/` are guest **payloads**, not cores,
and this seam does not compile or ship them. Compiling operator-supplied
`.kotoba` routes at startup is what this repository is *for* — precompiling one
into a shipped artifact would defeat it. `mesh_drama_profile.kotoba` is a ported
mesh app; `mesh_no_answer.kotoba` and `mesh_bad_route.kotoba` are fixtures for
the 204 and the startup-rejection branches. They stay where they are, compiled
fresh by `compile-route`.

## Test

```sh
clojure -M:test
clojure -M:lint
```

Regenerate the shipped decision core after editing `route.kotoba` (the compiler
lives in `:test`, so it never reaches a consumer):

```sh
clojure -M:test:gen
```
