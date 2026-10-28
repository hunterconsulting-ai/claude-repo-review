# License checks — read the file, not the badge

The API's `license` field is what GitHub's classifier *guessed*. The file is what the user is *bound by*. Quote the file.

## The procedure

1. **Probe every filename, not just `LICENSE`:** `LICENSE`, `LICENSE.md`, `LICENSE.txt`, `COPYING`, `LICENSE_ADDITIONAL_TERMS`.
   *Why:* `nocodb` keeps its license in `LICENSE.md`; plain `LICENSE` 404s on every branch. A `LICENSE`-only probe reported it as open source and missed its relicense from AGPL-3.0 to the Sustainable Use License.
2. **Read to the end of the file.** Clauses get appended after "END OF TERMS AND CONDITIONS".
   *Why:* `crawl4ai` reports clean Apache-2.0 through the API, but its file appends an **attribution requirement** — a visible credit line in any website or application built with it. A deliverable constraint invisible from the API.
3. **An API `NOASSERTION` means "go read the file"** — not "non-standard", not "missing".
   *Why:* `TencentDB-Agent-Memory` returned `NOASSERTION` only because a corporate preamble sat above verbatim MIT text.
4. **No license file at all?** Check the package manifest's `license` field, a copyright or license notice in the README, and source-file headers. These are weaker signals than a file — report exactly what they say, and that no file exists.
5. **Report the license by name in every review**, quoting any non-standard clause.

## The traps

- **No LICENSE file = all rights reserved.** Not a formality and not "effectively open". Under default copyright, a public GitHub repo grants the right to view and fork *on GitHub* and nothing more. Copying any of it into a deliverable infringes a named, identifiable rights-holder — and the risk lands on the user (or their client), not the author. State it prominently; for a reference or inspiration repo it is usually the decisive finding.
- **Badge ≠ license.** Repos display an MIT badge in the README while the API returns `license: null` and every license path 404s (seen on `CloudAI-X/threejs-skills` and `MeiGen-AI/LongCat-Video-Avatar`).
- **Split licenses.** Look for per-directory LICENSE files carving out proprietary subtrees, and trace which directory the feature you actually want lives in. (Stirling-PDF: core tools MIT in `app/core/`; `app/proprietary/` is a paid EULA.) **Compare each per-directory license's text and date against the root** — after a relicense they drift: nocodb's root license covers `master` and `develop`, while three package-level copies, a version older, still license `master` only.
- **Branch-scoped licenses.** Some licenses cover named branches only. Check that the license covers the branch or tag users actually install from — not just the one you reviewed.
- **License vs business model.** "Internal business use only" or "non-commercial" terms can block a consultant hosting the software for clients even when internal use is fine. Say so in Relevance and fit (Step 5), and leave the legal call to a lawyer.
- **Source-available ≠ open source.** tldraw is `NOASSERTION` and forbids production use without a paid key, enforced in code.
