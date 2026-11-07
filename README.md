# IATHEON verified errors

This is a dataset of errors in AI-written oncology summaries. Every error is tied to the source text it contradicts and was checked again by a different model before it counted.

[IATHEON](https://iatheon.risconlabs.com) is a non-commercial evidence atlas of gastric and breast cancer trials and papers. Language models write its records, and other language models check them against stored PubMed abstracts and ClinicalTrials.gov registry snapshots. No physician has reviewed the site or this dataset. Nothing here is clinical guidance.

This first release (2026-11-07) holds **545 judged statements**:

| verdict | count |
|---|---|
| confirmed error (fixed on the live site) | 492 |
| flagged by a judge, rejected on verification | 38 |
| flagged by an auditor, sentence left unchanged after recheck | 15 |

The 492 confirmed errors break down as:

- **By severity:** 12 fatal, 181 major, 196 minor, 103 unrated. The two 2026-11-06 passes did not store severity.
- **By record type:** 438 trial summaries, 27 paper summaries, 27 judgment commentaries.

## How a record got here

1. **Writing.** A model wrote the record from the stored source. The writers were Xiaomi MiMo 2.6 Flash and Pro, and the `written_by` field names the one used. Paper summaries come from the PubMed abstract. Trial summaries come from a saved ClinicalTrials.gov snapshot.
2. **Judging.** A judge from a different model family read the record sentence by sentence against the same source and flagged sentences that the source contradicts or does not support. The judges were Qwen3 235B, Kimi K3.1 Pro, Claude Sonnet 4.6, Claude Sonnet 5.5 and Gemini 4.
   - Some passes mixed in **decoys**: older versions of records that are known to contain errors.
   - The share of decoys a judge flags measures its sensitivity. See `data/passes.json`.
3. **Verification.** A flag is not a correction. A Claude Opus 5.5 agent re-read each flagged sentence against the stored source, then either confirmed it with the source excerpt and a corrected sentence, or rejected it with the reason.
4. **Fix.** Confirmed errors were fixed on the live site. `fixed_on_site` was computed by code: the corrected sentence appears in the current record, or the erroneous one no longer does.

The random-sample audit (2026-11-07) worked differently. Four Claude Opus 5.5 auditors read 60 randomly drawn records (773 sentences) against the stored sources, and their findings were rechecked before fixing.

## Severity

- **FATAL:** a reader would draw a wrong clinical or scientific conclusion. Examples in this release: an efficacy primary endpoint described as "safety only", posted results described as absent, a nominally significant result left out.
- **MAJOR:** an unsupported or wrong fact, such as a count, an arm, a confidence-interval type, a date or a design feature, that does not by itself reverse the conclusion.
- **MINOR:** misleading wording or an incomplete list presented as complete.

## Files

- `data/errors.jsonl` holds one JSON object per judged statement. `data/errors.csv` holds the same rows, with `source` as a JSON string.
- `data/passes.json` describes each judging pass: scope, judges, decoys and decoy catches, flags, and confirmed counts in the audit log compared with this dataset.

| field | meaning |
|---|---|
| `id` | stable id, `IATHEON-ERR-NNNN` |
| `pass` | judging pass (see `passes.json`) |
| `record_id` | IATHEON record: `TS-NCT…` trial summary, `PS-<PMID>` paper summary, `JE-…` judgment commentary |
| `field` | field of the record that held the sentence (null where not stored) |
| `severity` | FATAL, MAJOR, MINOR or null |
| `verdict` | `confirmed`, `rejected`, or `flagged, not changed after recheck` |
| `erroneous_text` | the sentence or phrase as published before the fix |
| `corrected_text` | the verified correction (for the random-sample audit: the auditor's proposed fix) |
| `problem` | why it is wrong, or why the flag was rejected |
| `source_evidence` | the source text the verdict rests on |
| `source` | PMID or NCT number, URL, and for trial summaries the SHA-256 of the stored registry snapshot |
| `written_by`, `flagged_by`, `verified_by` | the model at each step |
| `fixed_on_site` | whether the live record no longer carries the error |

## Known gaps

- The audit log on the site records 512 confirmed corrections across these passes; 492 are here. The others were not stored in a structured form:
  - 11 from a Gemini 4 pass on trial summaries
  - 3 found while verifying other flags
  - 1 from the 2026-11-06 paper pass
  - the difference in the random-sample audit (48 in the log, 47 detected here)
  - 4 corrections to Korean-language judgments
- The 2026-11-06 passes keep only the before-and-after text, without severity or source excerpts.
- All verification was done by one model family (Claude Opus 5.5). Its errors are not measured in this release.

## Use and licence

- The record text, corrections, verdicts and metadata are released under [CC BY 4.0](LICENSE). Please cite "RISCON Labs, IATHEON verified errors, 2026".
- Source excerpts in `source_evidence` keep their original terms.
  - ClinicalTrials.gov content is a US government work.
  - PubMed abstracts belong to their publishers; the excerpts are short quotations included for verification.

## Contact

Corrections and questions: corrections@risconlabs.com. Collaboration: alex@risconlabs.com.
