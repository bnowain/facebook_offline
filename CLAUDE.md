# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a **Facebook personal data archive** — an offline extract of account data downloaded from Meta's data export tool on 2026-02-10. It contains ~44,800 JSON files across two main areas: profile/account data and granular data logs. There is no application source code, build system, or tests.

## Repository Layout

```
Raw Data/                  # 11 original ZIP files (~26 GB total, do not modify)
DataArchives/facebook_full/  # Extracted and organized data
facebook_structure.csv       # Index of all 44,789 files with paths, sizes, timestamps
```

## Data Structure

All data under `DataArchives/facebook_full/` is JSON. The top-level categories are:

| Directory | Contents |
|---|---|
| `ads_information/` | Ad preferences, advertiser interactions, targeting categories |
| `apps_and_websites_off_of_facebook/` | Connected apps, off-Meta activity |
| `connections/` | Friends, followers, friend requests (sent/received/rejected/removed) |
| `data_logs/content/` | **Bulk of the archive** — 14 subcategories with paginated JSON files (e.g., `ads_impressions_12/`, `content_and_social_interactions_22/`) |
| `logged_information/` | Search history, location history |
| `personal_information/` | Profile fields (name, emails, birthday, gender, family) |
| `preferences/` | Privacy settings, notification preferences |
| `security_and_login_information/` | Login history, recognized devices |
| `your_facebook_activity/` | Posts, comments, reactions, messages, marketplace, groups, reels, stories, events, etc. |

## Working With This Data

- **Use `facebook_structure.csv`** to find files. Columns: `FullName`, `RelativePath`, `Name`, `Extension`, `Length`, `SizeMB`, `CreationTime`, `LastWriteTime`.
- **Data logs are paginated**: each subcategory under `data_logs/content/` contains numbered subdirectories (pages), each holding multiple JSON files.
- **JSON encoding**: Facebook exports use escaped Unicode for non-ASCII characters (e.g., `\u00e9` for é). Timestamps are typically Unix epoch seconds.
- **Large files**: Some JSON files (especially advertiser lists and activity logs) exceed 1 MB. The CSV index has size info to identify them.

## Privacy Notice

This archive contains **personally identifiable information** (real names, email addresses, phone numbers, location data, private messages). Do not expose, share, or transmit this data externally.

## Atlas Integration

This project is a spoke in the **Atlas** hub-and-spoke ecosystem. Atlas is a central orchestration hub that routes queries across spoke apps. It lives in a sibling directory (`E:\0-Automated-Apps\Atlas`).

**Rules:**

1. Only modify **this** project by default. Do not modify other spoke projects or Atlas unless explicitly asked.
2. If approved, changes to other projects are allowed — but always propose first and wait for approval.
3. Suggest API endpoint changes in other spokes if they would improve integration, but never write code in another project without explicit approval.
4. This app must remain **independently functional** — it works on its own without Atlas or any other spoke.
5. **No spoke-to-spoke dependencies.** All cross-app communication goes through Atlas.
   **Approved exceptions** (documented peer service calls):
   - `Shasta-PRA-Backup → civic_media POST /api/transcribe` — Transcription-as-a-Service
   New cross-spoke calls must be approved and added to this exception list.
6. If modifying or removing an API endpoint that Atlas may depend on, **stop and warn** before proceeding.
7. New endpoints added for Atlas integration should be general-purpose and useful standalone, not tightly coupled to Atlas internals.
8. **Privacy**: This archive contains personal data. Never expose its content through any public-facing endpoint. All queries must remain local.

**Spoke projects** (sibling directories, may be loaded via `--add-dir` for reference):

- **civic_media** — meeting transcription, diarization, voiceprint learning
- **article-tracker** — local news aggregation and monitoring
- **Shasta-DB** — civic media archive browser and metadata editor (FastAPI/HTMX)
- **Facebook-Offline** — local personal Facebook archive for LLM querying (this project, private, local only)
- **Shasta-PRA-Backup** — public records requests browser
- **Shasta-Campaign-Finance** — campaign finance disclosures from NetFile
- **Facebook-Monitor** — automated public Facebook page monitoring

### Lazy ChromaDB Sync (Atlas RAG)
Atlas maintains a centralized ChromaDB vector store. This project does NOT need its
own vector DB. Atlas fetches candidate records from this spoke's search API, chunks
deterministically, validates against ChromaDB cache, and embeds only new/stale chunks.
ChromaDB is a cache — this spoke's SQLite DB is the source of truth.

See: `Atlas/app/services/rag/deterministic_chunking.py` for this spoke's chunking strategy.

## Master Schema & Codex References

**`E:\0-Automated-Apps\MASTER_SCHEMA.md`** — Canonical cross-project database
schema and API contracts. **HARD RULE: If you add, remove, or modify any database
tables, columns, API endpoints, or response shapes, you MUST update the Master
Schema before finishing your task.** Do not skip this — other projects read it to
understand this project's data contracts.

**`E:\0-Automated-Apps\MASTER_PROJECT.md`** describes the overall ecosystem
architecture and how all projects interconnect.

> **HARD RULE — READ AND UPDATE THE CODEX**
>
> **`E:\0-Automated-Apps\master_codex.md`** is the living interoperability codex.
> 1. **READ it** at the start of any session that touches APIs, schemas, tools,
>    chunking, person models, search, or integration with other projects.
> 2. **UPDATE it** before finishing any task that changes cross-project behavior.
>    This includes: new/changed API endpoints, database schema changes, new tools
>    or tool modifications in Atlas, chunking strategy changes, person model changes,
>    new cross-spoke dependencies, or completing items from a project's outstanding work list.
> 3. **DO NOT skip this.** The codex is how projects stay in sync. If you change
>    something that another project depends on and don't update the codex, the next
>    agent working on that project will build on stale assumptions and break things.
