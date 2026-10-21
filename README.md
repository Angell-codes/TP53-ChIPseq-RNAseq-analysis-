### Question: How does TP53 binding influence gene expression in DNA damage response pathways? 

### Integrative Genomics:
The human genome is made of 3 billion base pairs organised into 23 chromosome pairs. 
A gene is a sequence of DNA. Humans have 20,000 of these protein-coding genes.
All genes aren't active all the time. The activity of a gene is determined by the number of RNA copies made of it: the more copies, the more "active" the gene is expressed. This is also known as gene expression. 
The promoter, the region around the transcription site, acts as an on/off switch where the machinery and TF bind. 
This initiates the copying of genes. 

### TP53:
Also known as the guardian of the genome, plays a role in the Damage DNA Repair Pathway. 
It will bind to the genome, switch on genes that halt the cell cycle. Repair the DNA and if required, leads the cell to apoptosis. This process is common in chemotherapy where individuals treated with doxorubicin suffer with DNA damage. Hence, tp53 is also called a tumour supressor.  

p53, the protein, recognizes a motif (binding sequence) that tells it where to bind on the genome. The consensus is RRRCWWGYYY-N-(0-13)-RRRCWWGYYY(R=A/G, W=A/T, Y=C/T). 

### The Data:
1. ChIP-seq(GSE176658): The ChIP-seq data shows where p53 binds the genome. DNA fragments bound by TP53 are pulled out with an antibody and sequenced, giving binding peaks that can be linked to nearby genes.
2. Motif scan(FIMO): positions in the DNA are given scores based of how well they match the TP53 motif. This lets us check whether ChIP-seq contain the canonical TP53 motif.
3. RNA-seq(GSE274604): Shows which genes are switched on or off between controlled and treated cells.

### Data Formats:
1. BED file: stores genomic regionss: chromosome name, the start, end, name, score and strand. This file is used to define the peaks located on the genome. The motif hits are saved in BED format. 
2. Gene Transfer Format (GTF): annotates the genes: chromosome name, source, feature, start, end, score, strand, frame, attributes.  It is used to map the peaks to genes, linking peaks to genes.
3. BigWig file: it stores a number for every position on the genome. It is used to extract the signal intensity over peaks to understand the binding strengths. It stores continuous signal data across the genome. Hence we can check how strongly tp53 binds at every binding site.
4. FASTA file: contains the raw protein or DNA data. We scan this file to find out motifs using tools like FIMO or MEME.
5. PFM: position frequency matrix. Each column is a position in the motif and each row is a base (A, C, G, T). The values are raw counts.
6. PWM: the PFM is first converted to probabilities for each base at each position. A log-odds PWM goes a step further and compares each base to its background frequency. I used probabilities.
7. MEME: a standardised text format use by MEME suite. It contains the motif name, width, alphabet and background frequency and letter-probability matrix.

### The computing environment: 
- The initial plan was to continue on a jupyter notebook on my windows software, however I was unable to import tools like pybedtools and pyBigWig.
- I tried to install through the terminal as well and restart the kernel multiple times, however there was some issue which was not resolved.
- Hence, I moved on to using Ubuntu. I created a mini anaconda environment and created a separate environment called genomics- a private toolbox which opens with the python(genomic) kernel.
- Then i installed all the required packers like BEDTools, pybedtools, pyBigWig, BioPython, pandas, numpy, seaborn, scikit, sciPy, matplotlib, seaborn, and gseapy.
- Then I started a jupyter lab through the ubuntu environment and used that to carry out my analysis.
- I also ran a FIMO command in the ubuntu terminal itself: fimo --oc fimo_out TP53.meme promoters.fa

### Pipeline:
1. Load and preprocessing the data
2. First look through plots
3. Normalize the RNA-seq counts and run PCA
4. Read and clean the motif scan results
5. Decide which genes tp53 binds
6. Link the binding to expressional changes
7. Test whether peaks really contain the motif, and whether stronger peaks contain it more often
8. Test whether strong binding predicts induction, and find the hidden confounder
9. Run pathway enrichment on the bound and induced genes
10. Check known p53 target (e.g, CDKN1A)

## Data and References

### Input data
- **TP53 ChIP-seq peaks**: `GSE176658_ENCFF041VRQ_IDR_ranked_peaks_GRCh38.bed` (GEO GSE176658, ENCODE file ENCFF041VRQ). Binding peaks, GRCh38.
- **TP53 ChIP-seq signal**: `GSE176658_ENCFF608GZS_signal_p-value_GRCh38.bigWig` (GEO GSE176658, ENCODE file ENCFF608GZS). Signal (−log10 p-value) track, GRCh38.
- **RNA-seq counts**: GEO GSE274604, [exact file name]. 8 samples: WT/MT, control/AMO, DMSO/DOXO. Raw count matrix, which I normalised in the notebook.
- **TP53 motif**: JASPAR MA0106.1 (`MA0106.1.pfm`, converted to `TP53.meme`).
- **Gene annotation for the analysis**: [GENCODE v38 GTF file name]. Used for gene positions, TSS and promoters (TSS ±2 kb).

