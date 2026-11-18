# dergipark-academic-search

Claude Skill for **DergiPark** (dergipark.org.tr) - Turkey's national journal platform
run by TUBITAK ULAKBIM.

Reads article metadata, walks journal archives, resolves DOIs and links PDFs. DergiPark
has no public API and its search page is bot-protected, so the skill routes around both
instead of fighting them. Every endpoint and extraction snippet was verified against the
live site before being written down.

## What it does

- Article page -> title, authors (+ institution, ORCID), DOI, journal, ISSN, volume,
  issue, pages, date, language, type, keywords, abstract, PDF URL
- Journal archive -> issue list -> article list
- DOI -> article page resolution
- Author profile -> publication list
- Crossref for topic discovery

## What it deliberately does NOT do

- No scraping of /search - disallowed by robots.txt and gated by Cloudflare Turnstile
- No attempt to bypass bot protection
- No bulk downloading, no login, no authenticated areas

## Install

Claude Code:

    git clone https://github.com/dcaktan/dergipark-academic-search.git
    mkdir -p ~/.claude/skills
    cp -r dergipark-academic-search ~/.claude/skills/
    ls ~/.claude/skills/dergipark-academic-search/SKILL.md

Then restart Claude Code. The skill loads automatically from the SKILL.md frontmatter.

Cherry Studio: Settings -> Skills -> Import -> pick the cloned folder or a zip of it.

## Layout

    SKILL.md                            main instructions (required)
    references/metadata-mapping.md      citation_* / DC.* tag mapping
    references/extraction-recipes.md    tested browser-console snippets
    references/endpoints.md             working and 404 endpoints
    LICENSE                             MIT

## Disclaimer

This tool only reads publicly available metadata. The user is responsible for complying
with the terms of use of any site it touches. Running automated requests against a
third-party service is the user's decision and the user's responsibility.

All examples in this repository are **fictitious placeholders**. No real author names,
ORCID identifiers, DOIs or article titles are included.

## License

MIT - see [LICENSE](LICENSE).
