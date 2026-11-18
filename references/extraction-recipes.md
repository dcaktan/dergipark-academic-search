# Tested Extraction Snippets

Every snippet below was executed against live DergiPark pages and returned complete
data. Drop them into the browser console (or tab.evaluate) on the relevant page.

## 1. Article to structured metadata

Run on .../article/{id}. Verified: 100% field coverage on a live article.

```js
(() => {
  const M = (sel) => Array.from(document.querySelectorAll(sel));
  const one = (n) => { const e = M('meta[name="' + n + '"]')[0]; return e ? e.content : null; };
  const many = (n) => M('meta[name="' + n + '"]').map(e => e.content).filter(Boolean);
  const first = one('citation_firstpage'), last = one('citation_lastpage');
  return {
    title: one('citation_title'),
    authors: many('citation_author').map((name, i) => ({
      name,
      institution: many('citation_author_institution')[i] || null,
      orcid: many('citation_author_orcid')[i] || null
    })),
    doi: one('citation_doi'),
    journal: one('citation_journal_title'),
    journalAbbrev: one('citation_journal_abbrev'),
    issn: one('citation_issn'),
    publisher: one('citation_publisher'),
    date: one('citation_publication_date'),
    year: (one('citation_publication_date') || '').slice(0, 4),
    volume: one('citation_volume'),
    issue: one('citation_issue'),
    pages: first && last ? first + '-' + last : null,
    language: one('citation_language'),
    articleType: one('citation_article_type'),
    keywords: (one('citation_keywords') || '').split(';').map(s => s.trim()).filter(Boolean),
    abstract: one('citation_abstract'),
    pdfUrl: one('citation_pdf_url')
      ? new URL(one('citation_pdf_url'), location.origin).href : null,
    url: one('citation_abstract_html_url') || location.href,
    referenceCount: many('citation_reference').length
  };
})()
```

## 2. Issue page to article list

Run on .../issue/{id}. Verified: 21/21 cards, 0 null fields.

```js
(() => {
  const abs = (u) => { try { return new URL(u, location.origin).href; } catch (e) { return u; } };
  return Array.from(document.querySelectorAll('article.article-card')).map(card => {
    const titleEl = card.querySelector('a.art-title');
    const text = card.innerText.replace(/\s+/g, ' ').trim();
    const pages = text.match(/Sayfa\s*(\d+)\s*-\s*(\d+)/) || [];
    const date = (text.match(/(\d{1,2}\s+[A-Za-zCĞİÖŞÜcğiöşü]+\s+\d{4})/) || [])[1] || null;
    const pdf = card.querySelector('a[href*="article-file"]');
    const href = titleEl ? titleEl.getAttribute('href') : '';
    return {
      articleId: (href.match(/article\/(\d+)/) || [])[1] || null,
      title: titleEl ? titleEl.innerText.replace(/\s+/g, ' ').trim() : null,
      url: href ? abs(href) : null,
      type: card.querySelector('.art-type') ? card.querySelector('.art-type').innerText.trim() : null,
      authors: Array.from(card.querySelectorAll('a.author-link')).map(a => a.innerText.trim()),
      authorProfiles: Array.from(card.querySelectorAll('a.author-link')).map(a => abs(a.getAttribute('href'))),
      date,
      pages: pages.length === 3 ? pages[1] + '-' + pages[2] : null,
      pdfUrl: pdf ? abs(pdf.getAttribute('href')) : null
    };
  });
})()
```

Note: the wrapper is article.article-card, NOT .article-content. Using .article-content
silently drops the PDF link, which lives in a sibling .art-actions block.

## 3. Archive page to issue list

Run on .../archive.

```js
(() => Array.from(document.querySelectorAll('a[href*="/issue/"]')).map(a => ({
  label: a.innerText.replace(/\s+/g, ' ').trim(),
  url: new URL(a.getAttribute('href'), location.origin).href,
  issueId: (a.getAttribute('href').match(/issue\/(\d+)/) || [])[1]
})))()
```

## 4. Author page to publications

Run on /tr/pub/@handle; the same card extractor as section 2 applies to the result list.

## 5. Crossref keyword discovery (no captcha, no key)

```js
(async () => {
  const url = 'https://api.crossref.org/works'
    + '?query.bibliographic=' + encodeURIComponent('yapay zeka')
    + '&filter=from-pub-date:2023-01-01,type:journal-article'
    + '&rows=20&select=DOI,title,author,container-title,issued,abstract'
    + '&mailto=you@example.com';
  const r = await fetch(url);
  const j = await r.json();
  return j.message.items.map(i => ({
    doi: i.DOI,
    title: (i.title || [''])[0],
    journal: (i['container-title'] || [''])[0],
    year: ((i.issued && i.issued['date-parts']) || [['']])[0][0],
    authors: (i.author || []).map(a => ((a.given || '') + ' ' + (a.family || '')).trim()),
    hasAbstract: !!i.abstract
  }));
})()
```
