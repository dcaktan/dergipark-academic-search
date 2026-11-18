# DergiPark Metadata Mapping

Article pages carry Highwire Press meta tags plus Dublin Core. All tags below were
verified on a live article page. Multi-value tags repeat (authors, institutions,
ORCIDs, references).

## Complete verified tag inventory

| Tag | Repeats | Example | Maps to |
|---|---|---|---|
| citation_title | no | STRATEJIK RISK ANALIZINDE ... | title |
| citation_author | yes | Hasan Koseoglu | authors[].name |
| citation_author_institution | yes | SULEYMAN DEMIREL UNIVERSITY | authors[].institution |
| citation_author_orcid | yes | 0000-0003-0183-208X | authors[].orcid |
| citation_doi | no | 10.21923/jesd.1932518 | doi |
| citation_journal_title | no | Muhendislik Bilimleri ve Tasarim Dergisi | journal.title |
| citation_journal_abbrev | no | MBTD | journal.abbrev |
| citation_issn | no | 1308-6693 | journal.issn |
| citation_publisher | yes (dup) | Suleyman Demirel Universitesi | publisher |
| citation_publication_date | no | 2026-09-25 | date (ISO) |
| citation_volume | no | 14 | volume |
| citation_issue | no | 3 | issue |
| citation_firstpage | no | 544 | firstPage |
| citation_lastpage | no | 563 | lastPage |
| citation_article_type | no | Arastirma Makalesi | articleType |
| citation_language | no | tr | language |
| citation_keywords | no | Afet Yonetimi;CKKV;Deprem | keywords[] (semicolon-separated) |
| citation_abstract | no | full abstract text | abstract |
| citation_abstract_html_url | no | article URL | url |
| citation_pdf_url | no | /tr/download/article-file/5923395 | pdfUrl (relative!) |
| citation_reference | yes | Cajilig, P., 2024. ... | references[] |

## Dublin Core (secondary, use as fallback)

| Tag | Example |
|---|---|
| DC.Title | article title |
| DC.Source | journal title |
| DC.Source.Volume / DC.Source.Issue | 14 / 3 |
| DC.Identifier | 1932518 (article id) |
| DC.Identifier.DOI | 10.21923/jesd.1932518 |
| DC.Identifier.pageNumber | 544-563 (already joined) |
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

Only e-ISSN is reliably present as text (e-ISSN: 1308-6693). Print ISSN may be absent
entirely. Prefer citation_issn from an article page over parsing the journal page.
