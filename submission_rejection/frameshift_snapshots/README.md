## Snapshots of the genomic position that were flagged by GISAID or GenBank as frameshifts.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/type_count_b.png" width="300">

- `correct_assembly_real_frameshift` contains positions that were assembled correctly and contain real frameshifts. They needed no fix or edits.

- `erroneous_assembly` contains positions that had assembly error. they may or may not contain real framesshifts. They all needed fix.

- `random_1bp_insertion` contains positions that had 1bp insertion that were denoted as `N` in the consensus genome. We did not edit them for re-submission.
