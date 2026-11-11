# Omni Preprint Manager (`preprint`)

# About

**`preprint`** is an academic preprint and publication manager for the [Omni Shell Framework](../omni/README.md). It provides a unified CLI and interactive `fzf` interface across four major scholarly platforms:

- **arXiv** (`arxiv`) — Search, metadata inspection, PDF download, and SWORD/package submission preparation.
- **Zenodo** (`zenodo`) — REST API integration (production and sandbox) for listing depositions, searching records, inspecting metadata, downloading PDFs, uploading files, and publishing records.
- **HAL** (`hal`) — Open archive search, author publication listing, metadata inspection, PDF downloading, and SWORD v2 (`hal+xml` / `zip`) deposit support.
- **ORCID** (`orcid`) — Public and authenticated v3.0 API support for searching researchers, browsing author profiles, inspecting works, adding new work metadata records, and automatically resolving external DOIs/arXiv IDs to downloadable PDFs.

## Key Features

- **Unified Multi-Database Interface**: Use the same commands (`database`, `list`, `get`, `search`, `download`, `send`) across `arxiv`, `zenodo`, `hal`, and `orcid`.
- **Interactive `fzf` Pagination & Actions**: Browse search results and profile entries interactively with in-menu pagination (`..` for previous page, `...` for next page), press `Enter` to inspect full abstract/metadata, or press `Ctrl-D` to immediately download the PDF.
- **Cross-Repository PDF Resolution**: When downloading a record from ORCID or HAL that references an arXiv ID or external open-access PDF, `preprint` automatically resolves and fetches the PDF.
- **Persistent Credential Management**: Store profile IDs, API tokens, and sandbox settings cleanly in `$OMNIRC` (`PREPRINT_CONF`) or provide them via environment variables.

## Prerequisites

- `curl`, `awk`, `sort`, `tar`
- `fzf` (optional, recommended for interactive `search`, `get`, and `download` menus)

---

# Install

## Via the Official `omni` Provider

```bash
omni providers install omni
omni tools use preprint --provider omni
```

## From a Local Directory

```bash
omni tools use -p /path/to/omni/preprint
```

## Directly from GitHub

```bash
omni tools use -u https://github.com/omnishell/preprint.git
```

---

# Usage

```bash
preprint <database|list|get|search|download|send> [options]
```

## 1. Database & Credential Configuration (`preprint database`)

Configure profile identifiers, API tokens, or credentials for `arxiv`, `zenodo`, `hal`, and `orcid`. Settings are persisted in `$OMNIRC` (`PREPRINT_CONF`).

```bash
# View configuration status for all supported databases
preprint database ls
preprint db ls

# Configure arXiv profile / SWORD credentials
preprint database set arxiv --profile <author_id> [--orcid <orcid_id>] [--user <user>] [--pass <pass>]

# Configure Zenodo access token and sandbox mode
preprint database set zenodo --token <zenodo_access_token> [--sandbox|--no-sandbox]

# Configure HAL author ID (idHal) and credentials
preprint database set hal --profile <idhal_or_name> [--orcid <orcid_id>] [--user <user>] [--pass <pass>]

# Configure ORCID iD and optional OAuth access token (for writing works)
preprint database set orcid --profile <0000-0000-0000-0000> [--token <orcid_token>] [--sandbox|--no-sandbox]

# Remove stored configuration for a database
preprint database rm <arxiv|zenodo|hal|orcid>
```

### Supported Environment Variable Overrides

You can also supply credentials via environment variables:

- **arXiv**: `ARXIV_PROFILE`, `ARXIV_ORCID`, `ARXIV_USER`, `ARXIV_PASS`, `ARXIV_TOKEN`
- **Zenodo**: `ZENODO_TOKEN`, `ZENODO_SANDBOX=1`
- **HAL**: `HAL_PROFILE`, `HAL_ORCID`, `HAL_USER`, `HAL_PASS`, `HAL_TOKEN`
- **ORCID**: `ORCID_ID` (or `ORCID_PROFILE`), `ORCID_TOKEN`, `ORCID_SANDBOX=1`

---

## 2. Listing Your Publications (`preprint list`)

List publications associated with your configured profile/account in a database:

```bash
preprint list [entries] <arxiv|zenodo|hal|orcid> \
  [-s|--sort <time|alpha>] \
  [-r|--reverse] \
  [-n|--max <N>]
```

**Examples:**

