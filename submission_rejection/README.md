### Notes on how to deal with rejected sequences based on our submission experience to GISAID or GenBank. 

- [What it means when the genome submissions don't get accepted immediately and what is expected of us.](#what-it-means-when-some-submitted-genomes-dont-get-accepted-immediately) 

- [How to fix the frameshift errors and when not to with some interesting examples.](#types-of-frameshifts-and-when-to-fix-them)

- [Where the real frameshift mutations tend on appear the genome for the curious minds.](#some-interesting-observations)

**Receiving error messages from GISAID or GenBank doesn't neccessarily mean the genome assembly is wrong and needs fix.** It means they need a quality check which not always lead to edits of the genomic sequence. 

# What it means when some submitted genomes don't get accepted immediately

### GISAID 

Behind the scene, GISAID runs the submitted genome through [CoVsurver](https://www.gisaid.org/epiflu-applications/covsurver-mutations-app/) and by default will not pass sequences with `FRAMESHIFT` (more about this tool and obtaining the error message see [note 1](#note-1)). Sometimes the comment looks like `Insertion of 11 nucleotide(s) found at refpos 27850 (FRAMESHIFT). NS7b without BLAST coverage. Stretch of NNNs.` but the only part that matters for submission is the frameshift. Knowing the number of base pairs of insertion or deletions that's causing the frameshift, and the genomic position of the frameshift is critical in evaluating and fixing them.

GISAID return those sequences to submitters and direct quote from them "**your sequences with frameshift detection have not been rejected**, even if you want us to release the sequences as they are, you just have to let us know by email". 

The submitters can do one of the following:

1) Check the read alignment (bam file) surrounding the frameshift, if it is due to assembly error, fix it; if the assembly is supported by enough reads, re-submit as is.

2) If the frameshift is real, check this box below during submission. GISAID told me "The sequences will be released with a frameshift verification comment by the Submitter."

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/gisaid_box.png" width="800"> 

3) If the frameshift cannot be verified (such as no access to the read alignment bam file), resubmit as is but let GISAID know it wasn't verified. GISAID told me "Sequences are released with a non-verification comment from the Submitter."

I believe the "verfied frameshift" and "non-verified frameshift" are distinguished by this icon on GISAID search table <img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/gisaid_mark.png" width="30"> ... except I'm not sure which is which XD 


### GenBank

GenBank runs [VADR](https://github.com/ncbi/vadr) and will detect many [potential problems](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#top) the genomes may have, and if any of those errors land within the essential genes, they will not let the sequence in. Installing and running VADR locally can be a task of its own, so I usually do a 1st submission to GenBank while checking this box <img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/genbank_box.png" width="450"> 
where they will not report errors, and collect the genomes that were removed and do a 2nd submission with that box unchecked to obtain the VADR error message `detailed-error-report.tsv`. Hopefully soon they will return error messages during the auto removal process.

There are a lot of [errors types](https://www.ncbi.nlm.nih.gov/genbank/sequencecheck/virus/) and some of the errors can be connected or originate from the same sequence problem, such as `CDS_HAS_FRAMESHIFT` can lead to `CDS_HAS_STOP_CODON` and `INDEFINITE_ANNOTATION_END` and `UNEXPECTED_LENGTH`. The goal here is not to "fix" the genome until there is no more VADR errors. The goal is to fix the assembly errors and leave whatever that is correctly-assembled and well-supported by reads as is, and convince GenBank staff that you have done the due diligence so they will accept the genomes. That being said, I have not succeeded in sending our rejected genomes in so I will complete this section once that is done...

<br>

# Types of frameshifts and when to fix them

For our 10,000+ genomes, GISAID returned ~650 genomes with ~700 frameshifts (one genome can have multiple frameshifts). Here is a breakdown of different types of frameshifts. Only the red slice requires fixing of the genomic sequence. 

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/type_count.png" width="700">

4 items are needed to inspect the frameshifts: the genome itself (the FASTA file), the genomic position of the frameshift so we know where to look (in the error message), the number of bases pairs of the insertion or deletion that is causing the frameshift (in the error message), the read alignment BAM files that the genome was assembled based off, and a genomic browser such as [IGV](https://igv.org/) to open the BAM files (whoa their new IGV site looks so nice that is distracting me from counting properly). Refresher for different related files types see [here](https://github.com/czbiohub/covidtracker_notes/blob/main/bioinformatics/file_types.md).

Note that Biohub only have **Illumina short read data** from metagenomic or ARTIC v3 libraries. Mis-assembly around deletions seems a lot more common for Nanopore data but I do not have enough experience to write about it. All our genomes were aligned with [minimap2 2.17](https://github.com/lh3/minimap2) and assembled with [iVar 1.2](https://github.com/andersen-lab/ivar). 

### A note on FASTA file limitation

A fasta file with a linear genomic sequence has its limitations representing a mixture of more than 1 type of genomes. Naturally occurring intra-host variation, sample contaminations, sequencing errors can all lead to more than 1 type of genomes in the sample. [IUPAC code](https://www.bioinformatics.org/sms2/iupac.html) can represent some of the diversity in nucleotide composition. For example, If a genomic position is `A` in some reads and `C` in others, this position can be represented as `M` (IUPAC code for `A` or `C`). However if a position has `A` in some reads, and is deleted in others, there is no corresponding IUPAC code and iVar will resort to put an `N` in that position for lack of a better choice. All the editing I have done to correct genome assembly error for COVID genomes, was to delete these Ns.

### If there is mis-assembly of the genome based aligned sequencing reads, the genome needs a fix.

With our Illumina data, most of the assembly error I see look like this example below. The error message said it had a 1bp deletion that led to frameshift. If you look at the read alignment, it is actually a 3bp deletion unambiguously representated by the reads containing these 3bp deletions. However, there are reads that align into the region of deletion such as the first read. This read ends with nucleotide `TA` which are the same as the first 2bp of the deletion, which are also the same as the nucleotides flanking the deletion (the 2 blue arrows), so mostly this read contains this deletion just like all the other reads. However when the read-aligner mapped this read to the reference genome, it can map it either as shown in the snapshot below, or map it as having the deletion, and they will be equally good alignment. In this case, the aligner would always favor against the deletion to avoid the gap-penalty of alignment. As a result, this read was mapped as not having the deletion, and instead extend into the deletion. 

Then this alignment was given to iVar to assemble the genome. From iVar's perspective, in 2 positions of the 3bp deletion, some reads have a deletion, some reads don't (such as the first read), and iVar would fill those 2 positions as N, and leave only 1bp of deletion. That is where this 1bp frameshift error comes from. To fix this assembly error, I would delete the 2 Ns that falls within the deletion to make it a 3bp deletion.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/mis1.png" width="600">

Here is another example. 

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/mis2.png" width="600">

In a rarely seen but related scenario below, the 4bp in the middle is `TTCA` in the refrence genome, and in the sample genome the sequence is `GTTC`. When the reads were aligned, they could either be aligned as having mutations in 3 positions, or a combination of an insertion and a deletion. When such a mixed alignment type were presented to iVar, iVar combined these 2 types of alignment and called the position of insertion `N` as explained above, followed by `K` (`T` or `G`), `T`, `Y` (`C` or `T`), and the deletion as `N`. That is 1bp longer than the reference genome and therefore a frameshift. To fix this assembly error, I replaced `NKTYN` with `GTTC` and the frameshift goes away.

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/mis3.png" width="500">

Another example:

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/mis4.png" width="500">

Whether there is assembly error is independent of whether after the assembly error is fixed, a real frameshift exist or not. Our goal is to fix all assembly errors and after that leave the rest alone regardless of frameshifts. 

### If it is a real frameshift and the genome assembly is correct, there is nothing to fix. Do not edit anything.

The rationale behind flag frameshifts as potential errors by the public repositories is that frameshifts are usually detrimental to the virus becuase they change the amino acid sequences. However in general, COVID (and all organisms) has some amount of tolerance to that. This frameshift below is right in the middle of ORF1a. The explanation is that some genes are not that critical (ORF8 can tolerate a large amount of deletion [REF](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7577707/)), I saw lots of frameshifts near the end of genes, and if there is more than one strain in a host their proteins can compensate for each other. 

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/real1.png" width="400">

Example 2. If you notice that in the very middle there is a read that looks just like the previous mis-assembly section. The reason iVar didn't put an `N` there, was because the fraction of such reads are too small to count. On the other hand, if iVar had indeed put an `N` there, this wouldn't be identified as a frameshift because there will be no deletion (the 1bp will be filled as N).

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/real2.png" width="300">

In an effort to let through real frameshifts, GISAID now provides an option to ignore all previously known frameshifts, and GenBank is ignoring all non-essential genes, which is great.

### If it is a random 1bp insertion, can go either way. For my own sanity, I did not fix them.

This is an interesting type of frameshifts. It is a 1bp insertion that lands seemingly random across the genome, most often precedes (a short stretch of) `A` or `T` (only 10/217 instances precede `C` or `G`), and is usually present in some of the reads but not all of them, which leads to a representation by `N` in the assembled genome. I am quite unsure about the original of this. Given it is an insertion instead of point mutation, it seem unlikely to be introduced by PCR processs; given the frequent occurrance, it's extremely unlikely to be sequencing error. It may have something to do with the virus replication - so very curious. 

For the purpose of submission, because the 1bp insertion is well-supported by reads, I usually don't remove them from the genome. Someone someday may find this feature interesting and do some study with it (or I am just too lazy).

A few examples:

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/N1bp1.png" width="400">

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/N1bp2.png" width="400">

Some rare cases:

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/N1bp3.png" width="400">

<img src="https://github.com/czbiohub/covidtracker_notes/blob/main/submission_rejection/images/N1bp4.png" width="400">

<br>

# Some interesting observations


<br>
  
##### Note 1
The error message can be found in the `comment` column from the output `query summary report` file found on the bottom of the CoVsurver result page. Sometimes GISAID will return that message together with rejection and that's all one needs. But when they don't, CoVsurver is the place to obtain it. Some people run their submissions through CoVsurver prior to submission to fix errors preemptively as well. If this tool gets busy and hangs, I usually just try a different time of the day (afternoon is better).

##### Note 2
