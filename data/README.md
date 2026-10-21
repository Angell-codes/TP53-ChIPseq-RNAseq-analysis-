# Data and inputs

Large files are not stored in this repository. Download them and place them in the notebook folder.

## Public data

| File | Source |
|---|---|
| `GSE176658_ENCFF041VRQ_IDR_ranked_peaks_GRCh38.bed` | https://www.encodeproject.org/files/ENCFF041VRQ/ (GEO GSE176658) |
| `GSE176658_ENCFF608GZS_signal_p-value_GRCh38.bigWig` | https://www.encodeproject.org/files/ENCFF608GZS/ (GEO GSE176658) |
| `GSE274604_count.txt` | https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE274604 |
| `gencode.v38.annotation.gtf` (used in the notebook for promoters) | https://www.gencodegenes.org/human/release_38.html |
| `Homo_sapiens.GRCh38.110.gtf` (used to build the FIMO input) | https://ftp.ensembl.org/pub/release-110/gtf/homo_sapiens/Homo_sapiens.GRCh38.110.gtf.gz |
| `genome.fa` (GRCh38 primary assembly, Ensembl release 110) | https://ftp.ensembl.org/pub/release-110/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz |
| `MA0106.1.pfm`, `TP53.meme` (MA0106.3) | https://jaspar.elixir.no (included in this folder) |

## How the motif scan input was made (terminal, outside the notebook)

```bash
# genome: Ensembl release 110, GRCh38 primary assembly
gunzip Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
mv Homo_sapiens.GRCh38.dna.primary_assembly.fa genome.fa
samtools faidx genome.fa

# gene intervals from the Ensembl 110 GTF (62,754 genes)
grep -w "gene" Homo_sapiens.GRCh38.110.gtf | awk '{OFS="\t"; print $1,$4-1,$5,$10,$7}' > genes.bed

# extend each gene by 1 kb, then extract the sequences
bedtools slop -i genes.bed -g genome.fa.fai -l 1000 -r 0 -s > promoters.bed
bedtools getfasta -fi genome.fa -bed promoters.bed -fo promoters.fa

# motif scan (FIMO 5.5.9)
fimo --oc fimo_out TP53.meme promoters.fa

# the DNA that FIMO actually scanned (used as the control universe in the notebook)
sort -k1,1 -k2,2n promoters.bed | bedtools merge -i - > scanned_regions.bed
```

### Important notes about `promoters.fa`

- **The name is misleading.** It contains whole genes plus 1 kb, not promoters only. It has 62,754 sequences, is 2 GB, and its longest sequence is 2,474,539 bases.
- **The strand was not used.** `genes.bed` has the strand in column 5, but `bedtools slop -s` reads column 6. The 1 kb was therefore added on the left of every gene, which is upstream for `+` genes but downstream for `-` genes.
- **FIMO scanned only part of the genome.** `scanned_regions.bed` covers 1,807,740,716 of 3,088,269,832 bases on chromosomes 1 to 22, X and Y (about 58.5%). All 53,550 FIMO hits fall inside it.
- **The notebook accounts for this.** Peaks are compared with random regions placed only inside the scanned DNA, and only peaks fully inside scanned DNA are tested (57,451 of 74,304 peaks on the scanned chromosomes).
- The weakest hit that FIMO reported has p = 1.22e-05. The notebook keeps sites with p < 1e-5.
