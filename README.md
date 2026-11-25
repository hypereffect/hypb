# hypb

`hypb` talks to [hyperb](https://github.com/hypereffect/hyperb) apps from the
command line: it prints their messages, inspects their signals and tree, fires
actions, and runs JavaScript scripts against them.

```text
hypb <address>                    the messages the app sends
hypb state <address>              its signals and tree, again after every change
hypb ui <address>                 the same, interactive, with the history
hypb action <address> <name>      fires an action
hypb run <address> [script.js]    runs a script against it
```

Addresses are the ones every hyperb renderer opens: HTTP, iroh and wasm apps
alike, with the session parameters in the fragment.

## Building

`hypb` depends on `hyperb-core`, which lives in the
[hyperb](https://github.com/hypereffect/hyperb) repository. A checkout of
`hyperb` is expected next to this repository:

```text
code/
├── hyperb/   # hyperb and its crates, including hyperb-core
└── hypb/     # this repository
```

Then:

```sh
cargo build --release
cargo run -- --help
```

The binary is `target/release/hypb`.

## Examples

```sh
hypb hyperb://example.com/app
hypb state hyperb://example.com/app --idle 2000
hypb ui hyperb://example.com/app
hypb action hyperb://example.com/app refresh
hypb run hyperb://example.com/app script.js
hypb run ~/apps/counter.wasm -e 'press("inc"); sleep(500); print(signal("count"))'
echo 'printTree()' | hypb run hyperb://example.com/app
```
