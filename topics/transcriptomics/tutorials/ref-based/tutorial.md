---
layout: tutorial_hands_on
topic_name: transcriptomics
tutorial_name: ref-based
---

# Introduction
{:.no_toc}

In the study of [Brooks *et al.* 2011](http://genome.cshlp.org/content/21/2/193.long), the Pasilla (PS) gene, *Drosophila* homologue of the Human splicing regulators Nova-1 and Nova-2 Proteins, was depleted in *Drosophila melanogaster* by RNAi. The authors wanted to identify exons that are regulated by Pasilla gene using RNA sequencing data.

Total RNA was isolated and used for preparing either single-end or paired-end RNA-seq libraries for treated (PS depleted) samples and untreated samples. These libraries were sequenced to obtain a collection of RNA sequencing reads for each sample. The effects of Pasilla gene depletion on splicing events can then be analyzed by comparison of RNA sequencing data of the treated (PS depleted) and the untreated samples.

The genome of *Drosophila melanogaster* is known and assembled. It can be used as reference genome to ease this analysis.  In a reference based RNA-seq data analysis, the reads are aligned (or mapped) against a reference genome, *Drosophila melanogaster* here, to significantly improve the ability to reconstruct transcripts and then identify differences of expression between several conditions.

> ### Agenda
>
> In this tutorial, we will deal with:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Pretreatments

## Data upload

The original data is available at NCBI Gene Expression Omnibus (GEO) under accession number [GSE18508](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE18508).

We will look at the 7 first samples:

- 3 treated samples with Pasilla (PS) gene depletion: [GSM461179](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM461179), [GSM461180](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM461180), [GSM461181](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM461181)
- 4 untreated samples: [GSM461176](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM461176), [GSM461177](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM461177), [GSM461178](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM461178), [GSM461182](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM461182)

Each sample constitutes a separate biological replicate of the corresponding condition (treated or untreated). Moreover, two of the treated and two of the untreated samples are from a paired-end sequencing assay, while the remaining samples are from a single-end sequencing experiment.

We have extracted sequences from the Sequence Read Archive (SRA) files to build FASTQ files.

> ### {% icon hands_on %} Hands-on: Data upload
>
> 1. Create a new history for this RNA-seq exercise
> 2. Import the FASTQ file pairs for
>       - `GSM461177` (untreated)
>       - `GSM461180` (treated)
>
>       To import the files, there is 2 options:
>       - Option 1: From a shared data library if available (ask your instructor)
>       - Option 2: From [Zenodo](https://dx.doi.org/10.5281/zenodo.290221)
>
>           > ### {% icon tip %} Tip: Importing data via links
>           >
>           > * Copy the link location
>           > * Open the Galaxy Upload Manager
>           > * Select **Paste/Fetch Data**
>           > * Paste the link into the text field
>           > * Press **Start**    
>           {: .tip}
>           
>           You can directly paste:
>
>           ```
>           https://zenodo.org/record/290221/files/GSM461177_1.fastq
>           https://zenodo.org/record/290221/files/GSM461177_2.fastq
>           https://zenodo.org/record/290221/files/GSM461180_1.fastq
>           https://zenodo.org/record/290221/files/GSM461180_2.fastq
>           ```
>
> 3. Rename the datasets according to the samples
> 4. Check that the datatype is `fastqsanger`, not `fastq`.
>    If the datatype is `fastq`, please change the file type to `fastqsanger`
>
>    > ### {% icon tip %} Tip: Changing the datatype
>    > * Click on the pencil button displayed in your dataset in the history
>    > * Choose **Datatype** on the top
>    > * Select `fastqsanger`
>    > * Press **Save**
>    {: .tip}
>
> 5. Add to each database a tag corresponding to the name of the sample (`#GSM461177` or `#GSM461180`)
> 
>    > ### {% icon tip %} Tip: Adding a tag
>    > * Click on the dataset
>    > * Click on **Edit dataset tags**
>    > * Add the tag: always starting with `#`
>    > * Check that the tag is apparing below the dataset name
>    > 
>    > These tags will propagate to the outputs of tools running using this dataset.
>    {: .tip}
{: .hands_on}

The sequences are raw sequences from the sequencing machine, without any pretreatments. They need to be controlled for their quality.

## Quality control

For quality control, we use similar tools as described in [NGS-QC tutorial]({{site.baseurl}}/topics/sequence-analysis): [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) and [Trim Galore](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/).

> ### {% icon hands_on %} Hands-on: Quality control
>
> 1. **FastQC** {% icon tool %}: Run FastQC on the FASTQ files to control the quality of the reads
>       - "Short read data from your current history"
>           - Click on "Multiple datasets"
>           - Select the four datasets
>
>       > ### {% icon tip %} Tip
>       >
>       > You can select several files by keeping the CTRL (or COMMAND) key pressed and clicking on the interesting files
>       {: .tip}
>
> 2. Inspect on the generated webpage for `GSM461177_1` sample
>
>    > ### {% icon question %} Questions
>    >
>    > 1. What is the read length?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>The read length is 37 bp</li>
>    >    </details>
>    {: .question}
>
> 3. **MultiQC** {% icon tool %}: Aggregate the FastQC report with
>      - "Which tool was used generate logs?" to `FastQC`
>      - "Type of FastQC output?" to `Raw data`
>      - "FastQC output" to the generated `Raw data` files (multiple datasets)
>
>    > ### {% icon question %} Questions
>    >
>    > 1. How is the quality score for the different files?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>Everything seems ok for 3 of the files. But for GSM461180_2, the quality seems to decrease quite a lot at the end of the sequences</li>
>    >    </details>
>    {: .question}
>
> 2. **Trim Galore** {% icon tool %}: Treat for the quality of sequences by running Trim Galore! with
>      - "Is this library paired- or single-end?" to `Paired-end`
>      - First "Reads in FASTQ format" to both `_1` datasets (multiple datasets)
>      - Second "Reads in FASTQ format" to both `_2` datasets (multiple datasets)
>
>    > ### {% icon question %} Questions
>    >
>    > Why is Trim Galore run once on the paired-end dataset and not twice on each dataset?
>    >
>    > <details>
>    > <summary>Click to view answers</summary>
>    > Trim Galore can remove sequences if they become too short during the trimming process. For paired-end files Trim Galore! removes entire sequence pairs if one (or both) of the two reads became shorter than the set length cutoff. Reads of a read-pair that are longer than a given threshold but for which the partner read has become too short can optionally be written out to single-end files. This ensures that the information of a read pair is not lost entirely if only one read is of good quality.
>    > </details>
>    {: .question}
>
{: .hands_on}

As the genome of *Drosophila melanogaster* is known and assembled, we can use this information and map the sequences on this genome to identify the effects of Pasilla gene depletion on splicing events.

# Mapping

To make sense of the reads, we need to determine to which they belong. The first step is to determine their positions within the *Drosophila melanogaster* genome. This process is known as aligning or 'mapping' the reads to the reference genome.

> ### {% icon comment %} Comment
>
> Do you want to learn more about the principles behind mapping? Follow our [training]({{site.baseurl}}/topics/sequence-analysis/)
{: .comment}

Because in the case of a eukaryotic transcriptome, most reads originate from processed mRNAs lacking introns, they cannot be simply mapped back to the genome as we normally do for DNA data. Instead the reads must be separated into two categories:

- Reads that map entirely within exons
- Reads that cannot be mapped within an exon across their entire length because they span two or more exons

    ![Spliced read](../../images/spliced_read.png)

Spliced mappers have been developed to efficiently map transcript-derived reads against genomes. 

<details>
<summary>Click for more details on the different assemblers</summary>
[TopHat](https://ccb.jhu.edu/software/tophat/index.shtml) was one of the first tools designed specifically to address this problem by identifying potential exons using reads that do map to the genome, generating possible splices between neighboring exons, and comparing reads that did not initially map to the genome agaisnt these in silico created junctions. 

In TopHat reads are mapped against the genome and are separated into two categories: (1) those that map, and (2) those that initially unmapped (IUM). 
"Piles" of reads representing potential exons are extended in search of potential donor/acceptor splice sites and potential splice junctions are reconstructed. IUMs are then mapped to these junctions.

TopHat has been subsequently improved with the development of TopHat2:

![Kim et al.](../../images/13059_2012_Article_3053_Fig6_HTML.jpg "TopHat2 (Kim et al, Genome Biology, 2013)")

To further optimize and speed up spliced read alignment Kim at al. [2015](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4655817/) developed [HISAT](https://ccb.jhu.edu/software/hisat2/index.shtml). It uses a set of [FM-indices](https://en.wikipedia.org/wiki/FM-index) consisting one global genome-wide index and a collection of ~48,000 local overlapping 42 kb indices (~55,000 56 kb indices in HiSat2). This allows to find initial seed locations for potential read alignments in the genome using global index and to rapidly refine these alignments using a corresponding local index:

![Hierarchical Graph FM index in HiSat/HiSat2](../../images/hisat.png "Hierarchical Graph FM index in HiSat/HiSat2 (Kim et al, Nat Methods, 2015)")

A part of the read (blue arrow) is first mapped to the genome using the global FM index. The HiSat then tries to extend the alignment directly utilizing the genome sequence (violet arrow). In (**a**) it succeeds and this read aligned as it completely resides within an exon. In (**b**) the extension hits a mismatch. Now HiSat takes advantage of the local FM index overlapping this location to find the appropriate matting for the remainder of this read (green arrow). The (**c**) shows a combination these two strategies: the beginning of the read is mapped using global FM index (blue arrow), extended until it reaches the end of the exon (violet arrow), mapped using local FM index (green arrow) and extended again (violet arrow). 

[STAR aligner](https://github.com/alexdobin/STAR) is a fast alternative for mapping RNAseq reads against genome utilizing uncompressed [suffix array](https://en.wikipedia.org/wiki/Suffix_array). It operates in two stages [(Dobin et al, Bioinformatics, 2013)](https://academic.oup.com/bioinformatics/article/29/1/15/272537). In the first stage it performs seed search:

![STAR's seed search](../../images/star.png "STAR's seed search (Dobin et al, Bioinformatics, 2013)")

Here a read is split between two consecutive exons. STAR starts to look for a *maximum mappable prefix* (MMP) from the beginning of the read until it can no longer match continuously. After this point it start to MMP for the unmatched portion of the read (**a**). In the case of mismatches (**b**) and unalignable regions (**c**) MMPs serve as anchors from which to extend alignments. [Dobin et al, Bioinformatics, 2013](https://academic.oup.com/bioinformatics/article/29/1/15/272537).

At the second stage STAR stitches MMPs to generate read-level alignments that (contrary to MMPs) can contain mismatches and indels. A scoring scheme is used to evaluate and prioritize stitching combinations and to evaluate reads that map to multiple locations. STAR is extremely fast but requires a substantial amount of RAM to run efficiently.
</details>

## Mapping

We can now map all the RNA sequences on the *Drosophila melanogaster* genome using STAR.

> ### {% icon hands_on %} Hands-on: Spliced mapping
>
> 1. Import the Ensembl gene annotation for *Drosophila melanogaster* ([`Drosophila_melanogaster.BDGP6.87.gtf`](https://zenodo.org/record/290221/files/Drosophila_melanogaster.BDGP6.87.gtf)) from the shared data library or from [Zenodo](https://dx.doi.org/10.5281/zenodo.290221) into your current Galaxy history
>    - Rename the dataset if necessary
>    - Verify that the datatype is `gtf` and not `gff`
>
> 2. **RNA STAR** {% icon tool %}: Map your reads on the reference genome with
>    - "Single-end or paired-end reads" to `Paired-end (as individual datasets)`
>    - "RNA-Seq FASTQ/FASTA file, forward reads" to the generated `trimmed reads pair 1` files (multiple datasets)
>    - "RNA-Seq FASTQ/FASTA file, reverse reads" to the generated `trimmed reads pair 2` files (multiple datasets)
>    - "Custom or built-in reference genome" to `Use a built-in index`
>    - "Reference genome with or without an annotation" to `use genome reference without builtin gene-model`
>    - "Select reference genome" to `Drosophila Melanogaster (dm6)`
>    - "Gene model (gff3,gtf) file for splice junctions" to the imported `Drosophila_melanogaster.BDGP6.87.gtf`
>    - "Length of the genomic sequence around annotated junctions" to ReadLength-1 (`36`)
>
> 3. **MultiQC** {% icon tool %}: Aggregate the FastQC report with
>      - "Which tool was used generate logs?" to `STAR`
>      - "Type of FastQC output?" to `Log`
>      - "STAR log output" to the generated `log` files (multiple datasets)
>
>    > ### {% icon question %} Question
>    >
>    > 1. Which percentage of reads were mapped 1 time for both sample?
>    > 3. What is the overall alignment rate?
>    >
>    >    <details>
>    >    <summary>Click to view answer</summary>
>    >    <ol type="1">
>    >    <li>37.84% and 14.69%</li>
>    >    </ol>
>    >    </details>
>    {: .question}
{: .hands_on}

**STAR** generates a BAM file with the mapped reads.

> ### {% icon question %} Question
>
> 1. What is a BAM file?
> 2. What does such a file contain?
>
>    <details>
>    <summary>Click to view answer</summary>
>    <ol type="1">
>    <li>a BAM file is the binary version of a SAM file</li>
>    <li>It contains information about the mapping: for each mapped read, the position on the reference genome, the mapping quality, ...</li>
>    </ol>
>    </details>
{: .question}

## Inspection of the mapping results

The BAM file contains information about where the reads are mapped on the reference genome. But it is a binary file and with the information for more than 3 million reads encoded in it, it is difficult to inspect and explore the file.

A powerful tool to visualize the content of BAM files is the Integrative Genomics Viewer IGV.

> ### {% icon hands_on %} Hands-on: Inspection of mapping results
>
> 1. **IGV** {% icon tool %}: Visualize the aligned reads for `GSM461177`
>     - Click on the STAR BAM output in your history to expand it.
>     - Towards the bottom of the history item, find the line starting with `Display with IGV`
>        
>        This is followed by 2 links:
>        - option 1: `local`. Select this option if you already have IGV installed on your machine
>        - option 2: `D. melanogaster (dm3)`. This will download and launch IGV on your local machine
>
>    > ### {% icon comment %} Comments
>    >
>    > In order for this step to work, you will need to have either IGV or [Java web start](https://www.java.com/en/download/faq/java_webstart.xml)
>    > installed on your machine. However, the questions in this section can also be answered by inspecting the IGV screenshots below.
>    >
>    > Check the [IGV documentation](https://software.broadinstitute.org/software/igv/AlignmentData) for more information.
>    >
>    {: .comment}
>
> 2. **IGV** {% icon tool %}: Zoom to `chr4:560,000-600,000`  (Chromosome 4 between 560 kb to 600 kb)
>
>    > ### {% icon question %} Question
>    >
>    > ![Screenshot of the IGV view on Chromosome 4](../../images/junction_igv_screenshot.png "Screenshot of IGV on Chromosome 4")
>    >
>    > 1. Which information does appear on the top in grey?
>    > 2. What do the connecting lines between some of the aligned reads indicate?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>The coverage plot: the sum of mapped reads at each position</li>
>    >    <li>They indicate junction events (or splice sites), *i.e.*, reads that are mapped across an intron</li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
> 3. **IGV** {% icon tool %}: Inspect the splice junctions using a **Sashimi plot**
>
>    > ### {% icon tip %} Tip: Creation of a Sashimi plot
>    >
>    > * Right click on the BAM file
>    > * Select **Sashimi Plot** from the context menu
>    {: .tip}    
>
>    > ### {% icon question %} Question
>    >
>    > ![Screenshot of a Sashimi plot of Chromosome 4](../../images/hisat_igv_sashimi.png "Screenshot of a Sashimi plot of Chromosome 4")
>    >
>    > 1. What does the vertical bar graph represent? And the numbered arcs?
>    > 2. What do the numbers on the arcs mean?
>    > 3. Why do we observe 4 different stacked groups of blue linked boxes at the bottom right?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>The coverage for each alignment track is plotted as a bar graph. Arcs represent observed splice junctions, *i.e.*, reads spanning introns</li>
>    >    <li>The numbers refer to the number of these observed junction reads. </li>
>    >    <li>The 4 groups of linked boxes on the bottom represent 4 transcripts from a single gene differing in the first exon.</li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
>    > ### {% icon comment %} Comment
>    >
>    > Check the [IGV documentation on Sashimi plots](https://software.broadinstitute.org/software/igv/Sashimi) to find some clues
>    {: .comment}
>
{: .hands_on}

After the mapping, we have in the generated mapping file the information about where the reads are mapped on the reference genome. So for each mapped read, we know where it is mapped and how good it was mapped.

The next step in the RNA-Seq data analysis is quantification of expression level of the genomic features (gene, transcript, exons, ...) to be able then to compare several samples for the different expression analysis. The quantification consist into taking each known genomic feature (*e.g.* gene) of the reference genome and then counting how many reads are mapped on this genomic feature. So, in this step, we start with an information per mapped reads to end with an information per genomic feature.

> ### {% icon comment %} Comment
>
> The quantification depends on the definition of the genomic features of the reference genome, and then on the annotations. We strongly recommend you to use an annotation corresponding to the same version of the reference genome you used for the mapping.
{: .comment}

To identify exons that are regulated by the Pasilla gene, we need to identify genes and exons which are differentially expressed between samples with PS gene depletion and control samples.
In this tutorial, we will then analyze the differential gene expression, but also the differential exon usage.

The mapping exercise worked for you? Great! :tada:

# Analysis of the differential gene expression

We will first investigate the differential gene expression to identify which genes are impacted by the Pasilla gene depletion

## Count the number of reads per annotated gene

To compare the expression of single genes between different conditions (*e.g.* with or without PS depletion), an essential first step is to quantify the number of reads per gene. 

*Add a small scheme to explain what is counting*

[**HTSeq-count**](https://www-huber.embl.de/users/anders/HTSeq/doc/count.html) is one of the most popular tools for gene quantification.

*FeatureCounts*

To quantify the number of reads mapped to a gene, an annotation of the gene position is needed. We have already uploaded the `Drosophila_melanogaster.BDGP6.87.gtf` with the Ensembl gene annotation for *Drosophila melanogaster* to Galaxy, which we can now make use of for this purpose.

In principle, the counting of reads overlapping with genomic features is a fairly simple task. But there are some details that need to be decided: for example the strandness...

### Estimation of the strandness

*Why do we need to estimate the strandness? Some scheme...*

> ### {% icon comment %} Comment
> This step is not necessary if you already know the library type of your data (information usually available with the preparation kit). 
>
> This information should usually come with your FASTQ files, ask your sequencing facility! If not, try to find them on the site where you downloaded the data or in the corresponding publication. Another option is to estimate these parameters with a *preliminary mapping* of a *downsampled* file and some analysis programs. Afterward, the actual mapping can be redone on the original files with the optimized parameters.
{: .comment}

The library type is determined by the library preparation protocol which, in turn, determines which of the two strands of cDNA obtained from the input RNA through reverse transcription ultimately get sequenced.

*Add scheme from Sailfish*

![Credit: Zhao Zhang](../../images/strand_library_type.png "Credit: Zhao Zhang(http://onetipperday.sterding.com/2012/07/how-to-tell-which-library-type-to-use.html)")

In the previous illustration, you can see that in the dUTP method, for example, only the first cDNA strand, synthesized through reverse transcription of the original RNA, gets sequenced, while the second cDNA strand corresponding to the original RNA strand is degraded because of the dUTP incorporated into it.

Examples of protocol | Description | Library Type (HISAT2)
--- | --- | ---
Standard Illumina | Reads from the left-most end of the fragment (in transcript coordinates) map to the transcript strand, and the right-most end maps to the opposite strand | Unstranded (default)
dUTP, NSR, NNSR | Same as above except we enforce the rule that the right-most end of the fragment (in transcript coordinates) is the first sequenced (or only sequenced for single-end reads). Equivalently, it is assumed that only the strand generated during first strand synthesis is sequenced. | First strand (FR/F)
Ligation, Standard SOLiD | Same as above except we enforce the rule that the left-most end of the fragment (in transcript coordinates) is the first sequenced (or only sequenced for single-end reads). Equivalently, it is assumed that only the strand generated during second strand synthesis is sequenced. | Second strand (RF/R)

If you do not know the library type, you can find it by yourself by mapping the reads on the reference genome and infer the library type from the mapping results by comparing reads mapping information to the annotation of the reference genome.

![Type of library, depending also of the type of sequencing](../../images/library_type_mapping.png "Type of library, depending also of the type of sequencing")

Sequencing proceeds from 5' to 3'. So, in the First Strand case, all reads from the left-most end of an RNA fragment (always from 5' to 3') are mapped to the transcript-strand, and (for pair-end sequencing) reads from the right-most end are always mapped to the opposite strand.

We can now try to determine the library type of our data.


*How do we do that? How is infer experiment working?*

> ### {% icon hands_on %} Hands-on: Determining the library strandness
>
> 1. **Infer Experiment** {% icon tool %}: Determine the library strandness with:
>    - "Input .bam file" to the STAR-generated `BAM` files (multiple datasets)
>    - "Reference gene model" to `Drosophila_melanogaster.BDGP6.87.gtf`
>    - "Number of reads sampled from SAM/BAM file (default = 200000)" to `200000`
>
> 2. Check the results and search the tool's documentation for help on the meaning
>    
>    > ### {% icon comment %} Comment
>    > As it is sometimes quite difficult to find out which settings correspond to those of other programs, the following table might be helpful to identify the library type:
>    >
>    > Sequencing type | **Infer Experiment** | **TopHat** | **HISAT2** | **htseq-count** | **featureCounts**
>    > --- | --- | --- | --- | --- | ---
>    > Paired-End (PE) | "1++,1--,2+-,2-+" | "FR Second Strand" | "Second Strand F/FR" | "yes" | "1"
>    > PE | "1+-,1-+,2++,2--" | "FR First Strand" | "First Strand R/RF" | "reverse" | "2"
>    > Single-End (SE) | "++,--" | "FR Second Strand" | "Second Strand F/FR" | "yes" | "1"
>    > SE | "+-,-+" | "FR First Strand" | "First Strand R/RF" | "reverse" | "2"
>    > SE,PE | undecided | "FR Unstranded" | default | "no" | "0"
>    >
>    {: .comment}
>
> *Adding an explanation of infer experiment output*
>    
>    > ### {% icon question %} Question
>    >
>    > 1. Which fraction of the reads in the BAM file can be explained assuming which library type for `GSM461177`?
>    > 2. Which library type do you choose for both samples?
>    >
>    >    <details>
>    >    <summary>Click to view answer</summary>
>    >    <ol type="1">
>    >    <li>Fraction of reads explained by "1++,1--,2+-,2-+": 0.4648 - Fraction of reads explained by "1+-,1-+,2++,2--": 0.4388</li>
>    >    <li>The library seems to be of the type unstranded for both samples. </li>
>    >    </ol>
>    >    </details>
>    {: .question}
{: .hands_on}

*FeatureCounts and strandness*

### Counting

In principle, the counting of reads overlapping with genomic features is a fairly simple task. But there are some details that need to be decided, such how to handle multi-mapping reads. **HTSeq-count** offers 3 choices ("union", "intersection_strict" and "intersection_nonempty") to handle read mapping to multiple locations, reads overlapping introns, or reads that overlap more than one genomic feature:

![HT-Seq methods to handle overlapping reads](../../images/htseq_count.png "HT-Seq methods to handle overlapping reads (HTSeq documentation: https://www-huber.embl.de/users/anders/HTSeq/doc/count.html)")

The recommended mode is "union", which counts overlaps even if a read only shares parts of its sequence with a genomic feature and disregards reads that overlap more than one feature.

> ### {% icon hands_on %} Hands-on: Counting the number of reads per annotated gene
>
> 1. **featureCounts** {% icon tool %}: Count the number of reads per genes using **featureCounts** with
>    - "Alignment file" to the STAR-generated `BAM` files (multiple datasets)
>    - "Gene annotation file" to `GTF file`
>    - "Gene annotation file" to `in your history`
>    - "Gene annotation file" to `Drosophila_melanogaster.BDGP6.87.gtf (dm6)`
>    - "Output format" to `Gene-ID "\t" read-count (DESeq2 IUC wrapper compatible)`
>    - "Advanced options"
>    - "GFF feature type filter" to `exon`
>    - "GFF gene identifier" to `gene_id`
>    - "Allow read to contribute to multiple features" to `No` ???
>    - "Strand specificity of the protocol" to `Unstranded`
>    - "Minimum mapping quality per read" to `10`
>    - "Count multi-mapping reads/fragments" to `Disabled; multi-mapping reads are excluded (default)`
>
>    - The "Union" mode ??
>
> 2. Inspect the result files
>
>    > ### {% icon question %} Question
>    >
>    > 1. Which information does the result files contain?
>    > 2. Which feature has the most reads mapped on it?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>The useful result file is a tabular file with two columns: the gene id and the number of reads mapped on the corresponding gene</li>
>    >    <li>To display the most abundantly detected feature, we need to sort the output file with the features and the number of reads mapped to them. This can be done using the Sort tool on the second column and in descending order, which reveals that FBgn0017545 is the feature with the most reads (7,650) mapped on it.</li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
> 2. **MultiQC** {% icon tool %}: Aggregate the FastQC report with
>      - "Which tool was used generate logs?" to `featureCounts`
>      - "Output of FeatureCounts" to the generated `summary` files (multiple datasets)
>
>    > ### {% icon question %} Question
>    >
>    > 1. Percentage of reads assigned?? Quality?
>    >
>    >    <details>
>    >    <summary>Click to view answers</summary>
>    >    <ol type="1">
>    >    <li>???</li>
>    >    </ol>
>    >    </details>
>    {: .question}
>
{: .hands_on}

## Identification of the differentially expressed features

In the previous section, we counted reads that mapped to genes for two sample. To be able to identify differential gene expression induced by PS depletion, all datasets (3 treated and 4 untreated) must be analyzed following the same procedure and for the whole genome.

> ### {% icon hands_on %} (Optional) Hands-on: Re-run on the other datasets
>
> You can do the same process on the other sequence files available on [Zenodo](https://dx.doi.org/10.5281/zenodo.290221)
>
> - Paired-end data
>     - `GSM461178_1` and `GSM461178_2`
>     - `GSM461181_1` and `GSM461181_2`
> - Single-end data
>     - `GSM461176`
>     - `GSM461179`
>     - `GSM461182`
>
> This is really interesting to redo on the other datasets, specially to check how the parameters are inferred given the different type of data.
{: .hands_on}

To save time, we have run the necessary steps for you and obtained 7 count files, available on [Zenodo](https://dx.doi.org/10.5281/zenodo.290221).

These files contain for each gene of Drosophila the number of reads mapped to it. We could compare the files directly and calculate the extent of differential gene expression, but the number of sequenced reads mapped to a gene depends on:

- Its own expression level
- Its length
- The sequencing depth of the sample
- The expression of all other genes within the sample

Either for within- or for between-sample comparison, the gene counts need to be normalized. We can then use the Differential Gene Expression (DGE) analysis, whose two basic tasks are:

- Estimate the biological variance using the replicates for each condition
- Estimate the significance of expression differences between any two conditions

This expression analysis is estimated from read counts and attempts are made to correct for variability in measurements using replicates that are absolutely essential for accurate results. For your own analysis, we advice you to use at least 3, but preferably 5 biological replicates per condition. You can have different number of replicates per condition.

[**DESeq2**](https://bioconductor.org/packages/release/bioc/html/DESeq2.html) is a great tool for DGE analysis. It takes read counts produced by **HTseq-count**, combines them into a big table (with genes in the rows and samples in the columns) and applies size factor normalization:

- Computation for each gene of the geometric mean of read counts across all samples
- Division of every gene count by the geometric mean
- Use of the median of these ratios as a sample's size factor for normalization

Multiple factors with several levels can then be incorporated in the analysis. After normalization we can compare, in a statistically reliable way, the response of the expression of any gene to the presence of different levels of a factor.

In our example, we have samples with two varying factors that can explain differences in gene expression:

- Treatment (either treated or untreated)
- Sequencing type (paired-end or single-end)

Here treatment is the primary factor which we are interested in. The sequencing type is some further information that we know about the data that might affect the analysis. This particular multi-factor analysis allows us to assess the effect of the treatment, while taking the sequencing type into account, too.

> ### {% icon comment %} Comment
>
> We recommend you to add as many factors as you think may affect gene expression in your experiment. It can be the sequencing type like here, but it can also be the manipulation (if different persons are involved in the library preparation), ...
{: .comment}

> ### {% icon hands_on %} Hands-on: Determines differentially expressed features
>
> 1. Create a new history
> 2. Import the seven count files from [Zenodo](https://dx.doi.org/10.5281/zenodo.290221) or the data library
>    - `GSM461176_untreat_single.deseq.counts`
>    - `GSM461177_untreat_paired.deseq.counts`
>    - `GSM461178_untreat_paired.deseq.counts`
>    - `GSM461179_treat_single.deseq.counts`
>    - `GSM461180_treat_paired.deseq.counts`
>    - `GSM461181_treat_paired.deseq.counts`
>    - `GSM461182_untreat_single.deseq.counts`
>
> 3. **DESeq2** {% icon tool %}: Run **DESeq2** with:
>    - "1: Factor"
>       - "Specify a factor name, e.g. effects_drug_x or cancer_markers" to `Treatment`
>       - "1: Factor level"
>           - "Specify a factor level, typical values could be 'tumor', 'normal', 'treated' or 'control'" to `treated`
>           - "Counts file(s)" to the generated count files (multiple datasets) with `treated` in name
>       - "2: Factor level"
>           - "Specify a factor level, typical values could be 'tumor', 'normal', 'treated' or 'control'" to `untreated`
>           - "Counts file(s)" to the generated count files (multiple datasets) with `untreated` in name
>    - "Insert Factor"
>    - "2: Factor"
>       - "Specify a factor name, e.g. effects_drug_x or cancer_markers" to `Sequencing`
>       - "1: Factor level"
>           - "Specify a factor level, typical values could be 'tumor', 'normal', 'treated' or 'control'" to `PE`
>           - "Counts file(s)" to the generated count files (multiple datasets) with `paired` in name
>       - "2: Factor level"
>           - "Specify a factor level, typical values could be 'tumor', 'normal', 'treated' or 'control'" to `SE`
>           - "Counts file(s)" to the generated count files (multiple datasets) with `single` in name   
>    - "Output normalized counts table" to `Yes`
{: .hands_on}

**DESeq2** generated 3 outputs

- A table with the normalized counts for each genes (rows) and each samples (columns)
- A graphical summary of the results, useful to evaluate the quality of the experiment:

    1. Histogram of *p*-values for all tests
    2. [MA plot](https://en.wikipedia.org/wiki/MA_plot): global view of the relationship between the expression change of conditions (log ratios, M), the average expression strength of the genes (average mean, A), and the ability of the algorithm to detect differential gene expression. The genes that passed the significance threshold (adjusted p-value < 0.1) are colored in red.
    3. Principal Component Analysis ([PCA](https://en.wikipedia.org/wiki/Principal_component_analysis)) and the first two axes

        Each replicate is plotted as an individual data point. This type of plot is useful for visualizing the overall effect of experimental covariates and batch effects.

        > ### {% icon question %} Questions
        >
        > 1. What is the first axis separating?
        > 2. And the second axis?    
        >
        >    <details>
        >    <summary>Click to view answers</summary>
        >    <ol type="1">
        >    <li>The first axis is seperating the treated samples from the untreated samples, as defined when DESeq2 was launched</li>
        >    <li>The second axis is separating the single-end datasets from the paired-end datasets</li>
        >    </ol>
        >    </details>
        {: .question}


    4. Heatmap of sample-to-sample distance matrix: overview over similarities and dissimilarities between samples

        > ### {% icon question %} Questions
        >
        > How are the samples grouped?
        >
        >    <details>
        >    <summary>Click to view answers</summary>
        >    <ol type="1">    
        >    <li>They are first grouped depending on the treatment (the first factor) and after on the library type (the second factor), as defined when DESeq2 was launched</li>
        >    </ol>
        >    </details>
        {: .question}

    5. Dispersion estimates: gene-wise estimates (black), the fitted values (red), and the final maximum a posteriori estimates used in testing (blue)

        This dispersion plot is typical, with the final estimates shrunk from the gene-wise estimates towards the fitted estimates. Some gene-wise estimates are flagged as outliers and not shrunk towards the fitted value. The amount of shrinkage can be more or less than seen here, depending on the sample size, the number of coefficients, the row mean and the variability of the gene-wise estimates.

- A summary files with for each gene:

    1.  Gene identifiers
    2.  Mean normalized counts, averaged over all samples from both conditions
    3.  Logarithm (to basis 2) of the fold change

        The log2 fold changes are based on primary factor level 1 vs. factor level 2, hence the order of factor levels is important. For example, for the factor 'Treatment', DESeq2 computes fold changes of 'treated' samples against 'untreated', *i.e.* the values correspond to up- or downregulation of genes in treated samples.

    4.  Standard error estimate for the log2 fold change estimate
    5.  [Wald](https://en.wikipedia.org/wiki/Wald_test) statistic
    6.  *p*-value for the statistical significance of this change
    7.  *p*-value adjusted for multiple testing with the Benjamini-Hochberg procedure which controls false discovery rate ([FDR](https://en.wikipedia.org/wiki/False_discovery_rate))

> ### {% icon comment %} Comment
>
> For more information about **DESeq2** and its outputs, you can have a look at [**DESeq2** documentation](https://www.bioconductor.org/packages/release/bioc/manuals/DESeq2/man/DESeq2.pdf).
{: .comment}

## Visualization of the differentially expressed genes

We would like now to draw an heatmap of the normalized counts for each sample for the most differentially expressed genes.

We would proceed in several steps
- Extract the most differentially expressed genes using the DESeq2 summary file
- Extract the normalized counts of these genes for each sample using the normalized count file generated by DESeq2
- Plot the heatmap of the normalized counts of these genes for each sample

> ### {% icon hands_on %} Hands-on: Extract the most differentially expressed genes
>
> 1. **Filter** {% icon tool %}: Extract genes with a significant change in gene expression (adjusted *p*-value below 0.05) between treated and untreated samples
>    - "Filter" to the DESeq2 summary file
>    - "With following condition" to `c7<0.05`
>
>    > ### {% icon question %} Question
>    >
>    > How many genes have a significant change in gene expression between these conditions?
>    >
>    > <details>
>    > <summary>Click to view answers</summary>
>    > We get 1,091 genes (6.21%) with a significant change in gene expression between treated and untreated samples.
>    > </details>
>    {: .question}
>
>    > ### {% icon comment %} Comment
>    >
>    > The file with the independent filtered results can be used for further downstream analysis as it excludes genes with only few read counts as these genes will not be considered as significantly differentially expressed.
>    {: .comment}
>
>    The generated file contains to many genes to get a meaningful heatmap. So we will take only the genes with an absoluted fold change > 2
>
> 2. **Filter** {% icon tool %}: Extract genes with an absolute log2FC > 1
>    - "Filter" to the differentially expressed genes
>    - "With following condition" to `abs(c3)>1`
>
>    > ### {% icon question %} Question
>    >
>    > How many genes have been conserved?
>    >
>    > <details>
>    > <summary>Click to view answers</summary>
>    > 11.92% (130) of the differentially expressed genes 
>    > </details>
>    {: .question}
>
>    The number of genes is still too high there. So we will take only the 10 most up-regulated and 10 most down-regulated genes
>
> 3. **Sort** {% icon tool %}: Sort the genes by log2 FC
>    - "Sort Dataset" to the differentially expressed genes with FC > 2
>    - "on column" to `3`
>    - "with flavor" to `Numerical sort`
>    - "everything in" to `Descending order`
>
> 4. **Select first lines** {% icon tool %}: Extract the 10 most up-regulated genes
>    - "File to select" to the sorted DE genes with FC > 2
>    - "Operation" to `Keep first lines`
>    - "Number of lines" to `10`
> 
> 4. **Select last lines** {% icon tool %}: Extract the 10 most down-regulated genes
>    - "Text file" to the sorted DE genes with FC > 2
>    - "Operation" to `Keep first lines`
>    - "Number of lines" to `10`
>
> 5. **Concatenate datasets** {% icon tool %}: Concatenated the 10 most up-regulated genes with the 10 most down-regulated genes
>    - "Datasets to concatenate" to the 10 most up-regulated genes and to the 10 most down-regulated genes
>
{: .hands_on}

We now have a table with 20 lines corresponding to the most differentially expressed genes. And for each of the gene, we have its id, its mean normalized counts (averaged over all samples from both conditions), its log2FC and other information.

We could plot the log2FC for the different genes, but here we would like to look at the heatmap with the read counts for these genes in the different samples. So we need to extract the read counts for these genes.

We will join the normalized count table generated by DESeq with the table we just generated to conserved in the normalized count table only the lines corresponding to the most differentially expressed genes

> ### {% icon hands_on %} Hands-on: Extract the normalized counts of most differentially expressed genes in the different samples
>
> 1. **Join two Datasets** {% icon tool %}: Join the two files
>    - "Join" to the normalized count table generated by DESeq2
>    - "using column" to `1`
>    - "with" to the concatenated file with 10 most up-regulated genes and the 10 most down-regulated genes
>    - "and column" to `1`
>    - "Keep lines of first input that do not join with second input" to `No`
>    - "Keep the header lines" to `Yes`
>
>    The generated files has too many columns: the ones with mean normalized counts, the log2FC and other information. We need to remove them
>
> 2. **Cut** {% icon tool %}: Conserve the columns with normalized counts
>    - "Cut columns" to `c1,c2,c3,c4,c5,c6,c7,c8`
>    - "Delimited by" to `Tab`
>    - "From" the joined dataset
>
{: .hands_on}

We have now a table with 20 lines (the most differentially expressed genes) and the normalized counts for these genes in the 7 samples.

> ### {% icon hands_on %} Hands-on: Plot the heatmap of the normalized counts of these genes for each sample
>
> 1. **heatmap2** {% icon tool %}: Plot the heatmap with 
>    - "Input should have column headers" to the generated table
>    - "Advanced - log transformation" to `Log2(value) transform my data`
>    - "Enable data clustering" to `Yes`
>    - "Coloring groups" to "Blue to white to red"
>
{: .hands_on}

You should obtain something similar to:

![Heatmap with the normalized counts for the most differentially expressed genes](../../images/ref_based_most_de_gene_heatmap.png "Normalized counts for the most differentially expressed genes")

> ### {% icon question %} Questions
>
> - Do you observe any tendency in the data?
> - What is changing if we select `Plot the data as it is` in "Advanced - log transformation"?
> - Can you generate an heatmap the normalized counts for the up-regulated genes with FC > 2?
>
> <details>
> <summary>Click to view answers</summary>
> <ol type="1">    
>   <li></li>
>   <li></li>
>   <li></li>
> </ol>
> </details>
{: .question}

## Analysis of the functional enrichment among the differentially expressed genes

We have extracted genes that are differentially expressed in treated (with PS gene depletion) samples compared to untreated samples. We would like to know the functional enrichment among the differentially expressed genes.

[Gene Ontology (GO)](http://www.geneontology.org/) analysis is widely used to reduce complexity and highlight biological processes in genome-wide expression studies, but standard methods give biased results on RNA-seq data due to over-detection of differential expression for long and highly expressed transcripts.

[goseq tool](https://bioconductor.org/packages/release/bioc/vignettes/goseq/inst/doc/goseq.pdf) provides methods for performing GO analysis of RNA-seq data, taking length bias into account. The methods and software used by goseq are equally applicable to other category based tests of RNA-seq data, such as KEGG pathway analysis.

goseq needs 2 files as inputs:
- A tabular file with the differentially expressed genes from all genes assayed in the RNA-seq experiment with 2 columns:
    - the Gene IDs (unique within the file)
    - True (differentially expressed) or False (not differentially expressed)
- A file with information about the length of a gene to correct for potential length bias in differentially expressed genes

> ### {% icon hands_on %} Hands-on: Prepare the datasets for GOSeq
>
> 1. **Compute** {% icon tool %} with 
>    - "Add expression" to `bool(c7<0.05)`
>    - "as a new column to" to DESeq summary file
>
> 2. **Cut** {% icon tool %} with
>    - "Cut columns" to `c1,c8`
>    - "Delimited by" to `Tab`
>    - "From" to the previously generated file
>
> 3. **Change Case** {% icon tool %} with
>    - "From" to the previously generated file
>    - "Change case of columns" to `c1`
>    - "Delimited by" to `Tab`
>    - "To" to `Upper case`
>
>    We just generated the first input for goseq
>
> 4. **Gene length and GC content** with
>    - "Select a built-in GTF file or one from your history" to `Use a GTF from history`
>    - "Select a GTF file" to `Drosophila_melanogaster.BDGP6.87.gtf (dm6)`
>    - "Select a built-in FASTA or one from your history" to `Use a built-in FASTA`
>    - "Select a FASTA file" to `Fly (Drosophila melanogaster): dm6 Full`
>    - "Output length file?" to `Yes`
>    - "Output GC content file?" to `No`
>
> 5. **Change Case** {% icon tool %} with
>    - "From" to the previously generated file
>    - "Change case of columns" to `c1`
>    - "Delimited by" to `Tab`
>    - "To" to `Upper case`
{: .hands_on}

We have now the two required files for goseq.

> ### {% icon hands_on %} Hands-on: Perform GO analysis
>
> 1. **goseq** {% icon tool %} with 
>    - "Differentially expressed genes file" to first file generated on previous step
>    - "Gene lengths file" to second file generated on previous step
>    - "Gene categories" to `Get categories`
>    - "Select a genome to use" to `Fruit fly (dm6)`
>    - "Select Gene ID format" to `Ensembl Gene ID`
>    - "Select one or more categories" to `GO: Cellular Component`, `GO: Biological Process`, `GO: Molecular Function`
>
{: .hands_on}

goseq generates a big table with for each GO terms:
1. `category`: GO category
2. `over_rep_pval`: *p*-value for over representation of the term in the differentially expressed genes
3. `under_rep_pval`: *p*-value for under representation of the term in the differentially expressed genes
4. `numDEInCat`: number of differentially expressed genes in this category 
5. `numInCat`: number of genes in this category 
6. `term`: detail of the term
7. `ontology`: MF (Molecular Function - molecular activities of gene products), CC (Cellular Component - where gene products are active), BP (Biological Process - pathways and larger processes made up of the activities of multiple gene products)
8. `p.adjust.over_represented`: *p*-value for over representation of the term in the differentially expressed genes, adjusted for multiple testing with the Benjamini-Hochberg procedure
9. `p.adjust.under_represented`: *p*-value for over representation of the term in the differentially expressed genes, adjusted for multiple testing with the Benjamini-Hochberg procedure

To identify categories significantly enriched/unenriched below some p-value cutoff, it is necessary to use the adjusted *p*-value.

> ### {% icon question %} Questions
>
> - How many GO terms are over represented? Under represented?
> - How are the over represented GO terms divided between MF, CC and BP? And for under represented GO terms?
>
> <details>
> <summary>Click to view answers</summary>
> <ol type="1">    
>   <li>48 GO terms (0.49%) are over represented and 65 (0.66%) - Filter on c8 (over-represented) and c9 (under-represented)</li>
>   <li>For over-represented, 31 BP, 6 CC and 10 MF - For under-represented, 36 BP, 25 CC and 3 MF - Group data on column 7 and count on column 1</li>
> </ol>
> </details>
{: .question}

# Inference of the differential exon usage

Next, we would like to know the differential exon usage between treated (PS depleted) and untreated samples using RNA-seq exon counts. We will rework on the mapping results we generated previously.

We will use [DEXSeq](https://www.bioconductor.org/packages/release/bioc/html/DEXSeq.html). DEXSeq detects high sensitivity genes, and in many cases exons, that are subject to differential exon usage. But first, as for the differential gene expression, we need to count the number of reads mapping to the exons.

## Count the number of reads per exon

This step is similar to the step of [counting the number of reads per annotated gene](#count-the-number-of-reads-per-annotated-gene) except that, instead of HTSeq-count, we are using DEXSeq-Count.

> ### {% icon hands_on %} Hands-on: Counting the number of reads per exon
>
> 1. **DEXSeq-Count** {% icon tool %}: Use the **DEXSeq-Count** to prepare the *Drosophila* annotations (`Drosophila_melanogaster.BDGP5.78.gtf`) to extract only exons with corresponding gene ids
>     - "Prepare annotation" of "Mode of operation"
>
>    The output is again a GTF file that is ready to be used for counting
>
> 4. **DEXSeq-Count** {% icon tool %}: Count reads using **DEXSeq-Count** with
>     - HISAT2 output as "Input bam file"
>     - The formatted GTF file
> 5. Inspect the result files
>
>    > ### {% icon question %} Question
>    >
>    > Which exon has the most reads mapped to it? From which gene has this exon been extracted? Is there a connection to the previous result obtained with HTSeq-count?
>    >
>    > <details>
>    > <summary>Click to view answers</summary>
>    > FBgn0017545:004 is the exon with the most reads mapped to it. It is part of FBgn0017545, the feature with the most reads mapped with HTSeq-count
>    > </details>
>    {: .question}
{: .hands_on}

## Differential exon usage

DEXSeq usage is similar to DESeq2. It uses similar statistics to find differentially used exons.

As for DESeq2, in the previous step, we counted only reads that mapped to exons on chromosome 4 and for only one sample. To be able to identify differential exon usage induced by PS depletion, all datasets (3 treated and 4 untreated) must be analyzed following the same procedure. To save time, we did that for you. The results are available on [Zenodo](https://dx.doi.org/10.5281/zenodo.290221):

- [dexseq.gtf](https://zenodo.org/record/290221/files/dexseq.gtf): the results of running DEXSeq-count in 'Prepare annotation' mode
- Seven count files generated in 'Count reads' mode

> ### {% icon hands_on %} Hands-on:
>
> 1. Create a new history
> 2. Import the seven count files and the dexseq.gtf from [Zenodo](https://dx.doi.org/10.5281/zenodo.290221)
>    - `dexseq.gtf`
>    - `GSM461176_untreat_single.dexseq.counts`
>    - `GSM461177_untreat_paired.dexseq.counts`
>    - `GSM461178_untreat_paired.dexseq.counts`
>    - `GSM461179_treat_single.dexseq.counts`
>    - `GSM461180_treat_paired.dexseq.counts`
>    - `GSM461181_treat_paired.dexseq.counts`
>    - `GSM461182_untreat_single.dexseq.counts`
>
> 3. **DEXSeq** {% icon tool %}: Run **DEXSeq** with
>    - "condition" as first factor with "treated" and "untreated" as levels and selection of count files corresponding to both levels
>    - "Sequencing" as second factor with "PE" and "SE" as levels and selection of count files corresponding to both levels
>
>    > ### {% icon comment %} Comment
>    >
>    > Unlike DESeq2, DEXSeq does not allow flexible primary factor names. Always use your primary factor name as "condition"
>    {: .comment}
{: .hands_on}

Similarly to DESeq2, DEXSeq generates a table with:

1.  Exon identifiers
2.  Gene identifiers
3.  Exon identifiers in the Gene
4.  Mean normalized counts, averaged over all samples from both conditions
5.  Logarithm (to basis 2) of the fold change

    The log2 fold changes are based on primary factor level 1 vs. factor level 2. The order of factor levels is then important. For example, for the factor 'Condition', DESeq2 computes fold changes of 'treated' samples against 'untreated', *i.e.* the values correspond to up- or downregulations of genes in treated samples.

6.  Standard error estimate for the log2 fold change estimate
7.  *p*-value for the statistical significance of this change
8.  *p*-value adjusted for multiple testing with the Benjamini-Hochberg procedure which controls false discovery rate ([FDR](https://en.wikipedia.org/wiki/False_discovery_rate))

> ### {% icon hands_on %} Hands-on:
>
> 1. **Filter** {% icon tool %}: Run **Filter** to extract exons with a significant differential usage (adjusted *p*-value equal or below 0.05) between treated and untreated samples
>
>    > ### {% icon question %} Question
>    >
>    > How many exons show a significant change in usage between these conditions?
>    >
>    > <details>
>    > <summary>Click to view answers</summary>
>    > We get 38 exons (12.38%) with a significant usage change between treated and untreated samples
>    > </details>
>    {: .question}
{: .hands_on}

# Annotation of the result tables with gene information

Unfortunately, in the process of counting, we loose all the information of the gene except its identifiant. In order to get the information back to our final counting tables, we can use a tool to make the correspondance between identifiant and annotation.

> ### {% icon hands_on %} Hands-on:
>
> 1. **Annotate DE(X)Seq result** {% icon tool %}: Run **Annotate DE(X)Seq result** on a counting table (from DESeq or DEXSeq) using the `Drosophila_melanogaster.BDGP5.78.gtf` as annotation file
{: .hands_on}

# Conclusion
{:.no_toc}

In this tutorial, we have analyzed real RNA sequencing data to extract useful information, such as which genes are up- or downregulated by depletion of the Pasilla gene and which genes are regulated by the Pasilla gene. To answer these questions, we analyzed RNA sequence datasets using a reference-based RNA-seq data analysis approach. This approach can be summarized with the following scheme:

![Summary of the analysis pipeline used](../../images/rna_quantification.png "Summary of the analysis pipeline used")
