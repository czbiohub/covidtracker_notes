## Snapshots of the genomic position that were flagged by GISAID or GenBank as frameshifts.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/type_count_b.png" width="300">

- `correct_assembly_real_frameshift` contains positions that were assembled correctly and contain real frameshifts. They needed no fix or edits.

- `erroneous_assembly_not_frameshift_after_fix` contains positions that had assembly error that needed fix. After fixing assembly errors, there were no more frameshifts.

- `erroneous_assembly_real_frameshift_after_fix` contains positions that had assembly error that needed fix. After fixing assembly errors, there were still frameshifts.

- `random_1bp_insertion` contains positions that had 1bp insertion that were denoted as `N` in the consensus genome. We did not edit them for re-submission.

- Filenames contain genomic positions followed by the ID for GISAID or GenBank. 

- The genomic position of interest is at 20bp from the left edge of the snapshots.

- Not all reads were captured in the snapshots. 
