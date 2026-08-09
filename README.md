# arwPyFormat

A plugin for [agentRW](https://github.com/linuxrebel/agentRW). Fixes PEP 8
style in a Python file with autopep8 — deterministically, in one subprocess,
with no model and no tokens.

```
/format_file path/to/file.py
```

---

## Why it exists

A model asked to fix indentation returned a line that kept the wrong indent and
silently dropped a `*` from `join(*lines)`. A style request caused a runtime
error.

Style is a solved problem. autopep8 solves it:

| | model | autopep8 |
|---|---|---|
| findings fixed per call | 1 | all of them |
| tokens | hundreds per finding | **zero** |
| can it alter logic | yes, and did | no |
| same input, same output | no | yes |

On a measured 17-finding file, autopep8 handled 11 with zero model
involvement. Every finding moved off the model is a win, not a shortfall — a
probabilistic system applied to a solved problem is the mistake, not the
shortcut.

**Deliberately not black.** black reformats the whole file to its own opinion,
producing a 40-line diff for an 11-line problem. That unrequested scope is the
exact failure this project keeps hitting. autopep8 fixes actual PEP 8
violations and leaves everything else alone.

---

## What this will do

**It will:**
- Rewrite **in place** the one `.py` file you name, running `autopep8
  --in-place` on it
- Change only whitespace: indentation, blank lines, spacing around operators
  and after commas, line continuation
- Run `autopep8` as a subprocess
- Return a count of changed lines to whoever called it

**It will not:**
- Touch any file except the one you named
- Change logic, rename anything, reorder imports, or reformat code that is
  already valid PEP 8
- Write a backup — **there is none.** autopep8 edits in place and this plugin
  does not snapshot first. Use version control, or call it from `/lint`, which
  takes its own snapshot and reverts the run if the result stops compiling
- Install packages, change configuration, or reach the network
- Send anything to a model. Nothing here leaves the machine

The one thing worth knowing: **no backup.** The changes are style-only and
autopep8 is deterministic, but the original is not recoverable from this plugin.

---

## Requirements

`autopep8` only.

| | |
|---|---|
| Fedora | `sudo dnf install python3-autopep8` |
| Debian/Ubuntu | `sudo apt install python3-autopep8` |
| pip | `python3 -m pip install --user autopep8` |

Distro packages are the better option — pip on top of a distro Python is a
known way to break it. The plugin invokes `autopep8` as an executable on
`PATH`, so either source works.

Without it, `/format_file` returns `autopep8_not_installed` and names what to
install. Nothing breaks and nothing is installed for you.

---

## Install

agentRW has no plugin installer yet, so a plugin is installed by copying two
files into place. Install agentRW first — see
[its README](https://github.com/linuxrebel/agentRW) — then:

```bash
git clone https://github.com/linuxrebel/arwPyFormat
```

Or, without git: download the **Source code (zip)** from the
[Releases](https://github.com/linuxrebel/arwPyFormat/releases) page and unzip
it. It contains everything the plugin needs. The folder it unpacks to is named
for the tag — `arwPyFormat-0.1.0` rather than `arwPyFormat` — so adjust the
paths below to match.

> **Do not unzip it straight into `tools/`.** A plugin has to sit exactly two
> levels down, at `tools/<owner>/<name>/`. One level too shallow and it is
> skipped in silence — `/plugins` reports nothing registered and says nothing
> about why. Copy the two files as shown below instead.

### Linux and macOS

agentRW installs to `/opt/agentRW`, which is owned by root, so copying a plugin
in needs `sudo`:

```bash
sudo mkdir -p /opt/agentRW/tools/linuxrebel/format
sudo cp arwPyFormat/plugin.py arwPyFormat/install.md /opt/agentRW/tools/linuxrebel/format/
```

### Windows

agentRW installs per-user, so no admin is needed. In PowerShell:

```powershell
$dest = "$env:LOCALAPPDATA\Programs\agentRW\tools\linuxrebel\format"
New-Item -ItemType Directory -Force -Path $dest
Copy-Item arwPyFormat\plugin.py, arwPyFormat\install.md -Destination $dest
```

### Check it took

Start `cagent` and run `/plugins`. You should see:

```
  linuxrebel/format ACTIVE   tools: format_file
      needs autopep8: found
```

---

## Uninstall

Remove the directory. The tool disappears with it.

```bash
sudo rm -rf /opt/agentRW/tools/linuxrebel/format
```

---

## Usage

```
/format_file <file> [aggressive]
```

`aggressive` passes `--aggressive` to autopep8, which will also fix things PEP 8
does not strictly require. Off by default, because the default is the one that
provably cannot change behaviour.

The tool is marked `model_facing = False`, so it is **not advertised in the
system prompt**. That saves ~57 tokens on every single turn — the prompt is
re-sent with every request, so an unadvertised tool is not a one-time saving.
It stays callable as `/format_file`, and by other plugins through `ctx.tools`,
which is how [arwLint](https://github.com/linuxrebel/arwLint) uses it to clear
every style finding in one pass without spending a token on any of them.

---

## License

MIT
