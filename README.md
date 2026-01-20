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

## Downloading software using conda

```bash
conda create -n phyluce
conda activate phyluce
conda install -c bioconda phyluce

