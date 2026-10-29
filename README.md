# nabu-skill

An [Agent Skill](https://agentskills.io) that teaches an AI agent to work with
[Nabu](https://github.com/arvicco/nabu), a local, read-only library of
ancient-text corpora, through Nabu's MCP server.

## What Nabu is

Nabu is open-source research infrastructure (MIT, Ruby 3.3+) that ingests
openly licensed digital corpora of antiquity into one searchable, citable
SQLite library on your own disk: Greek and Latin, documentary papyri,
cuneiform, the Hebrew and Christian Bibles in many witnesses, Classical
Arabic, Literary Chinese and the Buddhist canon, Sanskrit, Tamil, Tibetan,
Old English, Old Church Slavonic, Gothic and more, with dictionaries,
lemmatized treebanks and reconstruction data. The project's reference
library holds 2,480,397 documents and 119,725,893 passages in 255 language
codes from 249 live sources (as of 28 October 2026; current figures at
https://arvicco.github.io/nabu/). Your own library is built source by
source and starts much smaller.

Nabu ships a read-only stdio MCP server (`bin/nabu mcp`) with 14 tools.
Every passage it returns carries its urn, language and license class.

## What this skill does

`skills/nabu/SKILL.md` tells an agent:

- what a Nabu library holds and how documents, passages and urns relate;
- how to install Nabu and register its MCP server when the tools are missing;
- how to drive each of the 14 tools, with one worked call each;
- what each license class allows when quoting or exporting;
- the library's honest limits (an empty result is not absence of evidence,
  gold and silver lemmas are never summed, dates are typed).

The skill contains guidance only. It does not bundle Nabu, any data, or an
MCP server: install Nabu itself from https://github.com/arvicco/nabu.

## Install

First install Nabu and register its MCP server (see the Nabu
[quickstart](https://arvicco.github.io/nabu/quickstart/)):

```
git clone https://github.com/arvicco/nabu && cd nabu
bundle install
bin/nabu quickstart
claude mcp add --scope user nabu \
  --env BUNDLE_GEMFILE="$PWD/Gemfile" \
  -- bundle exec "$PWD/bin/nabu" mcp
```

Then add the skill.

Claude Code, as a plugin from this repository's marketplace:

```
/plugin marketplace add arvicco/nabu-skill
/plugin install nabu@nabu-skill
```

Any Agent Skills client, with the `skills` CLI:

```
npx skills add arvicco/nabu-skill
```

Or copy `skills/nabu/` into your client's skills directory
(`~/.claude/skills/` for Claude Code).

## More guidance

The Nabu repository ships per-discipline skills (cuneiform, classical,
biblical, Sinitic and others) under
[`skills/`](https://github.com/arvicco/nabu/tree/main/skills), and the full
tool reference in [`docs/mcp.md`](https://github.com/arvicco/nabu/blob/main/docs/mcp.md).

## License

The skill text is MIT (see `LICENSE`). The texts a Nabu library serves are
not: each carries its own license class, which the skill explains.
