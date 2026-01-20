# Phyluce
Building a phylogenetic tree using Phyluce



## Uniforming datasets

```bash

Globally replace all pipe symbols (|) with underscores (_)

sed -i 's/|/_/g' *.fasta

Truncate FASTA headers at the first space

sed -i 's/\s.*$//' *.fasta

Converting a fasta file into 2bit file

for file in *.fasta; do
    base_name="${file%.fasta}"
   path/for/faToTwoBit "$file" "${base_name}.2bit"
   done

Put all .2bit files into their own individual directories and create a directory named “name-lastz”

for file in *.2bit; do
    folder_name="${file%.2bit}"  # Remove the .2bit extension
    mkdir -p "$folder_name"     # Create a folder with the base name
    mv "$file" "$folder_name/"  # Move the .2bit file into the folder
done

```

## Running Phyluce pipeline

```bash
conda create -n phyluce
conda activate phyluce
conda install -c bioconda phyluce

**Step 0a: phyluce_probe_run_multiple_lastzs_sqlite**

phyluce_probe_run_multiple_lastzs_sqlite --db name.sqlite --output name-lastz --probefile /path/to/UCE_probes.fasta --scaffoldlist taxon_1 taxon_2  --genome-base-path ./ --identity 80 --coverage 80 --cores 16

Prepare a transcriptomes.conf file as follows and all taxas:
[scaffolds]
taxon_1:/path/to/taxon_1/taxon_1.2bit

**Step 0b: phyluce_probe_slice_sequence_from_genomes**

phyluce_probe_slice_sequence_from_genomes --lastz name-lastz --conf ./transcriptomes.conf --flank 800 --name-pattern "probe-name-pattern.fasta_v_{}.lastz.clean" --output output_UCEs

**Step 1: phyluce_assembly_match_contigs_to_probes**

phyluce_assembly_match_contigs_to_probes \
    --contigs /path/for/contigs/ \
    --probes /path/to/probe-name-pattern.fasta \
    --output uce-search-results

Prepare a taxon-set.conf file as follows and all taxas:
[all]
taxon_1
taxon_2

**Step 2: phyluce_assembly_get_match_counts**

phyluce_assembly_get_match_counts \
    --locus-db uce-search-results/probe.matches.sqlite \
    --taxon-list-config taxon-set.conf \
    --taxon-group 'all' \
    --incomplete-matrix \
    --output taxon-sets/all/all-taxa-incomplete.conf

**Step 3: phyluce_assembly_get_fastas_from_match_counts**

phyluce_assembly_get_fastas_from_match_counts \
    --contigs /path/for/contigs \
    --locus-db ../../uce-search-results/probe.matches.sqlite \
    --match-count-output all-taxa-incomplete.conf \
    --output all-taxa-incomplete.fasta \
    --incomplete-matrix all-taxa-incomplete.incomplete \
    --log-path log


**Step 4: phyluce_align_seqcap_align**

phyluce_align_seqcap_align \
    --input all-taxa-incomplete.fasta \
    --output mafft-nexus-internal-trimmed \
    --taxa (number) \
    --aligner mafft \
    --cores 12 \
    --incomplete-matrix \
    --output-format fasta \
    --no-trim \
    --log-path log

**Step 5: phyluce_align_get_gblocks_trimmed_alignments_from_untrimmed**

phyluce_align_get_gblocks_trimmed_alignments_from_untrimmed \
    --alignments mafft-nexus-internal-trimmed \
    --output mafft-nexus-internal-trimmed-gblocks \
    --cores 12 \
    --log log

**Step 6: phyluce_align_remove_locus_name_from_files**

phyluce_align_remove_locus_name_from_files \
    --alignments mafft-nexus-internal-trimmed-gblocks \
    --output mafft-nexus-internal-trimmed-gblocks-clean \
    --cores 12 \
    --log-path log

**Step 7: phyluce_align_get_only_loci_with_min_taxa**

phyluce_align_get_only_loci_with_min_taxa \
    --alignments mafft-nexus-internal-trimmed-gblocks-clean \
    --taxa (number) \
    --percent 0.50 \
    --output mafft-nexus-internal-trimmed-gblocks-clean-50p \
    --cores 12 \
    --log-path log

**Step 8: phyluce_align_concatenate_alignments**

phyluce_align_concatenate_alignments \
    --alignments mafft-nexus-internal-trimmed-gblocks-clean-50p \
    --output mafft-nexus-internal-trimmed-gblocks-clean-50p-raxml \
    --phylip \
    --log-path log

```

## Constructing a tree using IQTREE

```bash

conda create -n IQTREE
conda activate IQTREE
conda install -c bioconda IQTREE

**Extended Selection + Partition Merging**

/path/to/IQTREE/bin/iqtree2 -s ./mafft-nexus-internal-trimmed-gblocks-clean-50p-raxml.phylip -p ./mafft-nexus-internal-trimmed-gblocks-clean-50p-raxml.charsets -nt AUTO -bb 1000 -bnni -alrt 1000 -m  MFP+MERGE

**Standard Model Selection**

/path/to/IQTREE/bin/iqtree2 -s ./mafft-nexus-internal-trimmed-gblocks-clean-50p-raxml.phylip -nt AUTO -bb 1000 -m  TEST
