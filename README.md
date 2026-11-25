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
alike, with the session parameters in the fragment. Diagnostics go to stderr,
prefixed `hypb:`.

## Install

Prebuilt CLI binaries are attached to every
[release](https://github.com/hypereffect/hypb/releases): Linux, macOS and
Windows, for x86_64 and ARM64. On Linux or macOS, expand the snippet below and
paste it into bash: it detects your platform, downloads the latest release and
installs `hypb` into `~/.local/bin` (or `$HYPB_INSTALL_DIR`), adding it to your
`PATH` if needed.

<details>
<summary>Install script (bash)</summary>

```bash
(
  set -eu

  repo=hypereffect/hypb

  # Releases name their assets after the Rust target triple.
  case "$(uname -s)" in
    Linux)  platform=unknown-linux-gnu ;;
    Darwin) platform=apple-darwin ;;
    *) echo "hypb: unsupported OS: $(uname -s); download a binary from https://github.com/$repo/releases" >&2; exit 1 ;;
  esac
  case "$(uname -m)" in
    x86_64|amd64)  arch=x86_64 ;;
    arm64|aarch64) arch=aarch64 ;;
    *) echo "hypb: unsupported architecture: $(uname -m); download a binary from https://github.com/$repo/releases" >&2; exit 1 ;;
  esac
  target=$arch-$platform

  command -v curl >/dev/null || { echo "hypb: curl is required" >&2; exit 1; }
  tag=$(curl -fsSIL -o /dev/null -w '%{url_effective}' "https://github.com/$repo/releases/latest")
  tag=${tag##*/}
  case "$tag" in
    v*) ;;
    *) echo "hypb: no release found for $repo" >&2; exit 1 ;;
  esac
  asset=hypb-$tag-$target.tar.gz

  bindir=${HYPB_INSTALL_DIR:-$HOME/.local/bin}
  mkdir -p "$bindir"

  tmp=$(mktemp -d)
  trap 'rm -rf "$tmp"' EXIT
  curl -fsSL -o "$tmp/$asset" "https://github.com/$repo/releases/download/$tag/$asset"
  tar -xzf "$tmp/$asset" -C "$tmp"
  cp "$tmp/hypb-$tag-$target/hypb" "$bindir/hypb"
  chmod 755 "$bindir/hypb"
  "$bindir/hypb" --version

  # Add $bindir to the PATH of future shells, unless it is already on it.
  case ":$PATH:" in
    *":$bindir:"*) ;;
    *)
      case "${SHELL##*/}" in
        zsh)  rc=$HOME/.zshrc ;;
        bash) rc=$HOME/.bashrc; [ -f "$HOME/.bash_profile" ] && rc=$HOME/.bash_profile ;;
        *)    rc=$HOME/.profile ;;
      esac
      if ! grep -qsF "$bindir" "$rc"; then
        printf '\nexport PATH="%s:$PATH"\n' "$bindir" >>"$rc"
        echo "hypb: added $bindir to PATH in $rc; open a new shell to pick it up" >&2
      fi
      ;;
  esac
)
```

</details>

On Windows, download the `hypb-<version>-x86_64-pc-windows-msvc.zip` (or the
`aarch64-pc-windows-msvc` one) from the releases page and unzip `hypb.exe`
somewhere on your `%PATH%`. Every archive ships with the README and the release
page has a `SHA256SUMS` file.

Or build it from source, as described under [Building](#building).

## Commands

| Command | What it does |
|---|---|
| `hypb <address>` | The default: connects as a renderer, sends the `hello` and prints every message the app sends, parsed and encoded again, one per line. Filter with `--id`, `--type` or `--actions`, and stop with `--idle MS`. |
| `hypb state <address>` | Prints the signals and a short tree of the interface, again after every change: each element's type and id, what it shows, the signal it is bound to and what its handlers do. `--debounce MS` waits for changes to pause before printing, `--idle MS` closes the connection when the app goes quiet, `--max-text N` cuts long texts and `--no-color` disables colors. |
| `hypb ui <address>` | The page of `hypb state`, interactive in the terminal: type into inputs, press buttons, toggle checkboxes, change selects, sliders and tabs, with the page changing in place as the app sends. A sidebar lists every message both ways, each kept with the signals of the moment; picking one rebuilds the page as it was right after it and stays there read-only until `live` is picked again. |
| `hypb action <address> <name>` | Fires an action and prints it (after `> `) with what the app sends back (after `< `). `--payload JSON` sends a payload, `--signals JSON` merges signals into the current ones, `--settle MS` waits for the app to go quiet first and `--idle MS` closes the connection afterwards. |
| `hypb run <address> [script.js]` | Runs a JavaScript script against the app, from a file, standard input or `-e` for inline code. Besides the tree helpers (`signals`, `signal`, `query`, `press`, `input`, `trigger`, `wait`, `printTree`, …) it can send actions, write signals and settle on what the app sends. |

Every command that connects takes the app's address; `--verbose` prints the
client's diagnostics such as rejected elements and failing expressions, and
`--key HEX` connects with a given ed25519 secret key instead of the renderers'
identity.

`hypb help <command>` has the details and examples for each.

## Examples

```sh
hypb hyperb://example.com/app
hypb hyperb://example.com/app --type button --actions
hypb state hyperb://example.com/app --idle 2000
hypb ui hyperb://example.com/app
hypb action hyperb://example.com/app refresh
hypb action hyperb://example.com/app save --signals '{"form":{"email":"a@b.c"}}'
hypb run hyperb://example.com/app script.js
hypb run ~/apps/counter.wasm -e 'press("inc"); sleep(500); print(signal("count"))'
echo 'printTree()' | hypb run hyperb://example.com/app
```
