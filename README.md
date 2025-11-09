# Assignment10B — NobelPrize.org: Four Questions with the API

Author: Taha Malik  
Date: 2025-11-09

---

## Project overview

This project uses the NobelPrize.org REST API (v2 preferred) to fetch open JSON data about Nobel laureates, tidy the data, and answer four questions:

1. Top birth countries by count of laureates.
2. Birth countries that "lost" the most laureates (born in X, awarded under a different affiliation country).
3. Award years with the most laureates.
4. Age at award by category (median, mean, sd) and distributions (boxplots).

Everything is implemented as a single reproducible R Markdown: `Assignment10B_NobelPrize_org.Rmd`. The analysis fetches live data by default but includes an easy cache option to produce deterministic results for grading.

---

## Files in this project

- `Assignment10B_NobelPrize_org.Rmd` — The main R Markdown document. Contains all code, narrative, diagnostic messages, tables and plots. This is the file to knit to produce the HTML report.
- `README.md` — This file (how to run, design decisions, reproducibility notes).
- Optional cache files you may create locally:
  - `laureates_cached_v2.json` — cached raw JSON (optional)
  - `nobel_laureates_cached.rds` — cached raw RDS (optional)
  - `nobel_processed.rds` — cached processed tibbles (laureates_tidy, prizes_df2) (optional)

---

## Requirements

- R (version >= 4.0 recommended)
- R packages:
  - httr
  - jsonlite
  - dplyr
  - tidyr
  - lubridate
  - ggplot2
  - knitr
  - purrr
  - stringr
  - tibble

Install packages (example):

```r
install.packages(c("httr","jsonlite","dplyr","tidyr","lubridate",
                   "ggplot2","knitr","purrr","stringr","tibble"))
```

---

## Setup and quick run

1. Put `Assignment10B_NobelPrize_org.Rmd` and this `README.md` in the same directory.
2. Open the Rmd in RStudio (recommended) or run from R with `rmarkdown::render()`.

To knit in RStudio:
- Open the Rmd file and click *Knit*.

To render from R console:

```r
rmarkdown::render("Assignment10B_NobelPrize_org.Rmd", output_format = "html_document")
```

---

## Caching and reproducibility (recommended for grading)

The Rmd includes a `use_cache` toggle in the setup chunk. By default it is `FALSE` (fetch live data). For grading or reproducible submissions do the following:

1. Run the Rmd once with `use_cache <- FALSE` to fetch live data and produce processed tibbles.
2. After successful run, run these lines (or uncomment the same lines in the `fetch-data` chunk) to create cache files:

```r
jsonlite::write_json(laureates_list, "laureates_cached_v2.json", pretty = TRUE, auto_unbox = TRUE)
saveRDS(list(laureates_list = laureates_list), "nobel_laureates_cached.rds")
saveRDS(list(laureates_list = laureates_list,
             laureates_tidy = laureates_tidy,
             prizes_df2 = prizes_df2),
        "nobel_processed.rds")
```

3. Set `use_cache <- TRUE` at the top of the Rmd and re-knit. The document will load the cached RDS/JSON instead of calling the API.

Submit to the grader:
- `Assignment10B_NobelPrize_org.Rmd`
- `laureates_cached_v2.json` or `nobel_laureates_cached.rds` (at least one)
- Optionally `nobel_processed.rds` so the grader can reproduce tables/plots exactly.

---

## What each code block does (concise, focused)

- **setup** — Loads libraries, defines helper functions:
  - `safe_pluck_chr()` to safely extract nested list fields.
  - `coalesce_str()` for scalar fallbacks.
  - `country_normalize()` to standardize common country name variants.
  - `use_cache` flag and cache filenames.

- **fetch-data** — Fetches the laureates data:
  - Prefers v2 endpoint: `https://api.nobelprize.org/2.1/laureates`.
  - Set a polite `user_agent()` and a 30s timeout.
  - Handles HTTP 429 (rate-limited) and falls back to v1 if necessary.
  - Optionally saves JSON/RDS caches for reproducibility.

- **prepare-data** — Builds tidy tibbles:
  - `laureates_tidy`: one row per laureate with id, names, birth date and normalized birth country.
  - `prizes_df2`: one row per laureate-prize, with prize metadata and a list-column `affiliation_countries` containing all normalized affiliation-country values for that prize.
  - Joins birth info into prize rows and runs basic sanity checks.

- **Q1** — Top 10 birth countries by number of laureates:
  - Groups `laureates_tidy` by normalized birth country, counts unique laureates, outputs top 10 table.

- **Q2** — Top 10 birth countries that "lost" the most laureates:
  - For each prize, compares the prize's affiliation countries (all listed) to the laureate's normalized birth country.
  - Flags a laureate as "lost" if any prize has an affiliation country different from birth country.
  - Aggregates counts per birth country and outputs the top 10 table.

- **Q3** — Top 10 award years by number of laureates:
  - Counts distinct laureates per `awardYear` and outputs the top 10 years.

- **Q4** — Age at award by category:
  - Parses birth dates (partial/invalid dates become NA).
  - Uses `dateAwarded` when present; otherwise uses `awardYear` + Dec 10 as fallback.
  - Computes age in years, excludes nonsensical ages, summarizes by category and draws boxplots. Reports how many birth dates were excluded.

- **caching-note & session-info** — Instructions for caches and environment/session info for reproducibility.

---

## Design choices & assumptions (short)

- Affiliation vs. nationality: the code uses affiliation country (institution listed with the prize) as the location "under which" the prize was awarded. This is not identical to citizenship — it's an operational choice for "lost laureates".
- Multiple affiliations: all listed affiliation countries for a prize are collected and normalized; a difference in any of them triggers the "lost" flag for that prize.
- Partial birth dates: partial dates (e.g., `YYYY-00-00`) are excluded from age calculations. This is conservative; an imputation option can be added if desired, but must be documented.
- Country normalization is simple and extendable — update `country_normalize()` to add more mappings if you need higher historical accuracy.

---

## Troubleshooting

- If knitting fails because of missing packages: install them with `install.packages()`.
- If the R Markdown times out or API returns HTTP 429: set `use_cache <- TRUE` and use prepared cache files, or retry later.
- If you see unexpected NULLs in affiliation country, inspect the raw JSON for that laureate — the API sometimes uses different shapes; the code handles the common shapes.

---

## How to cite / API terms

- Data source: NobelPrize.org API (https://www.nobelprize.org/about/developer-zone-2/). Read and respect the API terms of use.
- If you reuse the data or visualizations publicly, include an attribution to NobelPrize.org.

---

## Contact

Author: Taha Malik   
---
