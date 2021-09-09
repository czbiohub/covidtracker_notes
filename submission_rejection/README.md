## Notes on how to deal with rejected sequences:

- [When the genome submissions don't get accepted immediately, what is expected of us.](#when-some-submitted-genomes-dont-get-accepted-immediately) 

- [How to fix the frameshift errors and when not to, with some interesting examples.](#types-of-frameshifts-and-when-to-fix-them)

- [Some interesting observations for the curious minds.](#some-interesting-observations)


<br>

## When some submitted genomes don't get accepted immediately

Receiving error messages from GISAID or GenBank doesn't neccessarily mean the genome assembly is wrong and needs fix. It means they need a quality check which not always lead to edits of the genomic sequence.

### GISAID 

GISAID by default will return sequences with frameshifts to the submitter, and below is direct quote from GISAID team (Jun 2021 email):

```
Your sequences with frameshift detection have not been rejected... The curation team is just waiting for the submitter's confirmation, waiting for three possibilities:

1. The Submitter notes that the frameshift was due to a bioinformatic error and then submit the sequences again, now without frameshift detection. The sequences are released.

2. The Submitter notes that the frameshift is real, and then resubmit the sequences notifying that the frameshift is real. The sequences will be released with a frameshift verification comment by the Submitter

3. The Submitter has no way to confirm or reject the presence of the frameshift. The Submitter resubmits the sequences and advises us that they cannot verify them. Sequences are released with a non-verification comment from the Submitter.
```

GISAID's approach is very reasonable: to alert people of potential errors, yet provide the option to submit the genomes as is without any additional work to check or fix anything. The problem is this information was not made clear anywhere and everyone felt burdened to "fix" their "rejected genomes".

Now GISAID added an option in the batch upload process so people can choose between a few options: 

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/gisaid_options.png" width="1000"> 

The ideal workflow is to select the middle option in the first submission, take all rejected sequences with frameshifts and then go through them to check, fix assembly errors, and resubmit with the last option (which corresponds to point 1-2 above). 

Alternatively for busy people, one can also upload and add a message with this button <img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/gisaid_contact.png" width="100"> to note that the frameshifts are not verified and ask GISAID to accept the data as is (corresponds to point 3 above). We believe the "verfied frameshift" and "non-verified frameshift" are distinguished by this icon on GISAID search table <img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/gisaid_mark.png" width="30"> ... except we're not sure which is which XD 

Behind the scene, GISAID runs the submitted genome through [CoVsurver](https://www.gisaid.org/epiflu-applications/covsurver-mutations-app/) and by default will not pass sequences with the word `FRAMESHIFT` in the comment column in the error message. Sometimes GISAID will send that message together with rejection to the submitter. But when they don't, it is the `query summary report` file found on the very bottom of the CoVsurver result page. If this tool gets busy and hangs, try a different time of the day (afternoon is better). Sometimes the comment looks like `Insertion of 11 nucleotide(s) found at refpos 27850 (FRAMESHIFT). NS7b without BLAST coverage. Stretch of NNNs.` but the only part that matters for submission is the frameshift. Knowing the number of base pairs of insertion or deletions that's causing the frameshift, and the genomic position of the frameshift is critical in evaluating and fixing them.

<br>

### GenBank

GenBank runs [VADR](https://github.com/ncbi/vadr) and will detect many [potential problems](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#top) the genomes may have, and if any of those errors land within the essential genes, they will not let the sequence in. Installing and running VADR locally can be a task of its own, so we usually do a 1st submission to GenBank while checking this box <img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/genbank_box.png" width="450"> 
where they will not report errors, and collect the genomes that were removed and do a 2nd submission with that box unchecked to obtain the VADR error message `detailed-error-report.tsv`. 

There are a lot of [errors types](https://www.ncbi.nlm.nih.gov/genbank/sequencecheck/virus/) and some of the errors can be connected or originate from the same sequence problem, such as `CDS_HAS_FRAMESHIFT` can lead to `CDS_HAS_STOP_CODON` and `INDEFINITE_ANNOTATION_END` and `UNEXPECTED_LENGTH`. The goal here is not to "fix" the genome until there is no more VADR errors. The goal is to fix the assembly errors and leave whatever that is correctly-assembled and well-supported by reads as is, and convince GenBank staff that you have done the due diligence so they will accept the genomes. That being said, we have not succeeded in sending our rejected genomes in so this section is to be completed...

<br>

## Types of frameshifts and when to fix them

For our 10,000+ genomes, GISAID and GenBank returned 800+ genomes with ~960 frameshifts (one genome can have multiple frameshifts). Here is a breakdown of different types of frameshifts. A collection of snapshots for all of them (removing duplicate cases) can be found in the subfolders.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/type_count.png" width="500">

4 items are needed to inspect the frameshifts: the genome itself (the FASTA file), the genomic position of the frameshift so we know where to look (in the error message), the number of bases pairs of the insertion or deletion that is causing the frameshift (in the error message), the read alignment BAM files that the genome was assembled based off, and a genomic browser such as [IGV](https://igv.org/) to open the BAM files (whoa their new IGV site looks so nice that is distracting for counting properly). Refresher for different related files types see [here](https://github.com/czbiohub/covidtracker_notes/blob/main/bioinformatics/file_types.md).

Note that Biohub only have **Illumina short read data** from metagenomic or ARTIC v3 libraries. Mis-assembly around deletions seems a lot more common for Nanopore data but we do not have enough experience to write about it. All our genomes were aligned with [minimap2 2.17](https://github.com/lh3/minimap2) and assembled with [iVar 1.2](https://github.com/andersen-lab/ivar). Full pipeline is [here](https://github.com/czbiohub/sc2-illumina-pipeline).

<br>

### Where did the `N` come from?

A fasta file with a linear genomic sequence has its limitations representing a mixture of more than 1 type of genomes. Naturally occurring intra-host variation, sample contaminations, sequencing errors can all lead to more than 1 type of genomes in the sample. [IUPAC code](https://www.bioinformatics.org/sms2/iupac.html) can represent some of the diversity in nucleotide composition. For example, If a genomic position is `A` in some reads and `C` in others, this position can be represented as `M` (IUPAC code for `A` or `C`). However if a position has `A` in some reads, and is deleted in others, there is no corresponding IUPAC code and iVar will resort to put an `N` in that position for lack of a better choice. Almost all the editing done to correct our genome assembly errors was to delete these Ns.

<br>

### If there is mis-assembly of the genome based on aligned sequencing reads, the genome needs a fix.

With our Illumina data, most of the assembly error look like this example below. The error message said it had a 1bp deletion that led to frameshift. If you look at the read alignment, it is actually a 3bp deletion unambiguously representated by the reads containing these 3bp deletions. However, there are reads that align into the region of deletion such as the first read. This read ends with nucleotide `TA` which are the same as the first 2bp of the deletion, which are also the same as the nucleotides flanking the deletion (indicated by 2 blue arrows), so most likely this read contains the deletion just like the other reads. However when the read-aligner mapped this read to the reference genome, it can map it either as not having the deletion (as shown in the snapshot below), or map it as having the deletion, and they will be equally good alignment. In this case, the aligner would always favor not to have the deletion to avoid the gap-penalty of alignment. As a result, this read was mapped as not having the deletion, and instead extend into the deletion. 

Then this alignment was given to iVar to assemble the genome. From iVar's perspective, in the first 2 positions of the 3bp deletion, some reads have a deletion, some reads don't (such as the first read), and therefore iVar would fill those 2 positions as `N`, and leave only 1bp of deletion. That is where this 1bp frameshift error comes from. To fix this assembly error in the consensus genome, we deleted the 2 Ns that falls within the deletion to make it a 3bp deletion.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/mis1.png" width="600">

Here is another example. 

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/mis2.png" width="600">

In a rarely seen but related scenario below, the 4bp in the middle is `TTCA` in the refrence genome, and in the sample genome the sequence is `GTTC`. When the reads were aligned, they could either be aligned as having mutations in 3 positions, or a combination of an insertion and a deletion. When such a mixed alignment type were presented to iVar, iVar combined these 2 types of alignment and called the position of insertion `N` as explained above, followed by `K` (`T` or `G`), `T`, `Y` (`C` or `T`), and the deletion as `N`. That is 1bp longer than the reference genome and therefore a frameshift. To fix this assembly error, we replaced `NKTYN` with `GTTC` and the frameshift went away.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/mis3.png" width="500">

Another example:

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/mis4.png" width="500">

<br>

### If the genome assembly is correct, there is nothing to fix. Regardless of whether frameshift or not, do not edit anything.

Whether there is assembly error is independent of whether a real frameshift exist or not. Our goal is to fix all assembly errors and after that leave the rest alone regardless of frameshifts. The key to tell whether the genome assembly is correct, is whether the number of base pair of deletion or insertion as flagged by the error message match the reads. Below matches the error message of "FRAMESHIFT of 8bp" so it needs no edits. 

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/real1.png" width="400">

Another example of real frameshift and correct genome assemlby. If you notice that in the very middle there is a read that looks just like the previous mis-assembly section. The reason iVar didn't put an `N` there, was because the fraction of such reads are too small to count. On the other hand, if iVar had indeed put an `N` there, this wouldn't be identified as a frameshift because there will be no deletion.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/real2.png" width="300">

<br>

### If it is a random 1bp insertion, can go either way. For our sanity, we did not fix them.

This is an interesting type of frameshift. It is a 1bp insertion that lands at seemingly random positions across the genome, most often precedes a short stretch of `A` or `T` (only 11/260 instances precede `C` or `G`), and is usually present in some of the reads but not all of them, which leads to a representation by `N` in the assembled genome (see [discussion here](https://github.com/andersen-lab/ivar/issues/45)). 

For the purpose of submission, because the 1bp insertion is well-supported by reads, we usually did't remove them from the genome. Someone someday may find this feature interesting and do some study with it (or yes, our submitter is simply too lazy).

A few examples:

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/N1bp1.png" width="400">

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/N1bp2.png" width="400">

Some rare cases:

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/N1bp3.png" width="400">

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/N1bp4.png" width="400">


<br>


## Some interesting observations

Our samples were collected between Mar 2020 and May 2021.

### Where on the genome the frameshifts tend to appear

The rationale behind flagging frameshifts as potential errors is that frameshifts are usually detrimental becuase they change the amino acid sequences. However in general, COVID (and all organisms) has some amount of tolerance to that. For example, some genes are not that critical, for example ORF8 can tolerate a large amount of deletion [REF](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7577707/)), frameshifts near the end of genes matters less, and if there is more than one viral genotype in a host, their proteins can compensate for each other. 

Below are frameshift positions we saw in more than 10 sample genomes. These numbers carry the caveat that we often sequenced outbreak samples that have identical or highly related genomes and are likely to share the same frameshifts.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/frameshift_count.png" width="300">

Below shows where frameshifts tend to appear on the genome. It counts, in each 50bp bin, how many **unique** frameshift positions there are (not counting the same frameshifts seen in different samples).

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/frameshift_plot.png" width="1000">

<br>
  
### Where on the genome the random 1bp insertion tend to appear

We are quite unsure about the origin of those random 1bp insertions. Given it is an insertion instead of point mutation, it seems unlikely to be introduced by PCR processs; given the frequent occurrance on so many reads, it's extremely unlikely to be sequencing error. It may have something to do with the virus replication? If anyone knows, please let us know!

Below are 1bp insertion positions we saw in more than 2 sample genomes. They showed up much less frequently in the same position as real frameshifts above. However we certainly did not capture all instances because they show up only in some of the reads, and if the percentage of reads is too low, iVar will not include it in the assembled consensus genomes.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/N1bp_count.png" width="300">

Below shows where these random 1bp insertions tend to appear on the genome. It counts, in each 50bp bin, how many **unique** insertion positions there are (not counting the same insertion seen in different samples).

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/documentation_images/N1bp_plot.png" width="1000">

<br>
  
### Please leave comments and questions in the [issues](https://github.com/czbiohub/covidtracker_notes/issues) section, or create a PR to edit!