```bash
# List your latest 20 arXiv preprints sorted by date
preprint ls arxiv -s time -n 20

# List your Zenodo depositions in alphabetical order
preprint ls zenodo -s alpha

# List works on your configured ORCID profile
preprint ls orcid
```

---

## 3. Searching Databases (`preprint search`)

Query `arxiv`, `zenodo`, `hal`, or `orcid` by keyword, title, author, or identifier:

```bash
preprint search <query> -d|--database <arxiv|zenodo|hal|orcid> \
  [-s|--sort <time|alpha>] \
  [-r|--reverse] \
  [-n|--max <page_size>] \
  [-D|--download] \
  [-o|--output <path>] \
  [-l|--no-fzf]
```

- By default, results open in an interactive `fzf` viewer:
  - Select `... (Next Page)` or `.. (Prev Page)` to paginate through remote API results.
  - Press `Enter` on an entry to view detailed metadata (and optionally download its PDF, or browse an author's works when searching `orcid`).
  - Press `Ctrl-D` on an entry to immediately download its PDF.
- Pass `-l` / `--no-fzf` for plain non-interactive terminal output.
- Pass `-D` / `--download` to proceed directly to PDF download upon selection.

**Examples:**

```bash
# Interactive search on arXiv with 25 results per page
preprint search "quantum error correction" -d arxiv -n 25

# Non-interactive search on HAL
preprint search "homomorphic encryption" -d hal --no-fzf

# Search researchers on ORCID and browse their publications interactively
preprint search "Ada Lovelace" -d orcid
```

---

## 4. Inspecting Entry Details (`preprint get`)

Fetch and display complete metadata (title, authors, publication date, DOI, abstract, PDF link) for a specific record ID, or pick interactively from your profile entries if `<id>` is omitted:

```bash
preprint get [<id>] -d|--database <arxiv|zenodo|hal|orcid> \
  [-D|--download] \
  [-o|--output <path>] \
  [-s|--sort <time|alpha>] \
  [-r|--reverse] \
  [-n|--max <N>]
```

**Examples:**

```bash
# Inspect a specific arXiv preprint and prompt to download
preprint get 2301.00001 -d arxiv -D

# Interactively pick one of your Zenodo records to inspect
preprint get -d zenodo
```

---

## 5. Downloading PDFs (`preprint download`)

Download the PDF for a specific entry ID (or choose interactively from your configured profile if `<id>` is omitted):

```bash
preprint download [<id>] -d|--database <arxiv|zenodo|hal|orcid> [-o|--output <file_or_dir>]
preprint dl [<id>] -d|--database <arxiv|zenodo|hal|orcid> [-o|--output <file_or_dir>]
```

**Examples:**

```bash
# Download an arXiv PDF to ./papers/
preprint dl 2301.00001 -d arxiv -o ./papers/

# Download a HAL record PDF
preprint download hal-01234567 -d hal
```

---

## 6. Submitting & Publishing Works (`preprint send`)

Upload a manuscript/archive or register a publication metadata record (`send`, `push`, or `publish`):

```bash
preprint send [<file>] -d|--database <arxiv|zenodo|hal|orcid> \
  [-m|--meta <meta_file>] \
  [-t|--title <title>] \
  [-a|--author "Last, First; Last2, First2"] \
  [-A|--abstract <abstract_text>] \
  [-c|--category <category>] \
  [--doi <doi>] \
  [--arxiv <arxiv_id>] \
  [--hal <hal_id>] \
  [--url <url>] \
  [--journal <journal_title>] \
  [--type <work_type>] \
  [--date <YYYY-MM-DD>] \
  [--publish]
```

### Metadata File Format (`-m|--meta`)

Instead of passing every flag on the command line, you can provide a `key: value` metadata file:

```yaml
title: Optimal Bounds for Distributed Consensus
authors: Doe, Jane; Smith, John
abstract: We present a tight lower bound...
category: cs.DC
doi: 10.5281/zenodo.1234567
date: 2025-05-01
```

**Examples:**

```bash
# Upload a draft PDF and metadata to Zenodo (creates draft deposition; add --publish to finalize)
preprint send paper.pdf -d zenodo -m paper.meta

# Add a new publication work record to your ORCID profile
preprint send -d orcid \
  --title "Optimal Bounds for Distributed Consensus" \
  --type "preprint" \
  --arxiv "2505.00001" \
  --date "2025-05-01"
```
