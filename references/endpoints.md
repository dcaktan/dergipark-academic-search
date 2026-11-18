# DergiPark Endpoint Reference

All entries verified against the live site. robots.txt is reproduced here in full
because it governs what this skill may do.

## robots.txt

```
User-agent: AhrefsBot
Disallow: /

User-agent: *
Disallow: /login
Disallow: /tr/login
Disallow: /en/login
Disallow: /search
Disallow: /tr/search
Disallow: /en/search
```

Consequence: /search is forbidden by the site owner in addition to being
Turnstile-protected. This skill must not fetch it.

## Working endpoints

| Purpose | URL | Verified result |
|---|---|---|
| Article page | /tr/pub/{slug}/article/{id} | 200, ~90 KB, citation_* meta tags |
| Article (EN) | /en/pub/{slug}/article/{id} | 200, English abstract |
| DOI resolver | /tr/doi/{doi} | 302 -> article page |
| Journal home | /tr/pub/{slug} | 200, ~275 KB |
| Archive | /tr/pub/{slug}/archive | 200, 53 issue links (sample) |
| Issue | /tr/pub/{slug}/issue/{id} | 200, article.article-card list |
| Author profile | /tr/pub/@{handle} | 200, article list |
| Publisher | /tr/pub/publisher/{id} | 200, ~157 KB |
| PDF | /tr/download/article-file/{fileId} | 200, application/pdf |
| DOI handle API | https://doi.org/api/handles/{doi} | 200, JSON, has DergiPark target URL |
| Crossref | https://api.crossref.org/works | 200, JSON, CORS-enabled |

## Confirmed NON-existent (verified 404)

Do not waste turns on these:

| URL | Status |
|---|---|
| /api/* (any) | 404 |
| /oai?verb=Identify | 404 (redirects to /en/pub/oai) |
| /tr/oai?verb=Identify | 404 |
| /{slug}/oai?verb=Identify | 404 |
| /tr/pub/{slug}/oai?verb=Identify | 404 |
| /search/suggest?q=... | 404 |
| /tr/search?q=... | Turnstile verification page, not results |

DergiPark does NOT expose OAI-PMH. If a user needs OAI, they must use the publisher's
own repository.

## Crossref recipes

```bash
# Keyword discovery
https://api.crossref.org/works?query.bibliographic={terms}
  &filter=from-pub-date:2023-01-01,type:journal-article
  &rows=20&select=DOI,title,author,container-title,issued,abstract&mailto={email}

# All articles of one journal (complete, exact)
https://api.crossref.org/works?filter=issn:{issn}&rows=100&select=DOI,title,issued

# Keyword within one journal
https://api.crossref.org/works?filter=issn:{issn}&query.bibliographic={terms}
  &rows=20&select=DOI,title,issued,abstract

# Date-bounded within one journal
https://api.crossref.org/works?filter=issn:{issn},from-pub-date:2024-01-01
  &rows=100&select=DOI,title,issued
```

Verified numbers on sample journal ISSN 1308-6693:
- all articles: 1,019
- keyword "yapay zeka": 22
- since 2024: 220

Crossref limits rows to 1,000 and paginates with &offset=.

## Coverage caveat

DOI registration is per journal or per publisher - not centralized under DergiPark.
Crossref members matching "dergipark" returns many separate publishers (e.g.
"Erciyes Universitesi (DergiPark)" = member 7004, "Acarological Studies, DergiPark" =
member 26688). There is no single member/prefix that covers all of DergiPark, and
Crossref records have no field marking a DOI as DergiPark-hosted.

Practical rule:
- Journal known -> use filter=issn: (exact and complete).
- Topic only -> Crossref keyword search, then resolve each DOI through
  https://doi.org/api/handles/{doi} and keep those resolving to dergipark.org.tr.
- Completeness critical -> ask the user to run the search UI manually; we cannot.
