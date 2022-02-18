# Unix Assignment

### Set Up
Files used in this assignment have been retrieved from GitHub.
```
git clone git@github.com:EEOB-BioData/BCB546-Spring2022.git
```

Unix assignment specific files are located in the UNIX_Assignment subdirectory within the cloned repository. To navigate there the following command was used.
```
cd ./BCB546-Spring2022/assignments/UNIX_Assignment
```
To complete the assignment, relevant files were copied to a separate directory outside of the cloned repository.
```
mkdir ../../../Unix_Assignment/
cp *.txt ../../../Unix_Assignment/
cp transpose.awk ../../../Unix_Assignment/
cd ../../../Unix_Assignment
```

## Data Inspection

### Attributes of `fang_et_al_genotypes`

```
du -b fang_et_al_genotypes.txt
```
Size of file is 11051939 bytes. The -b (--bytes) argument was used instead of the -h (--human-readable) argument because different results were returned between my personal machine and hpc-class due to how, by default, "du" measures space used not true file size.
```
wc -l fang_et_al_genotypes.txt
head -n 1 fang_et_al_genotypes.txt
awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
```
There are 986 columns and 2783 rows of data. The file is also tab-delimitated.

```
cut -f1-5,12,27,986 fang_et_al_genotypes.txt | head -n 10
cut -f1-5,12,27,986 fang_et_al_genotypes.txt | tail -n 10
```
It appears there are 3 columns of description information before the genotype data. Column 4 through the end of the document appear to be SNP sites, so there are 983 columns of SNP data. Row 1 has SNP site names and other header type information. Row 2 through the end of the document appear to be entries for the genotypes of sampled individuals so there are 2782 samples.

### Attributes of `snp_position.txt`
```
du -b snp_position.txt
```
Size of file is 82763 bytes
```
wc -l snp_position.txt
head -n 1 snp_position.txt
awk -F "\t" '{print NF; exit}' snp_position.txt
```
The `snp_position.txt` file appears to have 15 columns and 984 rows. The file is tab-delimitated.

```
head -n 15 snp_position.txt
```
SNP id's from column 1 of `snp_position.txt` seem to match the column labels from `fang_et_al_genotypes.txt`.

Additionally, the numbers of rows and columns from those respective files match when excluding the header row of `snp_position.txt` and descriptive columns from `fang_et_al_genotypes.txt`.

`snp_position.txt` also contains further information on SNP physical positions and associated gene information.

## Data Processing

### Maize & Teosinte Data
Run code in folder containing `snp_position.txt` `fang_et_al_genotypes.txt` and `transpose.awk` files.

Output will be 44 different files which are divided into 22 maize files which are located in `./output_maize` directory. The 22 teosinte files are located within `./output_teosinte` Within these directories, files should begin with either "maize" or "teosinte" as appropriate. Data has been further divided to separate markers by chromosomes in both ascending and descending physical coordinates.

For example a file named `maize_chr_1_asc.txt` contains maize sample genotypes of SNP markers located on chromosome 1 and which have been organized in ascending physical order.

Markers mapped to multiple positions are included in `*_chr_multiple.txt`files and markers with unknown positions are included in `*_chr_unknown.txt` files. Markers mapped to multiple positions located on a single chromosome have been diverted to `*_chr_multiple.txt` rather than their chromosome number specific file.

Each file contains Sample_ID information in the first row, SNP_IDs in the first column, chromosome number in the second column, physical position in the third column and genotypes in subsequent rows and columns.

Additionally two intermediate files will be created in the current directory. `subset_genotypes.txt` contains the subset of either maize or teosinte samples which will later be joined to the SNP position information. `joined_genotypes.txt` has the SNP position information concatenated into the first three columns of the `subset_genotypes.txt`. These two files are intermediate files and will get written over in the script so only the last run subset type (i.e. teosinte in the below script) will be available when the script finishes.

Run below code to generate above mentioned files.

```
for species in maize teosinte; do
  if [ $species == "maize" ]; then
    pattern="ZMMIL|ZMMLR|ZMMMR|Group"
  elif [ $species == "teosinte" ]; then
    pattern="ZMPBA|ZMPIL|ZMPJA|Group"
  fi
   grep -E $pattern fang_et_al_genotypes.txt | cut -f1,4- | awk -f transpose.awk | sed 's/Sample_ID/1SNP_ID/g' | sort -k1,1 >subset_genotypes.txt
   cut -f1,3,4 snp_position.txt | sed 's/SNP_ID/1SNP_ID/g'| sort -k1,1 | join - subset_genotypes.txt | sed 's/1SNP_ID/SNP_ID/g' >joined_genotypes.txt
   mkdir output_$species
   for chr in $(cut -d" " -f2 joined_genotypes.txt | sort -u | grep -v Chromosome); do
     if echo $chr | grep -Eq [0-9] - ; then
       awk -v chr=$chr '{if ($2~/Chromosome/ || ($2==chr && $3~/^[0-9]+$/)) {print $0}}' joined_genotypes.txt | sed 's;[^A-Z]\/[^A-Z];\?\/\?;g'| sort -k3,3 -n >./output_$species/${species}_chr_${chr}_asc.txt
       awk -v chr=$chr '{if ($2~/Chromosome/ || ($2==chr && $3~/^[0-9]+$/)) {print $0}}' joined_genotypes.txt | sed 's;[^A-Z]\/[^A-Z];\-\/\-;g' | sort -k3,3 -nr | sort -k2,2 -sr >./output_$species/${species}_chr_${chr}_desc.txt
     else
       awk -v chr=$chr '{if ($2~/Chromosome/ || $2==chr || ($3==chr && $3!~/^[0-9]+$/)) {print $0}}' joined_genotypes.txt | sed 's;[^A-Z]\/[^A-Z];\?\/\?;g' | sed 's/Chromosome/0Chromosome/g' | sort -k3,3 | sort -k2,2 -s | sed 's/0Chromosome/Chromosome/g' >./output_$species/${species}_chr_${chr}.txt
     fi
   done
done

```
