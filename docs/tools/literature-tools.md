# Literature & Reference Tools

Tools for searching scholarly literature and building verified bibliographies. Every search tool returns **abstract snippets** so the agent can verify claims before citing; references are saved to a report's bibliography and can be exported to BibTeX or Zotero.

## Search Tools

| Tool | Source | Description |
|------|--------|-------------|
| `search_pubmed` | PubMed/NCBI | Biomedical literature with abstract snippets |
| `search_crossref` | CrossRef | 150M+ scholarly works by keyword |
| `search_semantic_scholar` | Semantic Scholar | ~200M papers, any discipline |
| `search_arxiv` | arXiv | Open-access preprints (modelling, statistics, CS) |
| `search_preprints` | bioRxiv / medRxiv | Epidemiology & public-health preprints |
| `filter_openalex` | OpenAlex | Structured metadata filtering with abstract snippets |
| `list_zotero_collections` | Zotero | List collections across the user's configured libraries |

## Metadata & Verification Tools

Use these to fetch the **full** record for one known work — citation plus abstract — before saving it to a report.

| Tool | Description |
|------|-------------|
| `get_pubmed_citation` / `get_pubmed_citations` | One or many PubMed IDs, citations with abstracts |
| `get_pubmed_abstract` | Full abstract of a PubMed article |
| `get_crossref_metadata` | Full citation metadata for a DOI (CrossRef registrant) |
| `resolve_doi` | Resolve ANY DOI to full citation metadata (any registrant) |
| `get_openalex_work` | Formatted OpenAlex citation with full abstract |
| `get_semantic_scholar_paper` | Full paper metadata by any identifier |
| `get_arxiv_paper` | Full metadata for one arXiv ID |
| `get_preprint_details` | Full metadata for a bioRxiv/medRxiv DOI |

## Bibliography Tools

| Tool | Description |
|------|-------------|
| `save_report_references` | Save verified PubMed references to a report's bibliography |
| `save_openalex_references` / `save_doi_references` | Same for OpenAlex and DOI-identified references |
| `format_pubmed_bibliography` | Numbered bibliography from verified references, with abstracts |
| `export_references_bibtex` | Export a report's bibliography as BibTeX |
| `import_bibtex_references` / `import_bibtex_file` | Import references from BibTeX text or a `.bib` file |

## Zotero Integration

| Tool | Description |
|------|-------------|
| `list_zotero_collections` | List collections across configured Zotero libraries |
| `fetch_zotero_references` | Fetch references from a Zotero library |
| `export_to_zotero` | Export a report's references to Zotero |

Zotero tools are gated to the **Pro** plan (`zotero` feature) and require a Zotero API key configured in **Settings → Zotero**.

## Professional Profile

`build_professional_profile` assembles a researcher profile from **ORCID** and **Google Scholar** — publications, metrics and affiliations — useful for report author blocks and collaboration scoping.

## Typical Workflow

```
Encontre referências sobre transmissão de dengue por Aedes albopictus,
verifique-as e adicione à bibliografia do meu relatório.
```

1. `search_pubmed` — find candidates with abstract snippets
2. `get_pubmed_citations` — fetch full records for the shortlist
3. `save_report_references` — save verified references to the report
4. `format_pubmed_bibliography` — render the numbered bibliography

## Notes

- Search/metadata tools have a 300s timeout; NCBI rate limits are respected automatically.
- Literature search results are truncated to the global `max_tool_output_chars` cap — ask for narrower queries rather than full listings.
