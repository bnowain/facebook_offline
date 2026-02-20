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
6. If modifying or removing an API endpoint that Atlas may depend on, **stop and warn** before proceeding.
7. New endpoints added for Atlas integration should be general-purpose and useful standalone, not tightly coupled to Atlas internals.
8. **Privacy**: This archive contains personal data. Never expose its content through any public-facing endpoint. All queries must remain local.

**Spoke projects** (sibling directories, may be loaded via `--add-dir` for reference):

- **civic_media** — meeting transcription, diarization, voiceprint learning
- **article-tracker** — local news aggregation and monitoring
- **Shasta-DB** — civic media archive browser and metadata editor (FastAPI/HTMX)
- **Facebook-Offline** — local personal Facebook archive for LLM querying (this project, private, local only)
