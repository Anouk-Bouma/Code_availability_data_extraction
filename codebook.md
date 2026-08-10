# Codebook — Code Availability Data Extraction

Documents the variables produced by [`data_extraction.qmd`](data_extraction.qmd). Column names, source fields, and value codes below are taken directly from `recode_spec`, `shift_pilot_numbering`, and the raw fields observed in `backup_CODER01_2026-08-06.json`, `backup_Coder03_2026-07-30.json`, and `backup_Coder04_2026-07-27.json`.

**Question wording marked "*(inferred — please confirm)*" is a guess from the column name only** — this codebook was built from the code and data, not the original coding instrument, so those descriptions need a pass from whoever wrote the coding scheme.

Every coded variable appears in `combined` as `{variable}_{coderNN}` (one column per coder, e.g. `q1_supp_coder01`) plus `{variable}_agreement` (logical: `TRUE` if all coders who rated it agree, `FALSE` if they disagree, `NA` if fewer than 2 coders rated it). In `long_df` the same data is reshaped to one row per `paper_id` × `variable`, with columns `coder01`, `coder03`, `coder04`, `agreement`.

## ID and metadata columns

| Column | Role | Notes |
|---|---|---|
| `paper_id` | Join key (`id_col`) | Links records across coder files; never suffixed or excluded |
| `paper_title`, `meta_id`, `meta_year`, `meta_authors`, `meta_title`, `meta_journal`, `meta_doi`, `meta_reproducer` | Dropped (`meta_exclude`) | Metadata carried in the raw files but not needed in the combined comparison table |
| `coder_id` | Kept, excluded from agreement (`compare_exclude`) | Identifies which coder submitted the record |
| `meta_date` | Kept, excluded from agreement (`compare_exclude`) | Coding submission date |
| `_complete` | Kept, excluded from agreement (`compare_exclude`) | Coder-form completion flag |

## Recoded variables (from `recode_spec`)

These are collapsed from multiple raw dummy columns, or remapped from raw text, **inside `load_coder_data`** before the coder-file suffix is added. Raw source columns listed are their names *after* the pilot renumbering shift (see [Pilot renumbering map](#pilot-renumbering-map-raw--final) below) — this is the form `recode_spec` actually matches against.

| Final variable | Likely meaning | Type | Raw source column(s) | Value codes |
|---|---|---|---|---|
| `q1_supp` | Supplementary materials provided? *(inferred — please confirm)* | onehot | `q1_yes`, `q1_no` | `1` = yes, `0` = no |
| `q2_code` | Code shared/available? *(inferred — please confirm)* | onehot | `q2_yes`, `q2_no` | `1` = yes, `0` = no |
| `q3_rm` | README present? *("rm" = readme; inferred — please confirm)* | onehot | `q_readme_yes`, `q_readme_no`, `q_readme_unsure` | `1` = yes, `0` = no, `2` = unsure |
| `q4_sufficient` | Materials sufficient to reproduce? *(inferred — please confirm; `ps`/`ci`/`un` sub-labels unclear, please define)* | onehot | `q4_ps`, `q4_ci`, `q4_un` | `1` = `ps`, `0` = `ci`, `2` = `un` |
| `q_itr_main` | Main results reported in-text? *("itr" = in-text reporting; inferred — please confirm)* | onehot | `q_itr_main_yes`, `q_itr_main_no` | `1` = yes, `0` = no |
| `q_itr_supp` | Supplementary results reported in-text? *(inferred — please confirm)* | onehot | `q_itr_supp_yes`, `q_itr_supp_no` | `1` = yes, `0` = no |
| `q5_source` | Source of shared materials *(inferred — please confirm)* | value | `q5_source_0` (text) | `1` = "External Repository", `2` = "Journal website" |

A row gets `NA` for a onehot variable when none or more than one of its source dummies was coded `1` (including when the question didn't apply / wasn't reached for that paper).

## Other variables retained as-is (post pilot-rename)

Not part of `recode_spec` — these pass through unchanged (aside from the numbering shift) as individual columns.

**README free text**
- `q_readme_notes` — free-text notes on the README question (not shifted, no `q3`–`q6` in the name)

**Sufficiency / source free text** *(raw names shown pre-shift → final)*
- `q4_link_0` → `q5_link_0` — link associated with the materials-source question
- `q4_notes_0` → `q5_notes_0` — free-text notes on the materials-source question

**Language/tool used** *(one dummy per language; raw `q5_*` → final `q6_*`)*
`q6_r`, `q6_python`, `q6_matlab`, `q6_stata`, `q6_julia`, `q6_spss`, `q6_mplus`, `q6_c_cpp`, `q6_shell`, `q6_unclear`, `q6_other` (+ `q6_other_text` free text), `q6_notes` free text

**Repository/link status** *(inferred — please confirm; raw `q6_*` → final `q7_*`)*
`q7_broken`, `q7_empty`, `q7_not_mentioned`, `q7_request`, `q7_restricted`, `q7_other` (+ `q7_notes` free text)

**In-text reporting detail** (not renumbered — no `q3`–`q6` digit in the name)
- `itr_value`, `itr_page`, `itr_table_num`, `itr_supp_link` — free text

**General**
- `general_notes` — free-text coder notes, not tied to a specific question

## Pilot renumbering map (raw → final)

Applied to every raw column name except `paper_id`, when `apply_pilot_rename` is `TRUE` (default). Only affects names containing `q3`, `q4`, `q5`, or `q6`; each name is shifted once (`q6→q7`, `q5→q6`, `q4→q5`, `q3→q4`, applied in that order so a name is never re-matched by an earlier rule).

| Raw prefix | Final prefix |
|---|---|
| `q3_*` | `q4_*` |
| `q4_*` | `q5_*` |
| `q5_*` | `q6_*` |
| `q6_*` | `q7_*` |

Set `apply_pilot_rename <- FALSE` in the "Column identifications" chunk for a batch whose raw files already use the corrected numbering.

## Derived / bookkeeping columns

| Column pattern | Meaning |
|---|---|
| `{variable}_agreement` | `TRUE`/`FALSE`/`NA` — do the coders who rated `{variable}` for this paper agree? (`add_agreement_cols`) |
| `resolution`, `notes` (in `review_table` only) | Blank columns for manually recording how a disagreement was resolved |
| `alpha`, `n_cases`, `n_raters` (in `agreement_stats` only) | Krippendorff's alpha (nominal) per variable, and the number of papers/coders it was computed over |
