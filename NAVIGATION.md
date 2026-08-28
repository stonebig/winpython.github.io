# WinPython site — what this draft changes, and where it is heading

Working notes for the `draft-2026-03-redesign` branch. Two things are mixed here on purpose:
the 2026-03 announcement (which has to ship soon) and the slower move towards publishing
package sets rather than installers (which the announcement can start).

---

## 1. What is wrong with the current page

Not opinions about taste — these are things a first-time visitor actually hits.

**The page is 85 % archive.** `index.html` is 704 lines; the current release occupies 25 of
them and everything from line 55 to line 633 is history back to May 2020. The most important
information on the site is a rounding error on the page that carries it.

**Nothing on the page is a download.** Every "download" link goes to a SourceForge *folder
listing* or to a GitHub release page holding 14 assets. The visitor still has to work out
which of `WinPython64-3.14.7.0slimf.7z` and `WinPython64-3.14.7.0dot.exe` they wanted, from a
directory index, on a different site.

**The build suffixes are never explained in one place.** `dot`, `slim`, `whl`, `free`,
`slimf`, `cod` are glossed inline, once per release line, in wording that drifts between
releases — "with pre-installed wheels", "with ready to installed wheels", "= Python 3.14.5"
— so the same word means something slightly different three screens apart. A reader has to
reverse-engineer the naming scheme from examples.

**pylock and requirements files do not exist on the site at all.** They are the most
distinctive thing WinPython now publishes and they are invisible here.

**Accumulated markup rot.** An XHTML 1.1 doctype that has never matched the content; no
viewport meta tag, so phones render a 75 em page; `</li>` after `</p>`; `<p>` inside `<ul>`;
an unclosed `<p>` near the end that swallows the footnotes; a duplicated 3.15 block left over
from the 2025-05 edit sitting outside any list; a footer reading "Last updated 2024-04-13" on
a page edited this month; a Microsoft VC++ redistributable link that has been a 404 for years.

**The nav has three entries** — releases, overview, portable — and two of them point into
prose rather than at a decision the visitor is trying to make.

---

## 2. What the draft does

| | before | after |
|---|---|---|
| pages | 1 | `index.html` (current release + how to choose + how to reproduce + what it is) and `releases.html` (the archive) |
| current release | 25 lines among 610 lines of history | the whole top of the page |
| download links | folder listings | direct `.exe` / `.7z` / `.zip` URLs, with sizes |
| suffix meaning | glossed 31 times, inconsistently | one table, once |
| lock files | absent | a top-level section and a link on every build |
| history | flat wall | 31 collapsible entries, nothing lost |
| mobile | unusable | responsive |
| dark mode | no | follows the OS |

Still a static site: no generator, no JavaScript, no external requests, no fonts or CDNs to
fetch. Same deployment as today — commit and GitHub Pages serves it.

`releases.html` was generated from the old page by a script rather than retyped, so all 183
historical download lines and their links survive verbatim, back to 2020. Long-term
availability of old releases is a genuine WinPython advantage, so the archive stays complete;
it is collapsed, not truncated. The stray duplicated 3.15 block was dropped and the broken
tag nesting repaired on the way through.

**3.14 is marked Recommended**, not 3.13 — it is the minor that will keep getting updates
over the coming year. 3.13 stays visible as *Previous*, for the case where a dependency has
no 3.14 wheel yet. Each cycle this is a one-word edit: move the `featured` class and the
`Recommended` tag to whichever card leads.

---

## 3. Keeping the per-release edit small

An early version of this draft had a strip of package-version pills (numpy 2.5.2, polars
1.43.2, …) at the top of the download section. That was removed on purpose: it is a dozen
values to re-check by hand every cycle, for information that is already exact and already
generated in the `packages` changelog. Hand-maintained content is how the current page
acquired its broken tags and its 2024 footer date.

What is left to edit per release, and nothing more:

