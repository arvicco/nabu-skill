---
name: nabu
description: "Use when researching, reading, quoting or citing ancient and historical texts through a local Nabu library and its read-only MCP server (the nabu_* tools): Greek and Latin, cuneiform, papyri, the Hebrew and Christian Bibles in many witnesses, Classical Arabic, Literary Chinese and the Buddhist canon, Sanskrit, Tamil, Tibetan, Old English, Old Church Slavonic, Gothic and more, plus dictionaries, lemmatized treebanks and reconstruction data. Covers installing Nabu and registering its MCP server, driving each of the 14 tools, license-class etiquette for quoting and exporting, and the library's honest limits."
license: MIT
compatibility: "Needs a local Nabu checkout (https://github.com/arvicco/nabu, Ruby 3.3+) with a built library, and its MCP server (`bin/nabu mcp`, stdio, read-only) registered with the client. Without the server the skill can only explain how to install it."
metadata:
  upstream: "https://github.com/arvicco/nabu"
  census_as_of: "28 October 2026"
---

# Nabu — reading a local library of ancient texts

Nabu is an open-source pipeline that ingests openly licensed digital corpora
of antiquity into one searchable, citable SQLite library on the user's own
disk, and serves it to an agent through a read-only MCP server. This skill
teaches you to drive that server well. Every tool is read-only: you cannot
sync, edit or delete anything, so explore freely.

## What the library holds

