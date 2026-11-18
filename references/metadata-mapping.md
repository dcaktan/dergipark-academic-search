# DergiPark Metadata Mapping

Article pages carry Highwire Press meta tags plus Dublin Core. All tags below were
verified on a live article page. Multi-value tags repeat (authors, institutions,
ORCIDs, references).

> **All example values below are fictitious placeholders.** Author names, ORCID, DOI,
> article id, pages and keywords are invented so that a real person is not identified.
> The card selectors and verified counts were observed on the live site.

## Complete verified tag inventory

| Tag | Repeats | Example | Maps to |
|---|---|---|---|
| citation_title | no | Ornek Makale Basligi | title |
| citation_author | yes | Ornek Yazar Bir | authors[].name |
| citation_author_institution | yes | ORNEK UNIVERSITESI | authors[].institution |
| citation_author_orcid | yes | 0000-0002-1825-0097 | authors[].orcid |
| citation_doi | no | 10.12345/ornek.000001 | doi |
| citation_journal_title | no | Ornek Muhendislik Dergisi | journal.title |
| citation_journal_abbrev | no | OMD | journal.abbrev |
| citation_issn | no | 0000-0000 | journal.issn |
| citation_publisher | yes (dup) | Ornek Universitesi | publisher |
| citation_publication_date | no | 2025-01-01 | date (ISO) |
| citation_volume | no | 10 | volume |
| citation_issue | no | 2 | issue |
| citation_firstpage | no | 1 | firstPage |
| citation_lastpage | no | 10 | lastPage |
| citation_article_type | no | Arastirma Makalesi | articleType |
| citation_language | no | tr | language |
| citation_keywords | no | Ornek Anahtar Bir;Ornek Anahtar Iki | keywords[] (semicolon-separated) |
| citation_abstract | no | full abstract text | abstract |
| citation_abstract_html_url | no | article URL | url |
| citation_pdf_url | no | /tr/download/article-file/0000000 | pdfUrl (relative!) |
| citation_reference | yes | Author, A., 2025. Example reference. | references[] |

Note on citation_author_orcid: 0000-0002-1825-0097 is the ORCID foundation's public
example identifier, not a real researcher.

## Dublin Core (secondary, use as fallback)

| Tag | Example |
|---|---|
| DC.Title | article title |
| DC.Source | journal title |
| DC.Source.Volume / DC.Source.Issue | 10 / 2 |
| DC.Identifier | 0000000 (article id) |
| DC.Identifier.DOI | 10.12345/ornek.000001 |
| DC.Identifier.pageNumber | 1-10 (already joined) |
| DC.Identifier.URI | article URL |
| DC.Language | tr |
| DC.Type.articleType | Arastirma Makalesi |
| DC.Creator.PersonalName | repeats per author |

## Alignment caveat

citation_author, citation_author_institution and citation_author_orcid are three
separate lists that appear in document order. Pair them by index. If counts differ
(an author omitted their ORCID), index alignment breaks - match on name where possible
and leave the ORCID empty rather than shifting values.

## Issue-list card fields (not meta tags)

From article.article-card:

| Data | Source |
|---|---|
| title | a.art-title innerText |
| article id / URL | a.art-title[href] -> /article/{id} |
| type | .art-type innerText (uppercase in list) |
| authors | a.author-link (repeats) |
| author profile | a.author-link[href] -> /tr/pub/@handle |
| date | regex /(\d{1,2}\s+[A-Za-z...]+\s+\d{4})/ on card text |
| pages | regex /Sayfa\s*(\d+)\s*-\s*(\d+)/ on card text |
| PDF | a[href*="article-file"] |

Date and pages are not in dedicated elements - they only exist in the card's text,
which is why regex is required. Verified stable across a full 21-article issue.

## Journal home page

Only e-ISSN is reliably present as text (e-ISSN: 0000-0000). Print ISSN may be absent
entirely. Prefer citation_issn from an article page over parsing the journal page.
