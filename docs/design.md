# Design - Variant Triage Service

Write this BEFORE you write the code, then keep it up to date. It is marked by the lecturer.
Use your own words, your own thresholds and your own data. Delete the guidance in italics.

Group ID: <your group id>      Thresholds from data/params.json: min_depth = <...>, min_quality = <...>, top_k = <...>

## 1. Decomposition

| Question | Our answer |
|---|---|
| Input | |
| Output | |
| Constraints (named thresholds, what happens at exactly the threshold) | |
| Rule order and why the order matters | |
| Out of scope | |

## 2. Pseudocode or flowchart of classify_variant

*Give either structured pseudocode (indentation, IF / ELSE, RETURN) or a flowchart (a picture file
committed in docs/ and linked here, or a Mermaid block). Show all six rules.*

## 3. Hand trace

*Trace the first six rows of YOUR data/variants.csv (v1 to v6) with YOUR thresholds, then two more rows of
your choice that reach rules 3 to 6. Do it by hand BEFORE you run your code, then compare with your code.*

| Variant | Gene | Depth | Quality | Impact | Rule that fires | Decision | Reason |
|---|---|---|---|---|---|---|---|
| v1 | | | | | | | |

## 4. Test plan

Our thresholds: min_depth=8, min_quality=80. Boundaries are inclusive (equal passes).

| # | Type | Function | Input | Expected |
|---|---|---|---|---|
| 1 | Boundary | classify_variant | depth 7 (one below min_depth) | exclude, reason mentions depth 7 and 8 |
| 2 | Boundary | classify_variant | depth 8 (exactly min_depth) | passes the depth rule |
| 3 | Boundary | classify_variant | depth 9 (one above) | passes the depth rule |
| 4 | Boundary | classify_variant | quality 79 (one below min_quality) | exclude, reason mentions quality 79 and 80 |
| 5 | Boundary | classify_variant | quality 80 (exactly min_quality) | passes the quality rule |
| 6 | Boundary | classify_variant | quality 81 (one above) | passes the quality rule |
| 7 | Normal | classify_variant | priority gene + high, non-priority + low, etc. | rules 3 to 6 give prioritize / review / review / exclude |
| 8 | Rule order | classify_variant | depth 1 and quality 1 | exclude for low depth (rule 1 wins) |
| 9 | Awkward | classify_variant | gene ' egfr ', impact 'HIGH' | matches EGFR and high after strip() and casefold() |
| 10 | Normal | triage | two rows | new dicts, same order, input rows unchanged |
| 11 | Normal | decision_counts | 1 prioritize, 2 review, 1 exclude | {prioritize: 1, review: 2, exclude: 1} |
| 12 | Awkward | decision_counts | empty list | all three keys present with 0, in fixed order |
| 13 | Normal | impact_counts | mix with some excluded | excluded variants are not counted |
| 14 | Awkward | impact_counts | impact ' HIGH ' / 'Moderate' | counted as high / moderate; keys in fixed order |
| 15 | Awkward | impact_counts | empty list | all four keys with 0 |
| 16 | Normal | gene_summary | genes PTEN, CFTR, KRAS | keys in alphabetical order CFTR, KRAS, PTEN |
| 17 | Normal | gene_summary | EGFR depths 7, 8, 8 (one excluded) | counts 1/1/1, mean_depth 7.7 (excluded included) |
| 18 | Awkward | gene_summary | gene ' KRAS ', depth given as int | merged under KRAS, mean_depth 9.5 |
| 19 | Awkward | gene_summary | empty list | {} |
| 20 | Normal | rank_variants | mixed results, k = 3 | at most 3, no excluded, sorted with tie-breaks |
| 21 | Awkward | rank_variants | k = 0 and k larger than the data | [] and all candidates |
| 22 | Invalid | rank_variants | k = -1 | raises ValueError (pytest.raises) |
| 23 | Normal | results_to_csv / build_report | small results list | exact header line and report line formats |
| 24 | Side effects | all summary functions | any results | input list is not modified |

**How dictionaries are used for counting and summarising.** decision_counts and impact_counts
first create a dictionary with every allowed key set to 0 (`{key: 0 for key in DECISIONS}`), so the
keys are always present, in a fixed order, even when the count is zero. Each result then adds 1 to
the key for its (cleaned) label. Both functions share one helper, count_labels, so the counting
code is not copied. gene_summary first groups results in a dictionary of lists
(`gene -> [results]`, built with setdefault), then loops over `sorted(...)` gene names so the
output dictionary is in alphabetical order (dicts keep insertion order). Each gene's value is
itself a dictionary: total, the decision counts (reusing decision_counts) and mean_depth, which
is the mean of int(depth) over ALL the gene's variants (excluded ones too), rounded to 1 decimal.

## 5. Work plan

*Who builds which function, in which order, and how you will review each other's code.*