The reference library (the project's own build, figures as of 28 October 2026,
published at https://arvicco.github.io/nabu/) holds:

- **2,480,397 documents / 119,725,893 passages** in **255 language codes**,
  from **249 live sources**.
- **3,283,555 dictionary entries** on **587 dictionary shelves**.
- **25.5 million gold lemma annotations in 53 languages** (verified
  treebank and hand-lemmatized corpora) and a separately labeled
  **32.5 million silver** (upstream-automatic) in 18 languages.
- About **93%** of documents are in redistributable classes (public domain
  or attribution).
- Largest languages by passages: Classical Arabic (`ara`, 33.3M), English
  (`en` 24.3M — chiefly early-modern print), Literary Chinese (`lzh`, 20.6M),
  Japanese (5.6M), Latin (3.6M), Ancient Greek (3.2M), Sumerian (3.0M).

**The user's library is almost certainly smaller.** Every install is built
source by source; a fresh quickstart holds four sources. Call `nabu_status`
first in every session and treat its answer, not these figures, as what is
on this box.

## Install and connect (when the tools are missing)

If no `nabu_*` tools are available, tell the user how to get them; do not
pretend to search.

```
git clone https://github.com/arvicco/nabu && cd nabu
bundle install            # Ruby 3.3+
bin/nabu quickstart       # starter shelf: 4 sources, ~690 MB, a few minutes
bin/nabu enable <source> && bin/nabu sync <source>   # grow it, one source at a time
```

Register the server (absolute paths, so it works from any directory):

```
claude mcp add --scope user nabu \
  --env BUNDLE_GEMFILE=/path/to/nabu/Gemfile \
  -- bundle exec /path/to/nabu/bin/nabu mcp
```

Claude Desktop: the same command, args and `BUNDLE_GEMFILE` env in
`claude_desktop_config.json` under `mcpServers.nabu`. Opening Claude Code
inside the checkout needs nothing: the repository ships `.mcp.json`.
The full library is large (roughly 120 GB of fetched sources plus 160 GB of
derived indexes); recommend starting small. Details: `docs/mcp.md` and
`docs/quickstart.md` in the Nabu repository.

## The units you work with

- **Document**: what upstream delivers (a tablet, an inscription, a papyrus, a
  work in one edition, a dictionary entry). **Passage**: its citable subunit
  (line, verse, section, sentence). Both have a `urn`: CTS urns where upstream
  has them (`urn:cts:greekLit:tlg0012.tlg001.perseus-grc2:1.1`),
  `urn:nabu:<source>:…` otherwise.
- **A urn names an edition, not a work.** Two editions of the Iliad are two
  documents. Never cite one edition for a reading found in another.
- **Nothing is deleted.** Passages carry `revision`; withdrawn or upstream-
  removed content stays visible with `withdrawn` / `retired_upstream` flags.
  Report the flags when you cite.
- **Language codes are as upstream supplies them**, ISO 639 based; some
  sources use two-letter codes (`en`, `de`, `ko`) beside three-letter ones
  (`eng`). Check `nabu_status` before filtering by `lang`.

## The 14 tools

Every passage in every reply carries `urn`, `language`, `license_class` (and
usually `source`). Keep those fields with anything you quote. Replies are
bounded and say so ("showing k"); raise `limit` or narrow rather than assume
completeness.

### nabu_status: what is on this box
No arguments. Per-source counts, languages, license classes, index state,
what is excluded by default. Call it first, and again before ever concluding
that something is unattested.

### nabu_clarify: turn a vague reference into a urn
`{"query": "Odyssey 1.1"}` → candidates with `ref_urn` for each edition
holding that citation (here the Greek and the English Perseus editions).
It never guesses: an ambiguous reference resolves to nothing and lists every
reading; `match: "fuzzy"` marks a misspelling guess. Use it when the user
names a work rather than a urn.

### nabu_search: find passages
`{"query": "μηνιν αειδε", "lang": "grc", "limit": 5}`: full-text, words AND,
`"phrase"`, `prefix*`, diacritics optional. Give exactly one of `query`,
`lemma`, `similar_to`, `sense`. Useful variants:
- `{"lemma": "λέγω", "morph": "tense=aor,mood=opt"}`: dictionary-form search
  across every inflection, filtered by Universal Dependencies morphology.
  Non-gold hits carry a `tier` field; keep it.
- `near` + `window` for proximity in the same passage; `from`/`to`/`century`
  (signed years, negative = BCE) with `date_basis`; `place`, `script`,
  `lect`, `kind`, `facets` for document-level filters (undated, unplaced or
  unclassified documents drop out under them).
- `fuzzy: true` for fragment matching inside damaged documentary text;
  `translations: true` to search English/German translation layers and get
  the original passages back; `betacode: true` for TLG Beta Code input.
Snippets show where a match is; they are not quotable text.

### nabu_show: read the text
`{"urn": "urn:cts:greekLit:tlg0012.tlg001.perseus-grc2:1.1-1.2"}` → the first
two lines of the Iliad with title, source, revision, date block and license.
Accepts a passage, a document, an inclusive range, or a citation prefix
(lists everything beneath it). `parallel: true` (with `parallel_lang`, default
`eng`) aligns a translation line by line. `max_passages` caps at 200. **Quote
only what nabu_show returns.**

### nabu_concord: keyword in context
`{"lemma": "virtus", "lang": "lat", "width": 40}` → KWIC rows in corpus order,
keyword located in the pristine text. Use it to survey usage; use
`nabu_search` when you want relevance ranking.

### nabu_align: one verse across every witness
`{"ref": "MARK 2.3"}` → the verse in every registered witness (Greek, Latin,
Gothic, Classical Armenian, Old Church Slavonic, Old English, Coptic,
English…), each with its own license. Also takes a chapter (`"JON 1"`), a
same-book range (`"JON 1.1-1.16"`), or a passage urn to pivot from;
`collate: true` returns a variant apparatus instead. Statuses `no_match` /
`not_synced` are honest absences; report them as such.

### nabu_define: dictionary lookup
`{"lemma": "virtus", "lang": "lat"}` → the Lewis & Short entry with its
citations; cited passages held in the library come with a `resolved_urn` you
can open with `nabu_show`. The `lang` enum in the tool schema is the live set
of shelf languages. A leading asterisk (`*bogъ`) scopes to reconstruction
shelves. Bodies are capped at 6,000 characters (noted when truncated).

### nabu_etym: walk up to reconstructed ancestors
`{"lemma": "guþ", "lang": "got"}` → Proto-Germanic `*gudą` with its cognates
(each with `attested_count` in this library; `null` means not attested here,
not zero) and the chain further up to Proto-Indo-European. Data comes from
Wiktionary extracts: report it as Wiktionary's reconstruction, not as
settled fact.

### nabu_parallels: who quotes or echoes this passage
`{"urn": "urn:nabu:sblgnt:matt:4.4", "lang": "grc", "limit": 10}` → documents
sharing rare four-word phrases with the anchor, ranked by rarity-weighted
score, plus `lemma_echoes` for reworded allusions when the anchor has gold
lemmas. Other editions of the same work rank first; read past them to find
reception (Matthew 4:4 reaches LXX Deuteronomy 8:3, Philo and Origen). `explain: true` shows
the score arithmetic. The `evidence` phrases are folded; quote via
`nabu_show`.

### nabu_cognates: shared roots in aligned verses
`{"target": "LUKE 14.34", "langs": ["got", "chu"]}` → verse-and-root groups,
e.g. Gothic *salt* and Old Church Slavonic *соль* under one Indo-European
root. Check the root's shelf: a Proto-Germanic meet involving a Slavic
witness often signals borrowing, not inheritance; follow the `borrowed` flag.

### nabu_links: precomputed connections
`{"urn": "urn:nabu:sblgnt:matt:4.4"}` → edges a batch job already mined
(parallels, formulas, cognates, Arabic text reuse), with provenance in `runs`.
Empty means no batch covered this urn, not that no parallel exists; fall back
to `nabu_parallels`.

### nabu_place: an ancient place and what the library holds there
`{"query": "Lilybaeum"}` (or a Pleiades id `462281`, or `tm:2810`) →
the gazetteer card plus per-source document counts, with a separately
labeled `unlinked` tail of documents that merely mention the name. Titles
match exactly; there is no fuzzy matching. Preserve the card's `attribution`
(Pleiades is CC BY 3.0).

### nabu_char: a cuneiform sign or Egyptian hieroglyph card
`{"query": "MZL535"}` → the Oracc Sign List card for ŠEŠ (𒋀): readings,
sign-list concordances, glosses, corpus counts. Also takes a glyph, a sign
name or value (`szesz`), or a Gardiner code (`G5`). Ambiguous values list
all candidate signs. The Han character card is not served here; use
`nabu_define` for Chinese characters.

### nabu_signs: resolve a transliteration sign by sign
`{"text": "a-na {d}utu be-li2-ia", "lang": "akk"}` → each token mapped to
its sign name and Unicode codepoint with a status (`deterministic`,
`ambiguous`, `broken`, `unknown`…); determinatives flagged. Or pass a `urn`
from an ATF corpus (CDLI, Oracc, eBL, ETCSL).

## License classes: what you may quote and export

The guidance in this skill is MIT. The texts are not: every passage carries
its own `license_class`. Read it from the reply, never from memory, since
individual documents can differ from their source.

| Class | Meaning | What you may do |
| --- | --- | --- |
| `open` | Public domain, CC0 or equivalent | Quote and reuse freely; cite anyway. |
| `attribution` | CC BY, CC BY-SA and similar | Quote and reuse with credit to the source edition; share-alike terms travel with the text. |
| `nc` | Non-commercial (CC BY-NC variants) | Quote for research, teaching and discussion with credit; do not prepare it for commercial reuse. |
| `odbl` | Open Database License | Credit the database; derived databases are share-alike. |
| `research_private` | A personal research grant or private holding | Excluded by default. Do not set `include_restricted` unless the user asks and understands; never reproduce it for redistribution. |
| `restricted` | Other restricted terms | Excluded by default, as above. |

- When a reply carries a `credit` key, it is a grantor's display condition:
  keep it with the quotation.
- Reference cards (`nabu_place`, `nabu_char`, `nabu_signs`) carry their own
  license envelope; keep it when quoting their facts.
- Quote passages, not whole works. For bulk data the user should run
  `nabu export … --license-class open,attribution` on their own machine; it
  writes a manifest of every source used. Do not reconstruct a corpus by
  paging `nabu_show`.
- Verbatim upstream terms for every source: `docs/02-sources.md` in the Nabu
  repository.

A good citation names the edition, the urn, the class and the source:

> μῆνιν ἄειδε θεὰ Πηληϊάδεω Ἀχιλῆος — *Iliad* 1.1, Perseus edition
> (`urn:cts:greekLit:tlg0012.tlg001.perseus-grc2:1.1`, license: attribution)

## Honest limits

- **Empty is not absent.** A miss may mean the source is not synced here, the
  language is stored in another script or romanization, or the class is
  excluded by default. Check `nabu_status` and the reply's note before saying
  a word or text is unattested.
- **Search is folded; quotation is not.** Matching ignores case, diacritics
  and some per-language spelling variants. Snippets, KWIC keywords and
  parallels evidence are not readings.
- **Lemma tiers do not add up.** Gold is verified annotation and the only tier
  that counts as attestation; silver is upstream-automatic; equivalence is a
  gloss-level match. Report them side by side, never summed. Lemma search
  reaches only the lemmatized minority of the library.
- **Dates are typed.** `date_class` distinguishes artifact dating, attributed
  composition dates, edition years and model estimates. Most documents are
  undated, which is an absence, not a zero.
- **Optional layers may be missing on a box**: semantic `similar_to` needs a
  locally built vector store, `lect` filters need the lects registry module,
  `nabu_place` names need the gazetteer dump. The reply says so; relay it.
- **Notes are not upstream.** A `notes` field holds the library owner's own
  annotations; attribute them as commentary if you use them.
- **No interpretation is supplied.** Nabu returns editions and data; any
  translation, dating argument or etymology you add is yours and should be
  marked as such.

## For a reproducible claim, record

The urn and revision (with any flag), the source and license class (and
`credit` if present), the exact tool call with its arguments, the date of the
`nabu_status` snapshot, and for counts the tier and grain. Per-discipline
methodology (cuneiform, classical, biblical, Sinitic and more) ships in the
Nabu repository under `skills/`.
