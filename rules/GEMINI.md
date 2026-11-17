# Nabu — a local library of ancient texts

This extension connects a read-only MCP server over a research library
of ancient and historical texts. The bundled configuration runs the
DEMO image (Docker required): the SBL Greek New Testament, the Hebrew
Bible with morphology, the Clementine Vulgate, the World English Bible,
and the LSJ and Lewis & Short dictionaries, plus the registries the
tools read. The full library — millions of documents across hundreds of
sources, from cuneiform to early-modern print — is a local build from
the same repository: https://github.com/arvicco/nabu

## Using the tools

Every `nabu_*` tool is read-only; explore freely. The essentials:

- `nabu_search` — full-text search; `lemma:` for lemmatized search;
  filters for language, source and license class.
- `nabu_show` — a passage (or run of passages) by citation URN, the
  stable address every result carries.
- `nabu_define` — dictionary lookup (LSJ, Lewis & Short in the demo).
- `nabu_concord` — concordance lines for a form or lemma.
- `nabu_align` — cross-witness alignment of Bible verses.
- `nabu_status` — what this library actually holds; check it first.

## Etiquette

Every payload carries the license class of the text it quotes. Quote
with the credit line the result gives you; `open` and `attribution`
classes are safe to reproduce with credit. The deeper guide lives in
`skills/nabu/SKILL.md` in this repository.
