# Repository Update Notes

This package consolidates the curated GitHub project structure and provides
an update workflow for an existing repository.

## Included
- Project overview and key findings
- Editable DOCX and PDF investigation report
- 28 documented SPL searches and the `dnsf` extraction macro
- 30 numbered evidence screenshots
- Splunk ingestion guide and findings summary

## Important publication note
The report files originated as a draft. Personalise the cover and resolve
the URLhaus placeholder in the DOCX, then export a matching PDF before
publishing. The course dataset (`dns.log`) is intentionally excluded.

## Update strategy
Copy the contents of this package's `dns-log-analysis-splunk/` folder into
your existing local clone, preserve its `.git` directory, then review,
commit, and push. See `GITHUB_UPLOAD_GUIDE.txt` outside the repository root.