1. the release name and date (3 places: hero badge, download heading, footer);
2. one highlights line — the same sentence already written in the release notes;
3. the download card bodies: version numbers, sizes, and the tag in the URLs;
4. one new `<details>` block at the top of `releases.html`.

Even that is mechanical enough to generate. A `releases.toml` holding version, date, tag and
the per-build sizes, plus the small script that already built `releases.html` from the old
page, would reduce a release to a data edit and remove hand-written HTML from the loop
entirely. The draft does not depend on it — it is the obvious next cleanup, and the archive
generator is the proof it works.

---

## 4. The move from binaries to package sets

The honest framing of where WinPython is going:

> Today the site sells an installer and mentions what is inside it.
> The end state sells a **pinned, verifiable package set**, and the installer is one
> convenient way to obtain it.

2026-03 is the first release where that is literally true: `3.15.0.4 slim` and `slimf` exist
only as lock files, because at 3.15.0rc1 only ~60 % of the slim set has wheels and shipping a
half-empty 400 MB installer would mislead. That is not a gap in the release — it is the new
model arriving early, and the draft presents it that way rather than apologising for it.

### Stage 1 — this release (done in the draft)

- Lock and requirements files are linked from every single build, not buried.
- A dedicated section, *"The package set, without the 670 MB"*, with the three pip
  invocations copied from the release notes: hash-pinned requirements, PEP 751 pylock, and
  the offline-wheelhouse two-step.
- 3.15 slim/slimf presented as a usable deliverable.

### Stage 2 — next release, small infrastructure changes

**Attach the lock files to the GitHub release again.** In 2026-03 they live only in the repo
tree, so the only links available are `blob/master/changelogs/pylock.64-3_14_7_0slim.toml`
— which will point at 2026-04's content the moment 2026-04 lands. Release assets give a URL
that pins a release forever, which is the whole point of a lock file. (The tag-pinned raw URL
works, and the draft uses it in the copy-paste commands, but see the tag problem below.)

**Fix the tag shape.** `17.12.20260522/WinPython` contains a slash, so every URL to it needs
`%2F`, and the date reads 05-22 for an August 22nd release. A tag like `2026-03` would make
`github.com/winpython/winpython/releases/tag/2026-03` a link anyone can type, and would let
the site and the docs stop hardcoding build numbers.

**Add channel aliases.** `pylock.64-3_14_7_0slim.toml` embeds the exact build, so there is no
way to say "the current 3.14 slim set" in a CI file. A per-cycle copy named
`pylock.3.14-slim.toml` would let downstreams pin a *channel* and let the website link one
stable URL per row instead of rewriting seven of them every cycle — which also directly
shrinks the per-release edit described above.

### Stage 3 — shrink the binary matrix on purpose

2026-03 is already most of the way there. The rule this suggests writing down:

- `dot` for **every** supported Python — it is 18 MB, it costs nothing to publish, and it is
  the natural companion to a lock file.
- `slim` only where the package set is genuinely complete, and for the minor being
  recommended.
- pylock + requirements for **everything**, always, including the combinations with no binary.

Publishing fewer, better-justified binaries is easier to explain when the page already says
what a lock file is for.

### Stage 4 — a "latest" that does not need editing

A `latest` tag or a tiny `latest.json` in the site repo would let the download section, the
README and any CI pipeline resolve the current release without a human editing HTML. Worth
doing once the tag naming is fixed.

---

## 5. Navigation beyond this site

**The flavors are explained in four places and agree in none of them:** this site, the wiki,
`README.rst` and `README_PYPI.md`. Pick the *"which build should I take?"* table as the
canonical version and have the others link to it rather than paraphrase.

**The real changelog is a comment on a follow-up issue.** `issues/2026#issuecomment-…` is
excellent and thorough, but it is discoverable only by following a link from this site. The
GitHub Release body already sits next to the assets and is where people look first —
consider making it the canonical note and linking the issue for discussion.

---

## 6. Reviewing this draft

```
git checkout draft-2026-03-redesign
start index.html
```

Both pages carry a red DRAFT banner; remove the `div.draft-banner` from each before merging.
