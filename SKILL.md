---
name: dergipark-academic-search
description: Search and retrieve Turkish academic articles published on DergiPark (TUBITAK ULAKBIM's national journal platform) - articles, abstracts, authors, journals, issues and PDFs. Use when looking for Turkish journal articles, walking a DergiPark journal archive, resolving a DergiPark DOI, or building a Turkish literature review. Accounts for DergiPark having no public API and a bot-protected search page.
---

# DergiPark Academic Search

## Overview

DergiPark (dergipark.org.tr) is the publishing platform for ~1,800 Turkish academic
journals run by TUBITAK ULAKBIM. It hosts full-text articles, mostly open access.

Read this first - the access model is unusual and the obvious approach fails:

| Approach | Status | Why |
|---|---|---|
| Official JSON/REST API | Does not exist | /api/*, /oai, /tr/oai all return 404 (verified) |
| Scraping /search | Off-limits | robots.txt Disallows /search, /tr/search, /en/search; page is gated by Cloudflare Turnstile |
| Article pages | Works, server-rendered | Clean Highwire citation_* meta tags |
| Journal archive and issue pages | Works, server-rendered | Parseable HTML |
| DOI resolution | Works | /tr/doi/{doi} -> 302 -> article page |
| Crossref API | Works, no key needed | Use for keyword discovery (Workflow 3) |

Never attempt to bypass the Turnstile challenge or fetch /search. If the user wants a
keyword search across all of DergiPark, use Crossref or a search engine with
site:dergipark.org.tr (Workflow 3), or hand the search over to the user.

## Responsible use

This skill only reads publicly available metadata from dergipark.org.tr and from the
Crossref API. It does not bypass authentication, captchas or bot protection, and it does
not download the corpus in bulk. Before pointing it at a third-party service the user is
responsible for checking that service's current terms of use. See
references/endpoints.md for the detailed note and the verified robots.txt rules.

## URL Patterns (all verified)

```
Article      https://dergipark.org.tr/tr/pub/{journalSlug}/article/{articleId}
Article (EN) https://dergipark.org.tr/en/pub/{journalSlug}/article/{articleId}
DOI resolver https://dergipark.org.tr/tr/doi/{doi}
Journal home https://dergipark.org.tr/tr/pub/{journalSlug}
Archive      https://dergipark.org.tr/tr/pub/{journalSlug}/archive
Issue        https://dergipark.org.tr/tr/pub/{journalSlug}/issue/{issueId}
Author page  https://dergipark.org.tr/tr/pub/@{authorHandle}
Publisher    https://dergipark.org.tr/tr/pub/publisher/{publisherId}
PDF          https://dergipark.org.tr/tr/download/article-file/{fileId}
```

Swap /tr/ for /en/ anywhere to get the English UI (affects which abstract is shown).

## Workflow 1: DOI to full metadata (most reliable)

```bash
curl -sL "https://dergipark.org.tr/tr/doi/10.12345/ornek.000001" \
  | grep -oP '<meta name="citation_(title|author|doi|abstract|issn|journal_title|publication_date|volume|issue|firstpage|lastpage|keywords|language|article_type|pdf_url)" content="\K[^"]*'
```

Article pages are server-rendered, so curl works - no JS needed. For structured output
use the extractor in references/extraction-recipes.md (browser console) or map the meta
tags per references/metadata-mapping.md.

## Workflow 2: Walk a journal archive

1. GET /tr/pub/{slug}/archive -> collect every a[href*="/issue/"] link. The anchor text
   reads "Cilt: 14 Sayi: 3", which is also the volume/issue label.
2. GET each issue page -> collect article.article-card elements.
3. For full metadata on an article, fetch its article page (Workflow 1).

Verified extraction: 21/21 articles on a sample issue returned title, URL, authors,
date, pages and PDF link with zero missing fields.
See references/extraction-recipes.md for the tested snippet.

## Workflow 3: Topic discovery (when there is no DOI and no journal)

DergiPark search is unavailable to automation, so pick one:

### 3a. Crossref (preferred - free, no key, no captcha)

```bash
curl -s "https://api.crossref.org/works?query.bibliographic=yapay+zeka\
&filter=from-pub-date:2023-01-01,type:journal-article&rows=20\
&select=DOI,title,author,container-title,issued,abstract&mailto=you@example.com"
```

### 3b. Search inside a known journal (exact and complete)

```bash
# Every article in one journal (ISSN filter) - verified: 1,019 records for 1308-6693
curl -s "https://api.crossref.org/works?filter=issn:1308-6693&rows=100&select=DOI,title,issued"

# And keyword search within that journal - verified: 22 hits
curl -s "https://api.crossref.org/works?filter=issn:1308-6693\
&query.bibliographic=yapay+zeka&rows=20&select=DOI,title,issued,abstract"
```

To get a journal's ISSN, read citation_issn from any article page (verified) - it is more
reliable than parsing the journal home page.

### 3c. Search engine

```
site:dergipark.org.tr "makine ogrenmesi" 2024
```

Important: Crossref DOIs are registered per journal/publisher, not by DergiPark as a
whole. There is no single member filter that isolates DergiPark. A Crossref result is
likely DergiPark if resolving the DOI leads to dergipark.org.tr.

## Workflow 4: Author publication list

1. Get the author's handle from any issue page: a.author-link -> /tr/pub/@handle
2. GET that page - it lists the author's articles across all journals.
3. Deduplicate and sort by date.

## Workflow 5: Full text

citation_pdf_url (relative, e.g. /tr/download/article-file/0000000) returns
application/pdf. Always report the PDF URL so the user can open it directly.
Many articles are Turkish-language with an English abstract - fetch the /en/ variant
when the user wants the English abstract.

## Output format

```
Author(s). (Year). Title. *Journal*, Volume(Issue), pages. DOI: xxx
```

Always include the DOI and the DergiPark article URL. Never invent a DOI, page range or
author - if a field is absent in the metadata, omit it rather than guessing.

## Rate limits and politeness

- One request per second maximum against dergipark.org.tr; batch slowly for large archives.
- Set a descriptive User-Agent; for Crossref add &mailto= to the query.
- Cache issue pages locally - repeated runs should not re-hit the site.

## Important notes

- e-ISSN only is common; many DergiPark journals have no print ISSN.
- citation_pdf_url is relative - prefix https://dergipark.org.tr
- Some journals show a "TR Dizin" badge; for verified TR Dizin indexing and citation
  data, cross-check with the trdizin-academic-search skill.
- DergiPark metadata is author-supplied: expect occasional typos and inconsistent author
  name formats. Prefer ORCID when present for disambiguation.

## Resources

- references/metadata-mapping.md - every citation_* / DC.* tag, verified
- references/extraction-recipes.md - tested browser-console snippets
- references/endpoints.md - full URL table including confirmed negative results