### Reference files used to build the FIMO input
- `genome.fa`: human reference genome sequence, Ensembl release 110, GRCh38 primary assembly.
- `genome.fa.fai`: index file for `genome.fa`.
- `Homo_sapiens.GRCh38.110.gtf`: Ensembl release 110 gene annotation.

### Derived files
- `genes.bed`: gene intervals (chromosome, start, end, name, score) extracted from the Ensembl GTF.
- `promoters.bed`: each gene extended by 1 kb with `bedtools slop` (strand was not taken into account, so the extra 1 kb is on the left of every gene).
- `promoters.fa`: sequences for `promoters.bed`, extracted from `genome.fa`. 62,754 sequences covering about 58.5% of the genome.
- `fimo_out/fimo.tsv`: raw FIMO motif hits (MEME Suite 5.5.9, `fimo --oc fimo_out TP53.meme promoters.fa`).
- `data/scanned_regions.bed`: the regions FIMO scanned, used as the background for the motif enrichment test.
- `results/`: tables produced by the notebook, e.g. `TP53_motif_hits.tsv` (hits with p < 1e-6) and `TP53_bound_genes.tsv`.

### Tools
- MEME Suite 5.5.9 (FIMO), BEDTools / pybedtools, pyBigWig, Biopython, pandas, SciPy, scikit-learn, gseapy (Enrichr), Python 3.10.

### Papers and databases

**Datasets**
- ChIP-seq: TF ChIP-seq from A549 cells (ENCODE experiment ENCSR112XUO). GEO accession [GSE176658](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE176658). Peaks `ENCFF041VRQ`, signal `ENCFF608GZS`.
- RNA-seq: Kim D, Park SM, Lee SY, Kim J, et al. Antisense-mediated splicing correction as a therapeutic approach for p53 K120R mutation. *BMB Rep* 2024;57(11):503–508. PMID 39384177. GEO accession [GSE274604](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE274604).
- ENCODE: ENCODE Project Consortium. An integrated encyclopedia of DNA elements in the human genome. *Nature* 2012;489:57–74.
- IDR (peak ranking): Li Q, Brown JB, Huang H, Bickel PJ. Measuring reproducibility of high-throughput experiments. *Ann Appl Stat* 2011;5(3):1752–1779.

**Reference genome and annotation**
- Ensembl release 110 (genome and GTF): Harrison PW, et al. Ensembl 2024. *Nucleic Acids Res* 2024;52(D1):D891–D899.
- GENCODE (v38 annotation): Frankish A, et al. GENCODE 2021. *Nucleic Acids Res* 2021;49(D1):D916–D923.

**Motif and motif scanning**
- JASPAR (matrix MA0106.1): Rauluseviciute I, et al. JASPAR 2024: 20th anniversary of the open-access database of transcription factor binding profiles. *Nucleic Acids Res* 2024;52(D1):D174–D182. Matrix page: https://jaspar.elixir.no/matrix/MA0106.1/
- FIMO (MEME Suite 5.5.9): Grant CE, Bailey TL, Noble WS. FIMO: scanning for occurrences of a given motif. *Bioinformatics* 2011;27(7):1017–1018. https://doi.org/10.1093/bioinformatics/btr064

**Software**
- BEDTools: Quinlan AR, Hall IM. BEDTools: a flexible suite of utilities for comparing genomic features. *Bioinformatics* 2010;26(6):841–842. https://doi.org/10.1093/bioinformatics/btq033
- pybedtools: Dale RK, Pedersen BS, Quinlan AR. Pybedtools: a flexible Python library for manipulating genomic datasets and annotations. *Bioinformatics* 2011;27(24):3423–3424.
- Biopython: Cock PJA, et al. Biopython: freely available Python tools for computational molecular biology and bioinformatics. *Bioinformatics* 2009;25(11):1422–1423.
- SciPy: Virtanen P, et al. SciPy 1.0: fundamental algorithms for scientific computing in Python. *Nat Methods* 2020;17:261–272.
- scikit-learn: Pedregosa F, et al. Scikit-learn: machine learning in Python. *J Mach Learn Res* 2011;12:2825–2830.

**Pathway enrichment**
- Enrichr: Chen EY, et al. *BMC Bioinformatics* 2013;14:128. Kuleshov MV, et al. *Nucleic Acids Res* 2016;44(W1):W90–W97. Xie Z, et al. *Curr Protoc* 2021;1(3):e90.
- gseapy: Fang Z, Liu X, Peltz G. GSEApy: a comprehensive package for performing gene set enrichment analysis in Python. *Bioinformatics* 2023;39(1):btac757.
- Gene-set libraries used: MSigDB Hallmark 2020 (Liberzon A, et al. *Cell Syst* 2015;1(6):417–425), KEGG 2021 Human (Kanehisa M, Goto S. *Nucleic Acids Res* 2000;28(1):27–30), GO Biological Process 2023 (Ashburner M, et al. *Nat Genet* 2000;25:25–29).
