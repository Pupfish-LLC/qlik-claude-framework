---
name: qlik-load-script
description: Script syntax reference, QVD optimization, incremental load patterns (insert-only, insert/update, insert/update/delete, dual-timestamp for SCD2), master calendar generation, variable definitions, error handling, logging patterns, null handling patterns, diagnostic and validation patterns, subroutine integration, and platform gotchas (SET vs LET, dollar-sign expansion timing, SET variable comma limitation). Load when writing or reviewing Qlik load scripts.
---

<!-- TODO: Populate with script syntax patterns, incremental load patterns, null handling (vCleanNull, NullAsValue, string-encoded null cleaning), diagnostic patterns (TRACE templates, validation queries), subroutine integration, and platform edge cases. -->

For the complete incremental load pattern catalog including dual-timestamp SCD2 patterns, read `incremental-load-patterns.md` in this skill directory.
For null handling patterns (vCleanNull, NullAsValue, string-encoded null cleaning), read `null-handling-patterns.md` in this skill directory.
For diagnostic and validation patterns (TRACE templates, post-load queries), read `diagnostic-patterns.md` in this skill directory.
For reusable script templates, see the `script-templates/` subdirectory.
